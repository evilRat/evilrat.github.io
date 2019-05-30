---
layout: post
title: "Hexo+github搭建个人博客遇到的问题"
date: 2018-02-01
excerpt: "Hexo+github"
tags: [github,hexo,git]
categories: [blog,github,git]
comments: true
---

### hexo+github搭建个人博客遇到的问题

1. hexo博客_config.yml中的配置
 1. Site配置网站相关的信息，比如网站标题、子标题、描述和作者等。
 2. URL配置博客在网站的目录，这里需要注意一些问题。如果是在根目录，url就配置为https://githubID.github.io，root就配置为/，但是如果有二级目录，比如blog，就要对应的设置为url:https://githubID.github.io/blog root:/blog/。但是github pages有这么个问题，你访问的时候看起来是一级目录，其实是二级目录。所以我在设置为一记目录时，在本地hexo -s debug的时候是正常的，但是发布后网站是没有样式的，也就是说静态资源是访问不到的。所以我又尝试配置为二级目录url:https://githubID.github.io/blog root:/githubID.github.io/，然后样式就正常了。
 3. 我的博客是从jekyll迁移过来的，所以要把所有的_posts中的博文复制到source/_posts文件夹，然后修改_config.yml中的new_post_name参数，也就是让新的hexo生成的新博文符合原来jekyll博文的命名规则。
2. 相关的建站的方法这里不再说明，可以到`hexo.io`跟着做。
3. 安装服务器hexo-server
```
npm install hexo-server --save
```
  这个命令是安装hexo服务器，下面的命令是启动服务器，而且启动后修改文件无需重启，-p选项可以制定端口来运行server
```
hexo server
```
  运行成功后可以在命令行看到访问http://localhost:4000可以预览网站。
4. 我是在用github pages来建站，所以需要使用git来部署我的博客到github pages。
  所以要使用下面的命令安装git的部署器。
```
npm install hexo-deployer-git --save
```
  然后还需要修改配置_config.yml
```
deploy:
  type: git
  repo: https://github.com/githubID/githubID.github.io.git
  branch: #github会自动检测
  message: #自定义的提交信息。
```



<html>
<div class="ds-thread" data-thread-key="http://kongzheng1993.github.io/kongzheng1993-web_xml/" data-title="About trycatch" data-url="http://kongzheng1993.github.io/kongzheng1993"></div>
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
