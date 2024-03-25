---
title: 搭建 Git 服务器
date: 2024-03-22 00:25
updated: 2024-03-25 10:20
---

[Git - Git Hooks (git-scm.com)](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)

[CentOS 部署 Git 服务器 | 沈煜的博客 (shenyu.me)](https://shenyu.me/2017/01/03/git-server.html)

[搭建Git服务器 - 廖雪峰的官方网站 (liaoxuefeng.com)](https://www.liaoxuefeng.com/wiki/896043488029600/899998870925664)

[最简单的 Git 服务器 - 阮一峰的网络日志 (ruanyifeng.com)](https://www.ruanyifeng.com/blog/2022/10/git-server.html)

## 一、代码托管服务

一般情况下，都不建议自己搭建 Git 服务器，而要使用现成的服务，也就是代码托管服务。它们都是免费的。

> - [GitHub](https://github.com/)
> - [Gitlab](https://about.gitlab.com/)
> - [Bitbucket](https://bitbucket.org/)
> - [Codeberg](https://codeberg.org/)
> - [sourcehut](https://sr.ht/)
> - [Gitee](https://gitee.com/)

其中，除了最后一家 Gitee 是国内的服务，其他都是国外的服务。

这些外部服务，就不多做介绍了。本文的重点不是它们，而是想谈如果不得不自己搭建 Git 服务器，那该怎么做。

使用有 sudo 权限的帐号登录登录到服务器

## 二、Git 服务器软件

自己搭建 Git 服务器的原因，无非就是不方便访问外网，不愿意代码放在别人的服务器，或者有一些定制化的需求。

这时，你可以选择开源的 Git 服务器软件。

> - [Gitlab CE](https://about.gitlab.com/install/ce-or-ee/)
> - [Gitea](https://gitea.io/zh-cn/)
> - [Gogs](https://gogs.io/)
> - [Onedev](https://github.com/theonedev/onedev)

这些软件里面，Gogs 的安装是最简单的，但是功能相对比较弱。功能越强的软件，安装越复杂。

如果你只是想远程保存一份代码，并不在意有没有 Web 界面，或者其他功能，那么根本不用安装上面这些软件，一行命令就够了。

## 三、Git 仓库的 SSH 传输

熟悉 Git 的同学可能知道，Git 默认支持两种传输协议：SSH 和 HTTP/HTTPS。

服务器一般都自带 SSH，这意味着，**我们可以什么都不安装，只通过 SSH 就把仓库推到远程服务器。**

所以，一条命令就够了。我们只要在远程服务器上，建立同名的 Git 仓库，服务器就搭建好了。

第一步，安装 `git`：

```sh
apt-get install git # 或其他的安装工具 yum brew 等
```

第二步，创建一个 `git` 用户，用来运行 `git` 服务：

```sh
adduser git
```

设置密码

```sh
passwd git
# 输入密码，例如 abc.123
```

第三步，创建证书登录：

收集所有需要登录的用户的公钥，就是他们自己的 `id_rsa.pub` 文件，把所有公钥导入到 `/home/git/.ssh/authorized_keys` 文件里，一行一个。

> 本地运行 `ssh-keygen -t rsa -b 4096 -C "your_email@example.com"` 生成密钥，`cat ~/.ssh/id_rsa.pub` 查看密钥

第四步，初始化 Git 仓库：

先选定一个目录作为 Git 仓库，假定是 `/srv/sample.git`，在 `/srv` 目录下输入命令：

```sh
git init --bare sample.git
```

Git 就会创建一个裸仓库，裸仓库没有工作区，因为服务器上的 Git 仓库纯粹是为了共享，所以不让用户直接登录到服务器上去改工作区，并且服务器上的 Git 仓库通常都以 `.git` 结尾。然后，把 owner 改为 `git`：

```sh
chown -R git:git sample.git
```

也可以本地创建后直接上传到服务器

```bash
ssh git@192.168.1.25 git init --bare example.git
scp -r example/.git git@192.168.1.25:/home/git/example.git
```

第五步，禁用 shell 登录：

出于安全考虑，第二步创建的 git 用户不允许登录 shell，这可以通过编辑 `/etc/passwd` 文件完成。找到类似下面的一行：

```sh
git:x:1001:1001:,,,:/home/git:/bin/bash
```

改为：

```sh
git:x:1001:1001:,,,:/home/git:/usr/bin/git-shell
```

这样，`git` 用户可以正常通过 ssh 使用 git，但无法登录 shell，因为我们为 `git` 用户指定的 `git-shell` 每次一登录就自动退出。

第六步，克隆远程仓库：

为了后面方便连接，可以在 host 中添加一条映射（服务器 IP 映射到 server）

```
server 100.xxx.xx.xxx
```

现在，可以通过 `git clone` 命令克隆远程仓库了，在各自的电脑上运行：

```sh
git clone git@server:/srv/sample.git
```

可能会显示

```sh
Cloning into 'sample'...
warning: You appear to have cloned an empty repository.
```

剩下的推送就简单了。

## 管理公钥

如果团队很小，把每个人的公钥收集起来放到服务器的 `/home/git/.ssh/authorized_keys` 文件里就是可行的。如果团队有几百号人，就没法这么玩了，这时，可以用 [Gitosis](https://github.com/res0nat0r/gitosis) 来管理公钥。

这里我们不介绍怎么玩 [Gitosis](https://github.com/res0nat0r/gitosis) 了，几百号人的团队基本都在 500 强了，相信找个高水平的 Linux 管理员问题不大。

## 管理权限

有很多不但视源代码如生命，而且视员工为窃贼的公司，会在版本控制系统里设置一套完善的权限控制，每个人是否有读写权限会精确到每个分支甚至每个目录下。因为 Git 是为 Linux 源代码托管而开发的，所以 Git 也继承了开源社区的精神，不支持权限控制。不过，因为 Git 支持钩子（hook），所以，可以在服务器端编写一系列脚本来控制提交等操作，达到权限控制的目的。[Gitolite](https://github.com/sitaramc/gitolite) 就是这个工具。

这里我们也不介绍 [Gitolite](https://github.com/sitaramc/gitolite) 了，不要把有限的生命浪费到权限斗争中。

## Hook

我们将会使用 post-receive 钩子，更多钩子及含义可以参考 [git文档](http://git-scm.com/book/en/Customizing-Git-Git-Hooks) 在 your_site.git 文件夹中

```sh
$ cd hooks
```

如果继续输入 `ls` 可以看到目录下的钩子。

创建一个钩子

```sh
$ cat > post-receive
#!/bin/sh

git --work-tree=生产环境网站文件夹位置 --git-dir=/var/git/your_site.git checkout -f
```

其中生产环境网站文件夹要自行创建好，否则可能不会正常部署。

输入完成后按 `ctrl + D` 保存，`git-dir` 指的是仓库的地址， `work-tree` 则是存放代码的位置，也就是我们的网站的源代码的位置。 接下来则是要保证它可以运行：

```sh
chmod +x post-receive
```

本地端

一般情况下你应该有自己的项目了，给你的本地添加你的服务器的仓库即可。

```sh
git remote add myVPS-sitename ssh://user@mydomain.com/var/git/your_site.git
```

然后按照正常 push 的时候加上后面参数

```sh
git push myVPS-sitename master
```

如果你是密码登陆服务器输入密码即可，ssh 登陆服务器就可以直接传上去了。

说到这里，有时候输入太过于繁琐，也许有时候 ssh 登陆证书并不是我们想用了，这时候我们可以设置别名来登陆。

`ls -al ~/.ssh`   查看 ssh 密钥。默认情况下 Git push 的时候回采用  id_rsa

但是我们可能有专门的证书来做特定的 ssh 操作。

这时候可以利用到别名。

```sh
vim ~/.ssh/config
```

输入

```
Host alias
　　HostName www.ttlsa.com
　　Port 22
　　User root
　　IdentityFile ~/.ssh/id_rsa.pub
　　IdentitiesOnly yes
```

选项注释： 

- HostName 指定登录的主机名或 IP 地址
- Port 指定登录的端口号
- User 登录用户名
- IdentityFile 登录的公钥文件
- IdentitiesOnly 只接受 SSH key 登录
- PubkeyAuthentication

保存

然后 `ssh alias` 就可以直接登陆 ssh 了。

这时候我们可以先删除连接 `git  remote remove myVPS-sitename`

然后重新添加   `git remote add myVPS-sitename ssh://alias/var/git/your_site.git`

这时候就可以很方便的上传了。

参考资料：<https://blog.csdn.net/haozi3156666/article/details/40982085>

<https://www.cnblogs.com/sparkdev/p/6842805.html>

<https://blog.csdn.net/zhanlanmg/article/details/48708255>

[blog/git/搭建一个git服务器端仓库并且实现自动部署.md at master · hewq/blog (github.com)](https://github.com/hewq/blog/blob/master/git/%E6%90%AD%E5%BB%BA%E4%B8%80%E4%B8%AAgit%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%AB%AF%E4%BB%93%E5%BA%93%E5%B9%B6%E4%B8%94%E5%AE%9E%E7%8E%B0%E8%87%AA%E5%8A%A8%E9%83%A8%E7%BD%B2.md)
