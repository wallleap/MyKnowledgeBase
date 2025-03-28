---
title: AI 代码编辑器 Cursor
date: 2024-12-18T15:35:35+08:00
updated: 2025-01-09T17:32:10+08:00
dg-publish: false
---

## 价格

刚注册的账号能免费体验 Cursor Pro 10 天，你可以无限使用 AI。

10 天之后你的 AI 请求量就会受到限制，此时你有三个选择：

1. 花钱升级到 Cursor Pro，每月 20 美元
2. 网传可以换邮箱重新体验 10 天，如此反复
3. 打开 Cursor Settings / Models，设置自己的 API Key <https://api.v3.cm/register?aff=yH7I>

## 快捷键

你只需要知道 Cursor 的两个快捷键：Ctrl + K 和 Ctrl + L。

macOS 用户请把 Ctrl 自行替换成 Command。

Ctrl + K 用于生成代码，按下之后说出你的需求，比如“把数组 a 去重得到 b”，然后回车，就能看到代码自动生成了。

如果你对结果不满意，就继续打字，比如“使用 lodash 提供的 API”，然后代码就会被重写。

直到你对结果满意了，你就可以按下 Ctrl + Enter 来接受答案。

Ctrl + L 用于跟 AI 聊天，你可以问“这个文件做了什么”，然后回车。如果你想知道整个代码仓库做了什么，就把回车改成 Ctrl + 回车。

## 一句命令白嫖 Cursor VIP

### linux 和 mac 运行

```sh
$ bash <(curl -Lk https://ghp.ci/https://github.com/kingparks/cursor-vip/releases/download/latest/install.sh) 2402e433d97cfac826061b120973bdd0
```

### windows 使用 powershell 运行

第一句：

```sh
$ Set-ExecutionPolicy Bypass -Scope Process -Force
```

第二句：

```sh
Invoke-Expression (Invoke-WebRequest -Uri "https://ghp.ci/https://github.com/kingparks/cursor-vip/releases/download/latest/install.sh" -UseBasicParsing).Content; ./install.sh 9a05909bf9238aed3dbfd61c36cdbb71
```

1. 运行后要保持后台进程常开，其原理是网络代理。Windows 用户双击 exe 之后不要关闭黑色窗口。
2. 源码在 github，我看了，核心代码 auth 没有开源，是否信任请**自行考量**，已公开的代码不存在可疑代码，要求 root 权限是为了放置二进制文件到系统目录（介意者可以手动删除脚本并自行修改 PATH）
3. 代码里的最后一串字符串是推广码，可以改成你自己的，推广越多免费时间越长

其他项目：<https://github.com/chengazhen/cursor-auto-free>
