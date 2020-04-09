---
layout: post
title:  "properties文件的使用"
date:   2016-07-28
excerpt: "properties"
tag:
- oop
comments: true
---


## 简介

java中的properties文件是一种配置文件，主要用于配置信息，文件类型为.properties，格式为文本文件，文件内容是“键=值”的格式，在properties文件中可以使用“#”来做注释，properties文件在Java编程中用到的地方很多，操作很方便。

## Properties文件

config.properties

```
db_url=com.mysql.jdbc.Driver
db_mysql=jdbc:mysql
db_ip=127.0.0.1
db_port=3306
db_dbName=users
db_usn=root
db_pwd=root

```

## Properties类的方法

Properites类存在Java.util中，该类继承自Hashtable

1 getProperty ( String  key) ，   用指定的键在此属性列表中搜索属性。也就是通过参数 key ，得到 key 所对应的 value。
2 load ( InputStream  inStream) ，从输入流中读取属性列表（键和元素对）。通过对指定的文件（比如说上面的 test.properties 文件）进行装载来获取该文

件中的所有键 - 值对。以供 getProperty ( String  key) 来搜索。
3 setProperty ( String  key, String  value) ，调用 Hashtable 的方法 put 。他通过调用基类的put方法来设置 键 - 值对。 
4 store ( OutputStream  out, String  comments) ，   以适合使用 load 方法加载到 Properties 表中的格式，将此 Properties 表中的属性列表（键和元素

对）写入输出流。与 load 方法相反，该方法将键 - 值对写入到指定的文件中去。
5 clear () ，清除所有装载的 键 - 值对。该方法在基类中提供。

## 在JAVA文件中操作properties文件的方法

```

        pr=new Properties();
        inStream=this.getClass().getResourceAsStream("config.properties");
        InputStream inStream=DBUtil.class.getResourceAsStream("config.properties");
        try {
            pr.load(inStream);
            url=pr.getProperty("db_url");
            mysql=pr.getProperty("db_mysql");
            ip=pr.getProperty("db_ip");
            port=pr.getProperty("db_port");
            dbname=pr.getProperty("db_dbName");
            dbusn=pr.getProperty("db_usn");
            dbpwd=pr.getProperty("db_pwd");
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }

```

## 总结

java的properties文件需要放到classpath下面，这样程序才能读取到，有关classpath实际上就是java类或者库的存放路径，在java工程中，properties放到class文件一块。在web应用中，最简单的方法是放到web应用的WEB- INF\classes目录下即可，也可以放在其他文件夹下面，这时候需要在设置classpath环境变量的时候，将这个文件夹路径加到 classpath变量中，这样也也可以读取到。在此，你需要对classpath有个深刻理解，classpath绝非系统中刻意设定的那个系统环境变量，WEB-INF\classes其实也是，java工程的class文件目录也是。






<html>
<div class="ds-thread" data-thread-key="http://kongzheng1993.github.io/kongzheng1993-properties/" data-title="properties" data-url="http://kongzheng1993.github.io/kongzheng1993-properties/"></div>
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
