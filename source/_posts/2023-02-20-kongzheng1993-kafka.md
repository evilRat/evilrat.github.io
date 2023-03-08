---
title: Kafka常见问题总结
excerpt: 'Kafka'
tags: [Kafka]
categories: [Kafka]
comments: true
date: 2023-02-20 10:30:52
---


# 1.顺序问题

有些消息的消费是有顺序的，比如先创建订单，才能修改订单。

## 1. 如何保证消息的顺序性

kafka的topic是无序的，但是一个topic包含多个partition，每个partition内部是有序的，只要保证生产者写消息时，按照一定的规则写到同一个partition，不同的消费者读不同的partition的消息，就能保证生产和消费消息的顺序。比如同一个商户编号的消息写到同一个partition中，topic中创建了4个partition，然后部署4个消费者节点，构成消费者组，一个partition对应一个消费者节点，从理论上说，这套方案是能够保证消息顺序的。

## 2. 顺序消息失败了怎么办？

abcd四个顺序消息，如果a失败了，影响了bcd的消费怎么办？
加入重试，每个消息如果失败了，重试三到五次？可以解决大部分情况，但极端情况下，也可能会有问题。





https://mp.weixin.qq.com/s/YPkE3Tsu3RVbhfVZCBt1pQ