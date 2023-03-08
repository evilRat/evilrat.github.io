---
title: 用node.js做一个测试api服务
excerpt: ''
tags: [node.js]
categories: [node.js]
comments: true
top: 3
date: 2020-03-30 00:30:52
---


# 用node.js做一个测试api服务

前两天被安排做一体机的项目。这可是个大坑，兄弟们都不想搞。我和他们不一样了，迎难而上，哈哈。这个是嵌入到营业厅一体机或者pad的一个系统，为什么说这是一个大坑呢？没有测试环境，只能生产联调。要是平时，可以在公司搭一个测试环境嘛，可是疫情期间在家办公，诸多不便。还有一个问题是，没有后台系统啊，所有接口都不通，盲写代码？

## 怎么快速搞一个工具获得我想要的接口呢

记得之前看过一本书《架构探险 轻量级微服务架构》，阿里大佬黄勇老师的书，内容通俗易懂，是我很喜欢的风格。里面有一章用node.js讲解并实现微服务网关，npm安装node.js第三方模块，快速实现功能，就像Python和pip，当时就让我对node.js产生了浓厚的兴趣，js统一前后端了，哈哈。所以我想用node.js来实现一个简单的根据url来返回特定json的工具，并用这个工具来进行一体机项目的开发。

## 开始搞

1. 修改npm为淘宝镜像

```shell
npm install cnpm -g --registry=http://registry.npm.taobao.org
```

<img src='cnpm.bmp'>

2. 安装express

express是一款基于node.js的web应用框架，提供了大量的工具函数与中间件，使web应用开发效率更加高效。

```shell
npm install express
```
<img src='express.bmp'>

3. 编码

打开vscode，开始撸

```js
var express = require('express');
var port = 8080;

var app = express();
app.use(express.static('.'));

app.get('/querywriteCardBasicData', function (req, res) {
    res.send('{"id":"123",name:"jack",arg:11111}')
});

app.post('/querywriteCardBasicData', function (req, res) {
    res.send('{"id":"123",name:"jack",arg:11111}')
});

app.post('/querywriteCardBasicData', function (req, res) {
    res.send('{"id":"123",name:"jack",arg:11111}')
});

app.listen(port, function () {
    console.log('server is running at %d', port);
});
```

4. 运行

```shell
node app.js
```

<img src='run.bmp'>

5. 测试

浏览器访问`http://localhost:8080/querywriteCardBasicData`

<img src='test.bmp'>

## 总结

Node.js 是一个基于 Chrome V8 引擎的 JavaScript 运行环境。Node.js 使用了一个事件驱动、非阻塞式 I/O 的模型,使其轻量又高效。
我喜欢写一些小工具，之前因为工作上的需要，写过许多小工具，比如Python写的接口配置工具、文件下载工具、sql脚本生成工具。这些小工具都得到了同事和领导的好评，我也收获了极大的满足。我喜欢用工具完成一些简单但是重复的工作，如果有一个特别麻烦的事儿挡在我面前，我一定会在第一时间思考，怎么搞个工具，哈哈，加油。