---
title: 宝塔面板 Docker 部署 Bitwarden 密码管理器
date: 2023-08-09 20:34
updated: 2023-08-09 20:47
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - 宝塔面板 Docker 部署 Bitwarden 密码管理器
rating: 1
tags:
  - server
  - 密码
category: server
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: #
url: //myblog.wallleap.cn/post/1
---

## Bitwarden 简介

现每当注册一个网站或 APP 都会要求设置密码与登入时输入密码，无数网站和 APP 大量的账号和密码；如都设统一密码，就存在一旦一个密码被攻破，被撞库时由于使用相同密码而全军覆；如果为不同网站设置为不同密码，那么记住密码又成了一个困难的问题；所以密码管理工具也就应运而生，比较有名的密码管理工具有 1Password，OneSafe，LastPass 等，但是这些工具大多数需要收费并且密码存储在其提供的服务器上，密码不是掌管在自己手里，还是有些不放心，而开源的 Bitwarden 的出现让其成为不二之选

Bitwarden 开源可部署至自己的服务器，浏览器插件平台支持方面，**主流的如 Chrome、Edge 这些 Chromium 内核的浏览器可以使用插件，Mac、Windows、Linux 也有相应的应用，iOS 与 Android 平台也都有 APP 可以使用，都可实现在线同步，访问网站等自动填充功能**，安全性上 Bitwarden 采用 256-AES 加密算法，安全性有保障

## Bitwarden 部署

### 准备工作

- 域名（1.主域名处新增域名记录并解析到服务器 IP）
- 服务器（2.服务器安装宝塔面板）
- 安装 Nginx（3.宝塔面板→软件商店→Nginx）
- 安装 Docker（4.宝塔面板→软件商店→Docker 管理器）

### 1.新增域名记录并解析

