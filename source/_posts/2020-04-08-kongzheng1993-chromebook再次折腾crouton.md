---
title: chromebook再次折腾crouton
excerpt: ''
tags: [other]
categories: [other]
comments: true
date: 2020-04-08 00:30:52
---

# chromebook再次折腾crouton

## 学生时代的梦想

大学那会儿就梦想有个chromebook，想象着带着这个满满黑科技的笔记本，出入校园，秒杀一众macbook。无奈家境贫寒，而且找不到靠谱的渠道。16年9月刚毕业不久，拿到第一笔工资，我就下单买了这个asus flip c100p chromebook。

<img src="taobao.jpg">

## 无数次折腾

拿到这个chromebook，小巧精美，略微使用了一下，就开始着手折腾装Linux，毕竟这是我买它的初衷，低廉的价格、炫酷的外表、极客的内在，这是我对它爱不释手的原因。

当初不懂事，买的时候没有做攻略，光看着这款chromebook能各种翻折，小，而且会旋转就买了。没仔细看这款chromebook是arm架构的，后面折腾系统真的是折腾死个人啊。

x86架构的本子可以刷bios，从而装各种linux、win，走向它的人生巅峰，可我这个arm的，兼容它的linux发行版真的是少的很，而且折腾的人少，没有战友，也没有经验。

我知道的给asus flip c100p装Linux的方法有下面几种：

- **kali linux：** kali出了一个适配这个本的系统，按照官网的doc可以顺利安装。
- **arch linux：** arch也出过一个适配的系统，也是按照官网doc安装，但是我死活装不上，在github上提了好几个issue，好不容易装完了，却没法启动。说好的`ctrl+d`进chromeOS，`ctrl+u`进扩展卡里的arch呢?
- **crouton:** 这个方法是我最想成的，因为他是在chromeOS的系统之上的一个系统，类似于win10的WSL，两个系统可以随时切换而不用重启。可是我之前总是挂在装audio的时候，因为根据crouton的脚本，这时候要去google下载驱动，当时也用了代理，浏览器都能访问，他这掉链子，一怒之下差点卖了它。

我喜欢它的chromeOS，但是也垂涎Linux，开启它的更多使用场景。就这样，我来回折腾了很多次，断断续续，吃灰俩月，再拿出来折腾，折腾累了，再扔到角落吃灰，如此以往，几年都过去了。

## 重新启用

之前很久不用了，前段时间开始写公众号了，想着拿出它来，从chrome store找个好使的markdown编辑器，超强续航，写点东西，岂不美哉。

于是我拿出来开始折腾，看了下空间，之前倒腾系统，乱分区，导致可用空间太少了，所以我打算重新来过。

找了个U盘，装了`chromebook recovery utility`，重装了下系统，正好清一下磁盘。

<img src="recovery.bmp">

新装的系统，录了个视频。有兴趣可以点下面链接看下。

