---
title: DevTools的bug
excerpt: 'java'
tags: [java]
categories: [java]
comments: true
date: 2022-03-02 18:30:52
---

https://www.bilibili.com/video/BV1cZ4y1C717?spm_id_from=333.999.0.0

DevTools有一个类加载器，叫做Restart ClassLoader，负责加载我们src中自己写的类，但是依赖的jar包中的类，它不会加载，而是由Application ClassLoader加载。