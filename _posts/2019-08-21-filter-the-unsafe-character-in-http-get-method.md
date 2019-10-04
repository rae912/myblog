---
layout: post
title: "过滤和转义HTTP GET方法中的保留和不安全字符"
subtitle: "Translation and filter the unsafe characters in HTTP GET method"
author: "qingshan"
header-img: "img/home-bg-o.jpg"
header-mask: 0.4
tags:
  - 工作
  - Javascript
---

# 遇到问题：  
  今天接到用户报告，说在一个界面上获取数据超时无返回。根据报告内容很快就定位到了对应的接口，一个很常见的Get方法。将用户请求构造成URL通过postman复现，发现正常返回，并没有任何问题。没办法，只能手动模拟用户操作复现了。在chrome console上开启JavaScript debug模式，将构造请求的所有变量都打上断点，结果也是一切正常。最后，在接口返回的response中发现了问题：`居然没有response！！！`
  这就有点奇怪了，打开Network标签页，果然发现这个URL的请求状态是400。打开后端日志查看，后端并没有收到这条请求，怎么就400了呢？而且更奇怪的是我原样将request URL复制到postman里，是可以正常访问的！为什么浏览器会拦截这个URL呢？？！


# 排查过程：
  问题一下子陷入了僵局，开始怀疑是开启了gzip头引起的，但是很快就否定了这个原因，毕竟其他所有的接口都没有问题。于是只能逐个从这个request URL中入手分析。经过注意排查，发现仅当某个参数带入参数时，会触发这个问题，即浏览器会自动拦截带这个参数的URL呢？
  经过查询，发现原来浏览器URL有这么一天规范：`书写URL时要使用US-ASCII字符集可以显示的字符。`

```text
如果需要在URL中使用不属于此字符集的字符，就要使用特殊的符号对该字符进行编码。

如：最常使用的空格用%20来表示，例如：http://www.google.com/new%20123.html

除了那些无法显示的字符外，还需要在URL中对那些保留(reserved)字符和不安全(unsafe)字符进行编码。

所谓保留字符就是那些在URL中具有特定意义的字符。不安全字符是指那些在URL中没有特殊含义，但在URL所在的上下文中可能具有特殊意义的字符。例如双引号(“”)
```

部分保留字符和不安全字符及其URL编码
![](https://ae01.alicdn.com/kf/H031c1f348f8f4f3f99df86d3a71933875.jpg)

# 结论
通常情况下，如果对某个字符能否在URL中使用有疑问，那么应该始终使用该字符的编码。除字母、数字和字符$-_.+!*'()外的其它所有字符都应该使用编码。

# 问题解决方案
找到问题所在，其实就是用户在某个参数中传递了不安全的`|`竖线字符（正常情况下不应该有这样的特殊字符）。问题也要解决了。项目前端接口请求相关的网络通信库使用的是`axios`。`axios`已经封装了对URL特殊字符的编码，方法有两种

encodeURI()
对整个url进行编码，会避开url中的功能性字符，例如，& ? [ ]

编码前：http://10.10.67.67:8080/api/chain/basic/users?params=+[
编码后：http://10.10.67.67:8080/api/chain/basic/users?params=%2b[

encodeURIComponent()
对某个参数进行编码，会编码所有特殊字符

编码前：http://10.10.67.67:8080/api/chain/basic/users?params=+[
编码后：http://10.10.67.67:8080/api/chain/basic/users?params=%2b%5B

所以，只需要在需要转义特殊字符的参数上包装一层`&key=${encodeURIComponent(this.key)}`就可以了。

当然，基于“永远不要相信用户输入”的原则，所以在这类不确定字符的情况下，使用post传递json body也许是更好的方法。