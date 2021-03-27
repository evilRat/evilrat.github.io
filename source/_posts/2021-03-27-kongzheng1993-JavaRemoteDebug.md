---
title: Java Remote Debug
excerpt: 'Java'
tags: [Java]
categories: [Java]
comments: true
date: 2021-03-27 19:30:52
---

记得刚毕业后刚上班的时候，写过一段时间的C++，用过GDB来调试程序，其实Java也有相应的远程调试的工具----JDB

## 增加JVM配置开启远程调试

启动参数中增加：

```

-Xdebug -Xrunjdwp:transport=dt_socket,server=y,address=5000,suspend=n,onthrow=java.io.IOExpection,launch=/sbin/echo

```

-Xdebug 启动调试
-Xrunjdwp 加载JVM的JPDA参考实现库


- transport指定了调试数据的传送方式：dt_socket是指用Socket模式，dt_shmem是指用共享内存方式，其中dt_shmem只适用于Windows平台。
- server参数是指是否支持在server模式的VM中
- onthrow是指定，当产生该类型的Exception时，JVM就会中断下来，进行调试
- launch是指当JVM被中断下来时，执行的可执行程序
- suspend指定是否在调试客户端建立起来后，再执行JVM
- onuncaught是指明出现uncaught exception后，是否中断JVM的执行
- address指指定连接地址，当transport为dt_socket时，address就是我们远程连接过去的端口号

## 如何进行调试

1. JDK自带了JDB：

```console

jdb -connect com.sun.jdi.SocketAttach:port=5000,hostname=192.168.100.100

```

上面的命令就是通过socket连接到`192.168.100.100:5000`进行远程调试，连接后就可以通过jdb的命令来进行断点，调试了。


2. 使用IDEA远程调试


使用我们的ide工具，以IDEA为例，配置一个Remote JVM Debug的Run/Debug Configurations就好了，像下面一样

<img src="idea_remote_debug.png">

接下来就是像本地调试一样，直接在源码上断点，就好了！所以，用工具吧，人类！