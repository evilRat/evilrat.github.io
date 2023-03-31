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

一般我们将它分成以下几类：
- BIO：同步阻塞IO
- NIO：同步非阻塞IO
- AIO：异步非阻塞IO

**何为同步，何为阻塞？**

同步/异步： 同步IO是指用户空间(进程或者线程)是主动发起IO请求的一方，系统内核是被动接收方。异步IO则反过来，系统内核是主动发起IO请求的一方，用户空间是被动接收方。 

阻塞： 阻塞IO指的是需要内核IO操作彻底完成后才返回到用户空间执行用户程序的操作指令。“阻塞”指的是用户程序(发起IO请求的进程或者线程)的执行状态。可以说传统 的IO模型都是阻塞IO模型，并且在Java中默认创建的socket都属于阻塞IO模型。 



## BIO(Blocking IO)

同步阻塞IO，需要内核IO操作彻底完成后，才返回用户空间执行用户的操作。

<img src='blockingIO.png'>

用户程序调用内核read后，会一直等待，直到返回数据。
特点就是在内核执行IO操作的两个阶段，用户线程/进程一直是阻塞的。
优点：程序开发简单，阻塞期间用户线程挂起，基本不会占用CPU资源。
缺点：一个连接就是一个线程，在高并发场景下，需要大量的线程来维护大量的网络连接，内存和线程切换的开销非常大，性能很低，基本不可用。


## NIO(Non-blocking IO)

同步非阻塞IO，指用户空间的程序不需要等待内核IO操作彻底完成，可以立即返回用户空间执行用户的操作，用户空间的线程处于非阻塞状态。非阻塞IO要求socket被设置为NONBLOCK

<img src="nonBLockingIO.png">

同步非阻塞IO模型中，用户程序发起read调用，如果内核数据没有准备好，内核会立即返回。用户程序为了读到数据，会一直发起系统调用，轮询数据是否准备好，直到数据返回为止。

优点：每次发起IO系统调用，内核准备数据的过程中，调用会被直接返回，用户线程不会阻塞，实时性较好。

缺点：不断轮询内核，将占用大量的CPU时间，效率低下。

## IO多路复用

为了避免同步非阻塞IO模型中轮询等待的问题，就有了IO多路复用模型。IO多路复用的系统调用有select、epoll等，几乎所有的操作系统都支持select，具有良好的跨平台特性。epoll是select的Linux增强版本。

<img src="multiplexing.png">

<img src="man2select.png">

<img src="man2kqueue.png">

**Todo：**
- 几种Java程序，strace 查看socket、bind、read、select、epoll_create等系统调用，观察阻塞、同步
- select和epoll的示意图

## 异步IO

<img src="asyncIO.png">

用户线程通过系统调用内核注册某个IO操作，内核在整个IO操作（包括数据准备、数据复制）完成后通知用户程序，用户执行后续的业务操作。异步IO是真正的异步输入输出，它的吞吐量高于IO多路复用模型。就目前而言，Windows系统下通过IOCP实现了真正的异步IO，在Linux系统下，异步IO模型在2.6版本才引入，JDK对它的支持并不完善。
而大多数高并发服务器的程序都是基于Linux系统的，因此目前高并发网络应用程序的开发大多采用IO多路复用模型，大名鼎鼎的Netty框架使用的就是IO多路复用模型，而不是异步IO模型。



# Netty
Netty是一个异步事件驱动的网络应用框架快速开发可维护的高性能协议服务器和客户端。
Netty是基于Java NIO（New IO）封装的，充分结合了Reactor线程模型，将Netty变成了一个基于异步事件驱动的网络框架。

## Java NIO和NIO

Java1.4之前Java IO类库是阻塞IO，1.4版本之后引进了新的异步IO库，称为Java New IO类库缩写也就是NIO。而老的阻塞式的IO被称为Old IO，OIO。

Java NIO包含以下3个核心组件：
- Channel--通道
- Buffer--缓冲区
- Selector--选择器
看到这三个组件，就能识别出它用的是IO多路复用模型
NIO和OIO的区别：
- OIO是面向流（Stream Oriented）的，NIO是面向缓冲区（Buffer Oriented）的。

**为什么不用Java远程NIO？**

