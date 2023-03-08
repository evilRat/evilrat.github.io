---
title: nohup和&
date: 2019-06-05 13:52:46
tags: [Linux]
categories: [Linux]
comments: true
---

昨天申请休年假，但是组长表示让我别休年假，给我两天调休，所以今天就开始休息了。。。
闲来无事，想起一个困扰多年的问题，RocketMQ启动时使用nohup和&命令，&我知道时后台运行，可以在后台进程中找到运行的程序，而且经常被程序的输出搞得无法执行下一条命令。。。今天就来总结一下。

### 使用&后台运行程序
- 结果会输出到终端
- 使用Ctrl + C发送SIGINT信号，程序免疫
- 关闭session发送SIGHUP信号，程序关闭

### 使用nohup运行程序
- 结果默认会输出到nohup.out
- 使用Ctrl + C发送SIGINT信号，程序关闭
- 关闭session发送SIGHUP信号，程序免疫

### 使用nohup和&配合来启动程序
- 同时免疫SIGINT和SIGHUP信号
