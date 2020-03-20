---
layout: post
title: "匪夷所思的中文编码问题"
subtitle: "Confused Chinese encoding problem"
author: "qingshan"
header-img: "img/post-bg-halting.jpg"
header-mask: 0.4
tags:
  - 工作
  - MySQL
---

今天通过Python程序在数据库里查询一系列数据，返回结果为空，无报错。但是我肯定数据是存在的，于是手动上数据库搜索，一下子又出来了。在用程序debug出搜索关键字，复制到数据库，又查询不出，简直奇怪。而且更奇怪的是，被查询的中文字符串肉眼上见是一模一样的，居然一个可以，一个不行。具体如下：

```txt
查询的出：FILORGA 菲洛嘉

查询不出：FILORGA 菲洛嘉
```

## 排查过程
经过一顿排查，怀疑是中文编码的问题，通过工具将其转成Unicode：
```txt
FILORGA 菲洛嘉 \u0046\u0049\u004C\u004F\u0052\u0047\u0041\u0020\u83F2\u6D1B\u5609\u0020

FILORGA 菲洛嘉 \u0046\u0049\u004C\u004F\u0052\u0047\u0041\u00A0\u83F2\u6D1B\u5609\u00A0
```

可以看到，在上面两段字符串中，空格符的部分的编码是不一样的！！！一个是u0020，一个u00A0。这就是罪魁祸首。

## 原因：三种空格unicode(\u00A0,\u0020,\u3000)表示的区别

```
1.不间断空格\u00A0,主要用在office中,让一个单词在结尾处不会换行显示,快捷键ctrl+shift+space ;
2.半角空格(英文符号)\u0020,代码中常用的;
3.全角空格(中文符号)\u3000,中文文章中使用;
```