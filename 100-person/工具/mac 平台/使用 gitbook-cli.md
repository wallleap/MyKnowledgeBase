---
title: 使用 gitbook-cli
date: 2024-10-28T03:32:04+08:00
updated: 2024-10-28T03:53:15+08:00
dg-publish: false
---

新版本官网没有这个了，但是直接下载又会报错，在 issue 中找到了方法

```sh
$ brew install nvm
$ nvm install 14.16.0
$ nvm use 14.16.0
$ npm install -g gitbook-cli@2.2.0
$ gitbook fetch 3.2.3
```

再接着就可以正常使用 gitbook 命令了

```sh
$ gitbook help
$ gitbook install
$ gitbook serve
$ gitbook build . ./dist/
```

不使用 `gitbook init`，直接在根目录新建两个文件

- `README.md` 主页
- `SUMMARY.md` 侧边栏

这个版本中也可以新建 `book.json` 配置文件

[简介 · GitBook详细教程 (jiangminggithub.github.io)](https://jiangminggithub.github.io/gitbook/)
