---
title: Alist 小雅搭建
date: 2025-02-06T13:21:14+08:00
updated: 2025-02-17T10:14:30+08:00
dg-publish: false
---

<https://xiaoyaliu.notion.site/xiaoya-docker-69404af849504fa5bcf9f2dd5ecaa75f>

可以直接使用：<https://alist.xiaoya.pro/>

但可能会卡顿，所以推荐自建

## 准备工作

### 获取阿里云盘的 Token

<https://alist.nn.ci/zh/guide/drivers/aliyundrive.html> 或 <https://aliyuntoken.vercel.app/>

点击上面任意一个链接，按照提示扫码获取 Token，记录为 `mytoken`

### 获取阿里云盘的 Open Token

<https://alist.nn.ci/tool/aliyundrive/request.html>

点击上方链接扫码获取，记录下来 `myopentoken`

### 转存文件夹

<https://www.aliyundrive.com/s/rP9gP3h9asE>

点击上方链接并且转存所有文件，之后不要删除这个目录（或者自己创建一个文件夹，名字为 `小雅`）

在浏览器中打开这个目录，记录下链接最后的路径 `temp_transfer_folder_id`

## 使用 Docker 安装

一键安装和更新

```sh
bash -c "$(curl http://docker.xiaoya.pro/update_new.sh)"
```

注意事项：

由于 Docker Hub 的原因，可能需要开**特殊网络**才能完成安装

有可能没有权限，可以修改下目录权限，例如

```sh
sudo chown $USER /etc/xiaoya
sudo chmod -R 755 /etc/xiaoya
```

**重启就会自动更新数据库及搜索索引文件**

```sh
docker restart xiaoya
```

## 设置需要登录

webdav 默认账号密码

用户: guest 密码: guest_Api789

添加 `/etc/xiaoya/guestlogin.txt` 文件，文件里可以没有内容

添加 `/etc/xiaoya/guestpass.txt`，里面存放的就是密码，用户名为 `dav`（设置之后 webdav 的帐号密码也是这个）

> 新增文件后需要重启容器

## data 下其他文件

一键安装脚本默认 `/data` 映射在 `/etc/xiaoya`

之前自动创建了 `mytoken.txt`、`myopentoken.txt`、`temp_transfer_folder_id.txt`

`pikpak.txt` 查看 Pikpak 分享

```txt
"+86xxxx" "password"
```

`show_my_ali.txt` 存在的话显示自己的阿里云盘文件

`docker_address.txt` 配合 TVBOX 的 alist 搜索

`docker_address_ext.txt` 外网地址
