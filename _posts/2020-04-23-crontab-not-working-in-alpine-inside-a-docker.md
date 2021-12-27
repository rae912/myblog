---
layout: post
title: "解决Crontab在 Alpine 容器中工作不正常"
subtitle: "Crontab not work in alpine inside a docker"
author: "qingshan"
header-img: "img/post-bg-rwd.jpg"
header-mask: 0.4
tags:
  - 工作
  - Linux
---


今天使用docker New了一个Alpine Linux作为测试系统。为了验证系统稳定，需要每间隔一段时间来触发一个测试脚本。很常见的场景，于是准备使用crontab。使用crontab -l后，看到如下信息：
```bash
# do daily/weekly/monthly maintenance
# min   hour    day     month   weekday command
*/15    *       *       *       *       run-parts /etc/periodic/15min
0       *       *       *       *       run-parts /etc/periodic/hourly
0       2       *       *       *       run-parts /etc/periodic/daily
0       3       *       *       6       run-parts /etc/periodic/weekly
0       5       1       *       *       run-parts /etc/periodic/monthly
```

和常见的linux发行版不通，这里crontab里预置了一些时间粒度的文件夹，并通过run-parts命令自动扫描对应文件夹下的`具有可执行权限`的脚本。这么一看，还是挺人性化的。于是写好了脚本，测试无误后，命名为`test.sh`放到`hourly`文件夹下，期望每小时执行。
```bash
/etc/periodic/hourly/test.sh
```

但是事与愿违，测试脚本并没有预期的执行。在crontab中改用直接执行的方式，又可以成功。说明问题不在crontab，而是可能出现在run-parts这个命令上：
```bash
# min   hour    day     month   weekday command
0       *       *       *       *       /etc/periodic/hourly/test.sh  # 成功执行
0       *       *       *       *       run-parts /etc/periodic/hourly  # 不执行
```

在`stackoverflow`上搜索后发现，不少人遇到和我一样的问题，最合理的解释如下：
``` bash
The problem is the name of your script. From man run-parts:

If neither the --lsbsysinit option nor the --regex option is given then the names must consist entirely of ASCII upper- and lower-case letters, ASCII digits, ASCII underscores, and ASCII minus- hyphens.

In other words, no extension. Oddly enough, even with the --lsbsysinit option, you can't specify a file like foo.sh since that matches none of the namespaces covered:

If the --lsbsysinit option is given, then the names must not end in .dpkg-old or .dpkg-dist or .dpkg-new or .dpkg-tmp, and must belong to one or more of the following namespaces: the LANANA assigned namespace (^[a-z0-9]+$); the LSB hierarchical and reserved namespaces (^_?([a-z0-9_.]+-)+[a-z0-9]+$); and the Debian cron script namespace (^[a-zA-Z0-9_-]+$).

So, while foo.sh fails, foo.s-h or foo.-sh will work. I have no idea why they've done it this way but presumably they are following some standard or other.

Anyway, you have 2 options, either rename your scripts to not have an extension (extensions are optional in *nix anyway) or you can skip using run-parts altogether. Use this in your crontab instead:

find /home/fabe/tmp/ -prune -type f -executable -exec {} \;
The command above will find all executable files in the target directory and run them. I think that -executable is a GNU extension but you have tagged this as Linux so I assume you have GNU find.
```

简单来说，就是run-parts扫描规则有两条。第一，脚本权限是可执行；第二，脚本名字满足`namespaces (^_?([a-z0-9_.]+-)+[a-z0-9]+$)`，到这里似乎问题就是出在脚本名字上了。再看另外一个详细的解释：
```bash
Every script placed in folder /etc/cron.hourly would run on hourly basis.

However your files needs to be:

executable,
match the Debian cron script namespace (^[a-zA-Z0-9_-]+$).
So for example if you've script with extension (.sh in this case), it won't work.

To print the names of the scripts which would be run, try:

sudo run-parts --report --test /etc/cron.hourly
```

这里明确提到脚本以`.sh`结尾的是不会触发的。于是，将后缀`.sh`去掉，果然就可以了。这里还可以使用`run-parts --test`参数来列出目录下所有将被执行的脚本。
```bash
> run-parts --test /etc/periodic/hourly
/etc/periodic/hourly/test
```

问题解决。就是脚本名字的问题。


