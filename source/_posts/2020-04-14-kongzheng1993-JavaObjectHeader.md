---
title: Java对象头
excerpt: ''
tags: [Java]
categories: [Java]
comments: true
date: 2020-04-14 00:30:52
---

## Java对象

在JVM中，实例对象在内存中的布局分为三块区域：对象头、实例变量和填充数据。如下：

<img src="Java_Monitor.png">
　　　　

**实例变量：** 存放类的属性数据信息，包括父类的属性信息，如果是数组的实例部分还包括数组的长度，这部分内存按4字节对齐。其实就是在java代码中能看到的属性和他们的值。 

**填充数据：** 由于虚拟机要求对象起始地址必须是8字节的整数倍。填充数据不是必须存在的，仅仅是为了字节对齐，这点了解即可。

**对象头：** Hotspot虚拟机的对象头主要包括两部分数据：Mark Word（标记字段）、Klass Pointer（类型指针）、Array length（数组长度，只有数组类型才有）。其中Klass Point【Class Metadata Address 】是对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例，Mark Word用于存储对象自身的运行时数据，它是实现轻量级锁和偏向锁的关键。

### 对象头-Mark Word（标记字段）

- Mark Word记录了对象和锁有关的信息，当这个对象被synchronized关键字当成同步锁时，围绕这个锁的一系列操作都和Mark Word有关。
- Mark Word在32位JVM中的长度是32bit，在64位JVM中长度是64bit。
- Mark Word在不同的锁状态下存储的内容不同。

<table border="1" cellspacing="0">
<tbody>
<tr>
<td style="background-color: #bfbfbf; width: 71pt;" rowspan="2">
<p style="margin-left: 0cm;">锁状态</p>
</td>
<td style="background-color: #bfbfbf; width: 142pt;" colspan="2">
<p style="margin-left: 0cm;">25bit</p>
</td>
<td style="background-color: #bfbfbf; width: 71pt;" rowspan="2">
<p style="margin-left: 0cm;">4bit</p>
</td>
<td style="background-color: #bfbfbf; width: 71.05pt;">
<p style="margin-left: 0cm;">1bit</p>
</td>
<td style="background-color: #bfbfbf; width: 71.05pt;">
<p style="margin-left: 0cm;">2bit</p>
</td>
</tr>
<tr>
<td style="background-color: #bfbfbf; width: 71pt;">
<p style="margin-left: 0cm;">23bit</p>
</td>
<td style="background-color: #bfbfbf; width: 71pt;">
<p style="margin-left: 0cm;">2bit</p>
</td>
<td style="background-color: #bfbfbf; width: 71.05pt;">
<p style="margin-left: 0cm;">是否偏向锁</p>
</td>
<td style="background-color: #bfbfbf; width: 71.05pt;">
<p style="margin-left: 0cm;">锁标志位</p>
</td>
</tr>
<tr>
<td style="vertical-align: top; width: 71pt;">
<p style="margin-left: 0cm;">无锁</p>
</td>
<td style="vertical-align: top; width: 142pt;" colspan="2">
<p style="margin-left: 0cm;">对象的HashCode</p>
</td>
<td style="vertical-align: top; width: 71pt;">
<p style="margin-left: 0cm;">分代年龄</p>
</td>
<td style="vertical-align: top; width: 71.05pt;">
<p style="margin-left: 0cm;">0</p>
</td>
<td style="vertical-align: top; width: 71.05pt;">
<p style="margin-left: 0cm;">01</p>
</td>
</tr>
<tr>
<td style="vertical-align: top; width: 71pt;">
<p style="margin-left: 0cm;">偏向锁</p>
</td>
<td style="vertical-align: top; width: 71pt;">
<p style="margin-left: 0cm;">线程ID</p>
</td>
<td style="vertical-align: top; width: 71pt;">
<p style="margin-left: 0cm;">Epoch</p>
</td>
<td style="vertical-align: top; width: 71pt;">
<p style="margin-left: 0cm;">分代年龄</p>
</td>
<td style="vertical-align: top; width: 71.05pt;">
<p style="margin-left: 0cm;">1</p>
</td>
<td style="vertical-align: top; width: 71.05pt;">
<p style="margin-left: 0cm;">01</p>
</td>
</tr>
<tr>
<td style="vertical-align: top; width: 71pt;">
<p style="margin-left: 0cm;">轻量级锁</p>
</td>
<td style="vertical-align: top; width: 284.05pt;" colspan="4">
<p style="margin-left: 0cm;">指向栈中锁记录的指针</p>
</td>
<td style="vertical-align: top; width: 71.05pt;">
<p style="margin-left: 0cm;">00</p>
</td>
</tr>
<tr>
<td style="vertical-align: top; width: 71pt;">
<p style="margin-left: 0cm;">重量级锁</p>
</td>
<td style="vertical-align: top; width: 284.05pt;" colspan="4">
<p style="margin-left: 0cm;">指向重量级锁的指针</p>
</td>
<td style="vertical-align: top; width: 71.05pt;">
<p style="margin-left: 0cm;">10</p>
</td>
</tr>
<tr>
<td style="vertical-align: top; width: 71pt;">
<p style="margin-left: 0cm;">GC标记</p>
</td>
<td style="vertical-align: top; width: 284.05pt;" colspan="4">
<p style="margin-left: 0cm;">空</p>
</td>
<td style="vertical-align: top; width: 71.05pt;">
<p style="margin-left: 0cm;">11</p>
</td>
</tr>
</tbody>
</table>

