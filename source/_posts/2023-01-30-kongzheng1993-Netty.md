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


# 网络

## OSI

[开放式系统互联通信参考模型](https://baike.baidu.com/item/OSI%E6%A8%A1%E5%9E%8B/10119902?fr=aladdin)（英语：Open System Interconnection Reference Model，缩写为 OSI），简称为OSI模型（OSI model），一种概念模型，由国际标准化组织提出，一个试图使各种计算机在世界范围内互连为网络的标准框架。定义于ISO/IEC 7498-1。



## TCP/IP

抓包，三次握手、四次挥手


# IO

    任何程序都有IO，不然没有意义

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