---
title: 技术总结— java基础
date: 2018/3/1 19:21
tags: JDK
category: 总结
---

- 优点
  - 擅长发现问题
    - 业务流程优化
    - 代码(结构、实现)优化
  - 解决问题、吸收经验
    - 根因定位
      - 例子：DB log表溢出，大量无用记录(删除)，原因：定时任务输出Error日志->直播转点播失败->直播设备被删除(清理失效课程)
    - 方案
    - 经验总结
      - 业务脏数据及时清理
      - 重复任务失败 要有相应措施，防止重复失败
  - 完美主义CodeReview
    - 各种工具进行代码审查
    - 日志审查
      - Error原因
        - 该不该输出
        - 输出信息是否完整
      - 日志级别
    - 异常处理

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