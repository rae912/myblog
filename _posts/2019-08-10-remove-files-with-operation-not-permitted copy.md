---
layout: post
title: "MacOS下删除文件遇到Operation not permitted"
subtitle: "Remove files failed with Operation not permitted on MacOS"
author: "qingshan"
header-img: "img/home-bg-o.jpg"
header-mask: 0.4
tags:
  - 工作
  - MacOS
---

#1 在 mac osx 下删除一个文件，却提示删除失败：Operation not permitted。

从提示上看就是没有权限删除这个文件，但用了 sudo 之后仍然是 Operation not permitted。

root 都没有权限，那就肯定有其它的原因导致文件无法删除了，Google一下发现是 FreeBSD 系统的文件 flag 在作怪。

通过 ls 命令可以看到文件的 flag，例如 ls -lhdO zhetenga.com.txt：

drwxr-xr-x 3 root staff schg,uchg 0B Mar 25 2014 zhetenga.com.txt

上面的 schg,uchg 就是所谓的 flag了，前面是系统锁文件属性，后面是用户锁文件属性。

现在需要将这些 flag 清除了，修改文件 flag 的命令是chflags，如下：

chflags noschg zhetenga.com.txt

chflags nouchg zhetenga.com.txt



#2 MAC上删除了一个APP，在Launchpad上还显示带问号的图标？

同时按住control、option、command，然后点击那个图标，图表就变成了可删除状态（带有叉叉标志），点击叉叉即可删除。