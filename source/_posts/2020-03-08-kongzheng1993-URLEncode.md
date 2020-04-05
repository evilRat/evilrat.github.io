---
title: URLEncoder
excerpt: ''
tags: [Web]
categories: [Web]
comments: true
date: 2020-03-08 00:30:52
---

# URLEncoder

今天写代码要发起一个HTTP GET请求，WebService接口我们随处可见，作为一个“业务程序员”我们也每天都在写。。。所以指尖跳动，分分钟就出现了下面的代码：

```java
 try {
            encode = URLEncoder.encode(JsonUtil.convertObject2Json(reqParamsMap), "UTF-8").replace("\\+", "%20");
        } catch (UnsupportedEncodingException e) {
            logger.error("URLEncoder.encodec出错： ", e);
        }
```

撸完码我回头看了一眼，我习以为常的URL编码，到底是为什么呢？我想大概是怕传输数据中的特殊字符造成什么不可预知的问题吧。

工作完成，测试没有问题。我今天打算打破砂锅问到底，让我们一探究竟，到底为什么要编码？

<font color=red>知道了How，我们还要知道Why！！！</font>

## 找资料

打开bing，国际搜索（梯子断了很久了……）在`stackoverflow`上看到也有很多人提出了这个问题，很多的人回答都指向了一个RFC

	RFC(Request For Comments)-意即“请求评议”，包含了关于Internet的几乎所有重要的文字资料。
	通常，当某家机构或团体开发出了一套标准或提出对某种标准的设想，
	想要征询外界的意见时，就会在Internet上发放一份RFC，
	对这一问题感兴趣的人可以阅读该RFC并提出自己的意见；
	
	绝大部分网络标准的指定都是以RFC的形式开始，经过大量的论证和修改过程，
	由主要的标准化组织所指定的，但在RFC中所收录的文件并不都是正在使用或为大家所公认的，
	也有很大一部分只在某个局部领域被使用或并没有被采用，
	一份RFC具体处于什么状态都在文件中作了明确的标识.

