---
title: volta 管理 JS  工具版本
date: 2024-10-08T04:39:31+08:00
updated: 2024-10-08T04:48:02+08:00
dg-publish: false
---

[Volta - The Hassle-Free JavaScript Tool Manager](https://volta.sh/)

可以管理更多工具的版本，会比 nvm 更好用

## 安装

使用 curl 或 brew 安装

```sh
$ curl https://get.volta.sh | bash
$ brew install volta
```

安装完成后查看版本号

```sh
$ volta -v
```

查看帮助

```sh
$ volta -h
Volta 1.1.1
The JavaScript Launcher ⚡

    To install a tool in your toolchain, use `volta install`.
    To pin your project's runtime or package manager, use `volta pin`.

USAGE:
    volta [FLAGS] [SUBCOMMAND]

FLAGS:
        --verbose    Enables verbose diagnostics
        --quiet      Prevents unnecessary output
    -v, --version    Prints the current version of Volta
    -h, --help       Prints help information

SUBCOMMANDS:
    fetch          Fetches a tool to the local machine
    install        Installs a tool in your toolchain
    uninstall      Uninstalls a tool from your toolchain
    pin            Pins your project's runtime or package manager
    list           Displays the current toolchain
    completions    Generates Volta completions
    which          Locates the actual binary that will be called by Volta
    setup          Enables Volta for the current user / shell
    run            Run a command with custom Node, npm, pnpm, and/or Yarn versions
    help           Prints this message or the help of the given subcommand(s)
```

## 使用 volta 管理 node

安装最新 LTS 版

```sh
$ volta install node
```

安装指定版本

```sh
$ volta install node@22.5.1
```

查看已经安装的 node 版本

```sh
$ volta list node
```

在项目中 pin 一个 node 版本

```sh
$ volta pin node@22.5.1
```

这样它就会在 `package.json` 中新加上相关版本的配置，之后进入这个项目并且安装过 volta，执行 `node -v` 就会自动安装并切换到这个版本
