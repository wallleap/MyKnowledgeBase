---
title: Docker 安装
date: 2024-08-26T09:50:21+08:00
updated: 2024-08-26T09:54:36+08:00
dg-publish: false
source: https://www.sqlsec.com/2019/10/docker2.html
---

Docker 目前用的越来越多，前段时间接到一个项目：使用 Docker 部署 AWD 靶场，实际操作起来还是有很多细节的，这里特地总结一下，日后自己查起来也比较方便。

## Docker 安装

这里我只写 Ubuntu、Kali/Debian、CentOS 下的安装，macOS 和 Windows 平台的安装也太简单了吧。

### Ubuntu/CentOS 安装 Docker

Ubuntu Linux 系统耳熟能详的操作系统。

```bash
curl -fsSL get.docker.com -o get-docker.sh
sudo sh get-docker.sh --mirror Aliyun
```

### Kali/Debian 安装 Docker

Kali 是基于 Debian 封装的，所以两者安装方法大同小异。
Kali Linux 安装 Docker 用网上那个一键安装脚本貌似有点问题，这里单独记录一下，以便自己和其他网友使用：

```bash
# 添加Docker PGP密钥
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -

# 配置docker apt源 我这里用的国内阿里云的docker下载源
echo 'deb https://mirrors.aliyun.com/docker-ce/linux/debian buster stable'> /etc/apt/sources.list.d/docker.list

# 更新apt源
apt update

# 如果之前安装了docker的话 这里得卸载旧版本docker
apt remove docker docker-engine docker.io

# 安装docker
apt install docker-ce

# 查看版本
docker version
```

### 开机自启

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

## Docker 优化

### Docker 国内加速器

不替换源对话，docker pull 拉去镜像对速度实在太龟速了，如果你很佛系对话可以不进行更换

```bash
# 编辑这个文件，如果没有对话就创建这个文件
vim /etc/docker/daemon.json
```

内容如下：

```json
{
  "registry-mirrors": [
    "http://hub-mirror.c.163.com"
  ]
}
```

这里我使用对是国内 163 网易源，其他源可以自行百度替换。
配置完成后重启服务才可以生效：

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### Docker Portainer

Portainer 是 Docker 一款可视化管理用具，用起来更加容易上手，部署的话也十分简单。

```bash
# 拉取镜像
docker pull portainer/portainer

# 一键部署
docker volume create portainer_data
docker run -d -p 9000:9000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
```

浏览器访问本地的 `9000` 端口即可进入到设置密码界面，设置一个密码，即可进入系统：

![](https://image.3001.net/images/20201011/16023973804305.png)

这里选择管理本地 Docker 的选项，点击就可以看到比较容易理解的 Docker 管理即界面了。

### DockStation

官网地址：[DockStation - Developing with Docker has never been so easy and convenient](https://dockstation.io/)

一个 Docker 的图形化客户端管理工具，支持 Windows、Linux、macOS，因为现在 Windows 和 macOS 自带的 docker desktop 功能越来越强，所以这个 DockeStation 在 Linux 上还是比较方便的，感兴趣的同学可以体验一下：

![](https://image.3001.net/images/20201011/16023972532611.png)

## Docker 基础命令

### 搜索镜像

```bash
docker search 关键词
```

### 下载镜像

```bash
docker pull 镜像名
```

### 查看已下载的镜像列表

```bash
docker image ls
```

### 创建并使用容器

```bash
docker run -it --name 容器名 镜像名/镜像ID /bin/bash
```

### 查看当前容器

```bash
docker ps -a
```

### 统计信息

```bash
docker stats
```

### 启动容器

```bash
docker start 容器名/容器ID
```

### 重启容器

```bash
docker restart 容器名/容器ID
```

### 终止容器

```bash
docker stop 容器名/容器ID
```

### 终止所有容器

```bah
docker stop $(docker ps -aq)
```

### 连接容器

```bash
docker exec -it 容器名/容器ID /bin/bash
```

### 删除容器

```bash
docker rm 容器名/容器ID
```

### 删除所有容器

```bash
docker rm $(docker ps -aq)
```

### 删除镜像

```bash
docker rmi 镜像名/容器ID
```

### 删除所有镜像

```bash
docker rmi $(docker images -q)
```

### 端口映射

部署一个容器，并将 80 端口映射到宿主机的 8000 端口上

```bash
# 可以使用--name自定义部署的容器名
docker run -d -p 8000:80 --name 容器名 镜像名

# 也可以直接通过镜像部署
docker run -d -p 8000:80 镜像名
```

### dockerfile 部署镜像

```bash
docker build -t 自定义镜像名称 .
```

### docker-compose 部署

```bash
docker-compose up -d
```

### 构建新的镜像

```bash
docker commit -a "提交的镜像作者" -m "提交时的说明文字" 容器的ID 要创建的新的镜像
docker commit -a "国光" -m "wordpress_phpmyadmin" d64655e87ccc wordpress_phpmyadmin:v1
```

### 保存离线镜像

```bash
docker save -o 镜像文件名.tar 要保持的镜像
docker save -o wordpress_phpmyadmin.tar wordpress_phpmyadmin:latest
```

### 导入离线镜像

```bash
docker load --input 镜像文件名.tar
docker load --input wordpress_phpmyadmin.tar
```

### 挂载卷

看下面的这个案例理解一下就明白了：

```bash
docker run -d -p 9088:80 --name wordpress_phpmyadmin -v "`pwd`/mysql":/var/lib/mysql/ -v "`pwd`/app":/app/ wordpress_phpmyadmin:latest
```

## Docker compose

docker compose 神器，国内的 vulhubs 靶场就是用的 docker compose 规范，所以这里有必要安装一下。
首先来查看最新版本 [https://github.com/docker/compose/releases](https://github.com/docker/compose/releases)

我写这篇文章的时候目前是 1.25.0-rc2 版本，具体根据新版本的变化自行调整下面命令来安装：

```bash
# 下载docker-compose
curl -L https://github.com/docker/compose/releases/download/1.25.0-rc2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

# 给docker-compose执行权限
chmod +x /usr/local/bin/docker-compose

# 查看docker compose版本
root@kali-linux:~# docker-compose  version
docker-compose version 1.25.0-rc2, build 661ac20e
docker-py version: 4.0.1
CPython version: 3.7.4
OpenSSL version: OpenSSL 1.1.0k  28 May 2019
```

## 点评

实际上 Docker 这玩意还是实操比较好，后面去安恒的网络空间学院工作，经常和 Docker 打交道，那个时候也是我 Docker 玩的最 6 的时候，233333

本文可能实际上也没有啥技术含量，但是写起来还是比较浪费时间的，在这个喧嚣浮躁的时代，个人博客越来越没有人看了，写博客感觉一直是用爱发电的状态。如果你恰巧财力雄厚，感觉本文对你有所帮助的话，可以考虑打赏一下本文，用以维持高昂的服务器运营费用（域名费用、服务器费用、CDN 费用等）
