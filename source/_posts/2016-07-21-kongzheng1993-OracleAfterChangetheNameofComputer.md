---
layout: post
title: "修改主机名oracle无法正常启动"
date: 2016-07-20
excerpt: "oracle"
tags: [oracle]
comments: true
---

在刚开始学习Oracle时，安装完Oracle后，我发现我的主机名不是很炫酷，就去计算机管理把电脑的名字改成了EvilRat，然后我的Oracle就连不上了，因为listener服务一直启动不了，显示类似下面的情况：（图是从网上找的，不好在弄回去截图了，大家知道是什么情况能解决问题就行）。
<center>
<img src="/assets/img/oracle_listener.bmp">
</center>
然后我从网上查了资料，发现是因为listener的配置文件里面的主机名还没有改过来，于是我就是手动改过来了。
<center>
<img src="/assets/img/listener.bmp">
</center>
将箭头指的地方修改后，应该就没问题了，这是我真是遇到的问题，希望可以帮助到大家。








<html>
<div class="ds-thread" data-thread-key="http://kongzheng1993.github.io/kongzheng1993-oraclepcname/" data-title="oraclepcname" data-url="http://kongzheng1993.github.io/kongzheng1993-oraclepcname/"></div>
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
