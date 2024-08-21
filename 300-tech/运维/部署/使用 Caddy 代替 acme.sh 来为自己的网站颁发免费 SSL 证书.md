---
title: 使用 Caddy 代替 acme.sh 来为自己的网站颁发免费 SSL 证书
date: 2024-08-16T09:18:00+08:00
updated: 2024-08-21T10:32:40+08:00
dg-publish: false
source: //ruby-china.org/topics/41147
---

最近我公司网站想做个 SSL 证书，然后就看了看 [使用 acme.sh 给 Nginx 安装 Let’s Encrypt 提供的免费 SSL 证书](https://ruby-china.org/topics/31983), 发现 [_@_huacnlee](https://ruby-china.org/huacnlee "@huacnlee") 建议使用 [Caddy](https://caddyserver.com/), 但是没提供教程。

简单的看了一下网站，一如既往的需要安装：

```sh
$ sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
$ curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo apt-key add -
$ curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee -a /etc/apt/sources.list.d/caddy-stable.list
$ sudo apt update
$ sudo apt install caddy
```

`Caddy` 支持两种配置方式，`Caddyfile` 和 `JSON` 格式的文件，官网有比较这两种文件的优缺点，不过我选择了比较简单的 `Caddyfile`

安装完成后创建 `Caddyfile` 文件：

```
yourdomain.com

response "Hello, privacy"
```

第一行是你的域名，第二行是指访问网站时返回的信息。

这里说明一下 `Caddy` 不单单是个颁发域名证书的免费工具，它是一个准备取代 nginx 的工具。

当然，这样配置对我们来说没有实际意义。

PS：记得操作之前先关了 nginx，因为它也是一个反向代理工具。

我们直接出一个 Rails 版的 `Caddyfile` 吧。

```
yourdomain.com

reverse_proxy :3000
```

然后运行命令：

```sh
$ caddy start
```

至此，你的证书已经安装好了，也不需要在 renew 了。

因为我部署的是个 Java 项目，所以没试 `passenger` 服务怎么安装 [_@_huacnlee](https://ruby-china.org/huacnlee "@huacnlee") 能补充一下吗？

另外，我测试了一下，证书达到了 A 级，而不是 A+。

[![](https://l.ruby-china.com/photo/rocLv/145ebe01-6427-42cb-8805-768701dee59a.png!large)](<https://l.ruby-china.com/photo/rocLv/145ebe01-6427-42cb-8805-768701dee59a.png!large>)

另外再补充一下：

单页面应用 Caddyfile 配置

```
example.com {
  root * /root/www/admin
  try_files {path} {file} /index.html
  file_server
  encode gzip
}
```

修改以后无需重启，用 `reload` 重新加载配置：

```sh
$ caddy reload
```

停止服务：

```sh
$ caddy stop
```
