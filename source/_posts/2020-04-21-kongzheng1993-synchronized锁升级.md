---
title: synchronized锁升级
excerpt: ''
tags: [Java]
categories: [Java]
comments: true
date: 2020-04-21 00:30:52
---

[本文整理自知乎](https://zhuanlan.zhihu.com/p/115561832)

## 可见性

使用synchronized的代码块或方法中使用变量，会发生以下：

1. 获取同步锁
2. 清空自己的工作内存的变量副本
3. 从主存获取最新的值，并加载到工作内存中
4. 对变量进行操作。

所以使用synchronized会从主存中取最新的值，从而保证可见性。

## 原子性

最常见的例子，启动多个线程将一个静态变量执行`++`操作，最终结果并不是所有线程`++`次数之和。

使用synchronized会在操作前monitorenter和操作后monitorexit，保证synchronized中的代码是原子的。

## 有序性

代码中程序执行的顺序，Java在编译和运行时会对代码进行优化，这样会导致我们最终的执行顺序并不是我们编写代码的书写顺序。

咱先来看一个概念,重排序，也就是语句的执行顺序会被重新安排。其主要分为三种：

1. 编译器优化的重排序：可以重新安排语句的执行顺序。
2. 指令级并行的重排序：现代处理器采用指令级并行技术，将多条指令重叠执行。
3. 内存系统的重排序：由于处理器使用缓存和读写缓冲区，所以看上去可能是乱序的。

`A a = new A();`可能被被JVM分解成如下代码：

```java
// 可以分解为以下三个步骤
1 memory=allocate();// 分配内存 相当于c的malloc
2 ctorInstanc(memory) //初始化对象
3 s=memory //设置s指向刚分配的地址
// 上述三个步骤可能会被重排序为 1-3-2，也就是：
1 memory=allocate();// 分配内存 相当于c的malloc
3 s=memory //设置s指向刚分配的地址
2 ctorInstanc(memory) //初始化对象  
```

一旦假设发生了这样的重排序，比如线程A在执行了步骤1和步骤3，但是步骤2还没有执行完。这个时候线程B进入了第一个语句，它会判断a不为空，即直接返回了a。其实这是一个未初始化完成的a，即会出现问题。

**synchronized如何解决有序性问题**
给上面的三个步骤加上一个synchronized关键字，即使发生重排序也不会出现问题。线程A在执行步骤1和步骤3时，线程B因为没法获取到锁，所以也不能进入第一个语句。只有线程A都执行完，释放锁，线程B才能重新获取锁，再执行相关操作。

## synchronized的常见使用方式

- 修饰代码块（同步代码块）

```java
synchronized (object) {
      //具体代码
}
```

- 修饰方法

```java
synchronized void test(){
  //具体代码
}
```

- 修饰静态方法

```java
synchronized static void test(){
   //具体代码
}
```

- 修饰类

```java
synchronized (Example2.class) {
    //具体代码
 }
```

## synchronized继承

假设父类test方法是synchronized修饰的。

- 如果子类继承父类，没有重写test方法，那么子类test方法也是同步的。
- 如果子类继承父类，重写了test方法，但是没有使用synchronized修饰，那么子类test不是同步的。
- 如果子类继承父类，重写了test方法，并且用synchronized修饰，是同步。

## 锁升级

<img src="v2-8f405804cd55a26b34d59fefc002dc08_r.jpg">










