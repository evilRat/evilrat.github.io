---
layout: post
title: "***.java:[1,1] illegal character"
date: 2019-05-30
excerpt: ""
tags: [编码]
categories: [编码]
comments: true
---

## 千万不要用记事本写代码改代码……

刚接到今晚发布留守同事等电话：“你的代码报错了，给你发截图了，qq！！！”。我心里一惊，mmp，不可能吧。。。
赶紧登陆qq，看了一下他发来等截图
![编译报错](2019-05-31-kongzheng1993-生产部署illegal-character/WechatIMG1.jpg)
mmp? 第一行，第一个字符就报错？
定睛一看，是非法字符。
仔细回想……
今天我提代码的时候在生产库用记事本改代码了……
“大哥，帮我把这个文件重新提一下，多谢多谢🙏”
重提这个文件，打包，发布，编译成功，总算松了口气。

之前就记得windows记事本会文本文件编码做修改，静下来后百度下：
```
某些编辑器会往utf8文件中添加utf8标记（editplus称其为签名），它会在文件开始的地方插入三个不可见的字符（0xEF 0xBB 0xBF，即BOM），它的表示的是 Unicode 标记（BOM）。 因此要解决这个问题的关键就是把这个标记选项去掉，可按如下方法操作。 
首先用editplus打开这个文件，从Doucument菜单中选择Permanet Settings,有三个分类，分别是General,File, Tools.点击File,右边会有一项是 UTF-8 signature: 选择 always remove signature. 点击OK 。中文版本的 Editplus 下操作的菜单结构如下: 文档->参数设置->文件->UTF-8签名->总是移除签名->确定 ，这样就设置了UTF-8格式不需要在文件前面加标记，最后把文件另存为utf-8格式就好了.
```


