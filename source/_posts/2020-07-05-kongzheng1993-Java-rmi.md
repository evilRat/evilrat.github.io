---
title: Java RMI
excerpt: ''
tags: [java]
categories: [java]
comments: true
date: 2020-07-05 00:30:52
---


今天是周日，昨天是挺忙碌的一天，上午去了一趟姥姥家，一年多没去了，最然都在北京，却还是不能经常见面。和舅舅聊了聊最近的生活和工作，他一直鼓励我努力。下午去找潇哥玩了，也是很久不见了，半年是有了。去他家坐了坐，有了女朋友的潇哥确实是精致了一些，晚上吃了顿露天烧烤，聊了聊这半年怎么被社会毒打的。。

今天坐在电脑前，不知道干点啥，b站首页停了半天不知道刷个什么视频。突然想起来之前面试又被问到java的RMI，今天来总结一下。


RMI是Java官方的一个RPC协议，像dubbo也支持RMI。

官方文档[Java RMI](https://docs.oracle.com/javase/tutorial/rmi/index.html)。

- Registry 注册中心，负责注册服务。
- Server 提供服务，实现接口。
- Client 调用方，调用服务。


官方文档中demo工程可以看。

Server端需要实现接口，并将接口暴露到网络。

```java
    public static void main(String[] args) {
        try {
            String name = "Compute";
            Compute engine = new ComputeEngine();
            Compute stub =
                    (Compute) UnicastRemoteObject.exportObject(engine, 0); // anonymous port
            Registry registry = LocateRegistry.createRegistry(1099);
            registry.rebind(name, stub);
            System.out.println("ComputeEngine bound");
        } catch (Exception e) {
            System.err.println("ComputeEngine exception:");
            e.printStackTrace();
        }
    }
```

Client端要获取Server端注册接口的注册中心，并从注册中心获取到接口，然后调用。

```java
    public static void main(String args[]) {
        try {
            String name = "Compute";
            Registry registry = LocateRegistry.getRegistry("127.0.0.1", 1099);
            Compute comp = (Compute) registry.lookup(name);
            Pi task = new Pi();
            BigDecimal pi = comp.executeTask(task);
            System.out.println(pi);
        } catch (Exception e) {
            System.err.println("ComputePi exception:");
            e.printStackTrace();
        }
    }
```

其实，RMI只是Java对RPC的一种实现而已。只不过，很多RPC都是网络协议发送方法、参数等实现远程调用;RMI是通过客户端的Stub对象作为远程接口进行远程方法的调用。

    注：Stub:为屏蔽客户调用远程主机上的对象，必须提供某种方式来模拟本地对象,这种本地对象称为存根(stub),存根负责接收本地方法调用,并将它们委派给各自的具体实现对象。