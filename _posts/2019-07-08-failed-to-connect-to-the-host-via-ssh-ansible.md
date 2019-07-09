---
layout: post
title: "解决MacOS上的Mysql不能被访问的问题"
subtitle: "Solve the problem which mysql on MacOS cannot be connected"
author: "qingshan"
header-img: "img/post-sample-image.jpg"
header-mask: 0.4
tags:
  - 工作
  - Ansible
---

今天在通过Ansible去操作服务器的时候，发现Ansbile报错：
```sh
Failed to connect to the host via ssh: Permission denied (publickey,password)
```
但是我确认过，在服务端直接ssh目标主机（事先配置好ssh-key）是可以成功连接上的。这台奇诡了！

在github上搜索了issue之后，发现有不少的用户遇到了和我一样问题：
![https://github.com/ansible/ansible/issues/19584](https://github.com/ansible/ansible/issues/19584)

最终在这个issue页的提示下，发现是因为ansible没有指定ssh-key造成的，需要手动配置一下才行。
步骤如下：

#### 1 debug ansible ssh连接过程
```sh
guru@tj-lp140:/etc/ansible$ ansible all -m ping -vvv
Using /etc/ansible/ansible.cfg as config file
Using module file /usr/lib/python2.7/dist-packages/ansible/modules/core/system/ping.py
<35.165.79.66> ESTABLISH SSH CONNECTION FOR USER: None
<35.165.79.66> SSH: EXEC ssh -C -o ControlMaster=auto -o ControlPersist=60s -o KbdInteractiveAuthentication=no -o PreferredAuthentications=gssapi-with-mic,gssapi-keyex,hostbased,publickey -o PasswordAuthentication=no -o ConnectTimeout=10 -o ControlPath=/home/guru/.ansible/cp/ansible-ssh-%h-%p-%r 35.165.79.66 '/bin/sh -c '"'"'( umask 77 && mkdir -p "` echo /tmp/ansible-tmp-1482309322.49-151682117578429 `" && echo ansible-tmp-1482309322.49-151682117578429="` echo /tmp/ansible-tmp-1482309322.49-151682117578429 `" ) && sleep 0'"'"'' 
```
从图中可以看出，`ESTABLISH SSH CONNECTION FOR USER: None`没有指定用户，也没有指定ssh-key。因此下一步就可以手动指定。

#### 2 在ansible的Host主机配置文件中指明用户和key
```sh
[webserver]
35.165.79.66 ansible_user=root ansible_ssh_private_key_file=~/.ssh/id_rsa
```

这样显式的指明，在-vvv进行debug就可以看到没有这个问题了。
