---
title: Lock源码浅析
date: 2018/3/13 19:25
tags:
category: 
---

### Lock
先看Lock接口
````java
public interface Lock {
    //获取锁
    void lock();
    //获取锁过程中监听中断状态
    void lockInterruptibly() throws InterruptedException;
    //尝试获取锁，立即返回true或false
    boolean tryLock();
    //一定时间内尝试获取锁，允许中断
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    //释放锁，一般在finally中
    void unlock();
    //返回一个新建的Condition，用于实现wait/signal机制，释放锁并等待唤醒；应用：阻塞队列、CyclicBarrier、线程池等
    Condition newCondition();
}   
````
Lock比synchronized更灵活
- 允许中断
- 可以设置等待时限
- 通过返回判断状态

还提供了Object wait/notify的增强版，方便线程协作。

### ReentrantLock
再来看Lock的实现ReentrantLock,除了Lock规定的方法，它还提供了一些额外的方法：
````java
public class ReentrantLock implements Lock, java.io.Serializable {

      public ReentrantLock(boolean fair) {
          sync = fair ? new FairSync() : new NonfairSync();
      }
      
      public int getHoldCount() {
          return sync.getHoldCount();
      }
      public boolean isHeldByCurrentThread() {
          return sync.isHeldExclusively();
      }
      public boolean isLocked() {
          return sync.isLocked();
      }
      public final boolean hasQueuedThreads() {
         return sync.hasQueuedThreads();
     }
     public final boolean hasQueuedThread(Thread thread) {
         return sync.isQueued(thread);
     }
     public final int getQueueLength() {
         return sync.getQueueLength();
     }
     protected Collection<Thread> getQueuedThreads() {
         return sync.getQueuedThreads();
     }
     public boolean hasWaiters(Condition condition) {
         if (condition == null)
             throw new NullPointerException();
         if (!(condition instanceof AbstractQueuedSynchronizer.ConditionObject))
             throw new IllegalArgumentException("not owner");
         return sync.hasWaiters((AbstractQueuedSynchronizer.ConditionObject)condition);
     }
     public int getWaitQueueLength(Condition condition) {
         if (condition == null)
             throw new NullPointerException();
         if (!(condition instanceof AbstractQueuedSynchronizer.ConditionObject))
             throw new IllegalArgumentException("not owner");
         return sync.getWaitQueueLength((AbstractQueuedSynchronizer.ConditionObject)condition);
     }
     protected Collection<Thread> getWaitingThreads(Condition condition) {
         if (condition == null)
             throw new NullPointerException();
         if (!(condition instanceof AbstractQueuedSynchronizer.ConditionObject))
             throw new IllegalArgumentException("not owner");
         return sync.getWaitingThreads((AbstractQueuedSynchronizer.ConditionObject)condition);
     }
}
````
以上方法主要提供3个方面的功能：
- 公平锁实现
- 锁的状态(包括持锁线程、重入次数、排队线程等)
- Condtion相关状态

### ReentrantReadWriteLock
读写锁
ReentrantReadWriteLock包含readLock和writeLock，共享一个Syn子，其中readLock为共享锁，writeLock为排他锁，实现读写锁分离。

### synchronized
修饰代码段或方法，非公平互斥锁，需指定对象。

### volatile
- 防止指令重排
- 变量使用前从主存刷新