**锁的级别从低到高：无锁、偏向锁、轻量级锁、重量级锁。**

其中无锁和偏向锁的锁标志位都是01，只是在前面的1bit区分了这是无锁状态还是偏向锁状态。

JDK1.6以后的版本在处理同步锁时存在锁升级的概念，JVM对于同步锁的处理是从偏向锁开始的，随着竞争越来越激烈，处理方式从偏向锁升级到轻量级锁，最终升级到重量级锁。

JVM一般是这样使用锁和Mark Word的：
1. 当没有被当成锁时，这就是一个普通的对象，Mark Word记录对象的HashCode，锁标志位是01，是否偏向锁那一位是0。
2. 当对象被当做同步锁并有一个线程A抢到了锁时，锁标志位还是01，但是否偏向锁那一位改成1，前23bit记录抢到锁的线程id，表示进入偏向锁状态。
3. 当线程A再次试图来获得锁时，JVM发现同步锁对象的标志位是01，是否偏向锁是1，也就是偏向状态，Mark Word中记录的线程id就是线程A自己的id，表示线程A已经获得了这个偏向锁，可以执行同步锁的代码。
4. 当线程B试图获得这个锁时，JVM发现同步锁处于偏向状态，但是Mark Word中的线程id记录的不是B，那么线程B会先用`CAS`操作试图获得锁，这里的获得锁操作是有可能成功的，因为线程A一般不会自动释放偏向锁。如果抢锁成功，就把Mark Word里的线程id改为线程B的id，代表线程B获得了这个偏向锁，可以执行同步锁代码。如果抢锁失败，则继续执行步骤5。
5. 偏向锁状态抢锁失败，代表当前锁有一定的竞争，偏向锁将升级为轻量级锁。JVM会在当前线程的线程栈中开辟一块单独的空间，里面保存指向对象锁Mark Word的指针，同时在对象锁Mark Word中保存指向这片空间的指针。上述两个保存操作都是`CAS`操作，如果保存成功，代表线程抢到了同步锁，就把Mark Word中的锁标志位改成00，可以执行同步锁代码。如果保存失败，表示抢锁失败，竞争太激烈，继续执行步骤6。
6. 轻量级锁抢锁失败，JVM会使用自旋锁，**自旋锁不是一个锁状态，只是代表不断的重试**，尝试抢锁。从JDK1.7开始，自旋锁默认启用，自旋次数由JVM决定。如果抢锁成功则执行同步锁代码，如果失败则继续执行步骤7。
7. 自旋锁重试之后如果抢锁依然失败，同步锁会升级至重量级锁，锁标志位改为10。在这个状态下，未抢到锁的线程都会被阻塞。

### 对象头-指向类的指针

该指针在32位JVM中的长度是32bit，在64位JVM中长度是64bit。
Java对象的类数据保存在方法区。

### 对象头-数组长度

只有数组对象保存了这部分数据。
该数据在32位和64位JVM中长度都是32bit。