---
layout: post
title: "java反射的学习笔记"
date: 2017-03-08
excerpt: "java reflect"
tags: [java,reflect,Class,对象，Class对象]
comments: true
---


## java反射学习笔记


复习java反射机制，发现我一直没理解Class的对象这个含义，Class是一个类名

```java

import java.lang.reflect.Method;

/**
 * Created by evilrat on 3/8/17.
 */
public class GetMethods {

    public static void main(String [] args){

        Students s = new Students();
        Class c = s.getClass();
        Method m = null;
        try{
            m = s.getClass().getDeclaredMethod("getSex",null);
            System.out.println("Students类中有方法："+m);

        }catch (Exception e){
            System.out.println(e);
        }
    }
}


```

这里先声明了一个Students类的对象，然后调用s.getClass()获得了s的类的对象。这个类的对象是Class类型的。



















<html>
<div class="ds-thread" data-thread-key="http://kongzheng1993.github.io/kongzheng1993-mto/" data-title="mto" data-url="http://kongzheng1993.github.io/kongzheng1993-mto/"></div>
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


