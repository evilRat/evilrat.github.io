---
title: RocketMQ OpenMessaging Example
excerpt: ''
tags: [RocketMQ]
categories: [RocketMQ]
comments: true
date: 2020-04-09 00:30:52
---

# OpenMessaging

OpenMessaging，包括建立行业指南和消息传递，流式传输规范，从而为金融，电子商务，物联网和大数据领域提供通用框架。在分布式异构环境中，设计原则是面向云，简单，灵活且独立于语言。符合这些规范将使跨所有主要平台和操作系统开发异构消息应用程序成为可能。

RocketMQ提供了OpenMessaging 0.1.0-alpha的部分实现，以下示例演示了如何基于OpenMessaging访问RocketMQ。

**OMS生产者**
以下示例显示了如何以同步，异步或单向传输将消息发送到RocketMQ代理。

```java
public class OMSProducer {
    public static void main(String[] args) {
        final MessagingAccessPoint messagingAccessPoint = MessagingAccessPointFactory
            .getMessagingAccessPoint("openmessaging:rocketmq://IP1:9876,IP2:9876/namespace");

        final Producer producer = messagingAccessPoint.createProducer();

        messagingAccessPoint.startup();
        System.out.printf("MessagingAccessPoint startup OK%n");

        producer.startup();
        System.out.printf("Producer startup OK%n");

        {
            Message message = producer.createBytesMessageToTopic("OMS_HELLO_TOPIC", "OMS_HELLO_BODY".getBytes(Charset.forName("UTF-8")));
            SendResult sendResult = producer.send(message);
            System.out.printf("Send sync message OK, msgId: %s%n", sendResult.messageId());
        }

        {
            final Promise<SendResult> result = producer.sendAsync(producer.createBytesMessageToTopic("OMS_HELLO_TOPIC", "OMS_HELLO_BODY".getBytes(Charset.forName("UTF-8"))));
            result.addListener(new PromiseListener<SendResult>() {
                @Override
                public void operationCompleted(Promise<SendResult> promise) {
                    System.out.printf("Send async message OK, msgId: %s%n", promise.get().messageId());
                }

                @Override
                public void operationFailed(Promise<SendResult> promise) {
                    System.out.printf("Send async message Failed, error: %s%n", promise.getThrowable().getMessage());
                }
            });
        }

        {
            producer.sendOneway(producer.createBytesMessageToTopic("OMS_HELLO_TOPIC", "OMS_HELLO_BODY".getBytes(Charset.forName("UTF-8"))));
            System.out.printf("Send oneway message OK%n");
        }

        producer.shutdown();
        messagingAccessPoint.shutdown();
    }
}
```

**OMSPull消费者**
使用OMS PullConsumer轮询来自指定队列的消息。

```java
public class OMSPullConsumer {
    public static void main(String[] args) {
        final MessagingAccessPoint messagingAccessPoint = MessagingAccessPointFactory
            .getMessagingAccessPoint("openmessaging:rocketmq://IP1:9876,IP2:9876/namespace");

        final PullConsumer consumer = messagingAccessPoint.createPullConsumer("OMS_HELLO_TOPIC",
            OMS.newKeyValue().put(NonStandardKeys.CONSUMER_GROUP, "OMS_CONSUMER"));

        messagingAccessPoint.startup();
        System.out.printf("MessagingAccessPoint startup OK%n");

        consumer.startup();
        System.out.printf("Consumer startup OK%n");

        Message message = consumer.poll();
        if (message != null) {
            String msgId = message.headers().getString(MessageHeader.MESSAGE_ID);
            System.out.printf("Received one message: %s%n", msgId);
            consumer.ack(msgId);
        }

        consumer.shutdown();
        messagingAccessPoint.shutdown();
    }
}
```

**OMSPushConsumer**
将OMS PushConsumer附加到指定的队列，并通过MessageListener使用消息

```java
public class OMSPushConsumer {
    public static void main(String[] args) {
        final MessagingAccessPoint messagingAccessPoint = MessagingAccessPointFactory
            .getMessagingAccessPoint("openmessaging:rocketmq://IP1:9876,IP2:9876/namespace");

        final PushConsumer consumer = messagingAccessPoint.
            createPushConsumer(OMS.newKeyValue().put(NonStandardKeys.CONSUMER_GROUP, "OMS_CONSUMER"));

        messagingAccessPoint.startup();
        System.out.printf("MessagingAccessPoint startup OK%n");

        Runtime.getRuntime().addShutdownHook(new Thread(new Runnable() {
            @Override
            public void run() {
                consumer.shutdown();
                messagingAccessPoint.shutdown();
            }
        }));

        consumer.attachQueue("OMS_HELLO_TOPIC", new MessageListener() {
            @Override
            public void onMessage(final Message message, final ReceivedMessageContext context) {
                System.out.printf("Received one message: %s%n", message.headers().getString(MessageHeader.MESSAGE_ID));
                context.ack();
            }
        });
    }
}
```