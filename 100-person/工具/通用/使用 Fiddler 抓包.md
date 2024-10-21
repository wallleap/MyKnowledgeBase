---
title: 使用 Fiddler 抓包
date: 2024-10-15T01:37:04+08:00
updated: 2024-10-15T01:42:53+08:00
dg-publish: false
---

## 下载软件

- Fiddler-Classic：免费，只有 Windows 版 <https://www.telerik.com/download/fiddler>
- Fiddler Everywhere：订阅付费，多平台 <https://www.telerik.com/download/fiddler-everywhere>

## 安装后进行配置

Classic 需要额外安装 Fiddler Cert Maker <https://telerik-fiddler.s3.amazonaws.com/fiddler/addons/fiddlercertmaker.exe>，把 Fiddler Cert Maker 放到 Fiddler-Classic 安装的根目录下运行

### HTTPS

进入设置，找到 HTTPS 相关的，开启 HTTPS 解密并且安装证书

### 手机

需要开启允许远程设备连接，关闭防火墙

手机和电脑连到同一个网络，获取电脑 IP，并记住上面远程连接的端口号

进入手机 WiFi 设置，修改连接的 WiFi，把代理改成手动，主机名填电脑IP，端口号填上点保存

然后使用浏览器打开 `IP:端口`

下载证书，后缀修改 `crt`、`cer`/`pem`，手机能够识别就行，然后直接进行安装，选 CA 证书，重启手机