[RFC1738](https://tools.ietf.org/html/rfc1738)

![image-20200317175415033](C:\Users\kongz\AppData\Roaming\Typora\typora-user-images\image-20200317175415033.png)

可以看到这个RFC是小LEE提出来的，里面规定了很多东西，比如url的格式、编码、字符，还有很多scheme，我没看全，看到了FTP和HTTP。

## 总结一波

URL（统一资源定位符）是万维网中资源的地址。URL具有明确定义的结构，该结构由[万维网](https://en.wikipedia.org/wiki/Tim_Berners-Lee)的发明者[Tim Berners-Lee](https://en.wikipedia.org/wiki/Tim_Berners-Lee)在[RFC 1738](https://tools.ietf.org/html/rfc1738)中提出。

每个网址都采用*通用语法*，如下所示：

```bash
scheme:[//[user:password@]host[:port]]path[?query][#fragment]
```

`[user:password@]`由于安全原因，不建议使用URL语法的某些部分，因此很少使用。以下是您在互联网上经常看到的URL的示例-

```bash
https://www.google.com/search?q=hello+world#brs
```

对定义统一资源定位符（URL）语法的初始RFC进行了许多改进。当前定义通用URI语法的[RFC](https://tools.ietf.org/html/rfc3986)是[RFC 3986](https://tools.ietf.org/html/rfc3986)。这篇文章包含最新RFC文档中的信息。

URL由属于US-ASCII字符集的有限字符集组成。这些字符包括数字（0-9），字母（AZ，az），以及一些特殊字符（`"-"`，`"."`，`"_"`，`"~"`）。

ASCII控制字符（例如退格，垂直制表符，水平制表，换行等），不安全的字符像`space`，`\`，`<`，`>`，`{`，`}`等等，以及ASCII字符以外的任何字符被不允许直接的URL内放置。

此外，URL中有些字符具有特殊含义。这些字符称为**保留**字符。的保留字符的一些例子是`?`，`/`，`#`，`:`等作为URL的一部分发送，无论是在查询字符串或路径段，必须不包含这些字符的任何数据。

那么，当我们需要在URL中传输包含这些不允许的字符的任何数据时，我们该怎么办？好吧，**我们对它们进行编码！**

> URL编码将URL中保留的，不安全的和非ASCII字符转换为所有Web浏览器和服务器普遍接受并理解的格式。它首先将字符转换为一个或多个字节。然后，每个字节由两个十六进制数字表示，后跟一个百分号（`%`）-（例如`%xy`）。百分号用作转义字符。

URL编码也称为百分比编码，因为它使用百分号（`%`）作为转义字符。

#### URL编码示例

**空格：**您可能会遇到的最常见的URL编码字符是`space`。`space`十进制字符的ASCII码为`32`，转换为十六进制时为`20`。现在，我们在十六进制表示形式之前加一个百分号（`%`），这使我们获得了URL编码值- `%20`。

**+:** 看到我一开始代码里的replace，那是因为：

![image-20200317181423078](C:\Users\kongz\AppData\Roaming\Typora\typora-user-images\image-20200317181423078.png)

w3c的规定是空格被替换为`+`，Java中的URLEncoder正式这种情况，会把空格转成`+`

![image-20200317182519016](C:\Users\kongz\AppData\Roaming\Typora\typora-user-images\image-20200317182519016.png)

这就会导致在一些遵循RFC的应用上无法解码或其他问题，所以我们取一个折中的方法，要在`URLEncoder.encode()`之后把`+`替换为`%20`

## ASCII字符编码参考

下表是ASCII字符对其相应的URL编码形式的引用。

> 请注意，不需要编码字母数字ASCII字符。例如，您不需要编码字符`'0'`以`%30`如图所示下表。可以原样发送。但是根据RFC编码仍然有效。表格中所有可以安全地在URL中传输的字符都显示为绿色。

**下表使用RFC 3986中定义的URL编码规则。**

| Decimal | Character                   | URL Encoding (UTF-8) |
| :------ | :-------------------------- | :------------------- |
| 0       | NUL(null character)         | %00                  |
| 1       | SOH(start of header)        | %01                  |
| 2       | STX(start of text)          | %02                  |
| 3       | ETX(end of text)            | %03                  |
| 4       | EOT(end of transmission)    | %04                  |
| 5       | ENQ(enquiry)                | %05                  |
| 6       | ACK(acknowledge)            | %06                  |
| 7       | BEL(bell (ring))            | %07                  |
| 8       | BS(backspace)               | %08                  |
| 9       | HT(horizontal tab)          | %09                  |
| 10      | LF(line feed)               | %0A                  |
| 11      | VT(vertical tab)            | %0B                  |
| 12      | FF(form feed)               | %0C                  |
| 13      | CR(carriage return)         | %0D                  |
| 14      | SO(shift out)               | %0E                  |
| 15      | SI(shift in)                | %0F                  |
| 16      | DLE(data link escape)       | %10                  |
| 17      | DC1(device control 1)       | %11                  |
| 18      | DC2(device control 2)       | %12                  |
| 19      | DC3(device control 3)       | %13                  |
| 20      | DC4(device control 4)       | %14                  |
| 21      | NAK(negative acknowledge)   | %15                  |
| 22      | SYN(synchronize)            | %16                  |
| 23      | ETB(end transmission block) | %17                  |
| 24      | CAN(cancel)                 | %18                  |
| 25      | EM(end of medium)           | %19                  |
| 26      | SUB(substitute)             | %1A                  |
| 27      | ESC(escape)                 | %1B                  |
| 28      | FS(file separator)          | %1C                  |
| 29      | GS(group separator)         | %1D                  |
| 30      | RS(record separator)        | %1E                  |
| 31      | US(unit separator)          | %1F                  |
| 32      | space                       | %20                  |
| 33      | !                           | %21                  |
| 34      | "                           | %22                  |
| 35      | #                           | %23                  |
| 36      | $                           | %24                  |
| 37      | %                           | %25                  |
| 38      | &                           | %26                  |
| 39      | '                           | %27                  |
| 40      | (                           | %28                  |
| 41      | )                           | %29                  |
| 42      | *                           | %2A                  |
| 43      | +                           | %2B                  |
| 44      | ,                           | %2C                  |
| 45      | -                           | %2D                  |
| 46      | .                           | %2E                  |
| 47      | /                           | %2F                  |
| 48      | 0                           | %30                  |
| 49      | 1                           | %31                  |
| 50      | 2                           | %32                  |
| 51      | 3                           | %33                  |
| 52      | 4                           | %34                  |
| 53      | 5                           | %35                  |
| 54      | 6                           | %36                  |
| 55      | 7                           | %37                  |
| 56      | 8                           | %38                  |
| 57      | 9                           | %39                  |
| 58      | :                           | %3A                  |
| 59      | ;                           | %3B                  |
| 60      | <                           | %3C                  |
| 61      | =                           | %3D                  |
| 62      | >                           | %3E                  |
| 63      | ?                           | %3F                  |
| 64      | @                           | %40                  |
| 65      | A                           | %41                  |
| 66      | B                           | %42                  |
| 67      | C                           | %43                  |
| 68      | D                           | %44                  |
| 69      | E                           | %45                  |
| 70      | F                           | %46                  |
| 71      | G                           | %47                  |
| 72      | H                           | %48                  |
| 73      | I                           | %49                  |
| 74      | J                           | %4A                  |
| 75      | K                           | %4B                  |
| 76      | L                           | %4C                  |
| 77      | M                           | %4D                  |
| 78      | N                           | %4E                  |
| 79      | O                           | %4F                  |
| 80      | P                           | %50                  |
| 81      | Q                           | %51                  |
| 82      | R                           | %52                  |
| 83      | S                           | %53                  |
| 84      | T                           | %54                  |
| 85      | U                           | %55                  |
| 86      | V                           | %56                  |
| 87      | W                           | %57                  |
| 88      | X                           | %58                  |
| 89      | Y                           | %59                  |
| 90      | Z                           | %5A                  |
| 91      | [                           | %5B                  |
| 92      | \                           | %5C                  |
| 93      | ]                           | %5D                  |
| 94      | ^                           | %5E                  |
| 95      | _                           | %5F                  |
| 96      | `                           | %60                  |
| 97      | a                           | %61                  |
| 98      | b                           | %62                  |
| 99      | c                           | %63                  |
| 100     | d                           | %64                  |
| 101     | e                           | %65                  |
| 102     | f                           | %66                  |
| 103     | g                           | %67                  |
| 104     | h                           | %68                  |
| 105     | i                           | %69                  |
| 106     | j                           | %6A                  |
| 107     | k                           | %6B                  |
| 108     | l                           | %6C                  |
| 109     | m                           | %6D                  |
| 110     | n                           | %6E                  |
| 111     | o                           | %6F                  |
| 112     | p                           | %70                  |
| 113     | q                           | %71                  |
| 114     | r                           | %72                  |
| 115     | s                           | %73                  |
| 116     | t                           | %74                  |
| 117     | u                           | %75                  |
| 118     | v                           | %76                  |
| 119     | w                           | %77                  |
| 120     | x                           | %78                  |
| 121     | y                           | %79                  |
| 122     | z                           | %7A                  |
| 123     | {                           | %7B                  |
| 124     | \|                          | %7C                  |
| 125     | }                           | %7D                  |
| 126     | ~                           | %7E                  |
| 127     | DEL(delete (rubout))        | %7F                  |