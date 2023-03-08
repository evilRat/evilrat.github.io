---
title: localStorage
excerpt: ''
tags: [Web]
categories: [Web]
comments: true
date: 2020-03-30 00:30:52
---

# Window.localStorage

只读的`localStorage`属性允许你访问一个`Document`源（origin）的对象`Storage`；存储的数据将保存在浏览器会话中。`localStorage` 类似 `sessionStorage`，但其区别在于：存储在 localStorage 的数据可以长期保留；而当页面会话结束——也就是说，当页面被关闭时，存储在 sessionStorage 的数据会被清除 。

应注意，无论数据存储在 localStorage 还是 sessionStorage ，它们都特定于页面的协议。

另外，localStorage 中的键值对总是以字符串的形式存储。 (需要注意, 和js对象相比, 键值对总是以字符串的形式存储意味着数值类型会自动转化为字符串类型)