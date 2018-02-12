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
  - 