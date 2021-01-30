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

## Stream

流（Stream）： 是数据通道，用于操作数据源（这里指的是集合、数组等）所生成的元素序列。

- Stream不会存储元素
- Stream不会改变源对象，而是生成一个持有结果的新Stream
- Stream操作是延迟执行的。这意味着他们会等到需要结果的时候再执行

### Stream的操作步骤

1. 创建Stream
2. 中间操作
3. 终止操作，产生结果。

```java

//1. 从集合Colllection提供的stream()或parallelStream()创建流
List<String> list = new ArrayList<>();
Stream<String> stream1 = list.stream();
//2. 通过数组Arrays的静态方法stream()获取数组流
String[] str = new String[10];
Stream<String> stream2 = Arrays.stream(str);
//3. 通过Stream类中的静态方法of()
Stream<String> stream3 = Stream.of("1", "2", "3");
//4. 创建无限流
//迭代
Stream<Integer> stream4 = Stream.iterate(0, (x) -> x + 2);
stream4.limit(10).forEach(System.out::println);
//生成
Stream.generate(() -> Math.random()).limit(5).forEach(System.out::println);

```