1. NIO的类库和API繁杂使用麻烦，你需要熟练掌握Selectol,ServerSocketChannel, SocketChannel,ByteBuffer 等。
2. 需要具备其他的额外技能做制垫，例如熟悉Java 多线程编程。这是因为NIO编程涉及到Reactor 模式，你必须对多钱程和网络编程非常熟悉，才能编写出高质量的NIO程序。
3. 可靠性能力补齐， 工作量和难度都非常大。例如客户端面临断连重连、网络间断、半包读写、失败缓存、网络拥塞和异常码流的处理等问题， NI0 编程的特点是功能开发相对容易，但是可靠性能力补齐的工作量和难度都非常大。
4. JDK NIO的BUG，比如epoll bug，这个BUG会在linux上导致cpu 100%，使得nio server/client不可用，这个BUG直到jdk 6u4才解决，但是直到JDK1.7中仍然有这个问题，该问题并未被完全解决，只是发生的频率降低了而已。

## Reactor模式

Netty的整体架构是基于Reactor模式的。Reactor模式由Reactor线程、Handlers处理器两大角色组成，两大角色的职责分别如下：
1. Reactor线程的职责：负责响应IO时间，并且分发到Handlers处理器。
2. Handlers处理器的职责：非阻塞的执行业务处理逻辑。

<img src="asyncIO.png">

Reactor模式中IO事件的处理流程大致分为4步：
1. 通道注册。IO事件源于通道Channel，IO是和通道（对应于底层连接而言）强相关的。一个IO事件一定属于某个通道。如果要查询通道的事件，首先要将通道注册到选择器。
2. 查询事件。在Reactor模式中，一个线程会负责一个反映器（或者SubReactor子反映器），不断轮询，查询选择器中的IO事件（选择键）。
3. 事件分发。如果查询到了IO事件，则分发给与IO事件有绑定关系的Handler业务处理。
4. 完成真正的IO操作和业务处理，这一步由Handler业务处理器负责。

上面第1、2步其实是Java NIO的功能，Reactor模式仅仅是利用了Java NIO的优势而已。

### Netty中的Channel

Netty中不直接使用Java NIO的Channel组件，对Channel组件进行 了自己的封装。Netty实现了一系列的Channel组件，为了支持多种通 信协议，换句话说，对于每一种通信连接协议，Netty都实现了自己的 通道。除了Java的NIO，Netty还提供了Java面向流的OIO处理通道。 

Netty中常见的通道类型：
- NioSocketChannel：异步非阻塞TCP Socket传输通道
- NioServerSocketChannel：异步非阻塞TCP Socket服务端监听通道
- NioDatagramChannel：异步非阻塞的UDP传输通道
- NioSctpChannel：异步非阻塞Sctp传输通道
- NioSctpServerChannel：异步非阻塞Sctp服务端监听通道
- OioSocketChannel：同步阻塞式TCP Socket传输通道
- OioServerSocketChannel：同步阻塞式TCP Socket服务端监听通道
- OioDatagramChannel：同步阻塞式UDP传输通道
- OioSctpChannel：同步阻塞式Sctp传输通道
- OioSctpServerChannel：同步阻塞式Sctp服务端监听通道


<img src="channel.png">


### Netty中的Reactor

Netty中的反映器组件有多个实现类，这些实现类与其通道类型相互匹配。对应NioSocketChannel通道，Netty的反映器类型为NioEventLoop（NIO事件轮询）。

NioEventLoop类有两个重要的成员属性：
- Thread线程类型成员
- Java NIO选择器成员

<img src="netty_reactor">

到这里可以看出，NioEventLoop和前面说的反映器思路一致：一个NioEventLoop拥有一个线程，负责一个Java NIO选择器的IO事件轮询。
EventLoop反映器和Channel的关系：一个EventLoop反映器和NettyChannel通道是一对多的关系：一个反映器可以注册成千上万的通道。

<img src="EventLoop.png">

### Netty中的Handler

在Netty中，EventLoop反映器内部有一个线程负责Java NIO选择器事件的轮询，然后进行对应的事件分发，事件分发（Dispatch）的目标就是Netty的Handler（含用户定义的业务处理器）。

Netty的Handler分为两大类：
- ChannelInboundHandler入站处理器
- ChannelOutboundHandler出站处理器
二者都继承了ChannelHandler处理器接口。

<img src="handler.png">

Netty的IO事件类型：
- 可读：SelectionKey.OP_READ
- 可写：SelectionKey.OP_WRITE
- 连接：SelectionKey.OP_CONNECT
- 接受：SelectionKey.OP_ACCEPT

### Netty中的Pipeline

Netty设计了一个特殊的组件，叫做ChannelPipeline（通道流水线）。它像一条管道，将绑定到一个通道的多个Handler处理器实例串联在一起，形成一条流水线。ChannelPipeline默认实现被设计成一个双向链表，所有Handler处理器实例被包装成双向链表的节点，被加入到ChannelPipeline中。

