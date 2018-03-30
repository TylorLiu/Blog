---
title: Linux下排查Java服务CPU飙升
date: 2018/3/23 20:08
tags: [Linux,java]
category: troubleshooting
---

大致步骤：
````
//拿到进程号
ps -ef|grep java
//查线程CPU占用
ps -mp PID -o THREAD,tid,time 
//查到的线程号转成 16进制
printf "%x\n" tid
// 打印线程堆栈
jstack PID|grep tid 
````