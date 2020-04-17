---
title: Java Monitor
excerpt: ''
tags: [Java]
categories: [Java]
comments: true
date: 2020-04-17 00:30:52
---

## 什么是Monitor

Monitor可以理解为一种同步工具，也可理解为一种同步机制，常常被描述为一个Java对象，也叫**管程**。

管程（Monitor）是一种和信号量（Sophomore）等价的同步机制。它在Java并发编程中也非常重要，虽然程序员没有直接接触管程，但它确实是synchronized和wait()/notify()等线程同步和线程间协作工具的基石：当我们在使用这些工具时，其实是它在背后提供了支持。简单来说：管程使用锁（lock）确保了在任何情况下管程中只有一个活跃的线程，即确保线程互斥访问临界区管程使用条件变量（Condition Variable）提供的等待队列（Waiting Set）实现线程间协作，当线程暂时不能获得所需资源时，进入队列等待，当线程可以获得所需资源时，从等待队列中唤醒。

**总结：**

- 互斥：一个Monitor在一个时刻只能被一个线程持有，即Monitor中的所有方法都是互斥的。

- signal机制：如果条件变量不满足，允许一个正在持有Monitor的线程暂时释放持有权，当条件变量满足时，当前线程可以唤醒正在等待该条件变量的线程，然后重新获取Monitor的持有权。

所有的Java对象是天生的Monitor，每一个Java对象都有成为Monitor的潜质，因为在Java的设计中 ，每一个Java对象自打娘胎里出来就带了一把看不见的锁，它叫做内部锁或者Monitor锁。

Monitor的本质是依赖于底层操作系统的Mutex Lock实现，操作系统实现线程之间的切换需要从用户态到内核态的转换，成本非常高。

Monitor 是线程私有的数据结构，每一个线程都有一个可用monitor record列表，同时还有一个全局的可用列表。每一个被锁住的对象都会和一个monitor关联（对象头的MarkWord中的LockWord指向monitor的起始地址），同时monitor中有一个Owner字段存放拥有该锁的线程的唯一标识，表示该锁被这个线程占用。其结构如下：

<img src="1.png">

- Owner字段：初始时为NULL表示当前没有任何线程拥有该monitor record，当线程成功拥有该锁后保存线程唯一标识，当锁被释放时又设置为NULL
- EntryQ字段：关联一个系统互斥锁（semaphore），阻塞所有试图锁住monitor record失败的线程
- RcThis字段：表示blocked或waiting在该monitor record上的所有线程的个数
- Nest字段：用来实现重入锁的计数
- HashCode字段：保存从对象头拷贝过来的HashCode值（可能还包含GC age）
- Candidate字段：用来避免不必要的阻塞或等待线程唤醒，因为每一次只有一个线程能够成功拥有锁，如果每次前一个释放锁的线程唤醒所有正在阻塞或等待的线程，会引起不必要的上下文切换（从阻塞到就绪然后因为竞争锁失败又被阻塞）从而导致性能严重下降；Candidate只有两种可能的值0表示没有需要唤醒的线程1表示要唤醒一个继任线程来竞争锁

## Monitor具体实现方式

1. Monitor是在jvm底层实现的，底层代码是c++
2. Monitor的enter方法：获取锁
3. Monitor的exit方法：释放锁
4. Monitor的wait方法：为java的Object的wait方法提供支持
5. Monitor的notify方法：为java的Object的notify方法提供支持
6. Monitor的notifyAll方法：为java的Object的notifyAll方法提供支持

## Monitor机制

见下图：
<img src="2.png">

Monitor可以类比为一个特殊的房间，这个房间中有一些被保护的数据，Monitor保证每次只能有一个线程能进入这个房间进行访问被保护的数据，进入房间即为持有Monitor，退出房间即为释放Monitor。

当一个线程需要访问受保护的数据（即需要获取对象的Monitor）时，它会首先在entry-set入口队列中排队（这里并不是真正的按照排队顺序），如果没有其他线程正在持有对象的Monitor，那么它会和entry-set队列和wait-set队列中的被唤醒的其他线程进行竞争（即通过CPU调度），选出一个线程来获取对象的Monitor，执行受保护的代码段，执行完毕后释放Monitor，如果已经有线程持有对象的Monitor，那么需要等待其释放Monitor后再进行竞争。

再说一下wait-set队列。当一个线程拥有Monitor后，经过某些条件的判断（比如用户取钱发现账户没钱），这个时候需要调用Object的wait方法，线程就释放了Monitor，进入wait-set队列，等待Object的notify方法（比如用户向账户里面存钱）。当该对象调用了notify方法或者notifyAll方法后，wait-set中的线程就会被唤醒，然后在wait-set队列中被唤醒的线程和entry-set队列中的线程一起通过CPU调度来竞争对象的Monitor，最终只有一个线程能获取对象的Monitor。

**注意：**
当一个线程在wait-set中被唤醒后，并不一定会立刻获取Monitor，它需要和其他线程去竞争
如果一个线程是从wait-set队列中唤醒后，获取到的Monitor，它会去读取它自己保存的PC计数器中的地址，从它调用wait方法的地方开始执行。
2.3、Monitor与java对象以及线程是如何关联 
1.如果一个java对象被某个线程锁住，则该java对象的Mark Word字段中LockWord指向monitor的起始地址
2.Monitor的Owner字段会存放拥有相关联对象锁的线程id