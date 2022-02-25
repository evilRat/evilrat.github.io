---
title: MySQL性能优化
excerpt: 'mysql'
tags: [mysql]
categories: [mysql]
comments: true
date: 2022-01-09 18:30:52
---

# MySQL的MVCC、事务隔离和锁

## MVCC

MVCC，全称`Multi-Version Concurrency Control`，即多版本并发控制。MVCC是一种并发控制的方法，一般在数据库管理系统中，实现对数据库的并发访问，在编程语言中实现事务内存。

记为一致性非锁定读，MySQL基于自己的***回滚机制**为并发场景的**读操作**做的一个优化。回滚机制也就是undo log，

## 隔离级别

MySQL有四种隔离级别，由低到高分别是：

- read uncommited 读取未提交数据
- read commited 读取已提交数据
- repeatable read 可重复读
- serializable 串行化

MySQL的默认隔离级别是repeatable read。

举例： A、B线程为写线程，C线程为读线程
repeatable read级别下：
A、C同时向一条数据发起了写和读的操作，C发起线程后（已开启事务了）sleep，A开始修改数据，在可重复读的隔离界别下，要先生成一版快照，修改之前的快照，也就是undo log，以便回滚 