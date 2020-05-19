---
title: Guava
excerpt: ''
tags: [Guava]
categories: [Java]
comments: true
date: 2020-03-06 00:30:52
---

# Guava（一）
## Guava是什么
Guava项目包含一些我们在基于Java的项目中依赖的Google核心库：集合，缓存，原语支持，并发库，通用批注，字符串处理，I/O等。这些工具中的每一种确实每天都会被Google员工用于生产服务中。

更详细的介绍可以去[github/guava](https://github.com/google/guava/wiki)的Wiki了解。

## 引入Guava
```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>19.0</version>
</dependency>
```

## Guava工具类
### 集合
普通的集合：
```java
List<String> list = Lists.newArrayList();
Set<String> set = Sets.newHashSet();
Map<String, String> map = Maps.newHashMap();
```
不可变集合：
```java
ImmutableList<String> iList = ImmutableList.of("a", "b", "c");
ImmutableSet<String> iSet = ImmutableSet.of("e1", "e2");
ImmutableMap<String, String> iMap = ImmutableMap.of("k1", "v1", "k2", "v2");
```
    什么是不可变？
    - 在多线程操作下，是线程安全的。
    - 所有不可变集合会比可变集合更有效的利用资源。
    - 中途不可改变

#### 一个简单的对比
我们搞一个Map中嵌套List的对象要几步？
```java
Map<String,List<Integer>> map = new HashMap<String,List<Integer>>();
List<Integer> list = new ArrayList<Integer>();
list.add(1);
list.add(2);
map.put("test", list);
```
如果用了Guava呢？
```java
Multimap<String,Integer> mapM = ArrayListMultimap.create();
mapM.put("test",1);
mapM.put("test",2);
```
哪个方便？
    
    代码量少了并不是简简单单的节省时间而已，而是减少了人为错误的机会，少一行代码就少一点犯错误的可能！

### 字符串连接
连接多个字符串，追加到StringBuilder
```java
StringBuilder stringBuilder = new StringBuilder("你好！，");
// 字符串连接器，以|为分隔符，同时去掉null元素
Joiner joiner1 = Joiner.on("|").skipNulls();
// 构成一个字符串jim|jack|kevin并添加到stringBuilder
stringBuilder = joiner1.appendTo(stringBuilder, "jim", "jack", null, "kevin");
System.out.println(stringBuilder); 
// 执行结果：嗨，jim|jack|kevin
```
可以看到这里过滤了null

### Map&字符串互转
```java
Map<String, String> testMap = Maps.newLinkedHashMap();
testMap.put("Cookies", "12332");
testMap.put("Content-Length", "30000");
testMap.put("Date", "2018.07.04");
testMap.put("Mime", "text/html");
// 用:分割键值对，并用#分割每个元素，返回字符串
String returnedString = Joiner.on("#").withKeyValueSeparator(":").joi(testMap);
System.out.println(returnedString);
// 执行结果：Cookies:12332#Content-Length:30000#Date:2018.07.04#Mime:text/html
```

```java
// 接上一个，内部类的引用，得到分割器，将字符串解析为map
Splitter.MapSplitter ms = Splitter.on("#").withKeyValueSeparator(':');
Map<String, String> ret = ms.split(returnedString);
for (String it2 : ret.keySet()) {
    System.out.println(it2 + " -> " + ret.get(it2));
}
```

### 字符串工具类
```java
System.out.println(Strings.isNullOrEmpty("")); // true
System.out.println(Strings.isNullOrEmpty(null)); // true
System.out.println(Strings.isNullOrEmpty("hello")); // false
// 将null转化为""
 System.out.println(Strings.nullToEmpty(null)); // "" 
// 从尾部不断补充T只到总共8个字符，如果源字符串已经达到或操作，则原样返回。类似的有padStart
System.out.println(Strings.padEnd("hello", 8, 'T')); // helloTTT
```

### 字符匹配器CharMatcher
```java
// 空白回车换行对应换成一个#，一对一换
String stringWithLinebreaks = "hello world\r\r\ryou are here\n\ntake it\t\t\teasy";
String s6 = CharMatcher.BREAKING_WHITESPACE.replaceFrom(stringWithLinebreaks,'#');
System.out.println(s6); 
//执行结果：hello#world###you#are#here##take#it###easy
```

### 连续空白替换成一个字符
```java
// 将所有连在一起的空白回车换行字符换成一个#，倒塌
String tabString = "  hello   \n\t\tworld   you\r\nare             here  ";
String tabRet = CharMatcher.WHITESPACE.collapseFrom(tabString, '#');
System.out.println(tabRet); 
```

### 去掉前后空白和缩成一个字符
```java
// 在前面的基础上去掉字符串的前后空白，并将空白换成一个#
String trimRet = CharMatcher.WHITESPACE.trimAndCollapseFrom(tabString, '#');
System.out.println(trimRet);
//执行结果： hello#world#you#are#here
```

### 只保留数字
```java
String letterAndNumber = "1234abcdABCD56789";
// 保留数字
String number = CharMatcher.JAVA_DIGIT.retainFrom(letterAndNumber);
System.out.println(number);//123456789
```

## 总结
使用Guava可以极大减少我们的代码量，减少犯错机会，提高工作效率。这次我们先介绍这些，下一篇我们深入Guava Caching！