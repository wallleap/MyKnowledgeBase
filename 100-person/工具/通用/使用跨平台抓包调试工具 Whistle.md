---
title: 使用跨平台抓包调试工具 Whistle
date: 2024-10-16T10:33:41+08:00
updated: 2024-10-16T10:54:26+08:00
dg-publish: false
---

<https://github.com/avwo/whistle>

<https://wproxy.org/whistle/>

Whistle 是基于 Node 实现的跨平台抓包调试工具，其主要特点：

1. **完全跨平台**：支持 Mac、Windows 等桌面系统，且支持服务端等命令行系统
2. **功能强大（理论上可以对请求做任意修改）**：
	- 支持作为 HTTP、HTTPS、SOCKS 代理及反向代理
	- 支持抓包及修改 HTTP、HTTPS、HTTP2、WebSocket、TCP 请求
	- 支持重放及构造 HTTP、HTTPS、HTTP2、WebSocket、TCP 请求
	- 支持设置上游代理、PAC 脚本、Hosts、延迟（限速）请求响应等
	- 支持查看远程页面的 console 日志及 DOM 节点
	- 支持用 Node 开发插件扩展功能，也可以作为独立 npm 包引用
3. **操作简单**：
	- 直接通过浏览器查看抓包、修改请求
	- 所有修改操作都可以通过配置方式实现（类似系统 Hosts），并支持分组管理
	- 项目可以自带代理规则配置并一键设置到本地 Whistle 代理，也可以通过定制插件简化操作

## 安装

```sh
# 安装了 node
# 使用 Node 安装
$ npm i -g whistle  # 如果不想后面配置 CA 和代理，可以加上 && w2 start --init
# mac 可以直接使用 brew
$ brew install whistle
```

安装完成如果不知道 IP 和端口可以重启 whsitle

```sh
$ w2 restart
[i] whistle@2.9.86 restarted
[i] 1. use your device to visit the following URL list, gets the IP of the URL you can access:
       http://127.0.0.1:8899/
       http://192.168.3.5:8899/
       Note: If all the above URLs are unable to access, check the firewall settings
             For help see https://github.com/avwo/whistle
[i] 2. set the HTTP proxy on your device with the above IP & PORT(8899)
[i] 3. use Chrome to visit http://local.whistlejs.com/ to get started
```

关闭防火墙

## 安装 CA 设置全局代理

```sh
$ w2 ca
$ w2 proxy
$ w2 proxy off
```

如果是要抓局域网其他设备，可以先下载 CA 证书，然后到那个设备上安装好

之后在 WiFi 设置中配置代理地址和端口号

访问 <http://local.whistlejs.com/>，右上角 online 有新 IP 出现

## 设置 mock 规则

````
# 普通的 Host
test1.wproxy.org 127.0.0.1
test2.wproxy.org 127.0.0.1
# 或
127.0.0.1 test1.wproxy.org test2.wproxy.org

# 带端口，匹配路径、协议、正则、通配符等 ^（通配路径表示符）、$（精确匹配）、*（通配符）、!（取非）
test1.wproxy.org/path/to 127.0.0.1:6001
https://test2.wproxy.org/path1/to1 127.0.0.1:6001
# 根据请求参数设置 host
/google/ 127.0.0.1:6001
# 或
127.0.0.1:6001 test1.wproxy.org/path/to https://test2.wproxy.org/path1/to1 /google/

# CNAME
test1.wproxy.org/path/to host://www.qq.com:8080

# 通过请求参数设置 Hosts
/host=([\w.:-]+)/ host://$1

# 修改 Cookie
www.qq.com reqCookies://custom_key1=123&custom_key2=789

# 替换本地内容
test.wproxy.org/test file:///Users/xx/statics

# 替换其它请求
test.wproxy.org/test https://ke.qq.com

# 查看 JS 报错及 console.log
ke.qq.com log://

# 模拟响应 500（请求不会到后台服务）
test3.wproxy.org/path/to statusCode://500

# 修改响应状态码（请求会到后台服务）
test4.wproxy.org/path/to replaceStatus://400

# 302 重定向
test5.wproxy.org/path redirect://https://ke.qq.com/

# 301 重定向
test6.wproxy.org/path redirect://https://ke.qq.com/ replaceStatus://301

# 操作值
www.test.com/index.html file://{test.html}
``` test.html
Hello world.
Hello world1.
Hello world2.
```
````
