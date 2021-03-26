---
title: Win10下的wls中git状态不对的问题
excerpt: 'Windows Linux'
tags: [Windows, Linux]
categories: [OS]
comments: true
date: 2021-03-26 19:30:52
---

最近买了个thinkpad的扩展坞，换回了windows，用习惯了mac下的shell，好像有点回不去win的cmd了，不过问题不大，毕竟win自带linux子系统，以前用win的时候也是必备的，现在配合上巨硬出品的Terminal，香的很！

但是最近遇到一个问题，我在win环境下的git，显示`working tree clean`，但是在wls的ubuntu下却是几乎所有的文件，还以为wls下git有什么bug，差点就卸载了。

查看下差别：

```shell

kongzheng1993@LAPTOP-KCIF5AIF:/mnt/c/evilRat/workspace/lixiang/workspace/myProject$ git diff
diff --git a/.gitignore b/.gitignore
index 6b9542c0..96fc7230 100644
--- a/.gitignore
+++ b/.gitignore
@@ -1,30 +1,30 @@
-/target/
-!.mvn/wrapper/maven-wrapper.jar
-
-### STS ###
-.apt_generated
-.classpath
-.factorypath
-.project
-.settings
-.springBeans
-.sts4-cache
-
-### IntelliJ IDEA ###
-.idea
-*.iws
-*.iml
-*.ipr
-api/target
-service/target
-model/target
-pipeline/target
-### NetBeans ###
-/nbproject/private/
-/build/
-/nbbuild/
-/dist/
-/nbdist/
-/.nb-gradle/
-/.DS_store
-/api/src/main/resources/static/.DS_Store
+/target/^M
+!.mvn/wrapper/maven-wrapper.jar^M
+^M
+### STS ###^M
+.apt_generated^M
+.classpath^M
+.factorypath^M
+.project^M
+.settings^M
+.springBeans^M
+.sts4-cache^M
+^M
+### IntelliJ IDEA ###^M
+.idea^M
+*.iws^M
+*.iml^M
+*.ipr^M
+api/target^M
+service/target^M
+model/target^M
+pipeline/target^M
+### NetBeans ###^M
+/nbproject/private/^M
+/build/^M
+/nbbuild/^M
+/dist/^M
+/nbdist/^M
+/.nb-gradle/^M
+/.DS_store^M
+/api/src/main/resources/static/.DS_Store^M


```

所有的文件都是删掉后又新增，而且后面又`^M`， 看到这里其实就已经明白了。就是简简单单的win和linux、unix下的换行符不同。

- windows下：CRLF（表示句尾使用回车换行两个字符，即windows下的"\r\n"换行）
- unix下：LF（表示句尾，只使用换行）
- mac下：CR（表示只使用回车）

而Git处理换行的配置是`core.autocrlf`，可以是设置为true、false、inout

执行`git config --global core.autocrlf true`后，git仓库中所有的文件都会将crlf变成lf，也就不再不同了