[我的chromebook](https://www.bilibili.com/video/BV1NE411G7fn/)

昨天晚上又疯了，想用chromebook写博客，毕竟它那么小巧，我把它安置在最舒服的地方----我的床头，毕竟躺床上最舒服了。

我的blog用的hexo，就算我能用chromeOS的terminal，vi写博客，也需要`git`、`node.js`来编译和发布啊。

所以没办法，在折腾一次装Linux，鉴于我的chromeOS已经整的比较完美了，想留着用，所以选择`crouton`。

## 如果这次不行，我真挂咸鱼卖了，200就卖！

之前折腾过无数次crouton，会卡在audio，当时翻遍了github，了解到有个大佬，搞了个代理地址，修改crouton源码，把地址替换，可以解决网络的麻烦。

[dubuqingfeng/Chromebook-For-Chinese](https://github.com/dubuqingfeng/Chromebook-For-Chinese)

```shell
声卡驱动修改说明：
1.下载Cronton，打包下载，Download Zip

2.更改targets/audio文件:

第47行：

( wget -O "$archive" "$urlbase/$ADHD_HEAD.tar.gz" 2>&1 \
                                    || echo "Error fetching CRAS" ) | tee "$log"
改为：

( wget -O "$archive" "http://t.cn/R46YOzM" 2>&1 \
                                    || echo "Error fetching CRAS" ) | tee "$log"

3.直接运行installer/main.sh,或者make自己的crouton。
```

这个方法我当年试过，代理地址肯定是不可用了，毕竟几年前的方案了，人家不可能一直负责维护。

关于这块代码我也尝试过修改，它是把驱动文件下载`/tmp`目录，我的思路是直接fq，把文件搞下来，这块改代码复制到`/tmp`目录下呗。

我把这块的代码全给注释，然后加上了下面的代码。

```shell
cp /home/chronos/user/Dwonloads/xxxxx.tar.gz "$archive"
```

然后满心欢喜的`sudo sh install/main.sh`，可是不行啊，每次都找不到文件，我单独执行这个命令却完全可以复制……后来怀疑是每次进chronos，用户目录名都会发生变化才导致这个原因。

通过查看crouton的帮助能知道`-P`选项可以设置代理地址，`-m`可以设置镜像地址。

于是使用自己的代理和中科大的镜像开始安装：

```shell
sudo crouton -r trusty -t core,xorg,x11,gtk-extra,xfce,keyboard,cli-extra  -m http://mirrors.ustc.edu.cn/ubuntu-ports -P 10.10.10.185:1077
```

卡在声卡驱动的地方，通过看输出的日志了解到并没有走代理地址。后来尝试修改代理地址为`http://10.10.10.185:1077`。
这次走代理了，声卡也能正常下载文件，但是最后还是有问题：

```shell
nstalling target audio...
Fetching CRAS (branch aefe8a7296df698ba86e49b181e93ca2fa2510d1)...
--2020-04-08 16:59:50--  https://chromium.googlesource.com/chromiumos/third_party/adhd/+archive/aefe8a7296df698ba86e49b181e93ca2fa2510d1.tar.gz
Connecting to 10.10.10.185:1077... connected.
Proxy request sent, awaiting response... 200 OK
Length: unspecified [application/x-gzip]
Saving to: '/tmp/crouton-cras.RSihAP/adhd.tar.gz'

     0K .......... .......... .......... .......... ..........  106K
    50K .......... .......... .......... .......... ..........  242K
   100K .......... .......... .......... .......... .......... 99.5K
   150K .......... .......... .......... .......... ..........  769K
   200K .......... .......... .......... .......... .......... 1.22M
   250K .......... .......... .......... .......... ..........  408K
   300K .......... .......... .......... .......... .......... 1.35M
   350K .......... .......... .......... .......... .......... 3.36M
   400K .......... .......... .......... .......... ..........  739K
   450K .......... .......... .......... .......... ..........  793K
   500K .......... .......... .......... .......... ..........  859K
   550K .......... .......... .......... .......... .......... 1.48M
   600K .......... .......... .......... .......... ..........  273K
   650K .......... .......... .......... .......... .......... 2.24M
   700K .......... .......... .......... .......... .......... 2.57M
   750K .......... .......... .......... .......... .....      4.13M=1.9s

2020-04-08 16:59:54 (416 KB/s) - '/tmp/crouton-cras.RSihAP/adhd.tar.gz' saved [814734]

Reading package lists... Done
Building dependency tree
Reading state information... Done
The following extra packages will be installed:
  libasound2-data libsamplerate0
Suggested packages:
  libasound2-plugins
Recommended packages:
  alsa-base
The following NEW packages will be installed:
  alsa-utils libasound2 libasound2-data libsamplerate0 libspeexdsp1
0 upgraded, 5 newly installed, 0 to remove and 0 not upgraded.
Need to get 2196 kB of archives.
After this operation, 4784 kB of additional disk space will be used.
WARNING: The following packages cannot be authenticated!
  libasound2-data libasound2 libsamplerate0 libspeexdsp1 alsa-utils
E: There are problems and -y was used without --force-yes
Failed to complete chroot setup.
Unmounting /mnt/stateful_partition/crouton/chroots/trusty...
```

启动试了下，还是之前的报错：

```shell
UID 1000 not found in trusty
```

往前翻了一下日志，发现从中科大镜像站下载竟然有很多失败的。我复制了其中一个url到浏览器，网络确实不通。

**陷入沉思。。。**

中间吃了个火锅，女朋友从菜市场买来的牛肉^^_^^

填饱肚子，静下心来，突然灵光乍现----我用了代理，代理服务器在国外，那么代理服务器到中科大不通？？我通过代理访问中科大镜像站，没问题，但是顺着目录一步一步往下走，到`trusty`下，就再也不能前进了，我突然明白了什么，但是又不明白为什么主站能访问到，这个目录却无法访问。。

怎么解决？

1. 代理服务器到中科大镜像站不通
2. 不用代理没法正常安装audio

综上两点，我决定换成ubuntu官方源试试，先通过代理试了下，能正常访问，然后开始安装：

```shell
sudo crouton -r trusty -t core,xorg,x11,gtk-extra,xfce,keyboard,cli-extra  -m http://ports.ubuntu.com/ubuntu-ports -P http://10.10.10.185:1077
```

**功夫不负有心人！！！**

<img src="finishInstall.jpg">

启动xfce试试：

```shell
sudo startxfce4
```

略微卡顿，来到了我期待已久的桌面！！！

<img src="xfce4.jpg">

看下系统版本

<img src="linuxRelease.jpg">

圆满成功！！！
接下来就是好好整整这个ubuntu了，装各种环境，啦啦啦。。
