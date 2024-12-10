---
title: 10分钟搭建一个自己的短链接服务 YOURLS
date: 2024-11-13T14:21:15+08:00
updated: 2024-11-13T14:29:44+08:00
dg-publish: false
source: https://blog.laoda.de/archives/docker-compose-install-yourls
---

## 1.介绍

YOURLS 是基于 PHP 的，一个可以让你在自己的服务器上运行的 URL 缩短服务。（已经有近 10 年的历史了！）

利用它，我们可以完全控制自己的数据，其中包括详细的统计、分析、还可以安装一些插件。

免费！开源！

见 [【服务器能干什么】搭建一个短网址平台，可以查看数据详情！](https://blog.laoda.de/archives/yourls-building)

## 2.项目展示

见 [【服务器能干什么】搭建一个短网址平台，可以查看数据详情！](https://blog.laoda.de/archives/yourls-building)

## 3.搭建环境

- 服务器：~腾讯香港轻量应用服务器 24 元/月 VPS 一台~展示用的服务器是 [Netcup](https://netcup-sonderangebote.de/) 特价款，本期搭建用的是 [Vultr](https://loll.cc/vultr) 的服务器，按小时计费，可随时销毁（最好是选**非大陆的服务器**）（[腾讯轻量购买链接](https://loll.cc/tx)）[Hetzner注册免费得25欧试用金有效期一个月](https://loll.cc/hz)
- 系统：Debian 10（[DD脚本](https://blog.laoda.de/archives/useful-script#dd%E7%9B%B8%E5%85%B3) 非必需 DD 用原来的系统也 OK）
- 域名一枚，并做好解析到服务器上（[域名购买、域名解析](https://blog.laoda.de/archives/namesilo) [视频教程](https://www.bilibili.com/video/BV1Sy4y1k7kZ/)）
- 安装好 Docker、Docker-compose（[相关脚本](https://blog.laoda.de/archives/hello-docker#5%E5%AE%89%E8%A3%85dockerdocker-compose)）
- 【非必需】提前安装好宝塔面板海外版本 aapanel，并安装好 Nginx（[安装地址](https://forum.aapanel.com/d/9-aapanel-linux-panel-6812-installation-tutorial)）
- 【非必需本教程采用】安装好 Nginx Proxy Manager（[相关教程](https://blog.laoda.de/archives/nginxproxymanager)）

## 4.搭建视频

YouTube：[https://youtu.be/pz3XZG\_QZ-U](https://youtu.be/pz3XZG_QZ-U)

哔哩哔哩【高清版本可以点击去吐槽到 B 站观看】：

## 5.搭建方式

### 5.1 搭建

服务器初始设置，参考

[**新买了一台服务器“必须”要做的6件小事**](https://blog.laoda.de/archives/vps-script)

[【Docker系列】不用宝塔面板，小白一样可以玩转VPS服务器！](https://blog.laoda.de/archives/hello-docker)

```sh
sudo -i # 切换到root用户

apt update -y  # 升级packages

apt install wget curl sudo vim git  # Debian系统比较干净，安装常用的软件
```

创建一下安装的目录：

```sh
mkdir -p /root/data/docker_data/yourls

cd /root/data/docker_data/yourls

nano docker-compose.yml
```

`docker-compose.yml` 中的镜像来源 [官方仓库](https://hub.docker.com/_/yourls?tab=description)，内容如下：

```yml
version: "3.5"
services:

  mysql:
    image: mysql:5.7.22              # 如果遇到不正确的数据库配置，或无法连接到数据库PDOException: SQLSTATE[HY000] [1045] 用户'yourls'@'yourls_service.yourls_default'的访问被拒绝（使用密码：是）   可以把5.7.22 改为 5.7
    environment:
      - MYSQL_ROOT_PASSWORD=my-secret-pw
      - MYSQL_DATABASE=yourls
      - MYSQL_USER=yourls
      - MYSQL_PASSWORD=yourls
    volumes:
      - ./mysql/db/:/var/lib/mysql
      - ./mysql/conf/:/etc/mysql/conf.d
    restart: always
    container_name: mysql
  
  yourls:
    image: yourls
    restart: always
    ports:
      - "8200:80"
    environment:
      YOURLS_DB_HOST: mysql
      YOURLS_DB_USER: yourls
      YOURLS_DB_PASS: yourls
      YOURLS_DB_NAME: yourls
      YOURLS_USER: admin      # 自己起一个名字
      YOURLS_PASS: admin      # 自己换一个登陆密码
      YOURLS_SITE: https://gao.ee  # 换成你自己的域名
      YOURLS_HOURS_OFFSET: 8
    volumes:
      - ./yourls_data/:/var/www/html   
    container_name: yourls_service
    links:
      - mysql:mysql
```

> 注意：如果 VPS 的内存比较小 ，推荐设置一下 SWAP，一般为内存的 1-1.5 倍即可～

设置 SWAP 可以用脚本:

```sh
wget -O box.sh https://raw.githubusercontent.com/BlueSkyXN/SKY-BOX/main/box.sh && chmod +x box.sh && clear && ./box.sh
```

没问题的话，`ctrl+x` 退出，按 `y` 保存，`enter` 确认。

查看端口是否被占用，输入：

```sh
lsof -i:8200  #查看8200端口是否被占用，如果被占用，重新自定义一个端口
```

如果出现：

```
-bash: lsof: command not found
```

运行：

```sh
apt install lsof  #安装lsof
```

如果端口没有被占用，可以运行：

```sh
docker-compose up -d 
```

访问：`http:服务ip:8200` 即可。

> **注意：**
>
> 1、不知道服务器 IP，可以直接在命令行输入：`curl ip.sb`，会显示当前服务器的 IP。
>
> 2、遇到访问不了的情况，请在宝塔面板的防火墙和服务商的后台防火墙里打开对应端口。

### 5.2 更新

```sh
cd /root/data/docker_data/yourls  # 进入docker-compose所在的文件夹
docker-compose pull    # 拉取最新的镜像
docker-compose up -d   # 重新更新当前镜像
```

利用 Docker-compose 搭建的应用，更新非常容易～

### 5.3 卸载

```sh
sudo -i  # 切换到root
cd /root/data/docker_data/yourls  # 进入docker-compose所在的文件夹
docker-compose down    # 停止容器，此时不会删除映射到本地的数据
cd ~
rm -rf /root/data/docker_data/yourls  # 完全删除映射到本地的数据
```

利用 Docker-compose 搭建的应用，删除也非常容易～

## 6.反向代理（必须）

### 6.1 利用 Nginx Proxy Manager

在添加反向代理之前，确保你已经完成了域名解析，不会的可以看这个：**域名一枚，并做好解析到服务器上**（[域名购买、域名解析](https://blog.laoda.de/archives/namesilo) [视频教程](https://www.bilibili.com/video/BV1Sy4y1k7kZ/)）

> 可以直接搜 nginx 反向代理，下面的内容可以不看

之后，登陆 Nginx Proxy Manager（不会的看这个：**安装 Nginx Proxy Manager**（[相关教程](https://blog.laoda.de/archives/nginxproxymanager)））

> **注意：**
>
> Nginx Proxy Manager（以下简称 NPM）会用到 `80`、`443` 端口，所以本机不能占用（比如原来就有 Nginx）

直接丢几张图：

![](https://cdn.wallleap.cn/img/pic/illustration/202411131429283.webp?imageSlim)

![](https://cdn.wallleap.cn/img/pic/illustration/202411131429284.webp?imageSlim)

![](https://cdn.wallleap.cn/img/pic/illustration/202411131429285.webp?imageSlim)

> 注意填写对应的 `域名`、`IP` 和 `端口`，按文章来的话，应该是 `8200`

**IP 填写：**

如果 Nginx Proxy Manager 和 reader 在同一台服务器上，可以在终端输入：

```sh
ip addr show docker0
```

查看对应的 Docker 容器内部 IP。

否则直接填 `docker所在的服务器IP` 就行。

![](https://cdn.wallleap.cn/img/pic/illustration/202411131429286.webp?imageSlim)

完成之后，记得再次打开这个，把 Force SSL 再勾选上（小 BUG）

然后就可以用域名 +`/admin`（即 `https://你的域名/admin`）来安装访问了。

![](https://cdn.wallleap.cn/img/pic/illustration/202411131429287.webp?imageSlim)

![](https://cdn.wallleap.cn/img/pic/illustration/202411131429288.webp?imageSlim)

![](https://cdn.wallleap.cn/img/pic/illustration/202411131429289.webp?imageSlim)

![](https://cdn.wallleap.cn/img/pic/illustration/202411131429290.webp?imageSlim)

## 7.使用教程

看咕咕的视频。

### 7.1 下载中文语言包

下载地址：`https://github.com/ZvonimirSun/YOURLS-zh_CN/archive/refs/tags/v1.7.3.zip`

```sh
wget https://github.com/ZvonimirSun/YOURLS-zh_CN/archive/refs/tags/v1.7.3.zip
apt install zip -y
unzip v1.7.3.zip
```

需要下载并解压到 `/root/data/docker_data/yourls/yourls_data/user/languages` 目录，

![](https://cdn.wallleap.cn/img/pic/illustration/202411131429291.webp?imageSlim)

```sh
chown -R www-data:www-data zh_CN.mo  # 修改文件拥有者和组
chown -R www-data:www-data zh_CN.po  # 修改文件拥有者和组
```

然后修改 `/root/data/docker_data/yourls/yourls_data/user/config.php`。

![](https://cdn.wallleap.cn/img/pic/illustration/202411131429292.webp?imageSlim)

如果你有宝塔，可以直接登陆宝塔：

![](https://cdn.wallleap.cn/img/pic/illustration/202411131429293.webp?imageSlim)

```sh
chown -R www-data:www-data zh_CN.mo
chown -R www-data:www-data zh_CN.po
```

![](https://cdn.wallleap.cn/img/pic/illustration/202411131429294.webp?imageSlim)

### 7.2 激活插件

![](https://cdn.wallleap.cn/img/pic/illustration/202411131429295.webp?imageSlim)

### 7.3 同一条链接对应多个短链接

同样是在刚刚修改语言的地方 `/root/data/docker_data/yourls/yourls_data/user/config.php`
下面一行，把这个改成 `true` 就可以了~ （正常应该改成 `false` 才对的，感觉是个 BUG = =）

![](https://cdn.wallleap.cn/img/pic/illustration/202411131429296.webp?imageSlim)

记得重启容器：

```sh
cd /root/data/docker_data/yourls

docker-compose restart
```

实测成功！

![](https://cdn.wallleap.cn/img/pic/illustration/202411131429297.webp?imageSlim)

### 7.5 换个主题试试 （失败了）

yourls 虽然功能强大，但界面比较复古，好在可以 [更换主题](https://github.com/YOURLS/awesome-yourls#themes-)，当然，你也可以自己开发。这里以 [Sleeky主题](https://github.com/Flynntes/Sleeky) 为例。

![](https://cdn.wallleap.cn/img/pic/illustration/202411131429298.webp?imageSlim)

Sleeky 主题包括前端和后台两个部分。下载解压 Sleeky 主题后可以看到两个文件夹 `sleeky-frontend` 和 `sleeky-backend` ，前端只需要将 `sleeky-frontend` 中的文件复制到 yourls 网站根目录即可，后端则需要将 `sleeky-backend` 文件夹放到 yourls 目录下的 `user/plugins` 中，然后在后台管理 (`yourdomain.com/admin/plugins.php`) 中启动主题插件即可看到效果。

同时也要注意权限问题：

```sh
chown -R www-data:www-data sleeky-frontend

chown -R www-data:www-data sleeky-backend
```

你可以选择只安装前端或者只安装后端主题，如果你的前端主题没有 css 样式的话，可能是因为你的网站开启了 https，只需修改一下前面的 `config.php` 配置文件：

```sh
chown -R www-data:www-data sleeky-frontend

chown -R www-data:www-data sleeky-backend
```

将你的网站设置为 `https://your-own-domain-here.com` 即可。

记得重启一下容器：

```sh
cd /root/data/docker_data/yourls

docker-compose restart
```

失败：

![](https://cdn.wallleap.cn/img/pic/illustration/202411131429299.webp?imageSlim)

![](https://cdn.wallleap.cn/img/pic/illustration/202411131429300.webp?imageSlim)

有成功的小伙伴欢迎评论区反馈！

## 8.结尾

祝大家用得开心，有问题可以去 GitHub 提 [Issues](https://github.com/YOURLS/YOURLS)，也可以在评论区互相交流探讨。

同时，有能力给项目做贡献的同学，也欢迎积极 [加入到项目](https://github.com/YOURLS/YOURLS) 中来，贡献自己的一份力量！

## 9.参考资料

[http://yourls.org/](http://yourls.org/)

[https://docs.yourls.org/](https://docs.yourls.org/)

[https://www.hellosanta.com.tw/knowledge/category-38/post-219](https://www.hellosanta.com.tw/knowledge/category-38/post-219)

[https://github.com/YOURLS/YOURLS](https://github.com/YOURLS/YOURLS)

[https://hub.docker.com/\_/yourls?tab=description](https://hub.docker.com/_/yourls?tab=description)
