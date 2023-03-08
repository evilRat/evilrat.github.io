---
title: ThreadPool总结
excerpt: ''
tags: [Java, 并发]
categories: [Java, 并发]
comments: true
date: 2020-05-18 00:30:52
---

## 线程池

线程池就是一个线程的集合，它帮我们来管理线程。

- 管理线程，避免创建和销毁线程的资源开销。创建一个对象要类加载，销毁一个对象要GC，都是需要占用资源的。
- 提高响应速度，不需要创建线程，直接将任务交给线程池运行。
- 重复利用，线程执行完毕，放回线程池，重复利用。

### 创建线程池

```java
public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize,long keepAliveTime,TimeUnit unit,
   BlockingQueue<Runnable> workQueue,
   ThreadFactory threadFactory,
   RejectedExecutionHandler handler)
```

几个核心参数的作用：
- corePoolSize： 线程池核心线程数最大值
- maximumPoolSize： 线程池最大线程数大小
- keepAliveTime： 线程池中非核心线程空闲的存活时间大小
- unit： 线程空闲存活时间单位
- workQueue： 存放任务的阻塞队列
- threadFactory： 用于设置创建线程的工厂，可以给创建的线程设置有意义的名字，可方便排查问题。
- handler： 线程池的饱和策略事件，主要有四种类型。

### 几个东东的关系

我们经常可以看到有通过new ThreadPoolExecutor()来创建线程池，也有通过Executors.newFixedThreadPool()等方法来创建线程池的，再加上Executor、ExecutorService，感觉乱七八糟的，屡屡关系。

先看下Executors：

```java
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }

    public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }

    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }

```

也就是Executors归根到底还是通过new ThreadPoolExecutor()来实现的。

再看看我们的ThreadPoolExecutor类：

```java
public class ThreadPoolExecutor extends AbstractExecutorService {
    ...
}
```

他继承了AbstractExecutorService抽象类，而AbstractExecutorService呢？

```java
public abstract class AbstractExecutorService implements ExecutorService {
    ...
}
```

他实现了ExecutorService接口。

```java
public interface ExecutorService extends Executor {
    ...
}
```

ExecutorService又继承了Executor。


### 线程池工作流程

1. 提交一个任务，线程池里存活的核心线程数小于线程数corePoolSize时，线程池会创建一个核心线程去处理提交的任务。
2. 如果线程池核心线程数已满，即线程数已经等于corePoolSize，一个新提交的任务，会被放进任务队列workQueue排队等待执行。
3. 当线程池里面存活的线程数已经等于corePoolSize了,并且任务队列workQueue也满，判断线程数是否达到maximumPoolSize，即最大线程数是否已满，如果没到达，创建一个非核心线程执行提交的任务。
4. 如果当前的线程数达到了maximumPoolSize，还有新的任务过来的话，直接采用拒绝策略处理。

### 四种拒绝策略

- AbortPolicy(抛出一个异常，默认的)
- DiscardPolicy(直接丢弃任务)
- DiscardOldestPolicy（丢弃队列里最老的任务，将当前这个任务继续提交给线程池
- CallerRunsPolicy（交给线程池调用所在的线程进行处理)

### 线程池的工作队列

线程池都有哪几种工作队列？

- ArrayBlockingQueue
- LinkedBlockingQueue
- DelayQueue
- PriorityBlockingQueue
- SynchronousQueue

1. ArrayBlockingQueue（有界队列）是一个用数组实现的有界阻塞队列，按FIFO排序量。
2. LinkedBlockingQueue（可设置容量队列）基于链表结构的阻塞队列，按FIFO排序任务，容量可以选择进行设置，不设置的话，将是一个无边界的阻塞队列，最大长度为Integer.MAX_VALUE，吞吐量通常要高于ArrayBlockingQuene，newFixedThreadPool线程池使用了这个队列
3. DelayQueueDelayQueue（延迟队列）是一个任务定时周期的延迟执行的队列。根据指定的执行时间从小到大排序，否则根据插入到队列的先后排序。newScheduledThreadPool线程池使用了这个队列。
4. PriorityBlockingQueue（优先级队列）是具有优先级的无界阻塞队列
5. SynchronousQueueSynchronousQueue（同步队列）一个不存储元素的阻塞队列，每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQuene，newCachedThreadPool线程池使用了这个队列。

