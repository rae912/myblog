---
layout: post
title: "提升 SSH 的连接速度"
subtitle: "How to improve connect speed for SSH"
author: "qingshan"
header-img: "img/post-bg-rwd.jpg"
header-mask: 0.4
tags:
  - SSH
  - TCP
  - UDP
  - Linux
---


从一段时间前开始，本地通过 SSH 连接远程云主机就非常的卡。卡到建立连接要几分钟，输入字符要几秒钟才能在屏幕上显示。通过让 SSH 走代理，的方式，临时缓解这种问题，可以通过如下命令：

```
ssh -o ProxyCommand="nc -X 5 -x 127.0.0.1:8888 %h %p" user@IP
```
上述命令就是让本次 SSH 连接通过本地 8888 端口的代理进行。连接速度由代理服务器的速度决定。

但是每次启动 SSH 连接还要先开一个代理，这个就有点麻烦了。有没有别的解决方案呢？答案肯定是有的，那就是 Mosh.


## Mosh 简介
```

Mosh (mobile shell) is a tool used to connect from a client computer to a server over the Internet, to run a remote terminal.[2] Mosh is similar[3] to SSH, with additional features meant to improve usability for mobile users. 

https://mosh.org/
```
上面是官方简介，简言之就是一个类似基于 SSH 的工具。但是实际上的通信协议是 UDP。这个工具是 MIT 的项目组孵化的。

### 工作原理

原理本质上很简单，就是将 SSH 通过 TCP 协议的数据包，换成了 UDP 协议来替换。因为 UDP 的通信交互和开销较 TCP 要轻量很多，所以在网络不稳定或者被限制的情况下，有更优的表现。

### 安装方法

`mosh` 从 2012 年发布开始，迄今已经内置于各大 Linux 发行版的内置仓库了。可以通过系统自带的软件管理命令安装：

```
# CentOS
yum install mosh

# MacOS
brew install mosh
```

## 使用方法

### 1 手动模式

`mosh` 包含两个手动命令 `mosh-server` 和 `mosh-client`. 在服务端输入前一个命令后，界面上将会返回一个一次性密码：

```bash
MOSH CONNECT 60001 PSSSWORD

mosh-server (mosh 1.3.2) [build mosh 1.3.2]
Copyright 2012 Keith Winstein <mosh-devel@mit.edu>
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
````

这个密码还包括一个 UDP 端口，用于在`mosh-client` 客户端使用：
```
export MOSH_KEY=PSSSWORD
mosh-client SERVER_IP 60001
```

上面客户端命令第一条是将一次性密码写入 Linux 系统环境变量，`mosh-client`将会自动从环境中读取。 `60001`则是生成的连接端口，放在`mosh-client`命令的最后面。不出意外的话，可以瞬间连上。

需要注意的是，这次连接的建立也是一次性的。`mosh-server` 会在 `mosh-client` 断开连接后自动退出进程。所以下次连接必须重新启动`mosh-server`。这是从安全角度出发考虑的。


### 2 自动模式

 `mosh`  推荐采用自动模式，可以更方便的建立连接。

```bash
mosh USER@SERVER_IP
```

上述命令和 SSH 格式几乎一模一样。其工作原理是，先通过传统的 SSH 建立一个普通 TCP 的通道，然后通过执行 `mosh-server` 命令来启动一个`mosh` 会话，并自动获得生成的一次性密码。再自动通过`mosh-client`建立 UDP 连接。相当于是自动化的完成了手动模式的两步操作。


但是自动模式有一个缺点，就是第一次建立通道时候还是采用传统的 TCP 协议。如果连这一步都行不通的话，那么依然无法使用`mosh-client`建立 UDP 通道。

### 3 一个小问题

在 MacOS 作为客户端的情况下，不论是自动模式，还是手动模式，都会有一个报错：

```bash
The locale requested by LC_CTYPE=UTF-8 isn't available here.
Running `locale-gen UTF-8' may be necessary.

The locale requested by LC_CTYPE=UTF-8 isn't available here.
Running `locale-gen UTF-8' may be necessary.

mosh-server needs a UTF-8 native locale to run.

Unfortunately, the local environment (LC_CTYPE=UTF-8) specifies
the character set "US-ASCII",

The client-supplied environment (LC_CTYPE=UTF-8) specifies
the character set "US-ASCII".

locale: Cannot set LC_CTYPE to default locale: No such file or directory
locale: Cannot set LC_ALL to default locale: No such file or directory
LANG=en_US.UTF-8
LANGUAGE=
LC_CTYPE=UTF-8
LC_NUMERIC="en_US.UTF-8"
LC_TIME="en_US.UTF-8"
LC_COLLATE="en_US.UTF-8"
LC_MONETARY="en_US.UTF-8"
LC_MESSAGES="en_US.UTF-8"
LC_PAPER="en_US.UTF-8"
LC_NAME="en_US.UTF-8"
LC_ADDRESS="en_US.UTF-8"
LC_TELEPHONE="en_US.UTF-8"
LC_MEASUREMENT="en_US.UTF-8"
LC_IDENTIFICATION="en_US.UTF-8"
LC_ALL=
Connection to hostname closed.
/usr/local/bin/mosh: Did not find mosh server startup message.
```
这个原因就是因为 MacOS 的终端，和`mosh`所在的目标服务器使用的字符集不一致。`mosh`会采用 UTF-8 作为字符集进行传输。如果客户端默认不一致，就会导致`mosh-server`启动失败。
解决办法就是，在本地和目标服务器的用户配置文件里，指定 UTF-8 作为字符集。

```bash
vi ~/.bash_profile
export LC_ALL=en_US.UTF-8
export LANG=en_US.UTF-8
export LANGUAGE=en_US.UTF-8
```

这样问题就可以圆满解决。

特此记录。

