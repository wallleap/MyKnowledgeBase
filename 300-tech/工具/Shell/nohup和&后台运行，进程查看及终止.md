---
title: nohup和&后台运行，进程查看及终止
date: 2022-11-22 09:29
updated: 2022-11-22 10:24
cover: //cdn.wallleap.cn/img/post/1.jpg
author: Luwang
comments: true
aliases:
  - nohup和&后台运行，进程查看及终止
rating: 10
tags:
  - Linux
category: web
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: //www.cnblogs.com/baby123/p/6477429.html
url: null
---

## 1. `nohup`

**用途**：不挂断地运行命令。

**语法**：

```sh
nohup Command \[ Arg … \] \[　& \]
```

- 无论是否将 `nohup` 命令的输出重定向到终端，输出都将附加到当前目录的 `nohup.out` 文件中。
- 如果当前目录的 `nohup.out` 文件不可写，输出重定向到 `$HOME/nohup.out` 文件中。
- 如果没有文件能创建或打开以用于追加，那么 `Command` 参数指定的命令不可调用。

**退出状态**：该命令返回下列出口值： 　　

- `126` 可以查找但不能调用 `Command` 参数指定的命令。 
- `127` `nohup` 命令发生错误或不能查找由 `Command` 参数指定的命令。
- 否则，`nohup` 命令的退出状态是 `Command` 参数指定命令的退出状态。

## 2.`&`

**用途**：在后台运行

一般两个一起用

```sh
nohup command &
```

**eg:**

```sh
nohup /usr/local/node/bin/node /www/im/chat.js >> /usr/local/node/output.log 2>&1 &
```

![](https://cdn.wallleap.cn/img/pic/illustrtion/202211220930459.png)

**进程号7585**

## 3. 查看运行的后台进程

### （1）`jobs -l`

![](https://cdn.wallleap.cn/img/pic/illustrtion/202211220930460.png)

`jobs` 命令只看当前终端生效的，关闭终端后，在另一个终端 `jobs` 已经无法看到后台跑得程序了，此时利用 `ps`（进程查看命令）

### （2）`ps -ef` 

```sh
ps -aux|grep chat.js
```

- `a`：显示所有程序
- `u`：以用户为主的格式来显示
- `x`：显示所有程序，不以终端机来区分

![](https://cdn.wallleap.cn/img/pic/illustrtion/202211220930461.png)

注：

- 用 `ps -def | grep` 查找进程很方便，最后一行总是会 `grep` 自己

- 用 `grep -v` 参数可以将 `grep` 命令排除掉

```sh
ps -aux|grep chat.js| grep -v grep
```

![](https://cdn.wallleap.cn/img/pic/illustrtion/202211220930462.png)

再用awk提取一下进程 ID　

```sh
ps -aux|grep chat.js| grep -v grep | awk '{print $2}'
```

![](https://cdn.wallleap.cn/img/pic/illustrtion/202211220930463.png)

### （3）`lsof -i:端口号`

如果某个进程起不来，可能是某个端口被占用

查看使用某端口的进程

```sh
lsof -i:8090
```

![](https://cdn.wallleap.cn/img/pic/illustrtion/202211220930464.png)

```sh
netstat -ap|grep 8090
```

![](https://cdn.wallleap.cn/img/pic/illustrtion/202211220930465.png)

查看到进程id之后，使用netstat命令查看其占用的端口

```sh
netstat -nap|grep 7779
```

![](https://cdn.wallleap.cn/img/pic/illustrtion/202211220930466.png)

使用 `kill` 杀掉进城后再启动

## 4. 终止后台运行的进程

```sh
kill -9 进程号
```

![](https://cdn.wallleap.cn/img/pic/illustrtion/202211220930467.png)
