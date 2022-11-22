---
title: 由一次github README.md引发的思考
excerpt: ''
tags: [other]
categories: [other]
comments: true
date: 2020-07-11 00:30:52
---


今天继续整理springcloud，到nacos那块了，然后写完文档，发现目录不好使？？？记得之前看很多大佬的github，README里有的目录点击也没反应。感觉也不能github的bug啊。。。

## 用vscode打开试试？

之前经常用vscode写blog，就是因为vscode自带markdown的预览，而且也有很多好用的markdown插件。打开试了一下，没问题，能正常跳转。。

## 观察github点击目录标题后的变化

比如我springcloud学习仓库。

点击前浏览器地址栏为：`https://github.com/kongzheng1993/SpringCloudLearn`

点击后：`https://github.com/kongzheng1993/SpringCloudLearn#Nacos注册中心&配置中心`

感觉好像对啊。。

反复看几遍！标题上有个 **&** ！！！

再联系我们熟知的http url参数。。

**&** 是分割参数的！！！

删掉目录和标题中的 **&**

没问题了，正常跳转！

```text
    + [spring-cloud-stream](#spring-cloud-stream)
      - [springcloud-stream流程](#springcloud-stream流程)
      - [使用方法](#使用方法)
      - [集群重复消费问题](#-集群重复消费问题)
      - [SpringCloud-stream消息持久化](#springcloud-stream消息持久化)
    + [springcloud-Sleuth分布式请求链路跟踪](#springcloud-Sleuth分布式请求链路跟踪)
    + [springcloud-alibaba](#springcloud-alibaba)
      - [Nacos注册中心配置中心](#Nacos注册中心配置中心)
```

## 总结

就是因为 **&** 符导致的！浏览器会以为你这里是参数分割。。

别让我再看到有大佬的README.md有这个问题，有我就提PR [doge]

