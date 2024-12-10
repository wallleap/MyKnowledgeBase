---
title: http-server 在本地启动 https 服务以及配置域名
date: 2024-08-29T09:44:24+08:00
updated: 2024-11-05T13:53:24+08:00
dg-publish: false
---

附上原文（[https://www.npmjs.com/package/http-server](https://www.npmjs.com/package/http-server "NPM http-server")）。

TLS/SSL

---

First, you need to make sure that [openssl](https://github.com/openssl/openssl) is installed correctly, and you have `key.pem` and `cert.pem` files. You can generate them using this command:

```shell
openssl req -newkey rsa:2048 -new -nodes -x509 -days 3650 -keyout key.pem -out cert.pem
```

You will be prompted with a few questions after entering the command. Use `127.0.0.1` as value for `Common name` if you want to be able to install the certificate in your OS's root certificate store or browser so that it is trusted.

This generates a cert-key pair and it will be valid for 3650 days (about 10 years).

Then you need to run the server with `-S` for enabling SSL and `-C` for your certificate file.

```shell
http-server -S -C cert.pem
```

If you wish to use a passphrase with your private key you can include one in the openssl command via the -passout parameter (using password of foobar)

e.g. `openssl req -newkey rsa:2048 -passout pass:foobar -keyout key.pem -x509 -days 365 -out cert.pem`

For security reasons, the passphrase will only be read from the `NODE_HTTP_SERVER_SSL_PASSPHRASE` environment variable.

This is what should be output if successful:

```shell
Starting up http-server, serving ./ through https

http-server settings:
CORS: disabled
Cache: 3600 seconds
Connection Timeout: 120 seconds
Directory Listings: visible
AutoIndex: visible
Serve GZIP Files: false
Serve Brotli Files: false
Default File Extension: none

Available on:
  https://127.0.0.1:8080
  https://192.168.1.101:8080
  https://192.168.1.104:8080
Hit CTRL-C to stop the server
```

本文转自 <https://www.cnblogs.com/DreamSeeker/p/16468040.html>，如有侵权，请联系删除。

## 1、安装 http-server

```sh
npm install --global http-server
```

也可以不全局安装

```sh
pnpm init
pnpm install -D http-server
```

接着在 `package.json` 中添加

```json
{
  "scripts": {
    "server": "http-server ."
  }
}
```

之后运行

```sh
pnpm run server
```

## 2、生成证书文件

有两个。一个是 cert.pem, 一个是 key.pem 

```sh
openssl req -newkey rsa:2048 -new -nodes -x509 -days 3650 -keyout key.pem -out cert.pem
```

这里成功的文件可以修改地址，最后在启动的时候 -C 后面跟 cert 的路径，-K 后面跟 key 的路径，注意填绝对路径，如 `/usr/local/etc/nginx/ssl/cert.pem`。路径在启动服务的时候对应上就行。默认是生成的当前目录的。

## 2、mac 推荐使用这个生成证书

```sh
brew install mkcert
brew install nss # for Firefox
# 把 mkcert 添加到本地根 CA
mkcert -install
# 为网站生成一个证书
mkcert localhost  # 只给 localhost
mkcert mysite.example  # 如果域名为这个的话
```

## 3、启动服务

启动服务的时候，我的是前端项目，所以会进入到 dist 目录，里面有 index.html 的文件夹，运行命令。

```sh
http-server -S -C cert.pem -K key.pem
```

注意大小写，这里指定目录的时候是大写；用上面代码执行的时候你的两个证书文件都要放在当前目录下，否则的话就要写明证书文件的路径。

```sh
# 这里是不写 —C 或者 —K 的时候的默认值：  
-C or --cert ssl cert 文件路径 (default: cert.pem)
-K or --key Path to ssl key file (default: key.pem)
```

看到下面的效果就是启动成功了。

```sh
Starting up http-server, serving ./ through https
Available on:
  https://127.0.0.1:8080
  https://10.0.51.99:8080
Hit CTRL-C to stop the server
```

## 4、配置域名

这一步其实和 http-server 没有关系，因为我用到也提一下。在某些设定下可以解决后端请求有域名要求，域名限制的问题。

配置其实很简单，就是利用 host，以为 http-server 都是在 127.0.0.1 上启动服务的，我们利用 host 转发过去就好了。

```sh
127.0.0.1 test.jd.com
```

配置上述 host 就可以在浏览器实现 <https://test.jd.com:8080> 来访问了。

注意：在最新的谷歌浏览器中，会出现拦截并无法访问的情况。应该是谷歌浏览器安全又升级了，这个换成火狐浏览器，选择高级，接受风险并继续就可以了。根据经验，谷歌浏览器通过忽略安全的方式启动应该也可以，有时间查一下补上吧。

http-server 参数说明：

```sh
-p 端口号 (默认 8080)
-a IP 地址 (默认 0.0.0.0)
-d 显示目录列表 (默认 'True')
-i 显示 autoIndex (默认 'True')
-e or --ext 如果没有提供默认的文件扩展名(默认 'html')
-s or --silent 禁止日志信息输出
--cors 启用 CORS via the Access-Control-Allow-Origin header
-o 在开始服务后打开浏览器
-c 为 cache-control max-age header 设置Cache time(秒) , e.g. -c10 for 10 seconds (defaults to '3600'). 禁用 caching, 则使用 -c-1.
-U 或 --utc 使用UTC time 格式化log消息
-P or --proxy Proxies all requests which can't be resolved locally to the given url. e.g.: -P http://someurl.com
-S or --ssl 启用 https
-C or --cert ssl cert 文件路径 (default: cert.pem)
-K or --key Path to ssl key file (default: key.pem).
-r or --robots Provide a /robots.txt (whose content defaults to 'User-agent: \*\\nDisallow: /')
```

## Node 搭建的服务器

```js
const https = require('https');
const fs = require('fs');
const options = {
  key: fs.readFileSync('{PATH/TO/CERTIFICATE-KEY-FILENAME}.pem'),
  cert: fs.readFileSync('{PATH/TO/CERTIFICATE-FILENAME}.pem'),
};
https
  .createServer(options, function (req, res) {
    // server code
  })
  .listen({PORT});
```
