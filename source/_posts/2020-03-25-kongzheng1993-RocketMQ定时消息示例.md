---
title: 定时消息示例
excerpt: ''
tags: [MQ]
categories: [MQ]
comments: true
date: 2020-03-25 00:30:52
---


# 定时消息示例

## 什么是定时消息

预定的消息与正常的消息的不同之处在于，它们要等到指定的时间后才能传递。

RocketMQ 支持定时消息，但是不支持任意时间精度，仅支持特定的 level，例如定时 5s， 10s， 1m 等。其中，level=0 级表示不延时，level=1 表示 1 级延时，level=2 表示 2 级延时，以此类推。

**如何配置：**
在服务器端（rocketmq-broker端）的属性配置文件中加入以下行：

```txt
messageDelayLevel=1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h
```

描述了各级别与延时时间的对应映射关系。

这个配置项配置了从1级开始各级延时的时间，如1表示延时1s，2表示延时5s，14表示延时10m，可以修改这个指定级别的延时时间。

- 时间单位支持：s、m、h、d，分别表示秒、分、时、天
- 默认值就是上面声明的，可手工调整
- 默认值已经够用，不建议调整

## 应用

- 启动消费者等待传入的订阅消息

```java
import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
 import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
 import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
 import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
 import org.apache.rocketmq.common.message.MessageExt;
 import java.util.List;
    
 public class ScheduledMessageConsumer {
    
     public static void main(String[] args) throws Exception {
         // Instantiate message consumer
         DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("ExampleConsumer");
         // Subscribe topics
         consumer.subscribe("TestTopic", "*");
         // Register message listener
         consumer.registerMessageListener(new MessageListenerConcurrently() {
             @Override
             public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> messages, ConsumeConcurrentlyContext context) {
                 for (MessageExt message : messages) {
                     // Print approximate delay time period
                     System.out.println("Receive message[msgId=" + message.getMsgId() + "] "
                             + (System.currentTimeMillis() - message.getStoreTimestamp()) + "ms later");
                 }
                 return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
             }
         });
         // Launch consumer
         consumer.start();
     }
 }
```

- 发送定时消息

```java
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.common.message.Message;
public class ScheduledMessageProducer {
     public static void main(String[] args) throws Exception {
         // Instantiate a producer to send scheduled messages
         DefaultMQProducer producer = new DefaultMQProducer("ExampleProducerGroup");
         // Launch producer
         producer.start();
         int totalMessagesToSend = 100;
         for (int i = 0; i < totalMessagesToSend; i++) {
             Message message = new Message("TestTopic", ("Hello scheduled message " + i).getBytes());
             // This message will be delivered to consumer 10 seconds later.
             message.setDelayTimeLevel(3);
             // Send the message
             producer.send(message);
         }    
         // Shutdown producer after use.
         producer.shutdown();
     }
 }
```

- 验证

可以看到，消息存储10s后才会消费消息。
