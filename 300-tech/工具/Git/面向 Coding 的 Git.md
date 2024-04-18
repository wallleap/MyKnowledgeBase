---
title: 面向 Coding 的 Git
date: 2023-02-27 10:31
updated: 2023-02-27 11:14
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - 面向 Coding 的 Git
rating: 1
tags:
  - tool
  - Git
category: web
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: null
url: //myblog.wallleap.cn/post/1
---

## 说明

此篇文章记录在日常 coding 的操作以及一些必要的 git 概念

## git 的初始化设置

```sh
git config --global user.name "Your Name"
git config --global user.email "email@example.com"
```

config 的三个作用域，缺省等同于 `local`

```sh
git config --local ## local只对某个仓库有效
git config --global ## global对当前用户所有仓库有效
git config --system ## system对系统所有登录的用户有效
```

显示 config 的配置，加 `--list`

```sh
git config --list --local ## local只对某个仓库有效
git config --list --global ## global对当前用户所有仓库有效
git config --list --system ## system对系统所有登录的用户有效
git config user.name ## 打印指定 config
```

## 仓库操作

### 初始化仓库

```sh
git init
```

### 克隆远程仓库

```sh
git clone <link>
```

### 将工作区文件加到暂存区

```sh
# 添加单个文件
git add index.html
# 添加多个文件
git add index.html 404.html
# 添加整个目录
git add ./img
# 添加多个目录
git add ./img ./js
# 添加所有文件
git add .
```

## 状态

```sh
git status
```

### 提交信息

```sh
git commit -m 'xxxx'
git commit -v
# 修复上一次提交
git commit -amend -m 'xxxx'
```

### 删除

```sh
# 工作区中删除
git rm index.html
# 只删除 git 仓库中的文件
git rm --cached 404.html
```

### 撤销重置

```sh
git reset HEAD index.html
git reset HEAD .
git reset --hard <commitID>
```

### 比较

```sh
git diff index.html
git diff --cached [commitID] index.html
git diff <commitID> <fileName>
git diff <commitID1> <commitID2>
```

## 查看提交记录

### 只看变更列表

```sh
git log --oneline
```

