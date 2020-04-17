---
title: Effectiv Java学习笔记（一）
excerpt: ''
tags: [Java]
categories: [Java]
comments: true
date: 2020-04-07 00:30:52
---

# 创建和销毁对象

## 1.考虑使用静态工厂方法代替构造器

我们平时创建对象都是new一个，而new会调用类对应的构造器。但是还有一个方法，就是**静态工厂方法**。

```java
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```

上面是一个典型的示例。

优点：

- **静态工厂方法有方法名。** 新建对象可以通过参数列表不同来调用不通的构造器，但是这样没有文字描述，调用者很难一下理解构造器做了哪些操作。
- **每次调用不必都创建一个新对象。** 这使得不可变类可以使用预先构造好的示例，或者将构建好的示例缓存起来，重复利用，从而避免创建不必要的重复对象。像上面的例子，就没有创建对象。这样有利于提高性能。单例的对象也能在这样的方法里得以实现。
- **可以返回原返回类型的任何子类型。** 可以返回对象，同时又不会使对象的类变成公有的。这种技术适用于基于接口的框架。
- **在创建参数化类型示例的时候，使代码变得更加简洁。**

缺点：

- **类如果不含共有的或者受保护的构造器，就不能被子类化。**
- **静态工厂方法与其他静态方法实际上没有任何区别。** 大家都知道用构造器实例化对象，却不能一眼看出这个静态方法是用来实例化对象的。

静态工厂方法惯用名：

- **valueOf**----主要用于类型转换。
- **of**----valueOf的简洁方案。
- **getInstance**----返回的示例是通过参数来描述的，对于Singleton来说，该方法应该没有参数，直接返回唯一实例。
- **newInstance**----返回一个新的实例。

## 2.遇到多个构造器参数时要考虑使用构造器

这里讲了把生活中一个对象抽象成类，我们实例化对象的三种设计思路：

- **重叠构造器模式** 编写各种成员变量组合的构造器，实例化时选择我们需要的来。但是参数太多的话，构造器的数量就太多了，难以编写。
- **JavaBean模式** 通过setter方法来设置参数，但是这样无法实现不可变对象，引发线程安全问题。
- **Builder模式** 通过一个Builder内部类，实现setter赋值，然后再构造函数里将Builder的成员赋给我们实例化的对象。

**Builder模式**：
```java

/**
 * @Description:
 * @Author: kongz
 * @Date: 2020/4/7 18:55
 */
public class People {

    private String name;
    private int age;
    private String gender;

    public People(Builder builder) {
        name = builder.name;
        age = builder.age;
        gender = builder.gender;
    }

    public static class Builder {

        private String name;
        private int age;
        private String gender;

        public Builder(String name) {
            this.name = name;
        }

        public Builder age(int age) {
            this.age = age;
            return this;
        }

        public Builder gender(String gender) {
            this.gender = gender;
            return this;
        }

        public People build() {
            return new People(this);
        }

    }

    @Override
    public String toString() {
        return this.name + "--" + this.age + "--" + this.gender;
    }

    public static void main(String[] args) {
        People people = new Builder("kongz").age(20).gender("男").build();
        System.out.println(people.toString());
    }

}
```

Builder模式的确也有不足，创建对象必须要先创建它的构造器，这也会浪费一些资源，虽然不是那么明显，但是比重叠构造器更加冗长。

## 3.用私有构造器或者枚举类型强化Singleton属性

在Java 1.5之前有两种方法实现Singleton，这两种方法都要把构造器保持为私有的，并导出共有的静态变量，以便客户端能够访问该类的唯一实例。

1. 第一种方法中，公有静态成员是个final域：

```java
//singleton with public final field
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();

    private Elvis(){ ... }

    public void leaveTheBuilding() {
        ...
    }
}
```

私有构造器仅被调用一次，用来实例化公有的静态final域Elvis.INSTANCE。由于外界无法调用构造器，所以保证了Elvis的全局唯一。**但是想有特权的客户端可以通过AccessibleObject.setAccessable方法，通过反射机制调用私有构造器。如果要抵御这种攻击，可以修改构造器，让它在被要求创建第二个实例的时候抛出异常。**

2. 第二种方法中，公有的成员是个静态工厂方法。

```java
//singleton with static factory
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis(){ ... }
    public static Elvis getInstance() { return INSTANCE; }

    public void leaveTheBuilding() { ... }
}
```

对于静态方法Elvis.getInstance方法每次调用都会返回同一个对象引用，所以永远不会创建其他的Elvis对象。

工厂方法的优势：

- **提供了灵活性：** 在不改变api的前提下，可以改变该类是否应该为singleton的想法，因为客户都都是通过`getInstance`方法获取对象的。
- **泛型**

上面两种方法要实现singleton类变成可序列化（Serializable），仅仅在生命中加上`implements Serializable`是不够的。**为了维护并保证Singleton，必须声明所有的实例域都是瞬时（transient）的，并提供一个`readResolve`方法，否则每次反序列化一个序列化的实例时，都会创建一个新的实例。**

```java
//readResolve method to preserve singleton property
private Object readResolve() {
    //Return the one true Elvis and let the garbage collector
    //take care of the Elvis impersonator
    return INSTANCE;
}
```

    在jdk中ObjectInputStream的类中有readUnshared（）方法，上面详细解释了原因。
    我简单描述一下，那就是如果被反序列化的对象的类存在readResolve这个方法，
    他会调用这个方法来返回一个“array”（我也不明白），然后浅拷贝一份，作为返回值，
    并且无视掉反序列化的值，即使那个字节码已经被解析。
    ————————————————
    版权声明：本文为CSDN博主「无始之名」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
    原文链接：https://blog.csdn.net/u011499747/java/article/details/50982956

一个单例模式：

```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis(){ ... }
    public static Elvis getInstance() { return INSTANCE; }

    public void leaveTheBuilding() { ... }
    private Object readResolve() {
        return INSTANCE;
    }
}
```
