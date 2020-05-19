---
title: jmap
excerpt: ''
tags: [jvm]
categories: [jvm]
comments: true
date: 2020-03-08 00:30:52
---

## jmap
打印进程，核心文件或远程调试服务器的共享对象内存映射或堆内存详细信息。此命令是实验性的，不受支持。

### 概要

```shell
jmap [ 选项 ] pid
jmap [ options ] 可执行 核心
jmap [ 选项 ] [ pid ] 服务器ID @] remote-hostname-or-IP
```
#### 选项
命令行选项。
- <无选择>
不使用任何选项时，该jmap命令将打印共享对象映射。对于目标JVM中加载的每个共享对象，将打印共享对象文件的开始地址，映射的大小和完整路径。此行为类似于Oracle Solaris pmap实用程序。

- -dump：[live，]format=b，file=filename
将Java堆以hprof二进制格式转储到filename。的live子选项是可选的，但是当指定时，仅在堆活动对象被倾倒。要浏览堆转储，可以使用jhat（1）命令读取生成的文件。

- -finalizerinfo
打印有关正在等待完成的对象的信息。

- -heap
打印所用垃圾收集的堆摘要，头配置和逐代堆使用情况。此外，还会打印实习字符串的数量和大小。

- -histo [：live]
打印堆的直方图。对于每个Java类，将打印对象数量，以字节为单位的内存大小以及完全限定的类名称。JVM内部类名称打印有星号（*）前缀。如果live指定了子选项，则仅计算活动对象。

- -clstats
打印Java堆的类加载器明智的统计信息。对于每个类加载器，将打印其名称，活动程度，地址，父类加载器以及已加载的类的数量和大小。

- -F
力。当pid不响应时，将此选项与jmap -dump或选项一起使用jmap -histo。该live子选项并不在此模式下支持。

- -h
打印帮助信息。

- -help
打印帮助信息。

- -Jflag
传递flag到jmap运行命令的Java虚拟机。

#### pid
要为其打印内存映射的进程ID。该进程必须是Java进程。要获取机器上运行的Java进程的列表，请使用jps（1）命令。

#### 可执行文件
生成核心转储的Java可执行文件。

#### 核心
要为其打印内存映射的核心文件。

#### 远程主机名或IP
远程调试服务器hostname或IP地址。参见jsadebugd（1）。

#### 服务器ID
当多个调试服务器在同一远程主机上运行时使用的可选唯一ID。

### 描述
该jmap命令显示指定进程，核心文件或远程调试服务器的共享对象内存映射或堆内存详细信息。如果指定的进程在64位Java虚拟机（JVM）上运行，则可能需要指定该-J-d64选项，例如：jmap -J-d64 -heap pid。

    注意：此实用程序不受支持，在以后的JDK版本中可能不可用。在dbgeng.dll不存在该文件的Windows系统上，必须安装Windows调试工具才能使这些工具正常工作。在PATH环境变量中应包含的位置jvm.dll所使用的目标进程或从中故障转储文件被产生，例如位置的文件：set PATH=%JDK_HOME%\jre\bin\client;%PATH%。

