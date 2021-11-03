---
title: Java异常和错误
excerpt: 'Java'
tags: [Java]
categories: [Java]
comments: true
date: 2021-11-03 16:30:10
---


copy from [http://doc.okbase.net/foxty/archive/120046.html](http://doc.okbase.net/foxty/archive/120046.html)


# Java的异常体系
- Throwable: Java中所有异常和错误类的父类。只有这个类的实例（或者子类的实例）可以被虚拟机抛出或者被java的throw关键字抛出。同样，只有其或其子类可以出现在catch子句里面。
- Error: Throwable的子类，表示严重的问题发生了，而且这种错误是不可恢复的。
- Exception: Throwable的子类，应用程序应该要捕获其或其子类（RuntimeException例外），称为checked exception。比如：IOException, NoSuchMethodException…
- RuntimeException: Exception的子类，运行时异常，程序可以不捕获，称为unchecked exception。比如：NullPointException.


## 应该catch什么

其实只要是Throwable和其子类都是可以throw和catch的，那么如果在需要统一处理异常的地方，我们应该catch (Throwable th) 还是 catch (Exception)呢？

这两种处理的区别在于，catch throwable会把Error和其他继承Throwable的类捕捉到。而catch Exception只会捕捉Exception极其子类，捕捉的范围更小。先不考虑有其他的类继承了Throwable的情况下（附录A），第一种catch相当于比第二种catch多捕捉了把Error和其子类。

那么究竟Error是否需要捕捉呢？**JDK中Error类的的注释（如下）里提到过，Error是一种严重的问题，应用程序不应该捕捉它。**

    An Error is a subclass of Throwable that indicates serious problems that a reasonable application should not try to catch. Most such errors are abnormal conditions. The ThreadDeath error, though a “normal” condition, is also a subclass of Error because most applications should not try to catch it.

    A method is not required to declare in its throws clause any subclasses of Error that might be thrown during the execution of the method but not caught, since these errors are abnormal conditions that should never occur.

Java Lanuage Spec 7 中也提到：Error继承自Throwable而不是继承自Exception，是为了方便程序可以使用 “catch (Exception)“来捕捉异常而不会把Error也捕捉在内，因为Exception发生后可以进行一些恢复工作的，但是Error发生后一般是不可恢复的。

    The class Error is a separate subclass ofThrowable, distinct from Exception in the class hierarchy, to allow programs to use the idiom “} catch (Exception e) { " (§11.2.3) to catch all exceptions from which recovery may be possible without catching errors from which recovery is typically not possible.

已经不难看出，Java本身设计思路就是希望大家catch Exception就足够了，如果有Error发生，catch了也不会有什么作用（附录B）。

## 引申，如何设计异常体系？

如何设计异常体系要根据你的项目的情况，类库框架，应用程序的异常设计方式都会有一些区别。下面简单谈谈个人对异常设计的一些看法

### 类库/框架

继承RuntimeException扩展一个新的异常作为整个类库的异常基类。这个异常应该可以满足大部分类库对异常的要求。
在实现中，在任何需要捕捉checked exception的地方都会把异常统一转化成这个新的异常。
对于有特殊需求，需要自定义异常的，就通过继承这个基类来实现自定义异常。
不对异常记录log（交给上层来处理）
案例
fastjson

spring
自定义异常比较多，不过都是继承自org.springframework.core.NestedRuntimeException，而这个异常也是继承自RuntimeException。

### 应用程序

设计上和框架异常类似，只是在捕捉checked exception的时候需要log
如果需要根据异常进行不同的处理，建议给自定义异常增加一个ERROR_CODE字段，这样无论在服务器还是客户端都可以根据不同的ERROR_CODE进行对应的处理。但是出现这种情况的时候，应该需要考虑一下设计思路了，一般来讲根据异常来决定业务流程不是一个好的设计方案。

# 附录A：是否应该直接继承Throwable来扩展新的异常？
个人认为异常都应该继承自Exception或者RuntimeException，而且Java本身对Exception和Error的规划就很清晰了，Java自己类库中没有异常是直接继承自Throwable的。

# 附录B：Error可以catch吗？ 可以catch了后做些其他处理吗？
Error是可以catch的，而且也可以向常规Exception一样被处理，而且就算不捕捉的话也只是导致当前线程挂掉，其他线程还是可以正常运行，如果有需要的话捕捉Error之后也可以做些其他处理。但是Error是一种系统内部的错误，这种错误不像Exception一样是可能是程序和业务上的错误是可以恢复的。

假设进行网络连接操作的时候，IOException 发生了，可能是网络中断，我可以再尝试几次。

假设OutOfMemoryError发生了，就算被捕捉了，可以有什么手段让程序正常运行下去吗？
假设ExceptionInInitializerError发生了，类无法被正常初始化，这个是可以通过捕捉来恢复的吗？