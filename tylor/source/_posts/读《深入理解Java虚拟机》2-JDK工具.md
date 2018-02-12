---
title: 读《深入理解Java虚拟机》2-JDK工具
date: 2018/2/11 13:47
tags: tools
category: [notes]
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

- jinfo [option] pid 查看配置信息 
  - -sysprops查看System.getProperties()
  - -flag name=value 查询/设置jvm参数