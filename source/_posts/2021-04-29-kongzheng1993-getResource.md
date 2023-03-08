---
title: java.lang.Class#getResource浅析
excerpt: 'JDK'
tags: [JDK]
categories: [JDK]
comments: true
date: 2021-04-29 19:30:52
---

最近工程重构，原来单module的maven项目拆分出来api、web、model等多个module，之前我们有一个生成pdf的类`PdfGenerator`，需要获取字体文件：

```java
private static String fontPath = PdfGenerator.class.getResource("/") + "static/pdf/XXXXX.TTF";
```

但是因为项目结构的调整，启动类再`app`模块下，其他的模块都被`app`模块依赖，`PdfGenerator`这个类和对应的字体文件都被挪到了`common`模块下，而上面的`getResoutce("/")` 则会返回classpath，也就是app的`target/classes`目录，显然就找不到`common`模块`resource`目录（当然，编译后应该在`target/classes/static/pdf/`中）下的字体文件了。

### 思考

多module的maven工程，app依赖其他module，这些依赖都是其他module package的jar包，现在要取到字体文件，就需要去common的jar包中取。当然，我们如果再IDE中debug，结果是不对的：

<img src="IDE.png"/>

因为这时的getResource会去所谓的target目录下找，而我们springboot项目通过`java -jar`启动，并不会解压这些jar包，也就不会存在这些所谓的目录。

`java.lang.Class#getResource`做了什么？

```java
    public java.net.URL getResource(String name) {
        name = resolveName(name);
        ClassLoader cl = getClassLoader0();
        if (cl==null) {
            // A system class.
            return ClassLoader.getSystemResource(name);
        }
        return cl.getResource(name);
    }
```

通过源码可以看到，先处理name，如果是`/`开头，说名要从根目录查找，就去掉第一位，如果不以`/`开头，就获取当前类的路径，然后拼接在name之前，也就是从当前类所在目录查找。然后获取`ClassLoader`，之后调用`ClassLoade`r的`getResource`方法，我们有时候会看到`xxx.class.getClassLoader().getResource()`，就是手动直接调用`ClassLoader`的`getResource`方法。

### 解决

尝试`java -jar`启动，使用IDEA remote debug查看。`PdfGenerator.class.getResource("/")`结果如下：

```
jar:file:/my-workspace/project-name/app/target/app.jar!/BOOT-INF/classes!/
```

也就是获取到了`jar`包里面的目录。

但是我们要取的是jar包里的jar包里的文件，也就是app里的common里的文件。

`PdfGenerator.class.getResource("/static/pdf/font.TTF")` 结果如下：

```
jar:file:/my-workspace/project-name/app/target/app.jar!/BOOT-INF/lib/common.jar!/static/pdf/font.TTF
```

`PdfGenerator.class.getResource("/static")` 结果如下：

```
jar:file:/my-workspace/project-name/app/target/app.jar!/BOOT-INF/lib/common.jar!/static
```

对比可以发现，`PdfGenerator.class.getResource("/")` 还是返回app.jar里的classpath，而`PdfGenerator.class.getResource("/static")`就会找到app.jar的`/BOOT-INF/lib/`下的`common.jar`里的`static`目录，也就是我们传入`/status`，resoveName方法会删除开头的`/`，然后开始从app的classpath开始查找，一直找到`/BOOT-INF/lib/`下的`common.jar`里有一个`static`的资源（目录也算资源），于是返回。

这样我们的问题就解决了。

这里其实后面还遇到一个问题，既然`PdfGenerator.class.getResource("/static/pdf/font.TTF")` 可以直接拿到资源，我还为什么`PdfGenerator.class.getResource("/static") + "pdf/font.TTF"`？

这里还有一个坑，就是URL的`getPath()`方法，而且URL还重写了`toString()`

```
getPath():
file:/my-workspace/project-name/app/target/app.jar!/BOOT-INF/lib/common.jar!/static/pdf/font.TTF

toString():
jar:file:/my-workspace/project-name/app/target/app.jar!/BOOT-INF/lib/common.jar!/static/pdf/font.TTF
```

少了一个`jar:`，使用`toString()`可以得到的字符串才能用于iText指定字体文件。