[![image-20220305220137649](https://cdn.wallleap.cn/img/pic/illustration/202302271031140.png)](https://file.acs.pw/picGo/2022/03/05/20220305220144.png)

### 只看最近 N 次

```sh
git log -n2 --oneline
```

> n2 表示所有分支最近两个。

[![image-20220305220229829](https://cdn.wallleap.cn/img/pic/illustration/202302271031141.png)](https://file.acs.pw/picGo/2022/03/05/20220305220229.png)

### 只看当前分支历史

```sh
git log
```

[![image-20220305220412495](https://cdn.wallleap.cn/img/pic/illustration/202302271031142.png)](https://file.acs.pw/picGo/2022/03/05/20220305220412.png)

## 提交记录

### 修改 commit 的 message

**此操作不建议对已发布到线上的 commit 做修改。**

**修改最新的 commit**

```sh
git commit --amend
```

[![image-20220305222413228](https://cdn.wallleap.cn/img/pic/illustration/202302271031144.png)](https://file.acs.pw/picGo/2022/03/05/20220305222413.png)
**修改老旧 commit**

> 修改老旧 commit 的 message, 需要使用其上一个 commitId。

```sh
git reabase -i commitId
```

[![image-20220305222826712](https://cdn.wallleap.cn/img/pic/illustration/202302271031145.png)](https://file.acs.pw/picGo/2022/03/05/20220305222826.png)

[![image-20220305223007339](https://cdn.wallleap.cn/img/pic/illustration/202302271031146.png)](https://file.acs.pw/picGo/2022/03/05/20220305223007.png)

在新窗口中重新编辑 commit 信息即可。

[![image-20220305223056685](https://cdn.wallleap.cn/img/pic/illustration/202302271031147.png)](https://file.acs.pw/picGo/2022/03/05/20220305223056.png)

### 将多个 commit 整理成一个 commit

**合并连续的 commit**

```sh
git rebase -i commit的父哈希值
# 然后将需要合并的改为s。
```

[![image-20220305223844839](https://cdn.wallleap.cn/img/pic/illustration/202302271031148.png)](https://file.acs.pw/picGo/2022/03/05/20220305223844.png)

**合并非连续的 commit**

将除第一个外需要合并的修改为 `s` 即可进行编辑信息。

```sh
git rebase -i commit的父哈希值
```

进入编辑页面后，将需要合并的写在一起。如果没有自动出现则手动键入即可。需要删除的使用 `s` 命令，保留的使用 `pick`。

[![image-20220305224945894](https://cdn.wallleap.cn/img/pic/illustration/202302271031149.png)](https://file.acs.pw/picGo/2022/03/05/20220305224945.png)

[![image-20220305225107076](https://cdn.wallleap.cn/img/pic/illustration/202302271031150.png)](https://file.acs.pw/picGo/2022/03/05/20220305225107.png)

接下来编辑 commit 信息即可。

> - pick：保留该 commit（缩写:p）
> - reword：保留该 commit，但我需要修改该 commit 的注释（缩写:r）
> - edit：保留该 commit, 但我要停下来修改该提交 (不仅仅修改注释)（缩写:e）
> - squash：将该 commit 和前一个 commit 合并（缩写:s）
> - fixup：将该 commit 和前一个 commit 合并，但我不要保留该提交的注释信息（缩写:f）
> - exec：执行 shell 命令（缩写:x）
> - drop：我要丢弃该 commit（缩写:d）

## 操作暂存区和工作区

### 恢复文件变更

1. 编辑了某文件，但想放弃修改，恢复至与暂存区一致

```sh
git checkout 文件名
```

### 恢复暂存区

**整个暂存区**

```sh
git reset HEAD
```

**部分文件**

```sh
git reset HEAD -- fileName1 fileName2
```

## 分支操作

### 查看分支

**所有本地分支**

```sh
git branch
```

**远程所有分支**

```sh
git branch -r
```

**本地与远程分支**

```sh
git branch -a
```

### 新建分支

**新建不切换**

```sh
git branch [branch-name]
```

**新建并切换**

```sh
git checkout -b [branch]
```

**指定 commit 创建**

```sh
git checkout -b [branch]
```

**新建一个指向 tag 的分支**

```sh
git checkout -b [branch] [tag]
```

**指定的远程分支建立追踪关系**

```sh
git branch --track [branch] [remote-branch]
```

### 切换分支

**切换到指定分支，并更新工作区**

```sh
git checkout [branch-name]
```

**切换到上一个分支**

```sh
git checkout -
```

## 代码合并

### 选择几个 commit

> 此操作建议将最底层开始选择，例如：`git cherry-pick A B`，其中 A 的提交必须早于 B 的提交。

```sh
git cherry-pick <commitHash>
```

转移连续的提交 A 到 B，并且包含 A

```sh
git cherry-pick A^..B
```

### merge 合并

> 此合并方式会生成一个新的 commit 信息

```sh
git merge <branchName>
```

### rebase 合并

```sh
git rebase -i HEAD~2
```

## 标签

### 查看 tag

**所有 tag**

```sh
git tag
```

**查看 tag 信息**

```sh
git show <tagName>
```

### 新建 tag

**在当前 commit 创建**

```sh
git tag <tagName>
```

**指定 commit 新建**

```sh
git tag <tagName> <commitId>
```

### 删除 tag

**删除本地 tag**

```sh
git tag -d [tag]
```

**删除远程 tag**

```sh
git push origin :refs/tags/[tag]
```

### 提交 tag

**提交所有 tag**

```sh
git push [remote] --tags
```

**提交指定 tag**

```sh
git push [remote] [tag]
```

## 常见场景

### 开发中临时加塞了紧急任务

```sh
# 将当前状态暂存起来
git stash
# 将之前的暂存信息弹出来，并会保存暂存队列
git stash apply
# 将之前的暂存信息弹出来，不保存暂存队列
git stash pop
# 查看暂存队列
git stash list
```

## 常用命令脑图

![](https://cdn.wallleap.cn/img/pic/illustration/202302271134599.svg)

