---
title: 技术总结— java基础
date: 2018/3/1 19:21
tags: jdk
category: 总结
---

### Java类库

- 数据类型
  - 基本类型 byte、int等
  - 字符串 CharSequence
  - 数据结构 (同步容器、并发容器)
     - 数组
     - Map(Hash、Tree)
     - Set
     - List()
     - Queue
  - Collections排序、Iterator遍历

- 多线程支持
  - 线程状态
    - new->running<->timedwait/wait/blocked->terminal
  - 锁(读写锁，锁拆分ConcurrentHashMap)
    - ReentrantLock
    - ReentrantReadWriteLock
    
  - 线程池
    - Executors: Factory提供ExecutorService、ScheduledExecutorService、ThreadFactory、Callable
    - ThreadPoolExecutor 由Spring ThreadPoolTaskExecutor配置
        
  - ForkJoin 任务拆分，并发计算
  - Stream.parallelStream()并行流
  - Callable/Future
  - 并发控制
    - CountDownLatch 倒数
    - CyclicBarrier 起跑线
    - Semaphore 信号量，类似多个线程持锁
  - ThreadLocal
  
- Time Api
- reflect
  - Class获取Annotation/method/field等
  - 动态代理
- io
  - InputStream/OutStream
  - Reader/Writer
- nio ->Netty封装
  - ByteBuffer
  - Selector
  - SocketChannel
- ClassLoader类加载机制
  - 双亲委派(Bootstrap、ExtClassLoader、AppClassLoader)