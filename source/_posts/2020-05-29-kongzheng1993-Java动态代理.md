---
title: Java动态代理
excerpt: ''
tags: [java]
categories: [java]
comments: true
date: 2020-05-29 00:30:52
---

## 静态代理

静态代理是我们直接定义在代码里的，也就是手动创建代理类，在调用原有类方法的前后加入逻辑，实现增强。但是这种方式违反了软件设计的开闭原则。

**开闭原则（开放/封闭原则）**

在面向对象编程领域中，开闭原则规定“软件中的对象（类，模块，函数等等）应该对于扩展是开放的，但是对于修改是封闭的”，这意味着一个实体是允许在不改变它的源代码的前提下变更它的行为。该特性在产品化的环境中是特别有价值的，在这种环境中，改变源代码需要代码审查，单元测试以及诸如此类的用以确保产品使用质量的过程。遵循这种原则的代码在扩展时并不发生改变，因此无需上述的过程。



## 动态代理

**定义：** 给目标对象提供一个代理对象，并由代理对象控制对目标对象对引用；
**目的：**
    
1. 通过引入代理对象的方式来间接访问目标对象，防止直接访问目标对象给系统带来的不必要的复杂性。
2. 通过代理对象对原有对业务增强。

**动态代理中几个概念：**
- 接口对象： 声明了真实对象和代理对象的公共接口
- 真实对象： 代理对象所代表的真实对象，最终被引用的对象
- 代理对象： 包含真实对象从而操作真实对象，相当于访问者与真实对象之间的中介


### JDK动态代理

- Proxy （java.lang.reflect.Proxy） 提供静态方法来创建动态代理类和实例，同时也是所有动态代理类的父类。通过静态方法`Proxy.newProxyInstance(真实对象.getClass().getClassLoader(), 真实对象.getClass().getInterfaces(), this)`，这里就声明了代理类要使用什么类加载器，继承什么接口，还有this（进行增强的InvocationHandler）。
- InvocationHandler （java.lang.reflect.InvocationHandler） 接口， 它只有一个invoke方法，通过实现这个方法来增加自己的逻辑。我们通过重写invoke方法，做一些增强，然后`method.invoke()`,通过反射来调用真实对象的方法。


```java
public class SimpleJDKDynamicProxyDemo {
    static interface IService {
        public void sayHello();
    }

    static class RealService implements IService {
        @Override
        public void sayHello() {
            System.out.println("hello");
        }
    }

    static class SimpleInvocationHandler implements InvocationHandler {
        private Object realObj;

        public SimpleInvocationHandler(Object realObj) {
            this.realObj = realObj;
        }

        @Override
        public Object invoke(Object proxy, Method method,
                             Object[] args) throws Throwable {
            System.out.println("entering " + method.getName());
            Object result = method.invoke(realObj, args);
            System.out.println("leaving " + method.getName());
            return result;
        }
    }

    public static void main(String[] args) {
        IService realService = new RealService();
        IService proxyService = (IService) Proxy.newProxyInstance(
                IService.class.getClassLoader(), new Class<?>[]{IService.class}, new SimpleInvocationHandler(realService));
        proxyService.sayHello();
    }
}
```

在上面的例子中，IService和RealService是我们原有的。通过Proxy.newProxyInstance()来创建代理对象。

```java
  public static Object newProxyInstance(ClassLoader loader,Class<?>[] interfaces, InvocationHandler h)
```

**这里要注意的一点：**
我们可以把获取到的对象强转为`Proxy.newProxyInstance()`参数中interfaces中的任意一个接口类型，却不能把它强转为某个类类型，比如RealService，即使它实际去代理的对象就是RealServcie实例。

SimpleInvocationHandler实现了InvocationHandler，它的构造方法接收一个参数realObj表示被代理对象。invoke方法处理所有的接口调用。invoke方法有三个参数：

- proxy表示代理对象本身。
- method表示正在被调用的方法。
- args表示方法的参数。

在SimpleInvocationHandler的invoke实现中，我们调用了method的invoke方法，传递了实际对象realObj作为参数，达到了调用实际对象对应方法的目的，调用方法的前后，我们加入了自己的代码。

**这里要注意的一点：**
不要将代理对象传入invoke方法：`Object result = method.invoke(proxy, args);` 这将造成死循环！！因为proxy就是代理对象，调用它的对应方法又会调到SimpleInvocationInvocationHandler的invoke方法。

**Proxy.newProxyInstance()都干了什么？**

```java
Class<?> proxyCls = Proxy.getProxyClass(IService.class.getClassLoader(), new Class<?>[] { IService.class });
Constructor<?> ctor = proxyCls.getConstructor( new Class<?>[] { InvocationHandler.class });
InvocationHandler handler = new SimpleInvocationHandler(realService);
IService proxyService = (IService) ctor.newInstance(handler);
```

1. 通过`Proxy.getProxyClass()`创建代理类定义，类定义会被缓存；
2. 通过代理类都构造方法，构造方法有一个InvocationHandler类型的参数；
3. 创建InvocationHandler对象；
4. 创建代理对象。

这里`Proxy.getProxyClass()`方法需要两个参数，一个是ClassLoader，另一个是接口数组。它会动态生成一个类，类名以$Proxy开头，后面跟一个数字。

通过下面的参数可以在编译的时候保存生成的代理类的代码。
```shell script
java -Dsun.misc.ProxyGenerator.saveGeneratedFiles=true shuo.laoma.dynamic.c86.SimpleJDKDynamicProxyDemo
```

```java
final class $Proxy0 extends Proxy implements SimpleJDKDynamicProxyDemo.IService {
    private static Method m1;
    private static Method m3;
    private static Method m2;
    private static Method m0;

    public $Proxy0(InvocationHandler paramInvocationHandler) {
        super(paramInvocationHandler);
    }

    public final boolean equals(Object paramObject) {
        return ((Boolean) this.h.invoke(this, m1,
                new Object[]{paramObject})).booleanValue();
    }

    public final void sayHello() {
        this.h.invoke(this, m3, null);
    }

    public final String toString() {
        return (String) this.h.invoke(this, m2, null);
    }

    public final int hashCode() {
        return ((Integer) this.h.invoke(this, m0, null)).intValue();
    }

    static {
        m1 = Class.forName("java.lang.Object").getMethod("equals", new Class[]{Class.forName("java.lang.Object")});
        m3 = Class.forName("laoma.demo.proxy.SimpleJDKDynamicProxyDemo$IService").getMethod("sayHello", new Class[0]);
        m2 = Class.forName("java.lang.Object").getMethod("toString", new Class[0]);
        m0 = Class.forName("java.lang.Object").getMethod("hashCode", new Class[0]);
    }
}
```

$Proxy0是Proxy的子类，有一个构造方法，接受一个InvocationHandler类型的参数，这个构造方法会调用父类Proxy的构造方法，将InvocationHandler对象传入，保存在父类成员变量h中。实现了IService接口，对于每个方法，比如例子中的`sayHello`，它将调用InvocationHandler的invoke方法，对于Object中的方法--heshcode、equals等方法，同样转发给了InvocationHandler对象来处理。

上面的分析可以看出，这个代理类与被代理的对象没有任何关系，与InvocationHandler的具体实现也没有关系，而主要与接口数组有关，给定这个接口数组，它动态创建每个接口的实现代码，实现的方式就是转发给InvocationHandler，与被代理对象的关系以及对它的调用由Invocationhandler实现管理。

