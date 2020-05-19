---
title: maven install
excerpt: ''
tags: [maven]
categories: [maven]
comments: true
date: 2020-05-19 00:30:52
---

最近给框架升级，由于总部研发给的新框架是deploy到私有maven库，我们也都是在本地开发，到私有远程maven库网络是不通的，所以我也只能跳到那边把几个jar包下载，然后sftp到一台两边都通都服务器上，再搞到本地。。不就是mvn install嘛，我以为我可以系列，哈哈。。正好今天也查了很多资料，就记录下来，希望能帮到有缘人。。

## mvn install

将项目的主要工件以及生命周期中其他插件附带的任何其他工件安装到本地存储库。

### 一般用法

我脑海里记得到参数也就是这些了

```shell
mvn install:install-file -DgroupId=com.xxx -DartifactId=xxx -Dversion=1.0.0 -Dpackaging=jar -Dfile=you-jar-file-path
```

相关的参数可以查看[install:install-file](http://maven.apache.org/plugins/maven-install-plugin/install-file-mojo.html)

这里的参数里`-Dfile`是必须的，毕竟没文件，你install个毛线。。
其他参数都是可选的，毕竟jar文件如果也是maven工程，它会默认用父pom的`groupID`、`artifactId`、`version`参数。

但就是这些可选参数，我们还是要记住几个的。。

`groupID`、`artifactId`、`version`就不说了。

比如`pomFile`这个参数我就瞎了眼，之前研究maven的时候就没好好记住，如果你install的jar依赖了很多东西，而jar文件里又没有pom文件，那就必须得用这个参数指定pom文件了，不然你install成功了，会有依赖它的时候，它依赖的东西啥都没有。

再比如`packaging`参数，如果你install的只是一个pom的话，比如你install了`struts2-core`，它的父工程`struts2-parent`就是一个pom，`struts2-core`继承了父工程的依赖，你得install一下`struts2-parent`，尽管它只是个pom，这时候`packaging`就应该是pom，而不是jar了。

唉，只能手动install真的难受，各种依赖，少什么我就得去公司内网下载回来install，整整一下午。。


