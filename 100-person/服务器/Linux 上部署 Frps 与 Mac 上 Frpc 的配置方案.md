---
title: Linux 上部署 Frps 与 Mac 上 Frpc 的配置方案
date: 2024-07-31 09:15
updated: 2024-07-31 09:16
---

远程办公在过去几年里经历了巨大的发展。随着技术的进步和全球互联网的普及，越来越多的公司开始意识到远程工作的潜力和优势。视频会议、实时协作工具、云计算和即时通讯工具的发展，使得远程办公变得更加高效和便捷。这些工具让团队可以随时随地共享信息和合作，无论他们身处何地。

**虽然方便但是没有公网 ip、并不能很好的使用，公网与内网的连接就成了刚性需求。**

> NAT（Network Address Translation，网络地址转换）穿透对远程办公非常重要。NAT 是一种网络技术，允许多个设备共享单个公共 IP 地址，在互联网上发送和接收数据。然而，在远程办公环境中，NAT 可能导致一些问题，而 NAT 穿透可以解决这些问题。

Nat 穿透有多种实现方式，比如虚拟局域网、Frp 等工具，今天我们就比较热门的 frp 进行分享。

FRP（Fast Reverse Proxy）是一个用于进行内网穿透的开源工具，它可以帮助将本地服务暴露到公共网络，实现远程访问私有网络中的服务。

> .FRP 的工作方式基于客户端 - 服务器模型。在内网中部署 FRP 客户端，同时在公网中部署 FRP 服务器，客户端和服务器之间建立连接。
>
> .客户端负责将本地服务的请求发送到服务器，服务器收到请求后将流量转发给客户端中的目标服务，从而实现内外网之间的数据通信。

下面我们就进入在 Linux 服务端部署流程：

**在Linux上部署Frps服务端**：

**需要的设备**：一台具有公网ip的服务器、一台或者多台无公网的mac、Win电脑。这里是具有公网ip的linux服务器配置

