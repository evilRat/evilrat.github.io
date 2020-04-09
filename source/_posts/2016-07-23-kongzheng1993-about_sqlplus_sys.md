---
layout: post
title: "使用sys登陆sqlplus的问题"
date: 2016-06-10
excerpt: "use user sys login sqlplus"
tags: [sys,sqlplus,sql]
comments: true
---


### 使用sys/root登陆sqlplus

我们用sys/root的账号密码正常登陆sqlplus会出现以下问题：

<img src="/assets/img/sqlplus.bmp">


但是我们如果在输入password时不输入我们的密码（root），而是输入sys as sysdba,就可以顺利登陆，我也不知道什么原理，如果有知道的朋友，可以给我留言。


如果我们先用其他用户登陆比如learner/learner,然后断开，再使用connect sys/root as sysdba就可以登陆。

<img src="/assets/img/sqlplusloginsuccess.bmp">












<html>
<div class="ds-thread" data-thread-key="http://kongzheng1993.github.io/kongzheng1993-sqlplusloginsys/" data-title="sqlplusloginsys" data-url="http://kongzheng1993.github.io/kongzheng1993-sqlplusloginsys/"></div>
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