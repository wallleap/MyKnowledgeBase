---
title: macOS 安装 NVM
date: 2023-01-29T02:20:00+08:00
updated: 2024-09-24T06:33:22+08:00
dg-publish: false
aliases:
  - macOS 安装 NVM
author: Luwang
category: web
comments: true
cover: //cdn.wallleap.cn/img/post/1.jpg
description: 文章描述
image-auto-upload: true
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
rating: 1
source: null
tags:
  - 工具
url: //myblog.wallleap.cn/post/1
---

NVM（Node Version Manager）是一个用于在基于 Linux 系统上安装和管理 Node.js 的 shell 脚本。macOS 用户可以使用 `homebrew` 来安装 NVM。 本教程帮助你在 macOS 系统上安装 NVM 并管理 Nodej.is 版本。

[Volta - The Hassle-Free JavaScript Tool Manager](https://volta.sh/) 也可以用于管理 Node 版本

> 前提条件 在 macOS 上使用安装 `homebrew`

```sh
/bin/bash -c "$(curl -fsSL https:/raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

[具体可参考官网](https://link.juejin.cn/?target=https%3A%2F%2Fbrew.sh%2F "https://brew.sh/")

## 第一步: 删除现有 Node 版本

如果你的系统已经安装了 node，请先卸载它。我的系统已经通过 Homebrew 安装了 node。所以先把它卸载了。如果还没有安装就跳过。

```sh
brew uninstall --ignore-dependencies node 
brew uninstall --force node 
```

## 第二步: 在 Mac 上安装 NVM

现在，你的系统已经准备好了，可以进行安装。更新 Homebrew 软件包列表并安装 NVM。

```sh
brew update 
brew install nvm
```

接下来，在 home 目录中为 NVM 创建一个文件夹。

```
mkdir ~/.nvm 
复制代码
```

现在，配置所需的环境变量。在你的 home 中编辑以下配置文件

```sh
vim ~/.bash_profile 
```

然后，在 `~/.bash_profile`（或 `~/.zshrc`，用于 macOS Catalina 或更高版本）中添加以下几行

```sh
export NVM_DIR=~/.nvm
source $(brew --prefix nvm)/nvm.sh
```

按 ESC + `:wq` 保存并关闭你的文件。 接下来，将该变量加载到当前的 shell 环境中。在下一次登录，它将自动加载。

```sh
source ~/.bash_profile
```

NVM 已经安装在你的 macOS 系统上。 下一步，在 nvm 的帮助下安装你需要的 Node.js 版本即可。

## 第三步 : 用 NVM 安装 Node.js

首先，看看有哪些 Node 版本可以安装。要查看可用的版本，请输入。

```sh
nvm ls-remote 
```

现在，你可以安装上述输出中列出的任何版本。你也可以使用别名，如 node 代表最新版本，lts 代表最新的 LTS 版本，等等。

```sh
nvm install node     ## 安装最后一个长期支持版本
nvm install 10
```

安装后，你可以用以下方法来验证所安装的 node.js 是否安装成功。

```sh
nvm ls 
```

![Snipaste_2022-04-05_15-55-04.png](https://cdn.wallleap.cn/img/pic/illustration/202301291423914.webp)

如果你在系统上安装了多个版本，你可以在任何时候将任何版本设置为默认版本。要设置节点 10 为默认版本，只需使用。

```sh
nvm use 10
```

同样地，你可以安装其他版本，如 Node 其他版本，并在它们之间进行切换。

之后每个项目在新建的时候就应该新建文件 `.nvmrc`，写入使用的 node 版本

```conf
v12.22.12
```

进入项目之后直接 `nvm use` 就能使用这个版本的 node

## 结束

> 本教程向你解释了如何在 macOS 系统上安装 NVM 和 node.js。
