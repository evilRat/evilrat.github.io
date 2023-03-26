---
title: Netty
excerpt: 'netty'
tags: [netty]
categories: [netty]
comments: true
date: 2023-01-30 10:30:52
---

# 简介

[Netty](https://netty.io/)是一个异步事件驱动的网络应用程序框架，用于快速开发可维护的高性能协议服务器和客户端。

Netty是基于Java NIO（New IO）封装的，充分结合了Reactor线程模型，奖Netty变成了一个基于异步事件驱动的网络框架。

为什么不用Java远程NIO？
1. NIO的类库和API繁杂使用麻烦，你需要熟练掌握Selectol,ServerSocketChannel, SocketChannel,ByteBuffer 等。
2. 需妥具备其他的额外技能做制垫，例如熟悉Java 多线程编程。这是因为NIO编程涉及到Reactor 模式，你必须对多钱程和网络编程非常如悉，才能编写出高质量的NIO程序。
3. 可靠性能力补齐， 工作量和难度都非常大。例如客户端面临断连重连、网络间断、半包读写、失败缓存、网络拥塞和异常码流的处理等问题， NI0 编程的特点是功能开发相对容易，但是可靠性能力补齐的工作量和难度都非常大。
4. JDK NIO的BUG，比如epoll bug，这个BUG会在linux上导致cpu 100%，使得nio server/client不可用，这个BUG直到jdk 6u4才解决，但是直到JDK1.7中仍然有这个问题，该问题并未被完全解决，只是发生的频率降低了而已。


# 网络

## OSI

[开放式系统互联通信参考模型](https://baike.baidu.com/item/OSI%E6%A8%A1%E5%9E%8B/10119902?fr=aladdin)（英语：Open System Interconnection Reference Model，缩写为 OSI），简称为OSI模型（OSI model），一种概念模型，由国际标准化组织提出，一个试图使各种计算机在世界范围内互连为网络的标准框架。定义于ISO/IEC 7498-1。



## TCP/IP

抓包，三次握手、四次挥手


# IO

    任何程序都有IO，不然没有意义

IO最开始的定义就是计算机的输入流和输出流。随着技术发展，IO产生了分类。分类的维度不同，比如可以从工作层面上分为磁盘IO（本地IO）和网络IO，也可以从工作模式上分为BIO、NIO、AIO。




## BIO(Blocking IO)

同步阻塞IO，需要内核IO操作彻底完成后，才返回用户空间执行用户的操作。

## NIO(Non-blocking IO)

同步非阻塞IO，指用户空间的程序不需要等待内核IO操作彻底完成，可以立即返回用户空间执行用户的操作，用户空间的线程处于非阻塞状态。非阻塞IO要求socket被设置为NONBLOCK

## NIO(New IO)

Channel、Buffer、Selector

Java IO与Java NIO的区别，改进是什么？

# 阻塞/非阻塞

阻塞指的是用户空间程序的执行状态。传统的IO模型都是同步阻塞的。

# 同步/异步

同步IO，是一种用户空间与内核空间的IO发起方式。同步是指用户空间的线程是主动发起IO请求的一方，内核是被动接受方。异步IO则反过来，是系统内核主动发起IO请求，用户线程被动接受。

Linux异步IO实现方案总结
https://blog.csdn.net/smilejiasmile/article/details/122469921

netty中异步的理解
https://blog.csdn.net/W664160450/article/details/123474165
# Netty