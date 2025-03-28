---
title: Git 追加修改到某个 commit
date: 2024-05-24T10:04:00+08:00
updated: 2025-03-05T10:25:13+08:00
dg-publish: false
---

## 上一个 commit

**使用场景**
 当你提交了代码，结果发现代码中还有的地方要改善，可以通过 git commit --amend 来追加提交，这样就可以避免生成两次提交

有以下两种情况：

### 如果还没有 push 到远程

先 git add 提交修改的文件，就是下面的操作了

```sh
git commit --amend     # 修改上一次的提交

# 进入提交信息编辑界面
# 修改保存退出

```

如果不希望修改提交消息，可以在命令中使用 `--no-edit` 选项。

```sh
git commit --amend --no-edit
```

### 已经 push 到远程了

先 `git add` 提交修改的文件

```sh
git commit --amend     # 修改上一次的提交

# 进入提交信息编辑界面
# 修改保存退出

# 推送 (本地分支:远程分支)
git push  origin master:master

# 强制推送 (本地分支:远程分支)！！！这个不能使用,加-f的命令都别用！！！
git push -f origin master:master

```

## 某个很早之前的 commit

```sh
git add file
git stash
git log # 找到需要提交的对应上一个 commit id
git rebase -i 181362549eaa6099f92726d3aede030e8eb396be
# 这里使用的是 Vim，直接修改对应 commit 前面的 pick 为 edit，保存退出
git stash pop
git add file
git commit -m "对应的提交信息" --amend
git rebase --continue
git push
```