![](https://pic2.zhimg.com/v2-d99adedf9fcf0d611b639eb520592e31_r.jpg)

### 2.添加站点

宝塔面板 ->网站 ->添加站点 `passwd.nfxwblog.com` ->PHP 纯静态

![](https://pic4.zhimg.com/v2-dd1b24c5f33f7b10143763849843936b_r.jpg)

### 3.Docker 拉取官方镜像

宝塔面板 ->Docker 镜像 ->从仓库中拉取 ->`vaultwarden/server`

或者命令运行：

```sh
docker pull vaultwarden/server:latest
```

项目地址：<https://github.com/dani-garcia/vaultwarden>

### 4.创建 Docker 容器

- Docker->容器 ->添加容器

容器名：自定义，选择 bitwardenrs 镜像；端口：随便写一个没被占用的端口就行，如：5555->80（当有流量访问 5555 端口时，流量会直接映射到容器内的 80 端口）；挂载卷：选择之前创建站点时创建的服务器目录路径就行如：`/www/wwwroot/passwd.nfxwblog.com`，容器目录为 `/data`，完成后提交。**（注：需到宝塔面板 ->安全 界面放行你使用的端口 5555）**

端口这里写反了，容器的 80 映射到服务器的 5555 或其他想设置的端口

![](https://pic3.zhimg.com/v2-94cb5875964a2c73a67f8438df51f3b6_r.jpg)

### 5.添加反向代理

- 宝塔面板 ->网站（选择之前添加的站点）->设置 ->反向代理
- 目标 URL：选择你在 Docker 映射的端口 如：5555

![](https://pic4.zhimg.com/v2-7fa7cefb793b2e32ae0a10355d86b0e3_r.jpg)

### 6.申请 SSL 并开启 HTTPS 访问

1. 由于 BitWarden 强制服务端必须提供 HTTPS 安全服务，所以必须申请 SSL 证书并开启 HTTPS
2. [Freessl](https://link.zhihu.com/?target=https%3A//www.nfxwblog.com/go/q2NVo2px/)，填入域名选择亚信，步骤默认下一步即可
3. 下载 KeyManager，按浏览器提示跳转至 KeyManager

![](https://pic4.zhimg.com/v2-8af1ff53168304fb8c1cc59c8b69d74b_r.jpg)

1. 按要求在域名解析处添加记录完成后点击验证

![](https://pic1.zhimg.com/v2-41bcf4c1f527166b47de56298e0b75bc_r.jpg)

1. 完成后提示保存到 KeyManager，导出为 Nginx 格式得到压缩包，解压后使用文本编辑器分别打开 crt 与 key 格式文件，并在站点设置处 SSL 里填入秘钥与证书，开启强制 SSL

![](https://pic4.zhimg.com/v2-0f4c8c9a2b1e3b26fe2ddfd0620adabb_r.jpg)

![](https://pic4.zhimg.com/v2-a56a209e027d9f632be5ad57226f42a7_r.jpg)

### 7.Bitwarden 搭建完成注册账号

1. 访问站点 [https://passwd.nfxwblog.com/](https://link.zhihu.com/?target=https%3A//passwd.nfxwblog.com/) 出现主界面说明搭建成功

![](https://pic2.zhimg.com/v2-e697e0339b72b0de46b35b4d115319e1_r.jpg)

1. 创建个人账号

![](https://pic3.zhimg.com/v2-e69bc727f9bda890a6c1608b0ddd30ba_r.jpg)

1. 后台预览

![](https://pic1.zhimg.com/v2-24c76dec6a95a078c0376c35555617c8_r.jpg)

### 8.Bitwarden 功能演示与数据迁移

#### 1.浏览器使用 Bitwarden 插件

1. 使用 Edge 为例，打开 Microsoft Edge 扩展商店，搜索 Bitwarden 并安装

![](https://pic3.zhimg.com/v2-21b8f24d11d3cc8494e29c71503bb7ba_r.jpg)

1. 配置服务器 URL，保存跳转至登入选项后输入密码完成登入

![](https://pic2.zhimg.com/v2-c9da38ae083597108a5349b276246445_r.jpg)

1. 当我们在注册网站时浏览器就会弹出 BitWarden 的自动保存密码功能，**如需把保存在浏览器中的密码库全部迁移到 BitWarden 密码管理器中的话请往下看**

![](https://pic1.zhimg.com/v2-e179197a74bfc1385eb9813fd3172ecc_r.jpg)

1. 浏览器自动填充功能在 BitWarden 插件 ->设置 ->选项 ->开启页面加载时自动填充

![](https://pic2.zhimg.com/v2-178361b5131f567820a0ef910837a5c5_r.jpg)

1. BitWarden 自动填充效果演示

![](https://pic4.zhimg.com/v2-2230f2478a763304a031e89fc43fc83f_r.jpg)

#### 2.迁移浏览器密码库至 Bitwarden

1. Edge 为例，Chrome 及其他浏览器同理，打开设置 ->密码 ->导出密码 ->得到.csv 后缀文件

![](https://pic3.zhimg.com/v2-50d9ab2c8ffa2e9e1c8aa2646f4201a6_r.jpg)

1. 打开搭建的 BitWarden 网页端，工具 ->导入数据 ->文件格式：csv（edge 使用的也是 Chromium 内核）

![](https://pic1.zhimg.com/v2-8733cf29c06484dfe22d89275588d414_r.jpg)

> 至此也就完成数据迁移至 Bitwarden

#### 3.IOS 上使用 Bitwarden

1. AppStore 上搜索 Bitwarden 并下载

![](https://pic4.zhimg.com/v2-ac568a5ace03b8e42a1ee55cdffdf387_r.jpg)

1. 打开 APP，点击设置按钮 ->服务器 URL 处添加域名，完成后保存并使用注册的 Bitwarden 账号登入（附：可在 Bitwarden 设置界面开启 Face ID 验证）

![](https://pic3.zhimg.com/v2-1af525c60e5f72150183ef4f708ebd22_r.jpg)

1. IOS 使用示例：如打开我的图床查看自动填充功能 [南风的公益图床](https://link.zhihu.com/?target=https%3A//www.nfxwblog.com/go/ERK8rRWj/)

![](https://pic2.zhimg.com/v2-61f40b0a74295a4c2a10149f85688225_r.jpg)

### 9.Bitwarden 安全设置（可选）

> 也就是在我们完成搭建 Bitwarden 网页端并注册好个人账户后，想要开启如禁止注册、禁止网页访问的功能，如果只是一个人用的话可以关闭注册功能

### 关闭 Bitwarden 注册

1. Docker 管理器中选择创建的容器 Bitwarden 点击停止运行，然后删除容器（删除容器并不会删除数据，拉取的镜像还会存在）

![](https://pic4.zhimg.com/v2-ed7315b0f584caf5dbad6aeedd63a21b_r.jpg)

1. 命令解释

```text
docker run -d --name bitwarden \
-e SIGNUPS_ALLOWED=false \
 -v /安装目录/:/data/ \
-p 你想要的端口:80 \
bitwardenrs/server:latest
```

1. 以关闭注册为例执行以下命令（之前按照我的步骤来的话只需修改第三行 Bitwarden 的容器路径改为自己的路径）

```text
docker run -d --name bitwarden \
  -e SIGNUPS_ALLOWED=false \
  -v /www/wwwroot/passwd.nfxwblog.com/:/data/ \
  -p 5555:80 \
  bitwardenrs/server:latest
```

1. 开启禁止注册后效果

![](https://pic2.zhimg.com/v2-974e2416b1466063ed1c26baa22cb335_r.jpg)

### 10.小结

> Bitwarden 开源且拥有跨平台与终端的能力，以及可自行部署服务端的功能，让密码只掌握在自己手里，不必担心多个网站或 APP 不同密码而管理不便的烦恼 ❤️
