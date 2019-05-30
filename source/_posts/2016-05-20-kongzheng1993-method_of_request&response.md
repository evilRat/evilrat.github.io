---
layout: post
title:  "request&response"
date:   2016-07-28
excerpt: "properties"
tag:
- oop
comments: true
---

#### request的方法：

客户端的请求信息被封装在request对象中，通过它才能了解到客户的需求，然后做出响应。它是HttpServletRequest类的实例。

序号/方法/说明 

* object getAttribute(String name) 返回指定属性的属性值 
* Enumeration getAttributeNames() 返回所有可用属性名的枚举 
* String getCharacterEncoding() 返回字符编码方式 
* int getContentLength() 返回请求体的长度（以字节数） 
* String getContentType() 得到请求体的MIME类型 
* ServletInputStream getInputStream() 得到请求体中一行的二进制流 
* String getParameter(String name) 返回name指定参数的参数值 
* Enumeration getParameterNames() 返回可用参数名的枚举 
* String[] getParameterValues(String name) 返回包含参数name的所有值的数组 
* String getProtocol() 返回请求用的协议类型及版本号 
* String getScheme() 返回请求用的计划名,如:http.https及ftp等 
* String getServerName() 返回接受请求的服务器主机名 
* int getServerPort() 返回服务器接受此请求所用的端口号 
* BufferedReader getReader() 返回解码过了的请求体 
* String getRemoteAddr() 返回发送此请求的客户端IP地址 
* String getRemoteHost() 返回发送此请求的客户端主机名 
* void setAttribute(String key,Object obj) 设置属性的属性值 
* String getRealPath(String path) 返回一虚拟路径的真实路径

#### response的方法：

序号/方法/说明

response对象包含了响应客户请求的有关信息，但在JSP中很少直接用到它。它是HttpServletResponse类的实例。
序号 方 法 说 明 
* String getCharacterEncoding() 返回响应用的是何种字符编码 
* ServletOutputStream getOutputStream() 返回响应的一个二进制输出流 
* PrintWriter getWriter() 返回可以向客户端输出字符的一个对象 
* void setContentLength(int len) 设置响应头长度 
* void setContentType(String type) 设置响应的MIME类型 
* sendRedirect(java.lang.String location) 重新定向客户端的请求






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