1、下载适合当前版本的Frps。（可以从[github下载frps](https://github.com/fatedier/frp/releases)对应的版本）比如我的系统是Centos7对应下载：frp_0.53.0_linux_amd64

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8|# 前往 FRP 的官方 GitHub releases 页面下载适用于 Linux 的版本，例如 v0.37.1<br><br>wget https://github.com/fatedier/frp/releases/download/v0.37.1/frp_0.37.1_linux_amd64.tar.gz<br><br># 解压下载的文件<br><br>tar -xvf frp_0.37.1_linux_amd64.tar.gz<br><br># 进入解压后的目录<br><br>cd frp_0.37.1_linux_amd64|

> **小提示**：如果使用了Linux管理面板可以直接通过电脑下载文件，并上传到方便管理的目录即可。上方的操作做的也是这个操作，可跳过上方的操作，选择适合你自己的方法来进行下载、上传。
> 
> 比如我将Frps与 Frps.toml文件拷贝到方的目录：/www/wwwroot/net/frps/

**2. 配置 FRP 服务器**  
创建 FRP 配置文件 frps.atoml，用于配置 FRP 服务器。

|   |   |
|---|---|
|1|nano frps.atoml|

在编辑器中添加以下配置，根据需求进行修改：

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13|bindPort = 1443<br><br>#用于fprs和frp进行连接验证通讯的端口，可自行设定，需要在防火墙中开启此端口<br><br># 如果指定了“oidc”，将使用 OIDC 设置颁发 OIDC（开放 ID 连接）令牌。默认情况下，此值为“令牌”。auth.method = “token”<br><br>auth.method = "token"<br><br># 身份验证令牌 auth.token = “密码”<br><br>auth.token = "123456"<br><br># 配置 Web 服务器以启用 frps 的仪表板。<br><br>webServer.addr = "0.0.0.0"<br><br>webServer.port = 16443<br><br>#用于fprs显示连接状态流量的端口，可自行设定，需要在防火墙中开启此端口<br><br>webServer.user = "admin"<br><br>webServer.password = "123456"|

3. 启动 FRPS 服务器。（这里我常用的三种启动方式都写在下方，每行一个都有解释）

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11|#1、进入安装目录后 带terminal实时输出启动，测试配置时非常有用<br><br>cd /www/wwwroot/net/frps<br><br>./frps -c ./frps.toml<br><br># 2、进入目录后执行、不带terminal实时输出启动，可关闭terminao<br><br>cd /www/wwwroot/net/frps<br><br>./frps -c ./frps.toml>/dev/null 2>&1 &<br><br>#3、将目录位置写入命令，直接启动<br><br>/www/wwwroot/net/frps/./frps -c /www/wwwroot/net/frps/frps.toml|

启动FRP服务器后，它就开始监听来自客户端的连接。

4、配置开机自动启动，这样每次开机frps就会自动启动

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14|[Unit]<br><br># 服务名称，可自定义<br><br>#/etc/systemd/system<br><br>Description = frp server<br><br>After = network.target syslog.target<br><br>Wants = network.target<br><br>[Service]<br><br>Type = simple<br><br># 启动frps的命令，需修改为您的frps的安装路径<br><br>ExecStart = /www/wwwroot/net/frps/./frps -c /www/wwwroot/net/frps/frps.toml<br><br>[Install]<br><br>WantedBy = multi-user.target|

5、这一步一般情况下可以不做，但如果要让进程始终处于运行状态，可以安装Supervisor进行配置，也可以使用宝塔面板的进程管理器，配置文件如下：

|                                                                                                                                                 |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| ----------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14<br><br>15<br><br>16 | [program:FRPS]<br><br># 启动frps的命令<br><br>command=/www/wwwroot/net/frps/./frps -c /www/wwwroot/net/frps/frps.toml<br><br># 启动frps的的frps的安装路径<br><br>directory=/www/wwwroot/net/frps/<br><br>autorestart=true<br><br>startsecs=3<br><br>startretries=3<br><br>stdout_logfile=/www/server/panel/plugin/supervisor/log/FRPS.out.log<br><br>stderr_logfile=/www/server/panel/plugin/supervisor/log/FRPS.err.log<br><br>stdout_logfile_maxbytes=2MB<br><br>stderr_logfile_maxbytes=2MB<br><br>user=root<br><br>priority=999<br><br>numprocs=1<br><br>process_name=%(program_name)s_%(process_num)02d |


**在Mac中部署frpc客户端并设置开机启动/进程守护**

1、下载Mac系统版本客户端：frp_0.53.0_darwin_arm64.tar.gz，如果是M芯片的选择darwin arm版，intel芯片的选择：darwin_amd64：

---

2、解压文件后得到四个文件，我们仅留下：frpc、frpc.toml俩个文件。

---

**3、配置frpc.toml,连接客户端并且分配远程与本地的端口映射**：

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14<br><br>15<br><br>16<br><br>17<br><br>18<br><br>19<br><br>20<br><br>21<br><br>22<br><br>23<br><br>24<br><br>25<br><br>26<br><br>27<br><br>28<br><br>29<br><br>30<br><br>31<br><br>32<br><br>33<br><br>34<br><br>35<br><br>36<br><br>37<br><br>38<br><br>39<br><br>40<br><br>41<br><br>42<br><br>43<br><br>44<br><br>45<br><br>46<br><br>47<br><br>48<br><br>49|serverAddr = "写入上文服务端的公网ip"<br><br>serverPort = 1443<br><br># 如果指定了“oidc”，将使用 OIDC 设置颁发 OIDC（开放 ID 连接）令牌。默认情况下，此值为“令牌”。auth.method = “token”<br><br>auth.method = "token"<br><br># 身份验证令牌 auth.token = “密码自行设置”<br><br>auth.token = "123456"<br><br>#开放本地ssh22端口，对应远程的122端口<br><br>#[[proxies]]<br><br>name = "ssh"<br><br>type = "tcp"<br><br>localIP = "127.0.0.1"<br><br>localPort = 22<br><br>remotePort = 122<br><br># frpc.toml<br><br>[[proxies]]<br><br>name = "xyz"<br><br>type = "tcp"<br><br>localIP = "127.0.0.1"<br><br>localPort = 443<br><br>remotePort = 1443<br><br>#服务端与此域名通讯的端口<br><br>customDomains = ["xyz.newvfx.com"]<br><br>#定义本地服务的域名1<br><br>transport.useEncryption = true<br><br>#启用加密传输<br><br>transport.useCompression = true<br><br>#启用压缩传输<br><br># 目前支持 v1 和 v2 两个版本的 proxy protocol 协议。<br><br>#transport.proxyProtocolVersion = "v2"<br><br># frpc.toml<br><br>[[proxies]]<br><br>name = "xyz2"<br><br>type = "tcp"<br><br>localIP = "127.0.0.1"<br><br>localPort = 443<br><br>remotePort = 2443<br><br>#服务端与此域名通讯的端口<br><br>customDomains = ["xyz2.newvfx.com"]<br><br>#定义本地服务的域名2<br><br>transport.useEncryption = true<br><br>#启用加密传输<br><br>transport.useCompression = true<br><br>#启用压缩传输<br><br>#transport.proxyProtocolVersion = "v2"|

3、配置好文件后，启动本地客户端进行连接测试，在Terminal（终端）中进入目录，并执行启动命令：

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8|#进入客户端所在目录，可贫喜好自行修改<br><br>cd /Applications/frp<br><br>#执行启动命令<br><br>./frpc -c ./frpc.toml<br><br>#执行启动命令后不输出信息，可关闭Terminal<br><br>nohup ./frpc -c frpc.toml >/dev/null 2>&1 &|

上方两条可以通过shortcuts，或者新建为一个frpc.command的脚本。

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4|#!/bin/bash<br><br>clear<br><br>cd /Applications/frp<br><br>./frpc -c frpc.toml|

---

**4、设置开机Mac开机启动与进程守护**

_Mac OS各目录决定了其启动的先后和拥有的权限_：

|   |   |
|---|---|
|1<br><br>2<br><br>3|~/Library/LaunchAgents # 以当前设置的用户登录后启动。<br><br>/Library/LaunchAgents # 系统组件所有用户登录后都以当前用户启动<br><br>/Library/LaunchDaemons # 系统启动时以root用户启动，无需登陆。|

4、1写入到用户启动项目，开机后输出密码登陆用户后启动：

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14<br><br>15<br><br>16<br><br>17<br><br>18|echo '<?xml version="1.0" encoding="UTF-8"?><br><br><!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd"><br><br><plist version="1.0"><br><br><dict><br><br>    <key>Label</key><br><br>    <string>com.jisongbin.frp</string><br><br>    <key>ProgramArguments</key><br><br>    <array><br><br>        <string>/Applications/frp/frpc</string><br><br>        <string>-c</string><br><br>        <string>/Applications/frp/frpc.toml</string><br><br>    </array><br><br>    <key>RunAtLoad</key><br><br>    <true></true><br><br>    <key>KeepAlive</key><br><br>    <true></true><br><br></dict><br><br></plist>' > ~/Library/LaunchAgents/com.jisongbin.frp.plist|

4、2、写入到系统启动项目：开机后即启动，不需要输入用户输入密码
```
sudo sh -c 'echo "<?xml version=\"1.0\" encoding=\"UTF-8\"?>

<!DOCTYPE plist PUBLIC \"-//Apple//DTD PLIST 1.0//EN\" \"http://www.apple.com/DTDs/PropertyList-1.0.dtd\">

<plist version=\"1.0\">

<dict>

    <key>Label</key>

    <string>com.jisongbin.frp</string>

    <key>ProgramArguments</key>

    <array>

        <string>/Applications/frp/frpc</string>

        <string>-c</string>

        <string>/Applications/frp/frpc.toml</string>

    </array>

    <key>RunAtLoad</key>

    <true></true>

    <key>KeepAlive</key>

    <true></true>

</dict>

</plist>" > /Library/LaunchDaemons/com.jisongbin.frp.plist'
```

检查服务进程是否启动：

|   |   |
|---|---|
|1|ps -e \|grep frpc|

**小提示**：如果进入开机自动启动后，kill掉进程会自动重新读取frpc.atoml,可以使用命令关掉进程，让其自动启动

|     |              |
| --- | ------------ |
| 1   | killall frpc |
**Frps部署ssl通讯方式后去掉端口号的方法与转发速度的提升**

1、tcp类型通讯协议的转发：

需要在服务端、即有公网的服务器上配置Nginx反向代理，将域名或者ip转发到带端口的网络接口，这样就不需要输入端口号才能访问。这里我全程使用了ssl安全链接，所以转发到了https的端口上。

> 转发目标：https://127.0.0.1:1443 转发host：¥host

完整写法如下：

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14<br><br>15<br><br>16<br><br>17<br><br>18<br><br>19<br><br>20<br><br>21<br><br>22<br><br>23<br><br>24<br><br>25<br><br>26<br><br>27<br><br>28<br><br>29|#PROXY-START/<br><br>location ^~ /<br><br>{<br><br>    proxy_pass https://127.0.0.1:1443;<br><br>    proxy_set_header Host $http_host;<br><br>    proxy_set_header X-Real-IP $remote_addr;<br><br>    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;<br><br>    proxy_set_header REMOTE-HOST $remote_addr;<br><br>    proxy_set_header Upgrade $http_upgrade;<br><br>    proxy_set_header Connection $connection_upgrade;<br><br>    proxy_http_version 1.1;<br><br>    # proxy_hide_header Upgrade;<br><br>    add_header X-Cache $upstream_cache_status;<br><br>    #Set Nginx Cache<br><br>    set $static_filexVoJAVYp 0;<br><br>    if ( $uri ~* "\.(gif\|png\|jpg\|css\|js\|woff\|woff2)$" )<br><br>    {<br><br>        set $static_filexVoJAVYp 1;<br><br>        expires 1m;<br><br>    }<br><br>    if ( $static_filexVoJAVYp = 0 )<br><br>    {<br><br>        add_header Cache-Control no-cache;<br><br>    }<br><br>}<br><br>#PROXY-END/|

_经验记录_：  
1、如果带端口访问，ssl只需要在本地服务器 或者转发服务器添加ssl证书即可

2、如果要去掉端口，需要在转发服务器上使用反向代理，将域名转发到转发服务器的ssl模式转发端口

访问此域名则转发到服务器的：1443端口，1443端口又转发到本地服务器端口：443

**在这个模式下，所有链接都被切换到https模式，需要在转发服务器和 本地服务器都部署当前域名的证书。转发为ssl时需要证书验证一次，到达本地服务器时候还需要本地服务器有证书。俩个证书可以不是同一个厂家的，但必须是同域名的。**

**2、http、https方式的Nginx反向代理转发去端口**

_2、1vhost_https_port方式Nginx反向代理配置_

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14<br><br>15<br><br>16<br><br>17<br><br>18<br><br>19<br><br>20<br><br>21<br><br>22<br><br>23<br><br>24<br><br>25<br><br>26<br><br>27<br><br>28<br><br>29<br><br>30<br><br>31<br><br>32<br><br>33<br><br>34<br><br>35<br><br>36<br><br>37<br><br>38<br><br>39|server<br><br>{<br><br>    listen 80;<br><br>listen 443 ssl http2;<br><br>    server_name test.newvfx.com;<br><br>    index index.php index.html index.htm default.php default.htm default.html;<br><br>    #SSL-START SSL相关配置，请勿删除或修改下一行带注释的404规则<br><br>    #error_page 404/404.html;<br><br>    ssl_certificate    /www/server/panel/vhost/cert/test.newvfx.com/fullchain.pem;<br><br>    ssl_certificate_key    /www/server/panel/vhost/cert/test.newvfx.com/privkey.pem;<br><br>    ssl_protocols TLSv1.1 TLSv1.2 TLSv1.3;<br><br>    ssl_ciphers EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:!MD5;<br><br>    ssl_prefer_server_ciphers on;<br><br>    ssl_session_cache shared:SSL:10m;<br><br>    ssl_session_timeout 10m;<br><br>    add_header Strict-Transport-Security "max-age=31536000";<br><br>    error_page 497  https://$host$request_uri;<br><br>    #SSL-END<br><br>#Jisongbin反向代理配置<br><br>location / {<br><br>                proxy_redirect off;<br><br>                proxy_set_header Host $host;<br><br>                proxy_ssl_server_name on;<br><br>                proxy_set_header X-Real-IP $remote_addr;<br><br>                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;<br><br>                proxy_set_header X-Forwarded-Proto $scheme;<br><br>                #htts://https必须是域名不能用ip:[vhost_https_port]<br><br>                proxy_pass https://test.newvfx.com:1443;<br><br>                proxy_ssl_session_reuse on;<br><br>                proxy_set_header Upgrade $http_upgrade;<br><br>                proxy_set_header Connection $connection_upgrade;<br><br>                proxy_http_version 1.1;                <br><br>        }<br><br>#Jisongbin反向代理配置结束<br><br>}|

_2、2、http方式Nginx反向代理配置： vhost_http_port_

|                                                                                                         |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| ------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12 | server{<br><br>        listen 80;<br><br>        server_name *.example.com;<br><br>        location / {<br><br>            //服务器的ip和端口http://[ip]:[port]   vhost_http_port<br><br>                proxy_pass http://your_ip:8080;<br><br>                proxy_set_header Host $host;<br><br>                proxy_set_header X-Real-IP $remote_addr;<br><br>                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;<br><br>                proxy_hide_header X-Powered-By;<br><br>        }<br><br>        access_log off; |

补充：**Nginx反向代理实现实时上传下载**

**Nginx反向代理默认采用**：用户将文件上传《—-》到Nginx反向代理服务器《—-》转发到后端服务器，并以网速最低的一端作为总传输速度，并在每一段传输中，以当前段的最大带宽作为区域传输速度。

[![](https://img.newvfx.com/j/2023/12/nginx-750x282.png)](https://img.newvfx.com/j/2023/12/nginx.png)

在小文件上传下载时候，这个过程非常快，所以给人造成一种实时的错觉。当进行大文件（视频、软件、音频）下载的时候，就会发现效率很低。

> **具体表现**：本地客户端输出流量正常，服务端输入流量正常，但从服务端到用户的速度很低、通常只有几十kb，这是由于反向代理过程中，用户需要将文件完全上传到服务端后，服务端才从服务端转发到后端，如果是大一点的视频、音频会造成超时无法播放的问题。

---

**用户下载、播放时的方式：**  
在nginx反向代理中关闭向用户传输文件的缓存（即Nginx出方向的缓存）添加：

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7|添加禁用缓存，不将转发数据存储到硬盘<br><br>proxy_buffering off;<br><br>proxy_cache off;<br><br># proxy_hide_header Upgrade;<br><br>注释掉以下行 不添加缓存头<br><br># add_header X-Cache $upstream_cache_status;|

**用户进行上传**  
如果站点是允许用户进行大文件（视频、软件、音频）上传即（Nginx入方向）的，同样会存在上述问题，我们可以通过关闭传入buffering来实现直接上传下载的目的，并设定客户端上传文件大小限制：

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6|#Jisongbin直接上传不缓存<br><br>  proxy_request_buffering off;<br><br>  # jisongbin代理文件上客户端最大请求体大小<br><br>  client_max_body_size 10G;<br><br>#add_header X-Cache $upstream_cache_status;|

---

以上是在部署与应用过程中的记录，如果你在部署中遇到问题可以跟帖留言，看到就回复啦。

