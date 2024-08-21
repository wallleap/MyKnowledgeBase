---
title: 修改 Hosts 访问外网
date: 2023-01-29T02:34:00+08:00
updated: 2024-08-21T11:08:54+08:00
dg-publish: false
aliases:
  - 修改 Hosts 访问外网
author: Luwang
category: network
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
  - DNS
url: //myblog.wallleap.cn/post/1
---

把要访问的地址用 [DNS站长查询工具](http://tool.chinaz.com/dns) 查询出来，选一个 ttl 大的用

![](https://cdn.wallleap.cn/img/pic/illustration/202301291437722.png)

## windows 修改

在 `C:\Windows\System32\drivers\etc` 目录下找到 hosts 文件，使用记事本或其他文件编辑器打开，将包含众多 ip 的地址添加到原来 hosts 文件内容的后面即可。（这里需要管理员权限，可以先以管理员身份打开文件编辑软件，如在开始屏幕搜索 “notepad”，右键选择 “以管理员身份运行” 记事本程序，按下 Ctrl O 定位到 hosts 文件修改并直接保存。）

之后刷新 DNS 缓存：按下 Windows R 键，运行 cmd ，在命令提示符运行命令

```text
ipconfig /flushdns
```

## Linux(ubuntu)

切换到 root 用户，在终端输入

```text
sudo gedit /etc/hosts
```

将打开的 hosts 修改后保存。

或者将 `/etc/hosts` 复制到桌面上，然后手动编辑后保存，再使用命令 copy 覆盖过去即可

```text
sudo cp hosts /etc/
```

接着输入运行命令

```text
/etc/rc.d/init.d/nscd restart
```

刷新 DNS 缓存。

**附上 hosts 资源链接**

<https://github.com/lennylxx/ipv6%2Dhosts>
