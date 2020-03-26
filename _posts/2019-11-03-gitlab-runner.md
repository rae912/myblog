---
layout: post
title: "GitLab和GitLab-Runner部署方法"
subtitle: "The way for gitlab and runner deploying"
author: "qingshan"
header-img: "img/post-bg-halting.jpg"
header-mask: 0.4
tags:
  - 工作
  - Git 
---


由于安全限制，内网中很多网络资源无法直接运行和下载，因此采用手动（脚本）和离线资源包的形式实现半自动安装。

# 一. 安装依赖

```bash
sudo yum install curl openssh-server openssh-clients postfix cronie &&
sudo yum install policycoreutils-python policycoreutils-python-utils &&
sudo service postfix start &&
sudo chkconfig postfix on &&
    #这句是用来做防火墙的，避免用户通过ssh方式和http来访问。
sudo lokkit -s http -s ssh 
```

# 二. 安装 GitLab 准备工作

1.先创建 yum 库 VIM /etc/yum.repos.d/gitlab_gitlab-ce.repo

```bash
[gitlab-ce]
name=gitlab-ce
baseurl=http://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7
repo_gpgcheck=0
gpgcheck=0
enabled=1
gpgkey=https://packages.gitlab.com/gpg.key
```

2.执行安装 GitLab

```bash
sudo yum makecache &&
sudo yum install gitlab-ce &&
sudo gitlab-ctl reconfigure  #Configure and start GitLab

# 如果出现报错：canonical path for /var/opt/gitlab/gitlab-rails/etc/gitlab_shell_secret restorecon: No such file or directory.  则执行下面语句即可解决
mkdir -p /var/opt/gitlab/gitlab-rails/etc
```

# 三. 离线安装 gitlab-runner

1.下载手动安装包

```bash
 # Linux x86-64
sudo wget -O /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64

# Linux x86
#sudo wget -O /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-386

# Linux arm
#sudo wget -O /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-arm

# 内网下载
wget -c http://10.107.68.143:8001/gitlab-runner-linux-amd64

```

2.指定权限并运行起来

```bash

 # 赋予可执行权限
sudo chmod +x gitlab-runner-linux-amd64 &&

# 创建软连接到/usr/local/bin
sudo ln -s /home/apps/software/gitlab-runner-linux-amd64 /usr/local/bin/gitlab-runner


# 创建 GitLab CI 用户
#sudo useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash
 
# 安装
#sudo gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
sudo /usr/local/bin/gitlab-runner install --user=apps --working-directory=/home/apps &&

# 运行
sudo /usr/local/bin/gitlab-runner start

```

3.一键配置 runner

```bash
 # 获取IP地址
host_ip=$(ip addr | grep 'state UP' -A2 | tail -n1 | awk '{print $2}' | awk -F"/" '{print $1}')

# 获取主机名
host_name=$HOSTNAME

# token
token=LCveTas1jyh1tkPibTem

sudo gitlab-runner register \
  --non-interactive \
  --url "http://10.107.68.75/" \
  --registration-token "$token" \
  --executor "shell" \
  --description "[测试]离线平台CI/CD $host_name" \
  --tag-list "$host_ip, $host_name" \
  --run-untagged="true" \
  --locked="false" \
  --access-level="not_protected"
```

4.runner 高级配置
高级配置包括指定 build 的路径`builds_dir`等等

```bash
https://docs.gitlab.com/runner/configuration/advanced-configuration.html#the-runners-section
```

5.错误处理

```bash
fatal: git fetch-pack: expected shallow list
ERROR: Job failed: exit status 1
```

目标机器 Git 版本为默认的 1.8.3.1，这个版本太低，需要升级 Git

```bash
sudo yum install -y curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker &&
sudo yum remove git -y &&
wget -c http://10.107.68.143:8001/git-2.24.0.tar.gz &&
tar -xvf git-2.24.0.tar.gz &&
cd git-2.24.0 &&
autoconf && ./configure && make && sudo make install &&
git --version
```

如果是下面的报错：

```bash
fatal: Unable to find remote helper for 'http'
```

是因为 /usr/libexec/git-core/ 或者 /usr/local/libexec/git-core 路径没在 PATH 环境变量中。追加设置到系统环境变量/etc/profile就可以了：

```bash
# 先确认一下git-core的位置
#ls -a /usr/local/libexec/git-core

# 正确的git-core目录下，有很多很多git-* 开头的应用程序， 追加到系统变量/etc/profile
sudo echo 'PATH=$PATH:/usr/local/libexec/git-core' | sudo tee -a /etc/profile
```

6.取消 runner 注册

```bash
sudo gitlab-runner unregister --all-runners
```

7.坑点
一切问题解决就绪后，就可以开始愉快地跑起来runner了。但是实际使用过程中，还是发现了很让人摸不着头脑得事情。我的使用场景是批量服务器里部署服务的。但是即使我选中了公共的tags，但是实际上任务只会随机的在其中一台服务器上运行。让我丈二和尚摸不到头脑。官方文档也没说明这是为什么，可能很少有人拿来批量部署相同代码的缘故？
经过仔细检索，发现下面这段话：

```bash
When a build for GitLab CI is triggered, it will execute jobs listed in the .gitlab-ci.yml file. Think of these jobs as independent, concurrent steps in your build. These jobs are executed by any available runner capable of completing that job. However, where I think you're getting tripped up is that a job will only be completed once, and by the first available runner. Think of the runners as a pool of resources, not as build steps. Having multiple runners allows you to execute jobs in parallel.
```

翻译成人话，就是每个job只会随机的运行在第一个被发现满足要求的runner上。如果要批量运行同一个任务，就必须创建多个任务，指定不同的runner。于是只能很二地将一个job复制N次，然后每个job指定不同的tags，问题解决。感觉这样很不优雅，不知道有没有更优雅的方式？

8. Docker下日志清理
Gitlab docker运行一段时间后，发现磁盘空间费得很快，经过排查，是docker的日志占用了，日志位置在下面，要root权限才可以看到：
```bash
# root user
cd /var/lib/docker/container/
du -h --max-depth 1
```

解决方法： 配置docker deamon.json，限制docker产生的日志量
```bash
# vim /etc/docker/deamon.json
/etc/docker/daemon.json file:

{
  "log-driver": "json-file",
  "log-opts": {"max-size": "100m", "max-file": "3"}
}
```

然后reload和重启：  

```bash
# Flush changes:
sudo systemctl daemon-reload

# Restart
sudo systemctl restart docker
```

*注意！！！* 新的deamon.json仅对新的容器有效。所以必要时需要重新生成容器。

# 参考资料：
1. https://docs.docker.com/config/daemon/systemd/
