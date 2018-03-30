---
title: 常用JVM参数
date: 2018/2/28 18:45
tags: JVM
category: memo
---

### 内存设置

参数 | 含义 | 说明
-| :-: | -: 
-Xms1024m| 初始堆大小
-Xmx1024m| 最大堆大小
-Xmn500m | 年轻代大小
-Xss1024k| 单个线程栈
-XX:PermSize=200m | 永久代|before java8
-XX:MaxPermSize=300m | 永久代|before java8
-XX:MetaspaceSize    | 元数据 | after 8
-XX:MaxMetaspaceSize | 元数据 | after 8
-XX:NewRatio=4 |Olden区与Young区比例| 4：1
-XX:SurvivorRatio=8|Eden与Survivor区比例|8：1：1
-XX:LargePageSizeInBytes|内存页大小|[JVM优化之调整大内存分页](http://blog.csdn.net/yangpl_tale/article/details/76851149)

### GC设置

参数 | 含义 | 说明
-| :-: | -: 
-XX:+DisableExplicitGC|	关闭System.gc()|慎用
-XX:MaxTenuringThreshold|晋升年龄
-XX:PretenureSizeThreshold=1024k|Olden区直接分配对象临界值
-Xnoclassgc|禁用类回收
-XX:SoftRefLRUPolicyMSPerMB=1s|软引用存活秒数/每空闲MB
-XX:+UseParNewGC|并行回收YOUNG区，CMS默认搭档
-XX:+UseConcMarkSweepGC|CMS回收老年代
-XX:CMSFullGCsBeforeCompaction=3|3次FullGC后，整理老年代
-XX:+CMSParallelRemarkEnabled|并行标记，降低停顿
-XX:CMSInitiatingOccupancyFraction=85|Olden区85%后开始GC
-XX:+CMSClassUnloadingEnabled|开启类卸载

### 调试参数

参数 | 含义 | 说明
-| :-: | -: 
-XX:+PrintGCDetails|GC日志
-XX:+PrintGCApplicationStoppedTime|GC停顿时间
-verbose:gc/class/jni |查看gc、类加载、本地方法调用
-XX:+PrintHeapAtGC|打印GC触发时的堆栈
-Xloggc:log/gc.log|输出gc log
-XX:+HeapDumpOnOutOfMemoryError|内存溢出时产生堆dump
-XX:+HeapDumpOnCtrlBreak|Crtl+Break产生dump
### 其他
-Xverify:none 跳过编译检查
服务器推荐GC参数：
-Xloggc:gc.log
-XX:+PrintGCDetails
-XX:+PrintGCApplicationStoppedTime 
-XX:+PrintGCApplicationConcurrentTime