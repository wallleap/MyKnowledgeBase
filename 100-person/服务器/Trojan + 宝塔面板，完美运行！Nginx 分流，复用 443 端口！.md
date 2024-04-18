---
title: Trojan + 宝塔面板，完美运行！Nginx 分流，复用 443 端口！
date: 2023-08-09 20:51
updated: 2024-04-14 03:09
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - Trojan + 宝塔面板，完美运行！Nginx 分流，复用 443 端口！
rating: 1
tags:
  - server
category: server
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: https://v2rayssr.com/trojan-bt.html
url: https://obsius.site/2q5q4s0j555i0b0x1f09
---

## 准备工作

1、VPS 一台内存最好 1GB 左右，重置任意可安装宝塔面板的系统（建站推荐 CentOS7X 以上，或者是 CentOS8X）

2、一级域名一个做好相应的解析（今天作者解析的域名为：`v2rayssr.net` , `www.v2rayssr.net`）

（分别解析主域名和 www 的二级域名到 VPS 主机。）

## 实现的效果

我们访问 `v2rayssr.net` 及 `www.v2rayssr.net` 均能使用 https 的方式，也就是正常访问我们的博客。

使用 Trojan 客户端连接 `www.v2rayssr.net` 可以正常科学上网。

## 教程开始

### 1、连接 VPS 并安装宝塔面板

以下一键安装宝塔面板的代码仅仅局限 CentOS 系统的用户，其他用户请访问 [这里](https://v2rayssr.com/go?url=https://bt.cn/bbs/thread-19376-1-1.html) 查看安装命令

```sh
yum install -y wget && wget -O install.sh http://download.bt.cn/install/install_6.0.sh && sh install.sh
```

安装完毕以后，请自行记录宝塔面板的登录地址，并登录宝塔面板。

### 2、安装建站需要的必要插件

登录以后，在弹出的窗口环境中选择自己需要安装的环境。

（若是使用 WordPress 的博客，请依次安装 Nginx1.18、MySQL5.6、PHP7.4，其他非必要环境作者不想安装就略过了）

### 3、创建网站

因为作者使用 WordPress 博客的 CMS，所以选择宝塔的一键部署。

分别点击 “软件商店” – “一键部署” ，找到 WordPress 并选择一键部署，填写我们刚才解析好的两个域名。

![](https://v2rayssr.com/wp-content/uploads/2020/07/%E6%88%AA%E5%B1%8F2020-07-29-12.16.22.png)

如果是纯静态的就选 PHP 纯静态就行，添加了域名然后按下面的配置

### 4、设置网站数据库并配置网站

部署完毕以后，输入我们绑定的域名进行数据库配置及网站设置。不明白请查看 视频教程

### 5、修改默认的 Nginx 配置

找到 “软件商店” – “已安装” – “Nginx1.18”，设置 Nginx 的配置信息

![](https://v2rayssr.com/wp-content/uploads/2020/07/%E6%88%AA%E5%B1%8F2020-07-29-12.19.51.png)

在 http 模块前面增加如下代码，按照自己的需求进行更改（主要是域名）：

```
stream {
    # 这里就是 SNI 识别，将域名映射成一个配置名
    map $ssl_preread_server_name $backend_name {
        v2rayssr.net web;
        www.v2rayssr.net trojan;
    # 域名都不匹配情况下的默认值
        default web;
    }
 
    # web，配置转发详情
    upstream web {
        server 127.0.0.1:4433;
    }
 
    # trojan，配置转发详情
    upstream trojan {
        server 127.0.0.1:10110;
    }
 
    # 监听 443 并开启 ssl_preread
    server {
        listen 443 reuseport;
        listen [::]:443 reuseport;
        proxy_pass  $backend_name;
        ssl_preread on;
    }
}
```

### 6、在线申请 SSL 证书

在网站设置里面，勾选两个绑定的域名，并为其申请 SSL 证书和开启强制 Https 的访问。

![](https://v2rayssr.com/wp-content/uploads/2020/07/%E6%88%AA%E5%B1%8F2020-07-29-12.24.18.png)

### 7、修改网站的 Nginx 配置文件

在网站设置里面开启 “伪静态” 为 WordPress ，然后转到配置文件设置。

![](https://v2rayssr.com/wp-content/uploads/2020/07/%E6%88%AA%E5%B1%8F2020-07-29-12.28.14.png)

删除在 server 模块下面的 server_name 里面的 二级域名，只保留主域名

更改 server 模块下面的 443 端口为 4433。

在原有的 server 模块下面增加如下代码（请自行替换域名）

```
server
{
    listen 10111;
    server_name www.v2rayssr.net;
    location / {
        
        if ($http_host !~ "^v2rayssr.net$") {
          rewrite  ^(.*)    https://v2rayssr.net$1 permanent;
        }
 
       if ($server_port !~ 4433){
        rewrite ^(.*)   https://v2rayssr.net$1 permanent;
    }
 
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        
    }
    access_log logs/aaa.com_access.log;
}
```

> **更改完毕以后，请回到 Nginx 设置界面，重启 Nginx 服务。**

### 8、安装官方 Trojan 服务

```sh
sudo bash -c "$(curl -fsSL https://raw.githubusercontent.com/trojan-gfw/trojan-quickstart/master/trojan-quickstart.sh)"
```

设置 Trojan 开启自动启动

```sh
systemctl enable trojan   #设置Trojan开启自动启动
```

### 9、修改 Trojan 配置文件

找到 VPS 以下文件 `/usr/local/etc/trojan/config.json` 修改为如下代码：（自行更改密码和域名证书路径）

```
{
    "run_type": "server",
    "local_addr": "127.0.0.1",
    "local_port": 10110,
    "remote_addr": "127.0.0.1",
    "remote_port": 10111,
    "password": [
        "321321321"
    ],
    "log_level": 1,
    "ssl": {
        "cert": "/www/server/panel/vhost/cert/v2rayssr.net/fullchain.pem",
        "key": "/www/server/panel/vhost/cert/v2rayssr.net/privkey.pem",
        "key_password": "",
        "cipher": "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384",
        "cipher_tls13": "TLS_AES_128_GCM_SHA256:TLS_CHACHA20_POLY1305_SHA256:TLS_AES_256_GCM_SHA384",
        "prefer_server_cipher": true,
        "alpn": [
            "http/1.1"
        ],
        "alpn_port_override": {
            "h2": 81
        },
        "reuse_session": true,
        "session_ticket": false,
        "session_timeout": 600,
        "plain_http_response": "",
        "curves": "",
        "dhparam": ""
    },
    "tcp": {
        "prefer_ipv4": false,
        "no_delay": true,
        "keep_alive": true,
        "reuse_port": false,
        "fast_open": false,
        "fast_open_qlen": 20
    },
    "mysql": {
        "enabled": false,
        "server_addr": "127.0.0.1",
        "server_port": 3306,
        "database": "trojan",
        "username": "trojan",
        "password": "",
        "key": "",
        "cert": "",
        "ca": ""
    }
}
```

更改完毕以后，上传并保存，然后重启 Trojan 服务

```sh
systemctl restart trojan
```

防火墙开放 10110、10111

### 10、搭建和设置完毕

现在，你就可以连接你的 Trojan 节点了，网站也可以正常的访问了。

安卓：[SS客户端 归档 – v2cross](https://v2cross.com/ss-client)
