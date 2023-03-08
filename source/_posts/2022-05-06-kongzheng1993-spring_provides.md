---
title: spring.provides是什么？
excerpt: 'Spring'
tags: [SpringBoot]
categories: [SpringBoot]
comments: true
date: 2022-05-06 18:30:52
---

今天看代码发现很多springboot的starter的META-INF/目录下都有一个spring.provides文件，本来想找spring.factories的。查了很多资料，原来是spring的大佬都用STS开发，STS会用到这个文件，这个文件也是他们为了STS创建的，为了创建索引。

[spring-boot issues](https://github.com/spring-projects/spring-boot/issues/1926)