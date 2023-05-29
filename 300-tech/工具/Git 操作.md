---
title: Git 操作
date: 2022-11-11 09:48
updated: 2022-11-11 17:31
cover: //cdn.wallleap.cn/img/pic.jpg
author: Luwang
comments: true
aliases:
  - Git 从零到精通
rating: 10
tags:
  - web
  - Git
category: web
keywords:
  - web
  - Git
  - GitHub
description: Git 操作
source: null
url: //wallleap.github.io/learnGit/
---

[Git学习 (wallleap.github.io)](http://wallleap.github.io/learnGit/)

## 1、初始化环境

 下载Git、注册Github等账号

 配置本地Git信息

```zsh
git config --global user.name "Your Name"
git config --global user.email "email@example.com"
```

添加了 `--global` 是针对全局生效的，如果某个仓库需要单独的，只需在这个仓库中使用 `--local` 替换掉 `--global` 配置一遍即可

 生成 keygen

```zsh
ssh-keygen -t rsa -C "youremail@example.com"
```

将生成的 id_rsa.pub 中的公钥放到需要提交的服务器上进行验证

 测试连接

```zsh
ssh -T git@github.com
```

显示用户名成功认证则说明配置成功了

![](https://cdn.wallleap.cn/img/pic/illustrtion/20221018002636.png)

## 2、理论

所有的 Git 信息都在 `.git` 目录下

### 仓库的本质

- 一个所有的历史都可以追溯的文件夹
	- 历史的最小单位是仓库在某个时刻的状态（快照）
	- 通常用 SHA-1 表示
- 哪些东西可以标识仓库的某个状态
	- commit 对象的 SHA-1（无需写全）
	- 分支（branch）
	- HEAD
	- 标签（tag）

### 提交的本质

- 仓库从一个状态转移到另外一个状态时发生的变化
- commit 对象是不可变的
	- `git commit --amend` 修改 commit 说明会创建一个新的节点，在树上会分叉
- 持续的工作和提交，在仓库中建立了一棵树

![](https://cdn.wallleap.cn/img/pic/illustrtion/202211111111423.png)

### 分支的本质

- ⼀个可以移动的指针，指向某个提交对象
	- 新建⼀个分⽀，就是新建⼀个指针
	- 可以基于某个状态新建分支 `git checkout -b new-branch {仓库的某个状态}`
- 特殊的指针 HEAD（指向当前的头）
	- `HEAD~1`  `master~1`  `a2dde3~3`
	- `HEAD^`

![](https://cdn.wallleap.cn/img/pic/illustrtion/202211111124028.png)

### 远程仓库/本地仓库与 fork

- Git 是分布式版本控制系统
- 远程仓库/本地仓库/ fork 本质上就是复制了整个⽂件夹
- pull/push/push --force
	- `git pull` = `git fetch` + `git merge`
	- `git pull --rebase` = `git fetch` + `git rebase`
	- 只有远程分⽀和本地分⽀历史相同时，才能 push

## 3、了解Git基本命令

### 初始化仓库

```zsh
git init
```

### 克隆代码

```zsh
git clone 仓库链接
```

### 创建 .gitignore 文件

```zsh
touch .gitignore
```

### 添加当前目录下文件到暂存区

```zsh
git add .
# 添加部分文件
git add 文件名|文件夹
```

工作区（working tree），暂存区（index /stage），本地仓库（repository）

### 查看仓库中文件状态

```zsh
git status
```

### 将文件从暂存区中移出

```sh
git rm --cached 文件名
# 移出文件夹或所有文件
git rm --cached -r 文件夹或.
```

### 将暂存区中的文件进行提交

```zsh
git commit -m "本次提交的说明"
```

### 查看远程仓库

```zsh
git remote -v
```

### 添加远程仓库

```zsh
git remote add origin 仓库链接(ssh或https)
git remote set-url origin 新仓库链接
```

### 推送代码

```zsh
git push origin master:master
# origin是远程仓库别名、master是分支名
# 第一次可以多加一个参数
git push -u origin master
```

### 拉取代码

```zsh
git pull origin master
```

### 查看文件修改内容

```zsh
git diff 文件名
```

### 查看历史记录

```zsh
git log
git log --pretty=oneline
```

### 查看历史 tag

```sh
git tag
```

### 查看历史的某一状态

（/用户名/仓库/tree/可以是以下四种中后面的 eg: `github.com/wallleap/repo/tree/89089adadhiui……`）

```sh
git checkout commitID（不需要写全 只要是唯一就行）
git checkout HEAD
git checkout 分支名
git checkout tag名字
```

### 查看命令记录

```zsh
git reflog
```

### 版本回退

```zsh
# 回退到上一个版本(Git中HEAD代表当前版本)
git reset --hard HEAD^
# 回退到上上个版本
git reset --hard HEAD^^
# 回退到上100个版本
git reset --hard HEAD~100
# 根据版本号回退
git reset --hard 版本号前几位或写全
```

### 分支管理

```zsh
# 创建分支
git branch 分支名
# 切换分支
# git checkout 分支名
git switch -c 分支名
# 可以合写为一条
git checkout -b 分支名
# 查看当前分支
git branch
# 切换回master分支
# git checkout master 或
git switch master
# 合并分支
git merge 分支名
# 删除分支
git branch -d dev
```

## 4、冲突解决

### git merge 与冲突解决

- 合并的本质
	- 将两个或者多个状态（代码快照/代码副本）合并
	- git 逐行检查冲突，并尝试自动解决
	- 解决失败，报错
- 解决冲突
	- 通过人类的智慧，挑选正确的版本
	- 严格按照命令提示进行冲突解决
		- `--continue`/`--abort`/`--skip`

### rebase 与 merge

- merge：直接将两个状态合并，并产生一个合并提交
	- 将两个或多个开发历史交汇在一起

- rebase：将某个状态上的变更（commit）挨个重演
	- 在另外一个分支的基础上重演提交

- merge 的优缺点
	- 优点
		- 简单，易于理解
		- 记录了完整的历史
		- 冲突解决简单
	- 缺点
		- 历史杂乱无章
		- 对 bisect 不友好

- rebase 的优缺点
	- 优点
		- 分支历史是一条直线，清楚直观
		- 对 bisect 友好
	- 缺点
		- 较为复杂，劝退新手
		- 如果冲突可能要重复解决（或者 squash）
		- ~~会干扰别人？（和别人共享的分支永远不要 force push）~~
	- 使用场景
		- 定时将你的分支和主干进行同步

- `git merge --squash`
	- 优点
		- 把所有变更合在一起，更容易阅读，bisect 友好
		- 想要回滚或者 revert 非常方便
	- 缺点
		- 丢失了所有的历史记录

- 充分了解 merge 和 rebase，然后看自己的喜好

- 在任何需要和其他人共享的分支上，不要用 force push

- 在自己独占的分支上，尽量使用 rebase

### git checkout

- 将代码复原到任何⼀个状态上，但不修改仓库中的任何信息
	- `git checkout v1.0.0`
	- `git checkout fd56bd`
	- `git checkout master`
	- `git checkout origin/master`
- 新建分⽀
	- `git checkout -b {任何可以代表分⽀状态的东⻄}`
- 在⼯作⽬录和暂存区移动⽂件
	- `git checkout --`

### git reset

- 强⾏将当前分⽀HEAD指针移动到指定状态
	- `git reset HEAD --hard`
	- `git reset origin/master`
	- `git reset 3da2dfe`
	- `git reset HEAD~3`
- --soft/--mixed/--hard
- 使⽤场景
	- 迅速将分⽀恢复到某个状态
	- squash commits，例如⽅便 rebase

### git cherry-pick

- 将某个提交在另外的分⽀上重放
	- `git cherry-pick 1a2b3c4d`
- 使⽤场景
	- 希望把⼀个 bug fix 同步到较⽼的产品中
	- 在 master 上进⾏的变更希望进⼊ release 环节

### git revert

- 产⽣⼀个「反向提交」，撤销某个 commit
	- `git revert 1a2b3c4d`
- 产⽣的「反向提交」可以再次进⾏ revert
- 使⽤场景
	- 撤销历史中的某个更改，如 bug 或不恰当的功能
	- 回滚某次发布

### git bisect

- 在历史中查找某处引⼊的 bug
	- `git bisect start/reset`
	- `git bisect good/bad`
	- `git bisect run`
- 使⽤场景
	- 在问题可以稳定重现的时候，迅速定位出问题的代码
	- 有的时候这⽐直接去 debug 更快捷

### git stash(储藏)

- 临时性将⼯作⽬录的变更储存起来，然后清空⼯作⽬录
	- `git stash`
	- `git stash list`
	- `git stash pop`
- 使⽤场景
	- 你正在开⼼的开发，突然来了⼀个线上 bug
	- 你只好将⼿头的⼯作全部储存起来之后，开始别的⼯作
	- 结果⼜来了⼀件优先级更⾼的 bug
	- 完成别的⼯作之后回来继续
	- 类⽐「存档/读档」

## 5、规范化提交信息

[[规范化 Commit 提交信息]]

## 6、缩写

给 Git 设置别名，可以少打很多字母

在 `~/.bashrc` 下添加：

```sh
alias gst='git status'
alias ga='git add'
alias gc='git commit'
alias gl='git pull'
alias gp='git push'
alias gd='git diff | mate'
alias gau='git add --update'
alias gc='git commit -v'
alias gca='git commit -v -a'
alias gb='git branch'
alias gba='git branch -a'
alias gco='git checkout'
alias gcob='git checkout -b'
alias gcot='git checkout -t'
alias gcotb='git checkout --track -b'
alias glog='git log'
alias glogp='git log --pretty=format:"%h %s" --graph'
alias gr='git reset'
alias grhh='git reset HEAD --hard'
alias gcp='git cherry-pick'
```

常用：

```sh
ga . # 添加到缓存区
gst # 查看状态
gc -v # 加提交说明 或者用下面这个
gc -m push # 提交说明
gp -f # push 到远端
```

## 7、git ⼯作流

- ⼀个或者两个共享的稳定分⽀
	- master/main
	- develop
- 好处：
	- 清楚直观
	- 每个提交代表⼀个单独的完整的功能点，⽅便回滚
	- 每个提交都是稳定的，⽅便进⾏bisect
- 临时性的功能分⽀
	- feature
	- bugfix
	- release

### 8、⾃⾏搭建Git服务器

- gogs
	- 轻量，简单
	- `docker run -v ~git/gogs:/data -p 127.0.0.1:10022:22 - p 3000:3000 gogs/gogs`
- As root
	- 【安装docker】`apt-get update && apt-get install docker.io`
	- 【创建git⽤户】`adduser git`
	- 【使git⽤户可以运⾏docker】`usermod -aG docker git`
	- 【创建gogs运⾏⽬录】`mkdir -p /app/gogs`
	- 【改变⽬录的所有⼈】`chown git:git /app/gogs`
- As git
	- 【创建SSH⽬录】`mkdir -p ~/gogs/git/.ssh`
	- `ln -s ~/gogs/git/.ssh ~/.ssh`
	- 【⽣成SSH key】`ssh-keygen -t rsa -P ''`
	- `cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys`
	- 【更严格的权限】`chmod 700 ~/.ssh && chmod 600 ~/.ssh/*`
	- `cat >/app/gogs/gogs <<'END' #!/bin/sh ssh -p 10022 -o StrictHostKeyChecking=no git@127.0.0.1 \ "SSH_ORIGINAL_COMMAND="$SSH_ORIGINAL_COMMAND" $0 $@" END`
	- `chmod 755 /app/gogs/gogs`
