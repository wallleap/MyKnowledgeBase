---
title: xui 与宝塔共存
date: 2023-11-15T03:36:00+08:00
updated: 2024-08-21T10:32:41+08:00
dg-publish: false
---

**这里是 trojan-docker 的**

[EXP-Tools/trojan-docker: docker 一键部署 trojan 服务端 ：自建 VPS 科学上网 (github.com)](https://github.com/EXP-Tools/trojan-docker)

教程：[trojan 睁眼看世界教程 | 眈眈探求 (exp-blog.com)](https://exp-blog.com/net/trojan-zheng-yan-kan-shi-jie-jiao-cheng/)

先在宿主机完成 docker、docker-compose、certbot 的安装，并按前述说明申请域名的 HTTPS 证书。

## Install Docker Compose

### Update Your System

Before you begin, update your system to check that all packages are up to date.

```bash
sudo apt update
```

### Get the Docker Compose repository

Use curl to download the [Docker](https://cloudinfrastructureservices.co.uk/how-to-install-ansible-awx-using-docker-compose-awx-container-20-04/) Compose repo to the /usr/local/bin/docker-compose directory. You may also get Docker Compose from its [official Github repository](https://github.com/docker/compose).

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/v2.0.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

Output:

```bash
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current

                               Dload  Upload   Total   Spent    Left  Speed

0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
```

### Modify File Permissions

After downloading Docker Compose, run the following command to give permission and make the downloaded file executable.

```bash
sudo chmod +x /usr/local/bin/docker-compose
```

### Check Docker Version

Next, check the docker-[compose](https://cloudinfrastructureservices.co.uk/how-to-setup-ruby-docker-container-using-docker-compose/) version to ensure that the installation was successful.

```bash
docker-compose --version
```

Copy

Output:

```bash
Docker Compose version v2.0.1
```

Copy

This result shows that Docker Compose was successfully installed.

**Also Read** 

[How To Install and Use Docker Compose on CentOS 8 Tutorial](https://cloudinfrastructureservices.co.uk/how-to-install-and-use-docker-compose-on-centos-8/)

安装 cerbot 以获取 SSL 证书

```sh
# 考虑到国内安装超时问题，已指定了安装源为清华源
python3 -m pip install certbot --default-timeout=600 -i https://pypi.tuna.tsinghua.edu.cn/simple
```

申请证书（替换 YOUR_DOMAIN 为域名）

```sh
# 第一次执行该命令，需要根据交互步骤先注册邮箱（注意保存好），以后不再需要 # 该命令会占用 80 端口，执行前注意要停止相关进程（如 nginx） 
/usr/local/bin/certbot certonly --standalone -d YOUR_DOMAIN -d www.YOUR_DOMAIN  
```

定时任务

```sh
# 编辑定时任务
crontab -e

# 每两个月更新一次证书
0 0 1 */2 0 /usr/local/bin/certbot renew
```

certbot 申请的证书存储在 `/etc/letsencrypt` 目录:

- 其他目录为 certbot 的注册账号信息
- `archive/YOUR_DOMAIN/` ： 存储 YOUR_DOMAIN 域名申请过的历史证书
- `live/YOUR_DOMAIN/` ： 存储 YOUR_DOMAIN 域名当前有效证书的链接文件

之后会用到的只有两个文件：

- `/etc/letsencrypt/live/YOUR_DOMAIN/fullchain.pem`
- `/etc/letsencrypt/live/YOUR_DOMAIN/privkey.pem`

```sh
# 下载项目源码 
git clone https://github.com/EXP-Tools/trojan-docker /usr/local/trojan-docker 
cd /usr/local/trojan-docker 
# 构建 docker 镜像 
# password 为之后客户端连接 trojan 的密码 
# domain 为前面准备好的域名 
password=YOUR_PASSWORD domain=YOUR_DOMAIN docker-compose build 
# 刷新证书有效期，并复制宿主机的 HTTPS 证书到 docker 容器 
# 建议把此脚本设置到 crontab 定时执行 
./renew_cert.sh 
# 在后台启动 trojan 服务 
docker-compose up -d
```

最后把 `/usr/local/trojan-docker/nginx/html` 下的内容替换为伪装站点的 HTML 内容即可。

**下面的是 x-ui 的**：

[可以与宝塔共存的一个“魔法”服务器状态监控应用——xui | 爱玩实验室 (iwanlab.com)](https://iwanlab.com/xui/)

之前有一期我们搭建了一个 [服务器监控](https://iwanlab.com/tag/%e6%9c%8d%e5%8a%a1%e5%99%a8%e7%9b%91%e6%8e%a7/ "服务器监控")，颜值非常不错，这期我们再来搭建一个“特殊”的服务器监控——XUI。

不仅可以监控服务器的数据，还可以干一些高级的，大家感兴趣的事情。

重要的是，它可以和 [宝塔面板](https://iwanlab.com/tag/%e5%ae%9d%e5%a1%94%e9%9d%a2%e6%9d%bf/ "宝塔面板") 共存！

## 普通安装&升级（推荐）

```bash
bash <(curl -Ls https://raw.githubusercontent.com/vaxilu/x-ui/master/install.sh)
```

设置好帐号、密码、端口，接着在宝塔面板和服务器开启该端口。

> 面板所有数据包括账号信息等都存在 /etc/x-ui/x-ui.db 中，只要备份此文件即可。在新服务器安装了面板之后，先关闭面板，再将备份的文件覆盖新安装的，最后启动面板即可。

## 面板设置

记得改默认的账号密码

接着点击入站列表

![](https://cdn.wallleap.cn/img/pic/illustration/202311151544917.png)

如图填写

![](https://cdn.wallleap.cn/img/pic/illustration/202311151545864.png)

这里需要注意监听 IP、额外 id、传输配置为 ws、tls 关掉

## 宝塔设置

点击随便一个网站，例如 `blog.oicode.cn` 然后点击配置文件，找个位置放下面的代码

![](https://cdn.wallleap.cn/img/pic/illustration/202311151548964.png)

```nginx
location /plogger { 
  proxy_pass http://127.0.0.1:35714;
  proxy_redirect off;
  proxy_http_version 1.1;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection "upgrade";
  proxy_set_header Host $http_host;
  proxy_read_timeout 300s;
}
```

修改两个地方 `/plogger` → 之前填的路径、`35714` → 前面的端口号

## 下载客户端使用

下载相应的客户端

复制或扫码

![](https://cdn.wallleap.cn/img/pic/illustration/202311151553042.png)

把地址栏显示的 `IP地址` 改为绑定的域名，端口改为 `443`，传输方式 websocket，里面 Host 填写绑定的域名，`ws路径` 改为你前面设置的路径，打开 `tls`，点击完成即可。

## 其他

```sh
x-ui 管理脚本使用方法: 
----------------------------------------------
x-ui              - 显示管理菜单 (功能更多)
x-ui start        - 启动 x-ui 面板
x-ui stop         - 停止 x-ui 面板
x-ui restart      - 重启 x-ui 面板
x-ui status       - 查看 x-ui 状态
x-ui enable       - 设置 x-ui 开机自启
x-ui disable      - 取消 x-ui 开机自启
x-ui log          - 查看 x-ui 日志
x-ui v2-ui        - 迁移本机器的 v2-ui 账号数据至 x-ui
x-ui update       - 更新 x-ui 面板
x-ui install      - 安装 x-ui 面板
x-ui uninstall    - 卸载 x-ui 面板
----------------------------------------------
```

### 忘记用户名和密码

使用以下命令重置用户名和密码，默认都为 admin

```sh
/usr/local/x-ui/x-ui resetuser
```

### 面板设置修改错误导致面板无法启动

使用以下命令重置所有面板设置，默认面板端口修改为 54321，其它的也会重置为默认值，注意，这个命令不会重置用户名和密码。

```sh
/usr/local/x-ui/x-ui resetconfig
```

### 因时间误差导致 为麦斯 无法连接

引用 为途锐 官方的一句话：为麦斯 依赖于系统时间，请确保使用 为途锐 的系统 UTC 时间误差在 90 秒之内，***时区无关***。在 Linux 系统中可以安装 ntp 服务来自动同步系统时间。

### 卸载面板

执行以下命令即可完全卸载面板，如果还需要卸载 为途锐，请自行找相关教程。

```sh
systemctl stop x-ui
systemctl disable x-ui
rm /usr/local/x-ui/ -rf
rm /etc/x-ui/ -rf
rm /etc/systemd/system/x-ui.service -f
systemctl daemon-reload
```
