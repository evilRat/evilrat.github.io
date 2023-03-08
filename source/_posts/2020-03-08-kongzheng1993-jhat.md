---
title: jhat
excerpt: ''
tags: [jvm]
categories: [jvm]
comments: true
date: 2020-03-08 00:30:52
---

# jhat

分析Java堆。此命令是实验性的，不受支持。

## 概要

```jhat [ options ] heap-dump-file```

## 选项

命令行选项。

- -stack false | true
关闭跟踪对象分配调用堆栈。如果堆转储中没有分配站点信息，则必须将此标志设置为false。默认值为true。

- -refs false | true
关闭对对象引用的跟踪。默认值为true。默认情况下，将为堆中的所有对象计算后向指针，即指向指定对象的对象，例如引用程序或传入引用。

- -port 端口号
设置jhatHTTP服务器的端口。默认值为7000。

- -exclude exclude-file
指定一个文件，该文件列出了应从可达对象查询中排除的数据成员。例如，如果文件列出java.lang.String.value，则无论何时o计算从特定对象可访问的对象列表，java.lang.String.value都不会考虑涉及字段的引用路径。

- -baseline exclude-file
指定基线堆转储。两个堆转储中具有相同对象ID的对象都标记为不是新对象。其他对象被标记为新对象。这对于比较两个不同的堆转储很有用。

- -debug int
设置此工具的调试级别。级别0表示没有调试输出。为更多详细模式设置较高的值。

- -version
报告发布编号并退出

- -h
显示帮助消息并退出。

- -help
显示帮助消息并退出。

- -Jflag
传递flag到在其jhat上运行命令的Java虚拟机。例如，-J-Xmx512m使用最大堆大小为512 MB。

## 堆转储文件

要浏览的Java二进制堆转储文件。对于包含多个堆转储的转储文件，您可以通过#<number>在文件名后附加名称来指定文件中的哪个转储myfile.hprof#3。

## 描述

该jhat命令解析Java堆转储文件并启动Web服务器。该jhat命令使您可以使用自己喜欢的Web浏览器浏览堆转储。该jhat命令支持预先设计的查询，例如显示已知类的所有实例MyClass以及对象查询语言（OQL）。OQL与SQL相似，除了查询堆转储。可从jhat命令显示的OQL帮助页面获得OQL帮助。使用默认端口，可以从http://localhost:7000/oqlhelp /获得OQL帮助。

有几种方法可以生成Java堆转储：

- 使用该jmap -dump选项在运行时获取堆转储。参见jmap（1）。
- 使用该jconsole选项HotSpotDiagnosticMXBean在运行时获取堆转储。请参阅jconsole（1）和HotSpotDiagnosticMXBean接口说明，网址为
http://docs.oracle.com/javase/8/docs/jre/api/management/extension/com/sun/management/HotSpotDiagnosticMXBean.html
- OutOfMemoryError通过指定-XX:+HeapDumpOnOutOfMemoryErrorJava虚拟机（JVM）选项引发时，将生成堆转储。
- 使用hprof命令。请参阅HPROF：堆/ CPU分析工具，网址为：
http://docs.oracle.com/javase/8/docs/technotes/samples/hprof.html
