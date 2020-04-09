---
layout: post
title: "关于comparator和comparable"
date: 2016-07-26
excerpt: "getRequestDispatcher,forword,sendRedirect"
tags: [re]
comments: true
---


## 关于comparator和comparable

### comparator

Comparator强行对某个对象collection进行整体排序的比较函数，可以将Comparator传递给Collections.sort或Arrays.sort。
也就是说Comeparator是需要用sort方法调用的。

```


import java.util.*;

/**
 * Created by evilrat on 3/9/17.
 */
public class CollectionTest {

    static List<Demo> d = new ArrayList<Demo>();
    static Random random = new Random();
    static CompareterTool ct= new CompareterTool();
    public static void main(String [] args){
        for (int i = 0; i < 10;i ++){
            d.add(new Demo(random.nextInt(10)));
        }
        //排序前输出
        for (Iterator<Demo> it = d.iterator(); it.hasNext();){
            System.out.print(it.next().getAge()+"\t");
        }
        System.out.print("\n");
        //排序
        Collections.sort(d,ct);
        //排序后输出
        for (Iterator<Demo> it = d.iterator(); it.hasNext();){
            System.out.print(it.next().getAge()+"\t");
        }
    }
}


```


这里是使用Collections.sort方法调用了我们预先写好的比较器。以下是比较器的定义：

```


import java.util.Comparator;

/**
 * Created by evilrat on 3/9/17.
 */
public class CompareterTool implements Comparator<Demo> {
        @Override
        public int compare(Demo o1, Demo o2) {
            if (o1.age > o2.age){
                return 1;
            }else
                return -1;

        }
}



```


在编写比较器的时候我无意中这样写了：


```

import java.util.Comparator;

/**
 * Created by evilrat on 3/9/17.
 */
public class CompareterTool {

    public Comparator<Demo> comparator = new Comparator<Demo>() {
        @Override
        public int compare(Demo o1, Demo o2) {
            if (o1.age > o2.age){
                return 1;
            }else
                return -1;

        }
    };
}



```

请注意这种写法，直接声明一个Comparator<Demo>对象，然后紧跟这一段大括号重写了他的compare（）方法，然后在大括号外写了分号。
反正我是没这么写过，可能是我还没见过的原因吧，记录以下，感觉这样很炫酷。但是随之main方法中的排序代码要改为:


```

Collections.sort(d,ct.comparator);

```


ct是ComparatorTool类的对象，但是从编码来看，compare方法是在Comparator的对象comparator中啊，疑惑？为什么这里是调用ct.comparator，这是一个对象啊，难道说是comparator对象声明时直接写了一个方法？Comparator有这么一种特殊的写法。







### Comparable

对于Comparable来说，需要在编写想要进行排序的类的时候实现Comparable接口，然后在需要排序的时候调用Arrays或者Collections的sort方法就可以直接调用到这个比较器了。

```

/**
 * Created by evilrat on 3/9/17.
 */
public class Demo implements Comparable<Demo>{

    String name;
    int age;

    @Override
    public int compareTo(Demo o) {
        return this.getAge()-o.getAge();
    }

    public Demo(int age){
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}




```


调用方法,这里会自动对应到实现Comparable接口而产生的比较器：



```

Collections.sort(d);


```
























<html>
<div class="ds-thread" data-thread-key="http://kongzheng1993.github.io/kongzheng1993-about_comparator_and_comparable/" data-title="about_comparator_and_comparable" data-url="http://kongzheng1993.github.io/kongzheng1993-about_comparator_and_comparable/"></div>
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

