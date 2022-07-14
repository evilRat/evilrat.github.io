---
title: NoSuchMethodError
excerpt: 'Java'
tags: [Java]
categories: [Java]
comments: true
date: 2022-07-14 18:30:52
---

NoSuchMethodError是一个运行时错误，在编译时一般不会出现。如果编译成功，就说明方法本身是存在的，方法所在的类也是存在的，而且可以正常的引用到。

如果编译没有报错，说明方法存在，但是运行时还是会报NoSuchMethodError，说明存在相同权限定名的类，而其中一个没有对应的方法。

比如有两个类：A和B，A中有一个方法test()，而B中没有方法test()。编译的时候使用的是A#test方法，编译通过。而运行时，使用的时B#test，而B类中没有test方法，所以就会报NoSuchMethodError。

解决办法就是排除其中一个依赖的jar，或者干脆不用这个方法了。这种问题比较难复现，全看jvm类加载的顺序（？？？不确定）
