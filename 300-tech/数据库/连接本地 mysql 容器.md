---
title: 连接本地 mysql 容器
date: 2023-08-09T10:20:00+08:00
updated: 2024-08-21T10:32:34+08:00
dg-publish: false
aliases:
  - 连接本地 mysql 容器
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
source: #
tags:
  - MySQL
  - web
url: //myblog.wallleap.cn/post/1
---

本文已参与「新人创作礼」活动，一起开启掘金创作之路。

## 前言

使用的是 macOS 系统，但是 docker 命令都是一样的。本地想简单 `连接个mysql数据库`，但是本地没有安装 mysql，先不用远程服务器了，就用 docker 简单的起个 mysql 容器。已经安装 Docker Desktop，mac 比较简单，直接下载 dmg 安装即可。

## 一、启动镜像

已经安装好 docker，可以用 `docker info` 查看状态和信息，也可以查看菜单栏有小🐳的图标，即启动成功。

### 1、下载 mysql 镜像

```
docker pull mysql  // 直接下载了最新版本的
```

### 2、查看镜像

```
➜  ~ docker image ls 或者 docker images
REPOSITORY   TAG       IMAGE ID       CREATED      SIZE
mysql        latest    7b94cda7ffc7   6 days ago   446MB
```

### 3、启动 mysql 实例

```
docker run --name mysql -d -p 6666:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql:latest

// --name 为mysql的实例设置别名
// -d 以守护进程运行（后台运行）
// -p 6666为对外暴露的端口（随便选的），3306是内部端口
// -e MYSQL_ROOT_PASSWORD 设置mysql登录密码
// 最后的mysql是启动的镜像名称
```

### 4、查看启动容器

```
➜  ~ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                               NAMES
0d265e254add   mysql:latest   "docker-entrypoint.s…"   7 minutes ago   Up 7 minutes   33060/tcp, 0.0.0.0:6666->3306/tcp   mysql

docker ps -a  // 查看所有容器（运行中、未运行）
docker inspect mysql // 查看容器信息
```

### 5、查看端口

如果忘记容器映射端口，可以通过 `docker port 容器名` 查看，

```
➜  ~ docker port mysql
3306/tcp -> 0.0.0.0:6666
```

或者连接数据库端口报错时，检验端口是否开放，容器外命令行执行 `nc -vz -w 2 127.0.0.1 6666`

```
➜  ~ nc -vz -w 2 127.0.0.1 6666
Connection to 127.0.0.1 port 6666 [tcp/*] succeeded!

```

### 6、连接 mysql 数据库

启动成功后可以通过 localhost:6666（暴露的端口），root 和 123456(密码) 连接 mysql 数据库了，工具可以使用 navicat 或者 DBeaver 都行。

这里使用的 DBeaver，填入容器对应的端口等信息，可以正常连接成功。

![](https://cdn.wallleap.cn/img/pic/illustration/202308091021100.png)

执行准备好的 sql 文件，成功导入数据。

![](https://cdn.wallleap.cn/img/pic/illustration/202308091021101.png)

### 7、停止启动

通过 `容器ID或容器名` 停止启动容器，数据不会丢失。不过如果删除后数据丢失了，可以挂载文件。

```
docker stop mysql  // 停止mysql（容器名）容器
docker start mysql  // 启动被停止的mysql（容器名）容器
```

## 二、操作容器

### 1、进入容器内部

```
docker container exec -it mysql /bin/bash
// mysql是上面起的容器别名
```

### 2、登录 mysql

```js
bash-4.4# mysql -u root -p
Enter password:
Welcome to the MySQL monitor.
```

### 3、mysql 相关操作

```js
show databases;  // 查看数据库
create database test;  // 创建数据库
```
