---
title: Nginx
excerpt: ''
tags: [Nginx]
categories: [Nginx]
comments: true
date: 2020-05-12 00:30:52
---

前两天客户那边说检测出我们的nginx有漏洞，让我们搞一下，其实就是nginx出新版本了，老版本有个重大bug，跟着我们也应该升级。但是一来二去不知道为啥落我头上了，我是开发啊。。不过也无所谓，正好趁着这个机会好好了解一下。

## 什么是Nginx

Nginx是一款轻量级的Web服务器、反向代理服务器，由于它的内存占用少，启动极快，高并发能力强，在互联网项目中广泛应用。除了http它还支持IMAP/POP3等协议。

目前比较常见的系统架构中，nginx一般都是在入口处，作为反向代理。

<img src="v2-e1826bab1d07df8e97d61aa809b94a10_r.jpg"/>

提到Nginx就不能不提反向代理。。


- **正向代理：**  代理客户端去请求服务器，然后将服务器的响应返回给客户端。我们平常科学上网用的代理服务器起到的作用就是正向代理。*正向代理代理的是客户端*。
- **反向代理：** 代理服务器来接受客户端的请求，然后将客户端的请求转发到内部网络，并将服务器上得到的结果返回给客户端。*反向代理代理的是服务端*。

## 安装Nginx

安装Nignx和一般软件一样，也是一顿configure、make、make install。。

1. 首先是安装依赖

```shell
yum -y install gcc openssl-devel pcre-devel zlib-devel
```

已经有了的就不用装了，pcre库是用来支持正则表达式的，也就是我们想在nginx的配置文件里用正则，还是老老实实把pcre编译进nginx。zlib是对于http包的内容做gzip格式的压缩，如果我们在nginx.conf中配置了`gzip on`，并指定对于某些类型（content-type）的http响应使用gzip来进行压缩以减少网络传输量，就需要把zlib编译进nginx。openssl是用来支持ssl协议，也包含MD5、SHA1等散列函数。

2. 下载解压nginx

```shell
wget -o XX/nginx-X.X.X.tar.gz
tar -zxvf nginx-X.X.X.tar.gz
```

3. configure配置

进入源码根目录，执行：

```shell
./configure --prefix=/**/**/**
```

通过`--prefix`来指定软件安装位置。

4. 编译安装

```shell
make && make install
```

在安装完后，就会在你制定的目录出现nginx目录了，一些可执行文件在`sbin`目录下。执行`./nginx`就可以启动nginx了。

使用`nginx -s $sign`可以来控制启动的nginx进程。

$sign有：
- stop  快速关闭
- quit  优雅的关闭
- reload  重新加载配置文件
- reopen  重新打开日志文件


## 运行中的nginx

nginx在启动后，在unix系统中会以daemon的方式在后台运行，后台进程包含一个master进程和多个worker进程。所以，nginx是以多进程的方式来工作的，当然nginx也是支持多线程的方式的，只是我们主流的方式还是多进程的方式，也是nginx的默认方式。我们也可以手动关掉后台模式，让nginx在前台运行，并且也可以通过配置取消master进程，也就是让nginx以单进程运行。这样可以让我们更方便debug。

master进程主要是来管理worker进程的，包含：

- 读取并验证nginx.conf
- 接受外来的信号
- 向各worker进程发送信号
- 监控worker的运行状态，当worker进程退出后，master会启动新的worker

worker进程处理的是基本的网络事件，多个worker之间是平等的，他们同等竞争来自客户端的请求，各进程之间是独立的。一个请求只会在一个worker中处理。worker的数量是可以设置的，一般会和cpu核心数一致。worker默认是单线程的，这样可以避免线程切换。

## nginx的热部署

我们在使用nginx一般修改的都是nginx.conf配置文件。每次我们修改nginx.conf后都不需要重启nginx，而是直接执行`nginx -s reload`，让nginx重新加载一下配置就好。

nginx在重新加载配置文件后，会新建新建worker来处理新的请求，旧的请求还是走之前的worker，等老worker处理完请求后，就会被kill掉。

## nginx的高并发

前面的配置worker个数和cpu核心数一致，worker单线程来防止线程切换浪费资源，都是nginx高并发的原因。

还有就是nginx使用的是epoll模型的IO，我理解的就是和redis一样的IO多路复用，通过一个选择器来监听成千上万的连接。（之前在看一本书，用netty、redis、zk做高并发实战，上面讲了文件句柄，nginx是不是也修改了？）

## nginx高可用

请求不要直接打在nginx上，应该先通过虚拟IP（VIP）。
然后就是通过keepalived来监控nginx的状态，通过执行脚本来检查nginx进程状态。

