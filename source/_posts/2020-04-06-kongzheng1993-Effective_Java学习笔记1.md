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

