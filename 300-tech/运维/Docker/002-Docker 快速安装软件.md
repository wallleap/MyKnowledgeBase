---
title: 002-Docker 快速安装软件
date: 2024-10-27T02:40:24+08:00
updated: 2024-10-27T02:50:54+08:00
dg-publish: false
---

## 直接安装的缺点

- 安装麻烦，可能有各种依赖，运行报错。例如：WordPress，ElasticSearch，Redis，ELK
- 可能对 Windows 并不友好，运行有各种兼容问题，软件只支持 Linux 上跑
- 不方便安装多版本软件，不能共存。
- 电脑安装了一堆软件，拖慢电脑速度。
- 不同系统和硬件，安装方式不一样

> 本文档课件配套 [视频教程](https://www.bilibili.com/video/BV11L411g7U1?p=2)

## Docker 安装的优点

- 一个命令就可以安装好，快速方便
- 有大量的镜像，可直接使用
- 没有系统兼容问题，Linux 专享软件也照样跑
- 支持软件多版本共存
- 用完就丢，不拖慢电脑速度
- 不同系统和硬件，只要安装好 Docker 其他都一样了，一个命令搞定所有

## 演示 Docker 安装 Redis

Redis 官网：<https://redis.io/>

> 官网下载安装教程只有源码安装方式，没有 Windows 版本。想要自己安装 windows 版本需要去找别人编译好的安装包。

Docker 官方镜像仓库查找 Redis ：<https://hub.docker.com/>

或者直接使用命令搜索 `docker search redis`

![Docker镜像官网](https://cos.easydoc.net/46901064/files/kv8zs4qr.png)

一个命令跑起来：`docker run -d -p 6379:6379 --name redis redis:latest`

命令参考：<https://docs.docker.com/engine/reference/commandline/run/>

`docker run 容器名` 是运行一个新的容器（会先拉取镜像）

- `-d` 后台运行
- `-e` 设置环境变量
- `--name` 命名
- `-p` 设置端口映射
- `-v` 挂载

![Docker运行Redis后](https://cos.easydoc.net/46901064/files/kv8zy4xn.png)

## 安装 Wordpress

docker-compose.yml

```
version: '3.1'

services:

  wordpress:
    image: wordpress
    restart: always
    ports:
      - 8080:80
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: exampleuser
      WORDPRESS_DB_PASSWORD: examplepass
      WORDPRESS_DB_NAME: exampledb
    volumes:
      - wordpress:/var/www/html

  db:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_DATABASE: exampledb
      MYSQL_USER: exampleuser
      MYSQL_PASSWORD: examplepass
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
    volumes:
      - db:/var/lib/mysql

volumes:
  wordpress:
  db:
```

## 安装 ELK

```
docker run -p 5601:5601 -p 9200:9200 -p 5044:5044 -it --name elk sebp/elk
```

[内存不够解决方法](https://docs.microsoft.com/en-us/windows/wsl/wsl-config#global-configuration-options-with-wslconfig)

转到用户目录 `cd ~`，路径类似这个：`C:\Users\<UserName>`

创建 `.wslconfig` 文件填入以下内容

```
[wsl2]
memory=10GB # Limits VM memory in WSL 2 to 4 GB
processors=2 # Makes the WSL 2 VM use two virtual processors
```

生效配置，命令行运行 `wsl --shutdown`

## 更多相关命令

`docker ps` 查看当前运行中的容器

- `-a` 查看所有的

`docker images` 查看镜像列表

`docker rm container-id` 删除指定 id 的容器

`docker stop/start container-id` 停止/启动指定 id 的容器

`docker rmi image-id` 删除指定 id 的镜像

`docker volume ls` 查看 volume 列表

`docker network ls` 查看网络列表

`docker exec -it my_container sh` 以 sh 打开容器终端
