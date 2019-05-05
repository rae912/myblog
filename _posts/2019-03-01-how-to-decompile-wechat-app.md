---
layout: post
title: "微信小程序反编译获取源码实战"
subtitle: "How to decompile wechat app"
author: "qingshan"
header-img: "img/post-bg-css.jpg"
header-mask: 0.4
tags:
  - 工作
  - 小程序
  - 反编译
---


2019年3月，因为好奇某小程序的实现细节，想查看其源代码进行学习。在网上检索后，总结获取小程序源代码的方式主要有三种：

> 1 找小程序作者索要；

> 2 通过小程序appid和版本号通过微信服务器下载；

> 3 通过小程序缓存文件反编译；

经过实践，第一种方法可能性为0，第二种方法也于2018年初被微信封杀。只有第三种方法可以尝试。经过实践，成功达到目的，过程记录如下。

### 准备工作

* 反编译工具
> wxappUnpacker: https://github.com/qwerty472123/wxappUnpacker

* 安卓模拟器
> 网易mumu: http://mumu.163.com

* 安卓adb工具
> adb tool

* 源码查看工具
> WebStorm: https://www.jetbrains.com/webstorm/

### 实操阶段
##### 获取缓存文件
要获取小程序的缓存文件。由于微信的缓存目录在系统目录下，不论是安卓系统还是iOS系统都需要越狱才可以。鉴于手头没有符合条件（越狱）的硬件，这里采用模拟器来进行。

打开模拟器后，安装微信。这里使用的默认是当前较新的7.0.0版本。登录微信后，打开目标小程序。此时会发现，小程序在加载完毕后，不是进入默认页面，而是退回到加载前的页面。

经过各模拟器论坛资料收集后发现，微信在**V6.2.5.0**版本起，针对模拟器做了一些限制，小程序故意不能在模拟器打开。但是并不影响后续工作，因为这一步的目的我只是要获取缓存，能不能运行无所谓。

在shell里输入以下命令，通过adb链接模拟器：
```
adb kill-server && adb server && adb shell
```
默认是root权限，cd到进入小程序缓存所在目录：
```
cd /data/data/com.tencent.mm/MicroMsg/{用户标识符}/appbrand/pkg && ls -l
```
在确认其中有以.wxapkg结尾的缓存文件之后，全部拉回到电脑上准备反编译。
```
adb pull /data/data/com.tencent.mm/MicroMsg/{用户标识符}/appbrand/pkg
```

##### 反编译
安装好NodeJs环境后，在wxappUnpacker工作目录利用以下命令安装wxappUnpacker工具依赖：
```
npm install
```

完成之后利用以下的命令进行反编译：
```
node wuWxapkg.js {目标缓存}.wxapkg
```

此时，应该有编译过程日志显示：
![](https://ae01.alicdn.com/kf/HTB1Bf8CJNYaK1RjSZFnq6y80pXaj.jpg)

尽管可以看到有错误提示，但是并不用紧张。当前目录下会出现与刚才缓存一样名字的文件夹。里面有一系列的前端工程文件，说明已经反编译成功了。

#### 代码可视化
由于只是观摩学习，所以需要能够比较直观的阅读。将代码导入到Webstorm新建工程，随意打开一个文件，可以发现代码是没有格式的，全部都缩在一起。而且，代码的变量名都是一个单字母的。所以要使用Webstorm的代码格式化功能格式化一下。

![](https://ae01.alicdn.com/kf/HTB1MwpwJMHqK1RjSZJnq6zNLpXaI.jpg)

在图中所示的code菜单下选择Reformat Code，片刻后，即可获取得到格式化之后的，适合阅读的代码了。

