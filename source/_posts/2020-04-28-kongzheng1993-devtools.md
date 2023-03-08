---
title: springboot devtools
excerpt: ''
tags: [tools, Java]
categories: [Java]
comments: true
date: 2020-04-28 00:30:52
---

## devtools是什么

debug代码的时候，发现是个小bug，都不值得劳资手动重启应用怎么办？

springboot提供了一个`spring-boot-devtools`的工具，让我们在上述场景中提升效率。

它可以监控`classpath`下的资源，一旦classpath下有变动，就会触发应用重启。

## 如何使用devtools

在项目中添加`spring-boot-devtools`依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

这里设置optional为true，是因为`spring-boot-devtools`一般只使用于开发环境，在生产环境是需要禁用的。这样我们通过`java -jar`去启动应用，就不会启用devtools了，当然也可以防止devtools传递给依赖本应用的应用。

#### 如何触发应用重启？

Eclipse，对源文件修改并保存，就会触发重启。
IDEA，构建项目触发重启，`Build -> Build Project`。

当然如果你的机器配置高，IDEA勾选下图选项，可以在你修改文件后一定时间自动重启。

<img src="截屏2020-04-28下午6.51.22.png">

机器好可以这么玩，性能差就算了，因为devtools重启的时候，你的机器可能还没变编译完，就是起来了，也是个不完整的应用，比如controller没加载，调接口直接404……**不要问我为什么知道，问就是我踩过**。

不过这也能操作，毕竟原因我们知道了，就是还没构建完成，devtools就重启了呗，那咱们就让devtools晚会儿重启呗。。

```yml
spring:
    devtools:
        restart:
            poll-interval: 3s
```

这里设置devtools检测周期长一点，我3s够了，你不够再长点呗。。

**有几点需要注意：**
- devtools 将会使用独立的类加载器。
- devtools 依赖应用程序关闭钩子，如果你的应用禁用了关闭钩子（SpringApplication.setRegisterShutdownHook(false)），devtools 将会失效。
- 在决定是否应该触发重启时，devtools 将会自动忽略名为 spring-boot ， spring-boot-devtools ， spring-boot-autoconfigure ， spring-boot-actuator 和 spring-boot-starter 的项目。



