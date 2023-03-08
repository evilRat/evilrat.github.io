---
title: 交易消息示例
excerpt: ''
tags: [RocketMQ]
categories: [RocketMQ]
comments: true
date: 2020-04-10 00:30:52
---

# 交易消息示例

## 什么是交易消息

可以将其视为两阶段提交消息实现，以确保分布式系统中的最终一致性。事务性消息可确保本地事务的执行和消息的发送能以原子方式执行。

## 使用限制

1. 事务消息没有时间表和批处理支持。
2. 为了避免对单个消息进行过多的检查并导致半个队列的消息堆积，我们默认将单个消息的检查数量限制为15次，但是用户可以通过更改`transactionCheckMax`来更改此限制。如果一条消息经过`transactionCheckMax`次检查，默认情况下，代理将丢弃此消息并同时打印错误日志。用户可以通过覆盖`AbstractTransactionCheckListener`类来更改此行为。
3. 将`broker`配置中由参数`transactionTimeout`来确定一定时间后检查交易消息。用户还可以在发送事务消息时通过设置用户属性`CHECK_IMMUNITY_TIME_IN_SECONDS`来更改此限制，该参数优先于`transactionMsgTimeout`参数。
4. 交易消息可能被检查或消耗了不止一次。
5. 提交给用户目标主题的已提交消息可能会失败。目前，它取决于日志记录。RocketMQ本身的高可用性机制可确保高可用性。如果要确保事务消息不会丢失并且事务完整性得到保证，建议使用同步双写机制。
6. 事务性消息的生产者ID不能与其他类型的消息的生产者ID共享。与其他类型的消息不同，事务性消息允许向后查询。MQ服务器通过生产者ID查询客户端。

## 应用

### 1，交易状态

事务消息有三种状态：

- TransactionStatus.CommitTransaction：提交事务，表示允许使用者使用此消息。
- TransactionStatus.RollbackTransaction：回滚事务，表示该消息将被删除且不允许使用。
- TransactionStatus.Unknown：中间状态，表示需要MQ进行回溯以确定状态。

### 2，发送交易信息

#### 2，1 创建事务生产方

使用`TransactionMQProducer`类创建生产方客户端，并指定唯一的`producerGroup`，然后可以设置自定义线程池来处理检查请求。执行本地事务后，需要根据执行结果对MQ进行回复，回复状态如上节所述。

```java
import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.common.message.MessageExt;
import java.util.List;

public class TransactionProducer {
    public static void main(String[] args) throws MQClientException, InterruptedException {
        TransactionListener transactionListener = new TransactionListenerImpl();
        TransactionMQProducer producer = new TransactionMQProducer("please_rename_unique_group_name");
        ExecutorService executorService = new ThreadPoolExecutor(2, 5, 100, TimeUnit.SECONDS, new ArrayBlockingQueue<Runnable>(2000), new ThreadFactory() {
            @Override
            public Thread newThread(Runnable r) {
                Thread thread = new Thread(r);
                thread.setName("client-transaction-msg-check-thread");
                return thread;
            }
        });

        producer.setExecutorService(executorService);
        producer.setTransactionListener(transactionListener);
        producer.start();

        String[] tags = new String[] {"TagA", "TagB", "TagC", "TagD", "TagE"};
        for (int i = 0; i < 10; i++) {
            try {
                Message msg =
                    new Message("TopicTest1234", tags[i % tags.length], "KEY" + i,
                        ("Hello RocketMQ " + i).getBytes(RemotingHelper.DEFAULT_CHARSET));
                SendResult sendResult = producer.sendMessageInTransaction(msg, null);
                System.out.printf("%s%n", sendResult);

                Thread.sleep(10);
            } catch (MQClientException | UnsupportedEncodingException e) {
                e.printStackTrace();
            }
        }

        for (int i = 0; i < 100000; i++) {
            Thread.sleep(1000);
        }
        producer.shutdown();
    }
}
```

#### 2，2实现TransactionListener接口

当发送一半消息成功时，使用`executeLocalTransaction`方法执行本地事务。它返回上一部分中提到的三个事务状态之一。
`checkLocalTransaction`方法用于检查本地事务状态并响应MQ检查请求。它还返回上一部分中提到的三个事务状态之一。

```java
public class TransactionListenerImpl implements TransactionListener {
   private AtomicInteger transactionIndex = new AtomicInteger(0);
      private ConcurrentHashMap<String, Integer> localTrans = new ConcurrentHashMap<>();
      @Override
   public LocalTransactionState executeLocalTransaction(Message msg, Object arg) {
       int value = transactionIndex.getAndIncrement();
       int status = value % 3;
       localTrans.put(msg.getTransactionId(), status);
       return LocalTransactionState.UNKNOW;
   }
      @Override
   public LocalTransactionState checkLocalTransaction(MessageExt msg) {
       Integer status = localTrans.get(msg.getTransactionId());
       if (null != status) {
           switch (status) {
               case 0:
                   return LocalTransactionState.UNKNOW;
               case 1:
                   return LocalTransactionState.COMMIT_MESSAGE;
               case 2:
                   return LocalTransactionState.ROLLBACK_MESSAGE;
           }
       }
       return LocalTransactionState.COMMIT_MESSAGE;
   }
```
