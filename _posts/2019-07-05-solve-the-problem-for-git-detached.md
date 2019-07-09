---
layout: post
title: "Git HEAD detached from XXX (git HEAD 游离) 解决办法"
subtitle: "Solve the problem for the git of HEAD detached"
author: "qingshan"
header-img: "img/post-bg-product-manager.webp"
header-mask: 0.4
tags:
  - 工作
  - git
---

Git HEAD detached from XXX (git HEAD 游离) 问题：
Git 中的 HEAD 可以理解为一个指针，我们可以在命令行中输入 cat .git/HEAD 查看当前 HEAD 指向哪儿，一般它指向当前工作目录所在分支的最新提交。当使用 git checkout < branch_name> 切换分支时，HEAD 会移动到指定分支。但是如果使用的是 git checkout < commit id>，即切换到指定的某一次提交，HEAD 就会处于 detached 状态（游离状态）。

HEAD 处于游离状态时，我们可以很方便地在历史版本之间互相切换，比如需要回到某次提交，直接 checkout 对应的 commit id 或者 tag 名即可。也就是说我们的提交是无法可见保存的，一旦切到别的分支，游离状态以后的提交就不可追溯了。
它的弊端就是：在这个基础上的提交会新开一个匿名分支！也就是说我们的提交是无法可见保存的，一旦切到别的分支，游离状态以后的提交就不可追溯了。解决办法就是新建一个分支保存游离状态后的提交：

解决步骤:

#### 1 git branch -v 查看当前领先多少 
```sh
git branch -v
```
#### 2 新建一个 temp 分支，把当前提交的代码放到整个分支 
```sh
git branch temp
git checkout temp
```
#### 3 checkout 出要回到的那个分支，这里是 dev1 
```sh
git checkout dev1
```
#### 4 然后 merge 刚才创建的临时分支，把那些代码拿回来 
```sh
git merge temp
```
#### 5 git status 查看下合并结果，有冲突就解决 
```sh
git status
```
#### 6 合并 OK 后就提交到远端，删除刚才创建的临时分支
```sh
git push origin dev1
```
#### 7 删除刚才创建的临时分支
```sh
git branch -d temp
```
