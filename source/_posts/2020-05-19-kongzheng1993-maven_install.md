---
title: 记一次Maven冒险
excerpt: ''
tags: [maven]
categories: [maven]
comments: true
date: 2020-05-21 00:30:52
---

最近给框架升级，由于总部研发给的新框架是deploy到私有maven库，我们也都是在本地开发，到私有远程maven库网络是不通的，所以我也只能跳到那边把几个jar包下载，然后sftp到一台两边都通的服务器上，再搞到本地。。不就是mvn install嘛，我以为我可以系列，哈哈。。正好今天也查了很多资料，就记录下来，希望能帮到有缘人。。

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


## 更新，关于maven本地库

由于依赖太复杂，我服了，所以在可以连通maven远程库的机器Aclone了一份代码，make install，一波搞下来整个项目需要的所有依赖，然后把那边的本地库tar了个包通过跳板机搞到我本地了。想着一波把所有jar包复制到我本地库，该合并合并，该替换替换。。。

在机器A`make install`的时候不管我怎么设置`settings.xml`文件，它都去找maven中向仓库！！！一度要疯。。

后来找了几个博客看了下，了解到**所有的自定义pom都是继承自super pom的。也就是maven项目在需要下载metadata、pom和jar的时候会优先去中央仓库。** super pom文件内有配置`repositories`为中央仓库地址。

所以我们需要在项目的pom文件里添加`repositories`来覆盖super pom的配置，让maven直接去我们自己的服务器下载。

一顿操作，所有的依赖终于都下载到机器A了。sftp搞到我本地机器。

**但是，却怎么都导不进来！！！**

```text
[WARNING]  The POM for com.xxx:xxx:jar:xxx is missing, no dependency information available...
```

所有的包都是上面这种报错，我本地库明明都有，怎么就不行呢。。

原来当我们用maven来构建项目时，离线模式下，他会去我们的本地库找依赖的包，但是并不是有就会用，他会做一个`Verifying availability`，也就是校验下这些包是否可用。

**Maven根据`xxx.repositories`、`xxx.lastUpdated`和`xxx.sha1`来校验的**

- `***.repositories`，例如`_remote.repositories`文件会让maven优先从私服下载。
- `***.lastUpdated`，当远程库根据我们pom文件的描述找不到对应的资源，或者因为网络原因导致下载失败或中断，都会生成一个lastUpdated文件。
- `xxx.sha1`，和md5类似，生成的对应文件的加密摘要，sha1比md5长32位，更安全。

一顿操作，删除本地库目录下的所有`repositories`和`lastUpdated`文件删除：

```shell script
kongzheng1993@Evil:~/workspace/java/ngcrmpf_bj$ find ./ -name *.repositories | xargs rm
kongzheng1993@Evil:~/workspace/java/ngcrmpf_bj$ find ./ -name *.lastUpdated | xargs rm
```

成功了，皆大欢喜，之前看过maven实战，看来是啥也每记住啊～～
有时间还得看看！