---
title: 读《深入理解Java虚拟机》1-内存管理
date: 2018/2/3 15:57
tags: JVM 
category: note
---
### JVM中的内存区域划分
![9ZHKpQ.png](https://s1.ax1x.com/2018/02/03/9ZHKpQ.png)

- 线程共享区
  * 堆 —— 存放Class实例和数组。 
    * 新生代
      * Eden
      * Survivor
        * From
        * To
    * 老年代
  * 方法区(Permgen/MetaSpace)
    * 类信息
    * 运行时常量池
    * 静态变量
    * JIT编译代码等
- 线程私有区
  * 程序计数器
  * 栈内存  默认-Xss1024K
    * 虚拟机栈
      * 栈帧
        * 局部变量表
        * 操作数栈
        * 动态链接
        * 方法返回地址等
    * 本地方法栈
- 直接内存 (NIO DirectByteBuffer)

### 对象创建
- 类加载检查
- 分配内存
  * 指针碰撞：内存规整，以指针分界空闲内存，移动指针分配内存
  * 空闲列表：内存不规整，查列表分配、更新列表
  * 线程安全
    * (Thread Local Allocation Buffer)线程本地缓存分配，锁同步；-XX:+/-UseTLAB
    * CAS+失败重试
- 生成对象头，filed置0，<init>初始化
- 对象引用入栈

### 垃圾收集GC
- 哪些内存需要回收
  - 引用计数 (无法解决相互引用)
  - 可达性分析 - GCROOTS
    * 栈帧的局部变量表中的对象
    * 类静态属性引用的对象
    * 方法区常量引用的对象
    * 本地方法栈中引用的变量
  - 引用
    * 强->正常引用
    * 软->即将OOM时回收
    * 弱->有无弱引用都不影响回收
    * 虚->queue监听回收
  - 方法区回收
    * 废弃常量 - 没有引用
    * 无用的类 (-XX:+CMSClassUnloadingEnabled )
      * 无实例
      * classloader已回收
      * 其Class实例无引用
      * 没有任何反射调用
- 什么时候回收
  - Minor GC : Eden空间不足以分配对象时触发
  - Major/Full GC : System.gc()或Minor GC解决不了问题时触发
- 如何回收
  - 垃圾收集算法->分代收集
    * 新生代 ：复制
      * Serial 单线程stw适用于新生代较小的场景，如桌面应用 (Client默认收集器)
      * ParNew 多线程版，一般与CMS共用，线程数=CPU数，-XX: ParallelGCThreads
      * Parallel Scavenge 与CMS不兼容
        * -XX:MaxGCPauseMillis 最大停顿时间->交互应用
        * -XX:GCTimeRadio x 吞吐量=x/(1+x) ->后台运算
        * -XX:+ UseAdaptiveSizePolicy自调节      
    * 老年代 ：标记 清除/整理
      * Serial Old 整理(Client默认收集器)
      * Parallel Old 多线程版，与Scavenge配合用
      * CMS(Concurrent Mark Sweep) 缩短停顿时间，适用于网站服务器
        * 线程数=(CPU+3)/4，适合多CPU服务器
        * 用户线程并发需预留空间 -XX:CMSInitiatingOccupancyFraction=80
        * 空间碎片问题 -XX:CMSFullGCsBeforeCompaction=1  
        [![9GdyJH.md.jpg](https://s1.ax1x.com/2018/02/11/9GdyJH.md.jpg)](https://imgchr.com/i/9GdyJH)
  - G1 ：标记整理+region间复制，实现可预测停顿
    * Region 
    * 优先列表
        
