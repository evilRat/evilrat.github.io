---
title: FeignClient遇到的小问题
excerpt: 'Java'
tags: [Java]
categories: [Java]
comments: true
date: 2021-03-15 19:30:52
---


之前做了一个通过配置实现页面截图并发送到飞书群卡片的工具，可以配置截图页面url、宽高、超时时间等参数，通过cron定时发送或者@飞书机器人触发。并且自己也配置好了两个在用的业务，已经在用几个月了。前两天其他组的同事有业务场景要用到我们这个工具，配置上之后，竟然没有反应，我查了下日志---空指针，我第一反应是配置的表单我没有校验，导致后面取配置属性的时候NPE了。

开始看代码，看到后面并没有会造成空指针的地方，报错信息也只是打印到Feign调用的那行代码，检查调用Feign接口的参数，并没有问题啊。在feign关键源码处打断点：

```java

    public Object invoke(Object[] argv) throws Throwable {
        RequestTemplate template = this.buildTemplateFromArgs.create(argv);
        Retryer retryer = this.retryer.clone();

        while(true) {
            try {
                return this.executeAndDecode(template);
            } catch (RetryableException var5) {
                retryer.continueOrPropagate(var5);
                if (this.logLevel != Level.NONE) {
                    this.logger.logRetry(this.metadata.configKey(), this.logLevel);
                }
            }
        }
    }

```

一顿操作后，发现根本就没到这。

那肯定是接口调用前，盯着FeignClient看了半天，发现有个参数是int类型的，而我调用时，传递过来的是Integer，这地方肯定是强转时拆箱炸了。。


把FeignClient接口的参数类型修改为Integer后，测试通过。

