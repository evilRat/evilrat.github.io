---
title: mysql超过最大连接数一次生产问题定位.md
date: 2019-06-14 12:34:44
excerpt: "mysql"
tags: [mysql,java]
categories: [mysql,java]
comments: true
---

今天营业厅一体机小姐姐又来找我了，生产又有问题了。。。

我查了下日志：

```java

Caused by: com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: User ngcrmpf_bj already has more than 'max_user_connections' active connections
    at sun.reflect.GeneratedConstructorAccessor266.newInstance(Unknown Source)
    at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
    at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
    at com.mysql.jdbc.Util.handleNewInstance(Util.java:404)

```

连接数超了。。。

之前听同事说经常遇见这个问题，每次都是把服务器最大连接数调大一些，然后就好了。。。

我想：就算每次都调大连接数上限，也终会达到最大连接数啊。为什么呢？ 应该是我们定期发布，重启tomcat后会断开所有连接，如果我们长时间不发布，就会到达上限。

然后我开始翻代码，看到配置的最大连接数：

```java
ngcrmpf_bj.jdbc.maxActive=5
```

只有5条啊，8台tomcat加起来也就是四十条啊，为什么服务器设置1200还会超呢。。。。

仔细看报错信息，发现有一个方法，不知道是哪位大哥写的，竟然自己写了jdbc，而且没有关闭连接。。。mmp。。。

我们的接口参数都会加密传输，密钥存在mysql，他竟然用jdbc去数据库查密钥，而且没关闭连接。。。为什么不去redis？为什么不用mybatis？再不济，为什么不关闭连接。。。

苍天啊，大地啊，我真的不知道怎么和客户解释。。。

唉。。。
