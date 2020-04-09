---
layout: post
title:  "Session笔记"
date:   2016-07-27
excerpt: "Session"
tag:
- oop
comments: true
---

### Session简介

在WEB开发中，服务器可以为每个用户浏览器创建一个会话对象（session对象），注意：一个浏览器独占一个session对象(默认情况下)。因此，在需要保存用户数据时，服务器程序可以把用户数据写到用户浏览器独占的session中，当用户使用浏览器访问其它程序时，其它程序可以从用户的session中取出该用户的数据，为用户服务。

### Session和Cookie的区别

* Cookie是把用户的数据写给用户的浏览器。
* Session技术把用户的数据写到用户独占的session中。
* Session对象由服务器创建，开发人员可以调用request对象的getSession方法得到session对象。

### Session实现原理

servlet中：

```

response.sendRedirect("pages/login.jsp");
String username=request.getParameter("usn");
HttpSession session=request.getSession();
 session.setAttribute("username",username);

```

通过`HttpSession session=request.getSession();`，如果此线程中已经存在一个session，就使用这个session，如果没有，就创建一个。这个getSession()方法可以添加boolean的参数，true表示如果没有就创建一个，如果有就使用存在的那一个，false表示，直接创建一个，默认是true。session.setAttribute()来创建一个属性，这样另一端就可以get了。


Jsp中：

```

 Object usn=session.getAttribute("username");

```

直接使用servlet中创建的Session对象，调用getAttribute();得到servlet中set的属性。这里即便是使用getSession()方法再次得到session也是在servlet中设置的Session对象。





<html>
<div class="ds-thread" data-thread-key="http://kongzheng1993.github.io/kongzheng1993-session/" data-title="session" data-url="http://kongzheng1993.github.io/kongzheng1993-session/"></div>
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
