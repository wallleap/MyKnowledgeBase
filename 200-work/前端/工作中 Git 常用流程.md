---
title: 工作中 Git 常用流程
date: 2024-08-30T05:22:48+08:00
updated: 2024-08-30T05:23:02+08:00
dg-publish: false
---

- 日常在 dev 分支进行开发
- 功能点则创建 feature-x 分支，开发完合并到 dev 分支
- 修复 bug 则创建 fix-x 分支，修复完成后合并到 dev 分支
- 由领导将 dev 分支合并到 main/master 分支

```sh
$ cd project
$ git init
# 或者克隆已有仓库 git clone ... project & cd project
$ git branch dev 
$ git branch feature-1
$ git add .
$ git stash
$ git branch fix-1
$ git branch dev 
$ git stash pop
$ git add .
$ git commit -m "chore: 提交描述"
$ git push
```
