---
title: 使用 Caddy 代替 acme.sh 来为自己的网站颁发免费 SSL 证书
date: 2024-08-16T09:18:00+08:00
updated: 2024-10-28T05:25:41+08:00
dg-publish: false
source: //ruby-china.org/topics/41147
---

最近我公司网站想做个 SSL 证书，然后就看了看 [使用 acme.sh 给 Nginx 安装 Let’s Encrypt 提供的免费 SSL 证书](https://ruby-china.org/topics/31983), 发现 [_@_huacnlee](https://ruby-china.org/huacnlee "@huacnlee") 建议使用 [Caddy](https://caddyserver.com/), 但是没提供教程。

官网：<https://caddyserver.com/>

## 安装

简单的看了一下网站，一如既往的需要安装：

```sh
$ sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
$ curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo apt-key add -
$ curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee -a /etc/apt/sources.list.d/caddy-stable.list
$ sudo apt update
$ sudo apt install caddy
```

mac 的直接通过 brew 安装

```sh
$ brew install caddy
$ brew services start caddy
```

docker 安装

```sh
$ docker pull caddy
$ echo "hello world" > index.html
$ vim Caddyfile  # 自定义配置文件
$ docker run -d -p 80:80 -p 443:443 -p 443:443/udp \
    -v $PWD/Caddyfile:/etc/caddy/Caddyfile \
    -v $PWD/index.html:/usr/share/caddy/index.html \
    -v caddy_data:/data \
    caddy
$ curl http://localhost/
hello world
```

## 基础命令

```sh
# 列出可用子命令
$ caddy

# 作为守护进程启动
$ caddy run

# 启动并让它在后台运行
$ caddy start
$ caddy stop  # 通过这个停止

# 重载配置
$ caddy reload
```

更多命令 [命令行 - Caddy 文档 - Caddy 中文 (golang.ac.cn)](https://caddy.golang.ac.cn/docs/command-line)

## 配置

`Caddy` 支持两种配置方式，`Caddyfile` 和 `JSON` 格式的文件，官网有比较这两种文件的优缺点，不过我选择了比较简单的 `Caddyfile`

安装完成后创建 `Caddyfile` 文件：

```
yourdomain.com

respond "Hello, world!"
```

第一行是你的域名（或端口，例如 `:20115` 省略了 `localhost`），第二行是指访问网站时返回的信息。

在当前目录下重新运行 `caddy run` 或选择 Caddyfile 路径 `caddy run --config /path/to/Caddyfile`

这样就可以通过这个域名或 `localhost:20115` 进行访问了

配置多个站点

```
localhost {
	respond "Hello, world!"
}

localhost:2016 {
	respond "Goodbye, world!"
}
```

## 更多功能

这里说明一下 `Caddy` 不单单是个颁发域名证书的免费工具，它是一个准备取代 nginx 的工具。

当然，这样配置对我们来说没有实际意义。

### 静态文件服务器

```sh
$ caddy file-server --root ~/mysite --listen :2015 --browse
```

可以不带参数，就默认以当前目录为站点，但可能监听不了低端口

- `--root` 指定文件夹作为网站根目录
- `--listen` 绑定其他端口号
- `--browse` 没有索引文件也可以显示文件列表

在配置文件中

```
localhost:2015

root * ~/mysite
file_server browse
```

### 反向代理

PS：记得操作之前先关了 nginx，因为它也是一个反向代理工具。

命令

```sh
$ caddy reverse-proxy --from :2080 --to :9000
```

代理 HTTPS，由于默认会给网站使用 443 端口，所以可以省略 `--from`

```sh
$ caddy reverse-proxy --to :9000
```

其他域名

```sh
$ caddy reverse-proxy --from example.com --to https://example.com:9000
```

如果两个主机名不同，则需要加参数

```sh
$ caddy reverse-proxy --from example.com --to https://localhost:9000 --change-host-header
```

如果使用的是 `Caddyfile`，只需将第一行更改为您的域名

```
yourdomain.com

reverse_proxy :3000
```

### SSL 证书

前提：已经把域名解析到服务器 IP 地址了（A 或 AAAA 解析）

直接新建 `Caddyfile`

```
yourdomain.com

respond "Hello, privacy!"
```

同级目录下运行 `caddy run`，它会自动申请证书

静态文件

```sh
$ caddy file-server --domain example.com
```

反向代理

```sh
$ caddy reverse-proxy --from example.com --to localhost:9000
```

JSON 格式配置

```json
{
	"apps": {
		"http": {
			"servers": {
				"hello": {
					"listen": [":443"],
					"routes": [
						{
							"match": [{
								"host": ["example.com"]
							}],
							"handle": [{
								"handler": "static_response",
								"body": "Hello, privacy!"
							}]
						}
					]
				}
			}
		}
	}
}
```

match 下 host 都将触发自动 HTTPs

## 补充

### API 管理

可以调用相应的 API 接口，直接上传配置或管理

### Caddyfile

**响应文本**

```
localhost

respond "Hello, world!"
```

等效于

```
# 这是注释，加大括号和上面一样，但多个网站就必须加大括号
localhost {
  respond "Hello, world!"
}
```

**文件服务器**

```
localhost

file_server browse
```

**模板页面**

`caddy.html`

```html
<!DOCTYPE html>
<html>
	<head>
		<title>Caddy tutorial</title>
	</head>
	<body>
		Page loaded at: {{now | date "Mon Jan 2 15:04:05 MST 2006"}}
	</body>
</html>
```

配置

```
localhost

templates
file_server browse
```

现在访问 <https://localhost/caddy.html> 就能看到具体的时间

**启用 Gzip 和 Zstandard 支持**

```
localhost

encode zstd gzip
templates
file_server browse
```

**多个网站共用配置**

```
:8080, :8081 {
  file_server browse
}
```

**匹配器令牌**

```
localhost

file_server
reverse_proxy /api/* 127.0.0.1:9005
```

### 单页面应用 Caddyfile 配置

```
example.com {
  root * /root/www/admin
  try_files {path} {file} /index.html
  file_server
  encode gzip
}
```
