---
layout: post
title:  "Servlet"
date:   2016-07-26
excerpt: "servlet"
tag: [servlet]
comments: true
---

## WHAT

当一个请求到了web服务器，接收请求和响应请求是web服务器完成的，可是处理请求呢？接收和响应都是固定的东西，但是处理请求是包含业务逻辑在里面的，需要我们自己处理，所以就抽取出来了Servlet让我们来完成请求的处理，当然请求处理咱么又拆分出三层架构：servlet+service+dao。每个servlet来处理对应映射的url来的请求，很多servlet需要统一管理，所以tomcat还是一个servlet容器。

后来spring家族出现，servlet就退居幕后了。现在咱们用的springMVC其实核心组件DispatcherServlet本质上就是Servlet，只是在原来的HttpServlet基础上封装了一层。

## Servlet是怎么工作的

![Servlet接口](servlet接口.jpg)

Servlet是一个接口，其中有init()，getServletConfig()，service()，getServiceInfo()，destroy()方法。Tomcat已经替我们完成了大部分工作，并且传入了三个对象ServletConfig、ServletRequest、ServletResponse。

!(servlet实例化过程.jpg)

1. ServletConfig是servlet配置，也就是我们在web.xml里的配置。
2. Request/Response，也就是请求和响应。tomcat在收到http请求后，tomcat就解析了报文中的字段，然后封装进了Request对象。所以我们通过调用request对象的一些get方法就能获取请求的信息。response在tomcat传给servlet的时候还是空的。servlet逻辑处理后得到结果，最后通过response.write()写入response，tomcat或在servlet处理结束后拿到response组装成http响应发给客户端。

Servlet接口五个方法，init、service、destroy是声明周期方法。init和destroy各自执行一次，即创建和销毁。而service是在每次有新请求的时候被调用，也就是我们写业务代码的地方。

如果我们直接实现Servlet接口，会很麻烦，要自己处理请求类型，所以提供了抽象类GenericServlet：

1. 提升了init方法中原本形参的servletConfig对象的作用域，方便其他方法使用
2. init方法中调用了一个init空参方法，如果我们希望servlet创建时做一些自定义的操作，可以继承GenericServlet后覆盖init空参方法。
3. 由于其他方法内也可以使用ServletConfig，于是写了一个getServletContext方法
4. service是没有实现的。

向下找，会找到HttpServlet抽象类，他继承了GenericServlet。虽然他是一个抽象类，但是他并没有抽象方法。HttpServlet类完成了请求方法判断。
一个类被声明为抽象类，一般有两个原因：

1. 有抽象方法。
2. 没有抽象方法，但是不希望别人直接实例化使用。

所以这里仅仅是为了不让我们直接使用httpServlet。

HttpServlet的doGet和doPost方法的默认实现是直接返回405，也就是请求不支持。所以我们要重写这两个方法。

设计模式： Filter用到了责任链模式，Listener用到了观察者模式，Servlet使用的就是模板方法模式。

所以到这里，我们写servlet需要重写的方法就是doGet和doPost。
