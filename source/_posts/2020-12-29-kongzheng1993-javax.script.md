---
title: JDK自带的脚本引擎javax.script.ScriptEngine
excerpt: 'JDK'
tags: [JDK]
categories: [JDK]
comments: true
date: 2020-12-29 10:30:52
---

最近在做一个需求，因为时免测的功能，所以要做一个开关，如果出现问题，不能影响之前的业务。像这种开关，我之前的办法就是在配置中心搞一个变量，如果有问题就修改开关变量的值。

这次这个“开关“是一个同事来做的，他的方法是使用javax.script包下的脚本引擎，在配置中心搞一个javascript函数，将后台的java对象传入，执行js函数，得出结果。根据结果来决定后面代码的执行。

这个方法就很灵活了，可以传入参数来计算，也就是不仅可以开关，还能调整逻辑。在不重启服务的情况下，也能及时调整业务逻辑。

我都不知道jdk还有这个包呢。。。

原来从java6开始，jdk就开始支持脚本语言。

### 使用Java Script的方法

1. 创建一个ScriptEngineManager对象
2. 从管理器对象中获取ScriptEngine对象
3. 使用脚本引擎的eval()方法来执行脚本

```java

ScriptEngineManager manager = new ScriptEngineManager();
ScriptEngine engine = manager.getEngineByName("js");
String script = "print('hello world')";
try {
    engine.eval(script);
} catch (Exception e) {
    e.printStackTrace();
}
```

### 传递变量

ScriptEngine对象的put方法，可以传入参数。

```java
engine.put("a", 3);
engine.put("b", 4);

Object result = engine.eval("function add(a,b) {return a+b} add(a,b)");
System.out.println("result=" + result);

```

