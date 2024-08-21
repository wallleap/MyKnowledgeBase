---
title: Linux 服务器部署内网穿透供 mac 使用
date: 2024-07-31T09:41:00+08:00
updated: 2024-08-21T10:32:41+08:00
dg-publish: false
---

[Releases · fatedier/frp (github.com)](https://github.com/fatedier/frp/releases)

## 服务器端配置

先查看架构

```sh
arch
```

根据架构下载对应的后缀文件

- amd64：适用于 x86_64 架构（64 位 Intel 和 AMD CPU）。如果您的系统是
- x86_64：这是与 amd64 相同的描述，通常也表示 64 位架构。
- i386 / i686：适用于 32 位 Intel 和 AMD CPU。如果您使用的是 32 位系统，请选择这些版本。
- arm：适用于 32 位 ARM 架构。
- arm64：适用于 64 位 ARM 架构（也称为 AArch64）。

例如服务器架构为 x86_64 则需要找后缀为 amd64 或 x86_64 的文件，复制其链接

```sh
wget https://github.com/fatedier/frp/releases/download/v0.59.0/frp_0.59.0_linux_amd64.tar.gz
# 或者用下面命令下载（二选一）
curl -L -O https://github.com/fatedier/frp/releases/download/v0.59.0/frp_0.59.0_linux_amd64.tar.gz
```

也可以先下载到本地然后使用 scp 命令上传到服务器，例如：

```sh
# 本地终端 进入到对应目录
scp -P 10022 frp_0.59.0_linux_amd64.tar.gz root@服务器地址:/www/wwwroot/services
```

解压文件

```sh
tar -zxvf frp_0.59.0_linux_amd64.tar.gz
# 重命名
mv frp_0.59.0_linux_amd64 frp
# 删除不用的文件
rm frpc frpc.toml
```

编辑配置文件

```sh
vim frps.toml
```

内容如下

```toml
bindPort = 7000
vhostHTTPPort = 8080
```

服务器上安全组和防火墙开放用到的端口：7000 和 8080

启动服务

```sh
./frps -c ./frps.toml 
```

## mac 本地配置

下载相应的文件

```sh
wget https://github.com/fatedier/frp/releases/download/v0.59.0/frp_0.59.0_darwin_arm64.tar.gz
```

解压后编辑配置文件

```sh
tar -zxvf frp_0.59.0_darwin_arm64.tar.gz
# 重命名
mv frp_0.59.0_darwin_arm64 frp
# 编辑
cd frp
vim frpc.toml
```

内容如下

```toml
serverAddr = "服务器IP"
serverPort = 7000

[[proxies]]
name = "web"
type = "http"
localPort = 80 # 本地端口号
customDomains = ["www.yourdomain.com"] # 对应这个端口的域名

[[proxies]]
name = "web2"
type = "http"
localPort = 8888 # 本地端口号
customDomains = ["www.yourdomain2.com"]
```

这两个域名应当已经解析到服务器 IP

例如需要访问本地端口为 80 的网页就：`http://www.yourdomain.com:8080`

本地 8888 的网页：`http://www.yourdomain2.com:8080`

## 服务器端设置为自启动服务

安装 systemd

```sh
apt install systemd
```

**创建 frps.service 文件**

```sh
sudo vim /etc/systemd/system/frps.service
```

内容为

```ini
[Unit]
# 服务名称，可自定义
Description = frp server
After = network.target syslog.target
Wants = network.target

[Service]
Type = simple
# 启动frps的命令，需修改为您的frps的安装路径
ExecStart = /www/wwwroot/services/frp/frps -c /www/wwwroot/services/frp/frps.toml

[Install]
WantedBy = multi-user.target
```

使用 systemd 命令管理 frps 服务

```sh
# 启动frp
sudo systemctl start frps
# 停止frp
sudo systemctl stop frps
# 重启frp
sudo systemctl restart frps
# 查看frp状态
sudo systemctl status frps
```

设置 frps 开机自启动

```sh
sudo systemctl enable frps
```

## mac 端设置开机自启动

使用任意方式编辑 `~/Library/LaunchAgents/ci.nn.frpc.plist` 并添加如下内容

```plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
    <dict>
        <key>Label</key>
        <string>ci.nn.frpc</string>
        <key>KeepAlive</key>
        <true/>
        <key>ProcessType</key>
        <string>Background</string>
        <key>RunAtLoad</key>
        <true/>
        <key>WorkingDirectory</key>
        <string>/Users/luwang/Workspace/tools/frp</string>
        <key>ProgramArguments</key>
        <array>
            <string>/Users/luwang/Workspace/tools/frp/frpc</string>
            <string>-c</string>
            <string>/Users/luwang/Workspace/tools/frp/frpc.toml</string>
        </array>
    </dict>
</plist>
```

然后，执行 `launchctl load ~/Library/LaunchAgents/ci.nn.frpc.plist` 加载配置，现在你可以使用这些命令来管理程序：

- 开启: `launchctl start ~/Library/LaunchAgents/ci.nn.frpc.plist`
- 关闭: `launchctl stop ~/Library/LaunchAgents/ci.nn.frpc.plist`
- 卸载配置: `launchctl unload ~/Library/LaunchAgents/ci.nn.frpc.plist`

## 进阶配置

### 配置 https

使用 `https2http` 插件将本地 HTTP 服务转换为 HTTPS 服务，以供外部访问。

**配置 frps.toml**

```toml
bindPort = 7000
vhostHTTPSPort = 443
```

**配置 frpc.toml**

```toml
serverAddr = "服务器IP"
serverPort = 7000

[[proxies]]
name = "test_htts2http"
type = "https"
customDomains = ["test.yourdomain.com"]

[proxies.plugin]
type = "https2http"
localAddr = "127.0.0.1:80" # 本地端口修改成对应的

# HTTPS 证书相关的配置
crtPath = "./server.crt" # 把这两个文件放在本地（可以用 keymanager 生成 test.yourdomain.com）
keyPath = "./server.key"
hostHeaderRewrite = "127.0.0.1"
requestHeaders.set.x-from-where = "frp"
```

请注意，您需要根据您的域名和证书路径自行更改上述配置。

使用命令启动 frps 和 frpc

**访问 HTTPS 服务**

打开您的 Web 浏览器，访问 `https://test.yourdomain.com`。

通过按照以上步骤进行配置，您将能够为本地 HTTP 服务启用 HTTPS，以实现安全的外部访问。

### SSH 访问内网机器

编辑 frpc.toml 文件

```toml
serverAddr = "服务器IP"
serverPort = 7000

[[proxies]]
name = "ssh"
type = "tcp"
localIP = "127.0.0.1"
localPort = 22
remotePort = 6000
```

服务器需要开放 6000 端口

使用以下命令通过 SSH 访问内网机器，假设用户名为 test：

```sh
ssh -o Port=6000 test@x.x.x.x
```

frp 将请求发送到 `x.x.x.x:6000` 的流量转发到内网机器的 22 端口。
