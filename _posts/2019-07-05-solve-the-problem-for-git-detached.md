---
layout: post
title: "Git HEAD detached from XXX (git HEAD 游离) 解决办法"
subtitle: "Solve the problem for the git of HEAD detached"
author: "qingshan"
header-img: "img/post-bg-product-manager.webp"
header-mask: 0.4
tags:
  - 工作
  - 趣闻
---

Git HEAD detached from XXX (git HEAD 游离) 解决办法:

#### 1 git branch -v 查看当前领先多少 
#### 2 新建一个 temp 分支，把当前提交的代码放到整个分支 
#### 3 checkout 出要回到的那个分支，这里是 dev1 
#### 4 然后 merge 刚才创建的临时分支，把那些代码拿回来 
#### 5 git status 查看下合并结果，有冲突就解决 
#### 6 合并 OK 后就提交到远端，删除刚才创建的临时分支
