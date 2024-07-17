---
title: Git 追加修改到某个 commit
date: 2024-05-24 10:04
updated: 2024-05-24 10:05
---

## 上一个 commit

**使用场景**  
 当你提交了代码，结果发现代码中还有的地方要改善，可以通过git commit --amend来追加提交，这样就可以避免生成两次提交

有以下两种情况：

1.如果还没有push到远程  
先git add提交修改的文件，就是下面的操作了

```
git commit --amend     // 修改上一次的提交

// 进入提交信息编辑界面
// 修改保存退出

```

2.已经push到远程了  
先git add提交修改的文件，就是下面的操作了

```
git commit --amend     // 修改上一次的提交

// 进入提交信息编辑界面
// 修改保存退出

// 推送 (本地分支:远程分支)
git push  origin master:master

// 强制推送 (本地分支:远程分支)！！！这个不能使用,加-f的命令都别用！！！
git push -f origin master:master

```

## 某个很早之前的 commit

```
git add
git stash
git log # 找到需要提交的对应上一个 commit id
git rebase -i 181362549eaa6099f92726d3aede030e8eb396be
# 这里使用的是 Vim，直接修改对应 commit 前面的 pick 为 edit，保存退出
git stash pop
git add
git commit -m "对应的提交信息" --amend
git rebase --continue
git push
```


