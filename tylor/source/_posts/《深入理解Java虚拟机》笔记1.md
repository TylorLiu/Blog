---
title: 《深入理解Java虚拟机》笔记1
date: 2018/2/3 15:57
tags: JVM 
category: note
---
JVM内存区域划分：
![9ZHKpQ.png](https://s1.ax1x.com/2018/02/03/9ZHKpQ.png)

- 线程共享区
  * 堆
    * 新生代
      * Eden
      * Survivor
        * From
        * To
    * 老年代
  * 方法区
    * 类信息
    * 常量
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