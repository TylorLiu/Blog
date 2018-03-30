---
title: 读《深入理解Java虚拟机》2-JDK工具
date: 2018/2/11 13:47
tags: tools
category: note
---
JDK bin目录下提供了很多工具，大部分基于lib/tools.jar。
工具命令命名规则参考了unix系统的命令

- jps [options] [hostid] 查询虚拟机进程状况
  - -q 只输出lvmid 
  - -m 输出main函数的参数
  - -l 输出主类全名，或jar包路径
  - -v 输出进程启动的JVM参数
- jstat [option vmid[interval[s|ms] [count]]] 监视进程状态,类加载、gc、编译等

[![9JlFu4.md.jpg](https://s1.ax1x.com/2018/02/12/9JlFu4.md.jpg)](https://imgchr.com/i/9JlFu4)
jstat -gcutil 2764
S0：幸存1区当前使用比例
S1：幸存2区当前使用比例
E：伊甸园区使用比例
O：老年代使用比例
M：元数据区使用比例
CCS：压缩使用比例
YGC：年轻代垃圾回收次数
FGC：老年代垃圾回收次数
FGCT：老年代垃圾回收消耗时间
GCT：垃圾回收消耗总时间

- jinfo [option] vmid 查看配置信息 
  - -sysprops查看System.getProperties()
  - -flag name 查询jvm参数
  - -flag name=value 设置jvm参数
  
- jmap [option] vmid 生成堆dump等功能
[![9YqUXj.md.png](https://s1.ax1x.com/2018/02/16/9YqUXj.md.png)](https://imgchr.com/i/9YqUXj)

- jhat 堆dump分析工具

- jstack [option] vmid 生成线程快照
  - -F 请求不响应时强制输出线程堆栈
  - -l 附加锁信息
  - -m 附加本地方法堆栈
  
  Thread.getAllStackTraces()查看线程堆栈
  ````java
   Map<Thread, StackTraceElement[]> map = Thread.getAllStackTraces();
   for (Entry<Thread,StackTraceElement[]> entry :map.entrySet()){
       if (entry.getKey().equals(Thread.currentThread()))
           continue;
       System.out.println("线程："+entry.getKey().getName());
       StackTraceElement[] traceElements = entry.getValue();
       for (int i = 0; i <traceElements.length ; i++) {
           System.out.println(traceElements[i].toString());
       }
   }
  ````
  
JDK可视化工具
- JConsole 监控管理控制台，可视化jstate、jinfo、jstack等功能
- JVisualVM 可集成各种插件的故障处理工具，常用插件：
  - BTrace动态日志跟踪
  - MBeans 
  - VisualGC
  - JConsole Plugins
