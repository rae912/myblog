---
layout: post
title: "Java Spring关于RFC 7230 and RFC 3986的报错"
subtitle: "RFC7230 and RFC3986 in Java Spring"
author: "qingshan"
header-img: "img/home-bg-o.jpg"
header-mask: 0.4
tags:
  - 工作
  - Java
---

最近在开发过程中，遇到一个很坑的问题，查了很久耽误了很多时间，记录一下：

### 现象
在后端为Java Spring时，当浏览器向其发起某个特定Get请求时候，后端直接报错：
```
java.lang.IllegalArgumentException: Invalid character found in the request target. The valid characters are defined in RFC 7230 and RFC 3986
```

###排查
将触发报错的URL通过Postman调试工具模拟浏览器再次向后端发送请求，没有任何问题。

网上搜索资料，有资料显示可能是`HttpHeaderSize`设置太小，需要修改Tomcat配置，新加一个字段`maxHttpHeaderSize`。配置后为：
```xml
<!--<Connector port="1080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="1443" />-->
    <Connector URIEncoding="UTF-8" acceptCount="100" compressableMimeType="text/html,text/xml,text/javascript,text/css,text/plain"
    compression="on" compressionMinSize="10240" connectionTimeout="20000" disableUploadTimeout="true"
    enableLookups="false" maxHttpHeaderSize="81920" maxSpareThreads="75" maxThreads="150"
    minSpareThreads="25" noCompressionUserAgents="gozilla, traviata" port="1080"
    protocol="HTTP/1.1" redirectPort="1443" />
```

乍一看，URL的长度的确很长，联想到Get方法是有长度限制的，很有可能是达到了Request的长度上限Postman可以正常发送，证明这种猜想不成立，继续寻找其他原因。

### 继续排查
根据关键字`RFC 7230 and RFC 3986`继续搜索，发现这是两个规范，核心描述如下：
>* RFC 3986文档对Url的编解码问题做出了详细的建议，指出了哪些字符需要被编码才不会引起Url语义的转变，以及对为什么这些字符需要编码做出了相应的解释。

>* RFC 3986文档规定，Url中只允许包含英文字母（a-zA-Z）、数字（0-9）、-_.~4个特殊字符以及所有保留字符（! * ' ( ) ; : @ & = + $ , / ? # [ ]）。

还有一些字符当直接放在Url中的时候，可能会引起解析程序的歧义，这些字符被视为不安全字符。
>* 空格：Url在传输的过程，或者用户在排版的过程，或者文本处理程序在处理Url的过程，都有可能引入无关紧要的空格，或者将那些有意义的空格给去掉。
引号以及<>：引号和尖括号通常用于在普通文本中起到分隔Url的作用
>* #：通常用于表示书签或者锚点
>* %：百分号本身用作对不安全字符进行编码时使用的特殊字符，因此本身需要编码
>* {}|\^[]`~：某一些网关或者传输代理会篡改这些字符

### 可能的解决方案

1、切换版本到7.0.73以下，这个不实际。
2、修改Tomcat源码，这个也不实际。
3、前端请求对URL编码。
4、修改Get方法为Post方法。
5、因{}是不安全字符，默认被tomcat拦截。如果需要在URL中传输json数据，在catalina.properties中添加支持。

### 最终的解决方案
原因找到后，知道原来是URL里面包含了“|”这个符号导致的。所以我们将其更改为其他符号即可。
最终，将URL中的`“|”`符号替换为`“+”`符号。再在后端debug，发现已经可以正常收到URL中的参数，但是后端会自动将`“+”`替换成空格`“ ”`。

因此，将`“+”`替换成空格`“ ”`之后的字符串进行处理即可。


