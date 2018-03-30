---
title: JVM内存溢出分析总结
date: 2018/1/29 16:03
tags: JVM
category: troubleshooting
---
开个文总结下我遇到的内存溢出问题，放个JVM的图以供参考：

![pzqqqs.jpg](https://s1.ax1x.com/2018/01/29/pzqqqs.jpg)

从内存溢出分析的角度来看，内存主要分为栈内存、堆内存、直接内存和MetaSpace(java8之前是堆内的永久代)。
下面主要讲下这几种内存溢出的场景：
### 1. StackOverflowError 栈内存溢出
这里stack指的是线程的独立内存空间，主要保存方法执行信息，每个方法调用时都会产生一个栈帧，当栈深度超过虚拟机分配给线程的栈大小时就会出现StackOverflowError。
出现这个问题有以下几种可能：
- 分配的栈内存太小，这种不太常见 JVM默认栈内存-Xss1024k，足够一般线程使用。
- 方法循环调用，A方法调用B，B调用C，C又来调用A，就会出现。大部分栈溢出都是这种情况。
- 方法迭代，比如分页查询，总数1W，每页查一个，查到hasNext为true就继续查，迭代1W次，也有可能出现溢出。

问题出现后，根据stackTrace错误堆栈基本上就可以定位到问题位置。

PS：高并发应用创建过多线程也可能引起栈内存OOM，有开源、节流两种思路应对：
- 开源：提升栈内存总量
  * 扩大服务器内存
  * 减少堆内存-Xms -Xmx
- 节流
  * 减少每个线程分配的栈内存 -Xss512k或256k 
  * 通过负载均衡、拒绝&延迟响应等策略减少线程并发数

### 2. OutOfMemoryError: Java heap space 堆内存溢出
堆内存主要用来存放java对象，堆内存不够了就会报错。
问题有两种可能的原因：
1. 堆内存设置太小，无法满足应用正常运行的需要(包括基本运行以及承载高峰期的内存压力)；
2. 应用存在内存泄漏，部分对象无法回收，导致占用内存越来越多。


遇到这种情况，可以
- 先把最大堆内存-Xmx改大一些
- 进行JVisualVM内存抽样或者MAT等工具dump分析，查看内存中对象分步情况。重点观察内存占比较大的实例和线程。
- 还可以监视应用的堆内存，看一下内存的增长是否有规律。

也可以使用jmap等java命令来进行dump分析，详情参考[深入解析OutOfMemoryError](http://shenzhang.github.io/blog/2014/04/24/understanding-out-of-memory-error/)

另外，如果多次报错时打出的strackTrace都在同一个位置，建议着重看一下。如果是某些框架内部报错，先去官网或者Google一下。


### 3. OutOfMemoryError: PermGen space / MetaSpace 永久代或Metaspace溢出

这里主要讲类信息导致的内存溢出，类信息在java8之前存放在永久代，java8之后存在Metaspace中。
应用加载的类信息主要分两种: class文件和运行过程中动态生成的类。除非特地限制-XX:MaxMetaspaceSize或-XX:MaxPermSize大小，一般该问题都是由动态类引起的。
可以通过JVisualVM监视MetaSpace和类加载数，MetaSpace的增长与类加载的关系。

可以通过以下方式定位问题原因：
- 从错误堆栈找到问题抛出点
- 本地调试抛出点代码,加上JVM参数 -verbose:class，打印类加载信息
- 找到动态加载类的代码，分析如何解决

之前解决过的案例是由JAXB解析XML数据引起的，JAXBContext.newInstance()会生成一些动态类和方法，在JDK的JAXB工具类中通过WeakReference< Cache >来缓存JAXBContext,但是容易被回收，而且不能存放多个JAXBContext，如果需要解析多种格式的XML数据，还是会经常调用JAXBContext.newInstance()。
````java
private static final class Cache {
    final Class type;
    final JAXBContext context;

    public Cache(Class type) throws JAXBException {
        this.type = type;
        this.context = JAXBContext.newInstance(type);
    }
}    
````
最后采用Map来存放JAXBContext。
````java
private static Map<Class<?>, JAXBContext> contextMap = new ConcurrentHashMap<>();

````
还有一种类似的情况，使用Apache-CXF生成的Webservice-client，每次初始化也会生成一些动态类，应该使用单例模式避免重复初始化。
另外，CGLib字节码增强实现的动态代理以及Groovy等动态语言、动态产生的JSP文件等都会动态加载类信息，使用时需要注意。
### 4. 堆外内存溢出
java NIO通信时默认会使用ByteBuffer缓存数据，如果用完忘记调用ByteBuffer.release()方法，容易导致内存泄漏。
Tips：
- JVisualVm 装上Buffer Pools插件就可以监视java进程堆外内存的使用情况了，分别显示Direct、Mapped
- 使用了Netty应用在启动时可以用-Dio.netty.leakDetection.level=PARANOID 检测一下是否有内存泄漏情况，根据错误堆栈可以定位到泄漏点

