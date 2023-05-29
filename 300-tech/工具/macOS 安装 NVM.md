---
title: macOS 安装 NVM
date: 2023-01-29 14:20
updated: 2023-01-29 14:24
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - macOS 安装 NVM
rating: 1
tags:
  - 工具
category: web
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: null
url: //myblog.wallleap.cn/post/1
---

NVM（Node Version Manager）是一个用于在基于Linux系统上安装和管理Node.js的shell脚本。macOS用户可以使用`homebrew`来安装NVM。 本教程帮助你在macOS系统上安装NVM并管理Nodej.is版本。

> 前提条件 在macOS上使用安装`homebrew`

```sh
/bin/bash -c "$(curl -fsSL https:/raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

[具体可参考官网](https://link.juejin.cn/?target=https%3A%2F%2Fbrew.sh%2F "https://brew.sh/")

#### 第一步:删除现有Node版本

如果你的系统已经安装了node，请先卸载它。我的系统已经通过Homebrew安装了node。所以先把它卸载了。如果还没有安装就跳过。

```sh
brew uninstall --ignore-dependencies node 
brew uninstall --force node 
```

#### 第二步:在Mac上安装NVM

现在，你的系统已经准备好了，可以进行安装。更新Homebrew软件包列表并安装NVM。

```sh
brew update 
brew install nvm
```

接下来，在home目录中为NVM创建一个文件夹。

```
mkdir ~/.nvm 
复制代码
```

现在，配置所需的环境变量。在你的home中编辑以下配置文件

```sh
vim ~/.bash_profile 
```

然后，在 `~/.bash_profile`（或`~/.zshrc`，用于macOS Catalina或更高版本）中添加以下几行

```sh
export NVM_DIR=~/.nvm
source $(brew --prefix nvm)/nvm.sh
```

按ESC + `:wq` 保存并关闭你的文件。 接下来，将该变量加载到当前的shell环境中。在下一次登录，它将自动加载。

```sh
source ~/.bash_profile
```

NVM已经安装在你的macOS系统上。 下一步，在nvm的帮助下安装你需要的Node.js版本即可。

#### 第三步 : 用NVM安装Node.js

首先，看看有哪些Node版本可以安装。要查看可用的版本，请输入。

```sh
nvm ls-remote 
```

现在，你可以安装上述输出中列出的任何版本。你也可以使用别名，如node代表最新版本，lts代表最新的LTS版本，等等。

```sh
nvm install node     ## 安装最后一个长期支持版本
nvm install 10
```

安装后，你可以用以下方法来验证所安装的node.js是否安装成功。

```sh
nvm ls 
```

![Snipaste_2022-04-05_15-55-04.png](https://cdn.wallleap.cn/img/pic/illustration/202301291423914.webp)

如果你在系统上安装了多个版本，你可以在任何时候将任何版本设置为默认版本。要设置节点10为默认版本，只需使用。

```sh
nvm use 10
```

同样地，你可以安装其他版本，如Node其他版本，并在它们之间进行切换。

#### 结束

> 本教程向你解释了如何在macOS系统上安装NVM和node.js。
