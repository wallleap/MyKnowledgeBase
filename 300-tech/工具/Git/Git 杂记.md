---
title: Git 杂记
date: 2024-08-22T02:25:26+08:00
updated: 2024-08-22T02:25:29+08:00
dg-publish: false
---

### commit 提交完怎么撤回

```
git reset HEAD~1
git reset HEAD~2  (撤回2次)
```

## commit

```
git add .
git commit -m "some str"
git push
```

- 无新文件添加用下面

```
git commit -am "some str"
git push
```

- 重新提交

```
git commit --amend -m "重新提交注释"
```

## Git 开发错分支了

- 提交代码时

```
git add .
git stash (把暂存区的代码放入到git暂存栈)
git checkout name(切换到正确的分支)
git stash pop(把git暂存栈的代码放出来)
```

- 提交代码后

```
git reset HEAD~1  （最近一次提交放回暂存区, 并取消此次提交）
git stash (暂存区的代码放入到git暂存栈)
git checkout (应该提交代码的分支)
git stash pop (把git暂存栈的代码放出来)
git checkout  (切换到刚才提交错的分支上)
git push origin 错误的分支 -f  (把文件回退掉)
```

## 只想把一个提交合并到其它分支上

```
git log
git cherry-pick **id

(有冲突)
git add .
git cherry-pick --continue
```

- --continue 解决冲突后，让 Cherry pick 过程继续执行。
- --abort 发生代码冲突后，放弃合并，回到操作前的样子.
- --quit 发生代码冲突后，退出 Cherry pick，不回到操作前

## git rebase

- 合并多次 commit 提交

```
git rebase -i [startpoint] [endpoint]

(进入交互界面， 将p -> s)

p, pick: 保留该 commit
s, squash: 将该 commit 和 前面一个 commit 合并

git checkout -b temp (基于无名的临时分支创建temp分支)
git checkout dev  (切回dev分支)
git rebase temp

```

- 合并其他分支

```
git rebase master (基于开发分支合并主分支)

git add .
git rebase --continue
git reabase --skip
git rebase --abort (终止当前rebase操作)
```

## 使用 rebase 和 merge 的基本原则

- 下游分支更新上游分支内容的时候使用 rebase
- 上游分支合并下游分支内容的时候使用 merge
- 更新当前分支的内容时一定要使用 --rebase 参数

## 撤销 merge 操作

> 还没 add 时，取消这次合并

```
git merge --abort
```

> 已经 git add

```
git checkout 【行merge操作时所在的分支】
git reflog
git reset --hard 【merge前的版本号 id】
```

## 暂存命令 stash 使用

```
git stash #将本地修改暂时存储起来
git stash list #查看暂存的信息
git stash pop  #应用最近一次暂存的内容
git stash apply stash@{1} #应用指定版本的暂存内容
git stash clear  #清空暂存栈
```

## 回退到某个版本并应用指定的几次提交

```
git reset --hard 1d7444 #回退到某个版本
git cherry-pick 626335 #将某次commit的更改应用到当前版本
git cherry-pick …
git push origin HEAD --force  #强制提交
```
