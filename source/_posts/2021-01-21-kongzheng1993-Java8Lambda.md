---
title: Java8 Lambda
excerpt: 'Java'
tags: [Java]
categories: [Java]
comments: true
date: 2021-01-21 10:30:52
---

## 四大内置核心函数式接口

1. Consumer<T>： 消费型接口

```java

void accept(T t);

```

2. Supplier<T>： 供给型接口

```java

T get();

```

3. Function<T, R>：函数型接口

```java

R apply(T t);

```

4. Predicate<T>： 断言型接口

```java

boolean test(T t);

```


## 方法引用

如果Lambda体中的内容已经有方法实现了，可以使用方法引用

### 语法格式

1. 对象::实例方法名
2. 类::静态方法名
3. 类::实例方法名