keepalived是一种高性能的服务器高可用或热备解决方案，Keepalived可以用来防止服务器单点故障的发生，通过配合 Nginx 可以实现 web 前端服务的高可用。Keepalived以VRRP协议为实现基础，用 VRRP协议来实现高可用性(HA)。VRRP(Virtual Router Redundancy Protocol)协议是用于实现路由器冗余的协议，VRRP 协议将两台或多台路由器设备虚拟成一个设备，对外提供虚拟路由器IP(一个或多个)，而在路由器组内部，如果实际拥有这个对外IP的路由器如果工作正常的话就是MASTER，或者是通过算法选举产生，MASTER 实现针对虚拟路由器 IP 的各种网络功能，如 ARP 请求，ICMP，以及数据的转发等；其他设备不拥有该虚拟 IP，状态是 BACKUP，除了接收 MASTER 的VRRP 状态通告信息外，不执行对外的网络功能。当主机失效时，BACKUP 将接管原先 MASTER 的网络功能。VRRP 协议使用多播数据来传输 VRRP 数据， VRRP 数据使用特殊的虚拟源 MAC 地址发送数据而不是自身网卡的 MAC 地址，VRRP 运行时只有 MASTER 路由器定时发送 VRRP 通告信息，表示 MASTER 工作正常以及虚拟路由器 IP(组)，BACKUP 只接收 VRRP 数据，不发送数据，如果一定时间内没有接收到 MASTER 的通告信息，各 BACKUP 将宣告自己成为 MASTER，发送通告信息，重新进行 MASTER 选举状态。


## nginx.conf

我们一般修改的都是nginx.conf文件。

下面是nginx官方给的完整例子：

```shell
#!nginx
: # 使用的用户和组
: user  www www;
: # 指定工作衍生进程数
: worker_processes  2;
: # 指定 pid 存放的路径
: pid /var/run/nginx.pid;

: # [ debug | info | notice | warn | error | crit ] 
: # 可以在下方直接使用 [ debug | info | notice | warn | error | crit ]  参数
: error_log  /var/log/nginx.error_log  info;

: events {
: # 允许的连接数
: connections   2000;
: # use [ kqueue | rtsig | epoll | /dev/poll | select | poll ] ;
: # 具体内容查看 http://wiki.codemongers.com/事件模型
: use kqueue;
: }

: http {
: include       conf/mime.types;
: default_type  application/octet-stream;

: log_format main      '$remote_addr - $remote_user [$time_local]  '
: '"$request" $status $bytes_sent '
: '"$http_referer" "$http_user_agent" '
: '"$gzip_ratio"';

: log_format download  '$remote_addr - $remote_user [$time_local]  '
: '"$request" $status $bytes_sent '
: '"$http_referer" "$http_user_agent" '
: '"$http_range" "$sent_http_content_range"';

: client_header_timeout  3m;
: client_body_timeout    3m;
: send_timeout           3m;

: client_header_buffer_size    1k;
: large_client_header_buffers  4 4k;

: gzip on;
: gzip_min_length  1100;
: gzip_buffers     4 8k;
: gzip_types       text/plain;

: output_buffers   1 32k;
: postpone_output  1460;

: sendfile         on;
: tcp_nopush       on;
: tcp_nodelay      on;
: send_lowat       12000;

: keepalive_timeout  75 20;

: #lingering_time     30;
: #lingering_timeout  10;
: #reset_timedout_connection  on;


: server {
: listen        one.example.com;
: server_name   one.example.com  www.one.example.com;

: access_log   /var/log/nginx.access_log  main;

: location / {
: proxy_pass         http://127.0.0.1/;
: proxy_redirect     off;

: proxy_set_header   Host             $host;
: proxy_set_header   X-Real-IP        $remote_addr;
: #proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;

: client_max_body_size       10m;
: client_body_buffer_size    128k;

: client_body_temp_path      /var/nginx/client_body_temp;

: proxy_connect_timeout      90;
: proxy_send_timeout         90;
: proxy_read_timeout         90;
: proxy_send_lowat           12000;

: proxy_buffer_size          4k;
: proxy_buffers              4 32k;
: proxy_busy_buffers_size    64k;
: proxy_temp_file_write_size 64k;

: proxy_temp_path            /var/nginx/proxy_temp;

: charset  koi8-r;
: }

: error_page  404  /404.html;

: location /404.html {
: root  /spool/www;

: charset         on;
: source_charset  koi8-r;
: }

: location /old_stuff/ {
: rewrite   ^/old_stuff/(.*)$  /new_stuff/$1  permanent;
: }

: location /download/ {

: valid_referers  none  blocked  server_names  *.example.com;

: if ($invalid_referer) {
: #rewrite   ^/   http://www.example.com/;
: return   403;
: }

: #rewrite_log  on;

: # rewrite /download/*/mp3/*.any_ext to /download/*/mp3/*.mp3
: rewrite ^/(download/.*)/mp3/(.*)\..*$
: /$1/mp3/$2.mp3                   break;

: root         /spool/www;
: #autoindex    on;
: access_log   /var/log/nginx-download.access_log  download;
: }

: location ~* ^.+\.(jpg|jpeg|gif)$ {
: root         /spool/www;
: access_log   off;
: expires      30d;
: }
: }
: }
```

## nginx缓存

nginx支持缓存，这个缓存是将URL及相关组合当作key，用md5编码哈希后保存在硬盘上。

## 均衡负载

在upstream中配置，有权重、轮询、ip_hash、least_conn四种方式。
