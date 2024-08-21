---
title: Cockpit 安装配置速记
date: 2024-04-22T10:04:00+08:00
updated: 2024-08-21T10:32:40+08:00
dg-publish: false
---

[Cockpit安装配置速记 – 知了 (zhile.io)](https://zhile.io/2023/12/30/cockpit-configuration.html#more-614)

事情起因是手头服务器日渐增多，需要一个一站式管理面板。我管理需求不复杂，能看一些基本状态、能管理文件和使用 shell 即可，主要是一站式管理。

调研了目前存在的产品，各种面板开源不开源的很多，但能一管多的面板不多。最后看到了 [Cockpit](https://cockpit-project.org/) 完美符合我的需求。它是 RedHat 扶持的开源面板项目，在红帽 8 和 CentOS8 中是默认面板。界面也是简洁精致，功能非常强大。

话不多说，这篇文章是速记一些安装配置，以便大家参考。其实官网上有各种安装配置的文档，大家可以去具体翻翻。

这里记录我自己用到的 `Ubuntu` 系统。成品：[https://cockpit.zhile.io](https://cockpit.zhile.io/)

## 安装

参考文档：[https://cockpit-project.org/running.html#ubuntu](https://cockpit-project.org/running.html#ubuntu)

```bash
. /etc/os-release
sudo apt install -t ${VERSION_CODENAME}-backports cockpit
```

Bash

Copy

## 启动

```bash
sudo systemctl enable cockpit.socket
sudo systemctl start cockpit.socket
```

Bash

Copy

## 安装插件

Cockpit 支持插件，目前有一批官方和第三方的插件，具体列表：[https://cockpit-project.org/applications.html](https://cockpit-project.org/applications.html)

目前我就用到了一个 `Cockpit Navigator` 插件，用来网页上管理服务器的文件。

```bash
wget https://github.com/45Drives/cockpit-navigator/releases/download/v0.5.10/cockpit-navigator_0.5.10-1focal_all.deb
sudo apt install ./cockpit-navigator_0.5.10-1focal_all.deb
```

Bash

Copy

## 配置

Cockpit 默认监听 `0.0.0.0:9090`，但我不希望直接暴露端口，想用 nginx 给它代理出来。所以需要配置一下。参考文档：[https://cockpit-project.org/guide/latest/listen.html#listen-systemd](https://cockpit-project.org/guide/latest/listen.html#listen-systemd)

```bash
sudo mkdir /etc/systemd/system/cockpit.socket.d
cat << "EOF" | sudo tee /etc/systemd/system/cockpit.socket.d/listen.conf > /dev/null
[Socket]
ListenStream=
ListenStream=127.0.0.1:9090
FreeBind=yes
EOF
sudo systemctl daemon-reload
sudo systemctl restart cockpit.socket
```

Bash

Copy

到这里 Cockpit 就算安装配置好了，所有的节点也只需要如此配置即可。

如果你希望禁止某些用户通过 Cockpit 页面登录，可以配置 `/etc/cockpit/disallowed-users`，把用户名放进去即可。

```bash
cat << "EOF" | sudo tee /etc/cockpit/disallowed-users > /dev/null
# List of users which are not allowed to login to Cockpit
root
EOF
```

Bash

Copy

## 配置反代

我们无需在每台节点安装配置反代，挑选其中一台主控配置即可。先配置一下 Cockpit，参考文档：[https://cockpit-project.org/guide/latest/cockpit.conf.5.html](https://cockpit-project.org/guide/latest/cockpit.conf.5.html)

```bash
cat << "EOF" | sudo tee /etc/cockpit/cockpit.conf > /dev/null
[WebService]
Origins = https://cockpit.zhile.io # 这里改成你自己的地址
ProtocolHeader = X-Forwarded-Proto # 与nginx中一致
ForwardedForHeader = X-Forwarded-For # 与nginx中一致
LoginTo = false
AllowUnencrypted = false # 我使用了https，这里设置为false
EOF
sudo systemctl restart cockpit.socket
```

Bash

Copy

然后配置 nginx：

```nginx
server {
    listen 443 ssl http2;
    server_name cockpit.zhile.io;

    charset utf-8;

    ssl_certificate      certs/cockpit.zhile.io.crt;
    ssl_certificate_key  certs/cockpit.zhile.io.key;

    ...省略若干其他配置...

    location /cockpit/socket {
        proxy_http_version  1.1;
        proxy_pass          http://127.0.0.1:9090;
        proxy_set_header    Connection          "upgrade";
        proxy_set_header    Upgrade             http_upgrade;
        proxy_set_header    Host                http_host;
        proxy_set_header    X-Forwarded-Proto   scheme;
        proxy_set_header    X-Forwarded-For     proxy_add_x_forwarded_for;
    }

    location / {
        proxy_http_version  1.1;
        proxy_pass          http://127.0.0.1:9090;
        proxy_set_header    Connection          "";
        proxy_set_header    Host                http_host;
        proxy_set_header    X-Forwarded-Proto   scheme;
        proxy_set_header    X-Forwarded-For     proxy_add_x_forwarded_for;
    }

    ...省略若干其他配置...
}
```

nginx

Copy

打完收工～

nginx 配置中：

- http_upgrade 其实是 $http_upgrade
- http_host 其实是 $http_host
- scheme 其实是 $scheme
- proxy_add_x_forwarded_for 其实是 $proxy_add_x_forwarded_for

博客 markdown 代码块有 bug，注意自行添加。

nginx 证书什么的自己想办法，现在各种免费的很多。这种涉及服务器管理的，最好还是上 https。你看我，上了 https 还整个域名上 cf 盾，美滋滋。

**强调一下，这种 web 配置不是每个节点都需要配置，只要配置其中一台即可**
