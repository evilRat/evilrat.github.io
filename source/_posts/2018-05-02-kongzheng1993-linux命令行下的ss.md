---
layout: post
title: "linux命令行下的ss"
date: 2018-05-02
excerpt: "linux ss"
tags: [linux,ss,console]
categories: [linux,ss]
comments: true
---

## 为什么不用shadowsocks-qt5

我的pop! os基于ubuntu，因为装了图形界面，所以在用shadowsocks-qt5，而且很好用，只需要打开软件，就会自动连接朋友的ss-server。但是我的chromebook的fq问题一直存在，一度让我产生了卖掉它的想法。昨天开始我准备好好搞一下在shell运行ss客户端，然后了解到了有sslocal和ssserver这样的东西，简直是欣喜若狂，看到了随身携带我的cb的希望。

## 开始搞

sslocal和ssserver都依赖python，所以要先安装python。

```bash
sudo apt-get update
sudo apt-get install python python-pip
```
之后开始安装shadowsocks

```bash
pip install shadowsocks
```

运行ss
sslocal -s server_ip -p server_port -k "password" -l local_port -t 600 -m aes-256-cfb

可以通过新建一个配置文件来省去这些参数
比如我们在/etc下新建一个shadowsock.json

```bash
{
"server":"server_ip",
"server_port":server_port,
"local_ip":"127.0.0.1"
"local_port":1080,
"password":"password",
"timeout":600,
"method":"aes-256-cfb"
}
```

然后就可以直接sslocal -c /etc/shadowsock.json来启动sslocal

因为我的设备是chromebook，所以要配置一下chrom代理，我使用的是SwitchyOmega，可以去google商店下载，但是如果环境允许，可以从github下载，然后托到chrome插件里。然后配置一下SwitchyOmega，新建个情景模式，选择代理服务器，用socks5，地址和端口就是sslocal的配置中的local_ip和local_port，然后设置一下自动切换，在按照规则列表匹配请求后面选择刚才新建的SS，默认情景模式选择直接连接。点击应用选项保存。再往下规则列表设置选择AutoProxy 然后将“https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt”填进去，点击下面的立即更新情景模式，会有提示更新成功！这样就配置完了。

在启动sslocal之后，点击chrome右上角的SwitchyOmega图表，选择自动切换，工具会根据gfwlist.txt的配置自动切换是否将请求转发到你的local_ip的local_port。可以节省ss服务器的流量。

## 总结

总体来说还是比较顺利的，以后遇到什么问题要学会思考，不要盲目的尝试。要了解原理，像这次我遇到了很多报错，我都`more sslocal`看了脚本代码了，而且前面尝试了一个自动安装配置ss的脚本，wget一个脚本shadowsocks.sh，然后`./shadowsocks.sh install`就可以，但是我遇到了很多问题，也是进去好好研究了一下人家的代码。虽然最后这个方法没有研究透，不过能看一下大神们写的脚本也很好啊！！！