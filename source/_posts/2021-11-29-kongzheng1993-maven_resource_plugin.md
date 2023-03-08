---
title: maven-resource-plugin
excerpt: 'Maven'
tags: [Maven]
categories: [Maven]
comments: true
date: 2021-11-26 16:30:10
---

每次上线后，总会用户过来问，怎么没有生效，新功能不好使等问题。大概率都是因为浏览器缓存造成的。这些静态文件一般都在maven项目的resource目录下。

浏览器的缓存是通过url来的。如果url变了，浏览器就会认为要请求的不是一个资源，就会重新发起请求获取资源。

所以如果我们在修改静态文件后，统一给这些文件修改名称，到时候浏览器就会重新获取拿到新文件了。

spring resource支持给资源名增加后缀：
1. 资源名-md5方式：此时会将请求增加后缀（-资源名的md5）
```ini
spring.resources.chain.strategy.content.enabled=true
spring.resources.chain.strategy.content.paths=/**
 ```
2. 资源名-版本号：会在请求后增加后缀（-版本号）
```ini
spring.resources.chain.strategy.fixed.enabled=true
spring.resources.chain.strategy.fixed.paths=/**
spring.resources.chain.strategy.fixed.version=v1.0.0
```

第一种方法不需要每次修改配置，每次都是崭新的md5，但是需要记住老的md5，以便判断是否加载了最新的资源。第二种需要每次修改版本号，好处是，一眼就能看出是否是新版本的资源。

第二种方式我们可以使用`@project.version@`变量直接使用maven项目的版本号，这样就不需要再来bootstrap配置文件修改resource的版本号了。

因为在Spring MVC中，资源的查找、处理使用的是责任链设计模式（Filter Chain）。

<img src="20161012101735543.png"/>

而springboot的相关配置就是spring.resources.chain。


以上配置之后，所有的resource的请求都会加后缀，浏览器会重新获取静态资源，解决了发版后静态资源缓存的问题。但是这样会影响一些二进制文件，比如我们导入功能的excel模版文件，tff字体，pdf模版等等。因为maven-resources-plugin在打包resources时，会对文件进行统一编码，导致二进制文件损坏。

下面的配置就可以排除一些文件，以免被重新编码导致不可用。

```yml

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-resources-plugin</artifactId>
            <version>3.1.0</version>
            <configuration>
                <encoding>UTF-8</encoding>
                <nonFilteredFileExtensions>
                    <nonFilteredFileExtension>xlsx</nonFilteredFileExtension>
                    <nonFilteredFileExtension>xls</nonFilteredFileExtension>
                    <nonFilteredFileExtension>svg</nonFilteredFileExtension>
                    <nonFilteredFileExtension>tff</nonFilteredFileExtension>
                    <nonFilteredFileExtension>pdf</nonFilteredFileExtension>
                    <nonFilteredFileExtension>wsf</nonFilteredFileExtension>
                    <nonFilteredFileExtension>jpg</nonFilteredFileExtension>
                    <nonFilteredFileExtension>jpeg</nonFilteredFileExtension>
                    <nonFilteredFileExtension>gif</nonFilteredFileExtension>
                    <nonFilteredFileExtension>bmp</nonFilteredFileExtension>
                    <nonFilteredFileExtension>png</nonFilteredFileExtension>
                </nonFilteredFileExtensions>
            </configuration>
        </plugin>
    </plugins>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
        </resource>
    </resources>
</build>

```

[Maven官网资料]https://maven.apache.org/plugins/maven-resources-plugin/examples/include-exclude.html