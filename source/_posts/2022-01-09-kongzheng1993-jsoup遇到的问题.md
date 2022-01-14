---
title: 使用jsoup遇到的问题
excerpt: 'Web'
tags: [Web]
categories: [Web]
comments: true
date: 2022-01-14 18:30:52
---

最近做了一个通过jsoup来处理html，实现页面table可以根据用户的配置来展示哪些列。

https://jsoup.org/apidocs/org/jsoup/select/Selector.html

```java
    public Elements children() {
        return new Elements(childElementsList());
    }
```