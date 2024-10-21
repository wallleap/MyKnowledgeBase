---
title: Windows 下三分钟搭建 Shadowoscks 服务器端
date: 2024-09-06T04:14:40+08:00
updated: 2024-09-06T04:38:33+08:00
dg-publish: false
source: https://www.librehat.com/three-minutes-to-set-up-shadowsocks-server-on-windows/
---

[libQtShadowsocks](https://github.com/librehat/libQtShadowsocks) 项目，轻轻松松只要三分钟，让 Shadowsocks 服务端在你的 Windows 机器上跑起来！不用自己编译，不用安装什么 Python、.Net 的。

## 步骤

先上步骤，内情在后面再叙。

1. 下载最新的 [shadowsocks-libqss](https://github.com/shadowsocks/libQtShadowsocks/releases)（Fedora 下软件包名为 shadowsocks-libQtShadowsocks）
2. 准备好 config.json 文件（[范例文件点此](https://github.com/shadowsocks/libQtShadowsocks/blob/master/shadowsocks-libqss/config.json)，点 Raw 另存为 config.json 就能下载下来了），server 字段填写”0.0.0.0″（如果走 IPv6 线路，填”::”），加密方式可以改成更新潮的”chacha20″（老旧的 Shadowsocks 客户端可能不支持 chacha20 加密）
3. 要 shadowsocks-libqss.exe 以 server mode 运行需要加上 -S 参数，你可以从命令行提示符去运行。或者，更简单的，用我写好的 bat 脚本（[点此查看](https://gist.github.com/librehat/c6a04effc8936ff93be5)，点 Raw 按钮另存为 bat 脚本即可）
4. 将 shadowsocks-server.bat 与 shadowsocks-libqss.exe, config.json 放在同一个文件夹下，执行 shadowsocks-server.bat 即可

停止运行？关掉就可以了。下次再运行？执行 shadowsocks-server.bat 脚本就好了。修改配置文件？别用自带的记事本和写字板，去下载一个 [Notepad++](http://notepad-plus-plus.org/) 吧，免得格式乱套了。

可以使用 instsrv.exe+srvany.exe 将应用程序安装为服务

```cmd
d:\instsrv.exe shadowsocks d:\srvany.exe
```

然后注册表 `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\shadowsocks` 添加 Parameters 项 新建以下几个字符串值：

- 名称 `Application` 值为你要作为服务运行的程序地址。
- 名称 `AppDirectory` 值为你要作为服务运行的程序所在文件夹路径。
- 名称 `AppParameters` 值为你要作为服务运行的程序启动所需要的参数。

```cmd  
sc start shadowsocks  
```

完成

## 具体

### 下载资源包

打包好的这个版本 <https://github.com/shadowsocks/libQtShadowsocks/releases/download/v2.0.2/shadowsocks-libqss-v2.0.2-win64.7z>

解压出来得到 `shadowsocks-libqss.exe`

然后新建 `config.json`，内容为

```json
{
    "server":"0.0.0.0",
    "server_port":8388,
    "local_address":"127.0.0.1",
    "local_port":1080,
    "password":"password",
    "timeout":600,
    "method":"rc4-md5",
    "http_proxy": true
}
```

新建 `shadowsocks-server.bat`，内容为

```bat
::This batch will run shadowsocks-libqss in server mode using the config.json file in current folder as the configuration

@echo off
::this script is updated for version 1.7.0
shadowsocks-libqss.exe -c config.json -S
```

之后把这三个文件放到同一个目录中，例如 `ssserver` 目录，接着双击 `shadowsocks-server.bat`

### 关闭防火墙或开放端口 8388

> [!IMPORTANT]
> 需要找到所有有关防火墙的地方关闭它

### 客户端配置

使用 SSR 客户端（ShadowsocksX-NG 等）

新增一个服务器配置

- Address：服务器的 IP 地址，例如 `192.168.1.10`
- Port：上面服务器的端口号 `8388`
- Encryption：加密方式，要和上面对应 `rc4-md5`
- Password：`password`
- 可以改个备注，其他保持默认

点击 OK

然后设置代理为 Global 全局模式，并且 Turn on 启动它

> 外网访问，路由器要设置好端口转发或者开启 DMZ，不然公网 IP 是不能直接访问你的电脑的

## “内情”

使用的这个程序 shadowsocks-libqss 本身是 [libQtShadowsocks](https://github.com/shadowsocks/libQtShadowsocks) 项目的一部分，最初始的目的是演示如何在 Qt 程序里调用 libQtShadowsocks，后来则成为一个命令行式的客户端/服务端了。因为本身 libQtShadowsocks 就兼具客户端和服务端的功能，两者的区别很不大，如果拆成两个程序反而奇怪。

所以默认情况下 shadowsocks-libqss 会读取当前文件夹下的 config.json 文件，并以客户端模式启动。使用 -c 可以指定配置文件，而 -S 参数则使其以服务端模式运行。

配置文件 config.json 的格式和其他 Shadowsocks 版本一样，双向兼容~~（说了无痛>.<)

简单说一下相比其他文章提到的其他版本，shadowsocks-libqss 的优势在哪里吧。shadowsocks-libev 在 Windows 下编译比较麻烦，如果你下载第三方编译的版本，总是有点不放心不是？（当然，你下载的是我编译的 shadowsocks-libqss，如果你不放心就别下拉倒）。~~shadowsocks-go 已经很久没有实质更新了~~。shadowsocks-csharp-server 支持的加密方式很少而且依赖.Net…主线的 python 版本依赖 Python 环境，而且要支持 Salsa20 和 ChaCha20 的话还要装 libsodium，值得注意的还有 shadowsocks-libqss 的 CPU、内存占用小。

最后，我的博客不是反映问题的地方，有问题请报到 [libQtShadowsocks的issues](https://github.com/shadowsocks/libQtShadowsocks/issues) 列表！

05 March 2015 Update:

从 1.4.2 版起，服务器地址填写:: 表示同时监听所有 IPv4 和所有 IPv6 地址，~~和 shadowsocks-libev 一致~~。之前的版本填写:: 仅监听所有 IPv6 地址。
