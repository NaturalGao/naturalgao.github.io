---
title: Git常用命令
categories:
  - Git
tags:
  - Git
abbrlink: caff8000
date: 2019-07-15 13:08:58

---

git 版本控制 常用的命令

<!-- more -->

#### 版本号

```
git --version
```

#### 配置

> 配置文件 为    **~/.gitconfig（全局）** **.git/config （局部）**

```git
git config --global user.email "xxxx@qq.com"
git config --global user.name "xxxx"

# 生成秘钥
ssh-keygen -t rsa -C "xxx@qq.com"

# 测试
ssh -T git@github.com
```

设置默认使用文本编辑器

```
git config --global core.editor sub1
```

#### alias

> 取别名，缩减代码

```
git config --global alias.c commit
```

#### 基础

> ##### 创建

```
git init
```

> ##### 拉取

```
git clone https://github.com/NaturalGao/naturalgao.github.io.git
```

> ##### 添加文件

```
git add .       //添加全部文件
git add a.php   //添加a.php文件
```

> ##### 修改文件名

```
git mv a.php b.php  //修改a.php文件名为b.php
```

> ##### 删除文件

```
git rm a.txt   //版本库和本地文件都会删除
git rm --cached a.txt //只删除版本库，保留本地

```

> ##### 推送

```
git commit -m "描述"

```

> ##### 查看状态

```
git status

```

#### 忽略文件

> 文件： .gitignore

```
*.txt     //忽略所有txt文件
/vendor   //忽略vendor文件夹

```

#### 日志

> 简单描述

```
git log

```

> 查看变动信息

```
git log -p

```

> 最近一次提交

```
git log -p -1

```

> 查看简短信息

```
git log --oneline

```

> 查看变动的文件

```
git log --name-only

```

> 查看类型变化

```
git log --name-status

```

#### 修改最新一次提交

```
git commit --amend

```

#### 分支

> 查看分支

```
git branch

```

> 创建分支

```
git branch dev  //创建 dev分支

```

> 切换分支

```
git checkout dev   //切换到 dev分支

```

> 创建&&切换分支

```
git checkout -b dev  //创建并且切换到dev分支

```

> 合并分支

```
git merge dev

```

> 删除分支

```
git branch -d dev

```

> 已合并的分支

```
git branch --merged

```

> 没有合并的分支

```
git branch --no-merged

```

> 删除没有合并的分支

```
git branch -D dev

```

#### 暂存区

> 添加

```
git stash

```

> 查看列表

```
git stash list

```

> 恢复

```
git stash apply    //恢复全部
git stash apply stash@{1}  //恢复第一个

```

> 删除

```
git stash pop

```

> 移除文件

```
git reset HEAD a.php  //移除a.php文件

```

> 恢复文件内容

```
git checkout -- a.php

```

#### 标签

> 添加

```
git tab '标签'

```

#### 生成压缩包

```
git archive master --prefix = "naturalGao/" --forma=zip > natural.zip

```

#### 移动master分支到最新

```
git rebase master

```

#### 查看远程分支

```
git branch -a

```

#### 拉取某个远程分支

```
git pull origin dev:dev

```

#### 删除远程分支

```
git push origin --delete dev

```