## 几种常用的线程池

几种常用的线程池
1. newFixedThreadPool (固定数目线程的线程池)

```java
public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>(),
                                      threadFactory);
    }
```

特点：

    - 核心线程数和最大线程数大小一样
    - 没有所谓的非空闲时间，即keepAliveTime为0
    - 阻塞队列为无界队列LinkedBlockingQueue

- newCachedThreadPool(可缓存线程的线程池)

```java
public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory) {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>(),
                                      threadFactory);
    }
```

特点：
        - 核心线程数为0
        - 最大线程数为Integer.MAX_VALUE
        - 阻塞队列是SynchronousQueue
        - 非核心线程空闲存活时间为60秒

- newSingleThreadExecutor(单线程的线程池)

```java
public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>(),
                                    threadFactory));
    }
```

特点：
        - 核心线程数为1
        - 最大线程数也为1
        - 阻塞队列是LinkedBlockingQueue
        - keepAliveTime为0

- newScheduledThreadPool(定时及周期执行的线程池)

```java
public ScheduledThreadPoolExecutor(int corePoolSize) {
        super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
              new DelayedWorkQueue());
    }
```

特点：
        - 最大线程数为Integer.MAX_VALUE
        - 阻塞队列是DelayedWorkQueue
        - keepAliveTime为0scheduleAtFixedRate() ：按某种速率周期执行
        - scheduleWithFixedDelay()：在某个延迟后执行

## 线程池状态

```java
    // runState is stored in the high-order bits
    private static final int RUNNING    = -1 << COUNT_BITS;
    private static final int SHUTDOWN   =  0 << COUNT_BITS;
    private static final int STOP       =  1 << COUNT_BITS;
    private static final int TIDYING    =  2 << COUNT_BITS;
    private static final int TERMINATED =  3 << COUNT_BITS;
```

**RUNNING**
- 该状态的线程池会接收新任务，并处理阻塞队列中的任务;
- 调用线程池的shutdown()方法，可以切换到SHUTDOWN状态;
- 调用线程池的shutdownNow()方法，可以切换到STOP状态;

**SHUTDOWN**
- 该状态的线程池不会接收新任务，但会处理阻塞队列中的任务；
- 队列为空，并且线程池中执行的任务也为空,进入TIDYING状态;

**STOP**
- 该状态的线程不会接收新任务，也不会处理阻塞队列中的任务，而且会中断正在运行的任务；
- 线程池中执行的任务为空,进入TIDYING状态;

**TIDYING**
- 该状态表明所有的任务已经运行终止，记录的任务数量为0。
- terminated()执行完毕，进入TERMINATED状态

**TERMINATED**
- 该状态表示线程池彻底终止


<img src="2020-05-18 12-50-13屏幕截图.png">

## 线程池异常处理

常用的几种方法：

- 在我们提供的Runnable的run方法中捕获（try/catch）任务代码可能抛出的所有异常，包括未检测异常
- 使用ExecutorService.submit执行任务，利用返回的Future对象的get方法接收抛出的异常，然后进行处理
- 重写ThreadPoolExecutor.afterExecute方法，处理传递到afterExecute方法中的异常
- 为工作者线程设置UncaughtExceptionHandler，在uncaughtException方法中处理异常 (不推荐)


查资料的时候看到有人引用了[crossoverJie的文章](https://mp.weixin.qq.com/s/dqOy2eeeOsDa1AN3nNUftg)，之前也在b站看过他分享的程序员的一天，也拜读了一下。

他分享的是一个生产问题的定位，线程池中，一个线程报错导致线程池remove掉这个worker，又new了一个worker，然后这个worker会卡在去队列take的地方。


