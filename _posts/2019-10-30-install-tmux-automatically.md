---
layout: post
title: "Tmux和Htop自动安装过程"
subtitle: "Install Tmux and Htop automatically"
author: "qingshan"
header-img: "img/post-bg-product-manager.webp"
header-mask: 0.4
tags:
  - 工作
  - Linux
---

# 1 Tmux自动安装
Tmux是我在工作中经常用的工具，除了可以取代screen、nohup等后台进程管理以外，还可以方便地将一个项目所有相关的进程整合到一个session界面中。我通常一个session开四个4X4的pane，一个项目运行主进程（或者日志监控）、一个数据库交互、一个资源监控、一个备用。

今天在新机器上发现通过yum安装的tmux的一直提示：faild to connect to server。应该是安装过程中有什么问题导致没有安装成功。考虑到以后经常会用，而且yum提供的也是很旧版本的tmux，索性整理一个自动安装的脚本，为未来提供方便。

```bash
sudo yum install gcc kernel-devel make ncurses-devel libevent-devel&&
wget -c https://github.com/tmux/tmux/releases/download/2.9/tmux-2.9.tar.gz &&
tar -xvf tmux-2.9.tar.gz &&
cd tmux-2.9 &&
./configure && make && sudo make install
```

以上脚本跑完后，输入命令看下是否有成功：`tmux ls`。如果没有错误提示，就是成功了。

# 2 Htop自动安装
htop是一款简单易用的命令行可视化系统资源监控工具，也是常用的。

```bash
wget -c https://hisham.hm/htop/releases/2.2.0/htop-2.2.0.tar.gz &&
sudo yum install -y ncurses-devel &&
tar -xvf htop-2.2.0.tar.gz &&
cd htop-2.2.0 &&
./configure && make && sudo make install
```

脚本跑完后，运行`htop`就可以看到资源监控界面了。