---
layout: post
title: "String str"
date: 2016-07-11
excerpt: "code or die"
tags: [String, new]
comments: true
---


String str=new String("abc"); 跟着这段代码之后，我们会想到一个问题，就是这行代码究竟创建了几个String对象呢？
答案是2个。
String str只是定义了一个名为str的String类型的变量，因此它并没有创建对象；=是对变量str进行初始化，将某个对象的引用（或者叫句柄）赋值给它，显然也没有创建对象；现在只剩下了new String("abc")了。那么，new String("abc")什么又能被看作成“abc”和
new String()呢？
这就要了解一下String的构造器：
public String (String original){
	//other code...
}
平常我们创建一个类的实例的方法有两种：
1.使用new创建对象；
2.调用Class类里面的newInstance方法，利用反射机制创建对象。

我们正是使用new调用了String类的上面那个构造器方法创建了一个对象，并将它的引用赋值给了str变量。同时我们注意到，被调用的构造器方法接受的参数也是一个String对象，这个对象正是"abc"。由此我们又要引入另外一种创建String对象的方式的讨论——引号内包含文本。

这种方式是String特有的，并且它与new的方式存在很大区别。  

String str="abc";  

毫无疑问，这行代码创建了一个String对象。  

String a="abc";  String b="abc";   那这里呢？

答案还是一个。  

String a="ab"+"cd";   再看看这里呢？

答案是三个。
说到这里，我们就需要引入对字符串池相关知识的回顾了。  

在JAVA虚拟机（JVM）中存在着一个字符串池，其中保存着很多String对象，并且可以被共享使用，因此它提高了效率。由于String类是final的，它的值一经创建就不可改变，因此我们不用担心String对象共享而带来程序的混乱。字符串池由String类维护，我们可以调用intern()方法来访问字符串池。  

我们再回头看看String a="abc";，这行代码被执行的时候，JAVA虚拟机首先在字符串池中查找是否已经存在了值为"abc"的这么一个对象，它的判断依据是String类equals(Object obj)方法的返回值。如果有，则不再创建新的对象，直接返回已存在对象的引用；如果没有，则先创建这个对象，然后把它加入到字符串池中，再将它的引用返回。因此，我们不难理解前面三个例子中头两个例子为什么是这个答案了。

 

只有使用引号包含文本的方式创建的String对象之间使用“+”连接产生的新对象才会被加入字符串池中。对于所有包含new方式新建对象（包括null）的“+”连接表达式，它所产生的新对象都不会被加入字符串池中，对此我们不再赘述。因此我们提倡大家用引号包含文本的方式来创建String对象以提高效率，实际上这也是我们在编程中常采用的。

 

栈（stack）：主要保存基本类型（或者叫内置类型）（char、byte、short、int、long、float、double、boolean）和对象的引用，数据可以共享，速度仅次于寄存器（register），快于堆。 
堆（heap）：用于存储对象










<html>
<div class="ds-thread" data-thread-key="http://kongzheng1993.github.io/kongzheng1993-String_Original/" data-title="String_Original" data-url="http://kongzheng1993.github.io/kongzheng1993-String_Original/"></div>
<!-- 多说评论框 end -->
<!-- 多说公共JS代码 start (一个网页只需插入一次) -->
<script type="text/javascript">
var duoshuoQuery = {short_name:"kongzheng1993"};
    (function() {
        var ds = document.createElement('script');
        ds.type = 'text/javascript';ds.async = true;
        ds.src = (document.location.protocol == 'https:' ? 'https:' : 'http:') + '//static.duoshuo.com/embed.js';
        ds.charset = 'UTF-8';
        (document.getElementsByTagName('head')[0] 
         || document.getElementsByTagName('body')[0]).appendChild(ds);
    })();
</script>
</html>
