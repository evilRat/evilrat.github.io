---
title: iframe重复加载的问题
excerpt: 'Web'
tags: [Web]
categories: [Web]
comments: true
date: 2020-12-18 19:30:52
---

最近需要开发一个可复用的dialog，用iframe实现，但是发现一个问题，当我打开显示这个dialog时，iframe引入的页面加载，没有问题。但是关闭这个iframe时，又重新加载了这个页面。从浏览器开发者工具可以看到两次请求这个页面，而且通过断点也能看到两次都进入了vue的creted、mount等生命周期函数。

```html
    <el-dialog title="iframe测试" :visible.sync="visiable" lock-scroll="false" @close="visiable = false"
               width="80%">
        <iframe id="iframeId" :src="'/XXX?YYY=' + yyy + '&ZZZ=' + zzz + '&r='+ Math.random()"
                style="width: 100% " frameborder="0" onload="this.height=400"></iframe>
    </el-dialog>
```

经过查询资料，当iframe的src属性被修改时，则会导致iframe再次被加载。上面的例子中，虽然YYY变量没有变动，但是当我关闭iframe的时候，再次检测src属性的时候，会执行`Math.random()`，每次都不同，所以会重新加载iframe。

### 如何修复

src属性的值用变量，不需要显示dialog的时候，将变量赋值`null`，只要src为`null`，iframe就不会加载，因为就算加载，也是请求到`null`。。。当我们需要用iframe的时候再给src赋值，关闭的时候，在修改会`null`。这样，iframe就只加载一次了～