---
title: NIO&BIO
excerpt: 'NIO BIO'
tags: [NIO BIO IO]
categories: [NIO BIO IO]
comments: true
date: 2019-09-11 18:30:52
---

## BIO

同步非阻塞， blocking I/O，服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器就需要启动一个线程进行处理，如果这个连接不做任何事情就会造成不必要的线程开销。这种情况可以通过线程池机制改善。tomcat bio就是通过线程池实现的，默认200并发。

```java


```
