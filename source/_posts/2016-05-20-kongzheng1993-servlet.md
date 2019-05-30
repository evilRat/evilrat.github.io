---
layout: post
title:  "Servlet笔记"
date:   2016-07-26
excerpt: "servlet"
tag:
- oop
comments: true
---

## servlet 笔记


- 注意如果是使用请求转发来切换页面，地址栏url不变，但是网页实际上已经变化，这时候要认清真正的地址，来确定本页面和其对应的servlet之间的相对位置，防止发生404错误。
- 在jsp中使用click动作有限制。页面上显示的同步机制的数据，click是有效果的；如果是异步遍历的数据，那么触发的事件click是无法捕捉到的，需要on来获取整个标签页面的属性。而且on的作用范围比较大，同步和异步中都可以使用。

click范例：

```

$("body").on("click","#firstPage",function(){
        //alert("首页:");
        changePages(1);
    }

```

on范例：

```

$("body").on("click","#lastPage",function(){
        //alert("last页"+totalPage);
        changePages(totalPage);
    }

```















<html>
<div class="ds-thread" data-thread-key="http://kongzheng1993.github.io/kongzheng1993-servlet/" data-title="servlet" data-url="http://kongzheng1993.github.io/kongzheng1993-servlet/"></div>
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
