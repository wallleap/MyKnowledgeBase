---
title: 国光 Kali Linux 配置记录
date: 2024-08-26T09:20:01+08:00
updated: 2024-08-26T09:23:10+08:00
dg-publish: false
source: https://www.sqlsec.com/2019/10/kali.html
---

该文章写于 446 天前，内容可能已经没有参考价值了，请自行判断。

Kali Linux 白帽子必备操作系统，有时候重新安装又得折腾下，特此这里总结一下常见的命令，日后再配置的话就可以减少一些不必要的坑了，而且还可以帮助到一些网友，何乐而不为呢。

## 更新源

```bash
vim /etc/apt/sources.list
```

## 官方源

记住这个官方源就好了，不要被网上辣鸡文章误导，官方源会**自动选择最近的源**服务器更新的，简单明了。

```bash
deb http://http.kali.org/kali kali-rolling main non-free contrib deb-src http://http.kali.org/kali kali-rolling main non-free contrib
```

~如果你很无聊的话 可以手动尝试如下更新源~：不对，官方源有时候会抽风，自动选择的源速度慢的一匹。

## 中科大

```bash
deb http://mirrors.ustc.edu.cn/kali kali-rolling main non-free contrib deb-src http://mirrors.ustc.edu.cn/kali kali-rolling main non-free contrib
```

## 浙大

```bash
deb http://mirrors.zju.edu.cn/kali kali-rolling main contrib non-free deb-src http://mirrors.zju.edu.cn/kali kali-rolling main contrib non-free
```

## 东软大学

```bash
deb http://mirrors.neusoft.edu.cn/kali kali-rolling/main non-free contrib deb-src http://mirrors.neusoft.edu.cn/kali kali-rolling/main non-free contrib
```

## 解决安装进程占用

在 apt 安装软件的时候可能会出现以下问题：

```bash
无法获得锁 /var/lib/apt/lists/lock - open (11: 资源暂时不可用)
```

这是因为 apt 进程被占用了，解决方法也比较简单粗暴，直接删除掉 lock 锁文件即可：

```bash
rm /var/cache/apt/archives/lock rm /var/lib/dpkg/lock
```

## 软件更新

不建议升级所有软件，耗费时间，用什么升级什么比较理性：

```bash
apt update # 检测某个软件是否有更新 apt list --upgradable|grep xxx # 更新软件 apt install xxx # 升级所有软件 apt-get upgrade
```

## Kali 2019.4 中文乱码

```bash
apt update # 设置编码字体为：zh_CN.UTF-8 UTF-8 dpkg-reconfigure locales # 安装中文字体 apt install xfonts-intl-chinese apt install ttf-wqy-microhei # 重启 reboot
```

## 虚拟机优化

## Parallels Desktop 14

```none
An error occurred when installing Parallels Tools.Please go to /var/log/parallels-tools-install.log for more information.
```

这里比较小众 如果你用的 macOS 系统的话 可能会出现这个问题 对于这个问题的解决方法 我放在了这篇文章里面了：

- [Parallel Tools 高版本内核的安装失败的解决方法](https://www.sqlsec.com/2019/12/pd.html)

- [Kali 2021.1 安装 Parallel Tools 疑难解答](https://www.sqlsec.com/2021/04/pdtools.html)

值得一提的是 Parallels Desktop 15 貌似没有这个问题 pd tools 可以顺利地直接安装

## VMware

Vmware 安装的 Kali Linux 的话，得安装 open-vm-tools，来优化使用体验:

```bash
apt update apt install open-vm-tools-desktop fuse # 安装完重启一下生效 reboot
```

## Gdebi

Gdebi 是一个安装 `.deb` 软件包的工具。 它提供了图形化的使用界面，但也有命令行选项，可以自动解决安装包的依赖问题，让安装软件少走弯路，省下宝贵的时间。

```bash
apt update apt install gdebi
```

## 酸酸乳客户端的安装

[https://github.com/the0demiurge/CharlesScripts/blob/master/charles/bin/ssr](https://github.com/the0demiurge/CharlesScripts/blob/master/charles/bin/ssr)
获取本脚本的最新版
使用方法：把该脚本放到 $PATH 里面并加入可执行权限就行
首次使用输入 `ssr install` 后安装时会自动安装

```bash
# 将ssr添加到bin目录下 sudo mv ssr /usr/local/bin sudo chmod +x /usr/local/bin/ssr # 显示帮助信息 ssr help # 安装ssr 执行此命令确保git命令安装成功 ssr install # 配置ssr ssr config # /usr/local/share/shadowsocksr/config.json 实际上编辑的是这个文件 # 相关参数 ssr start ssr stop ssr uninstall
```

下面附上一个 SSR 的配置案例文件：

```json
{ "server": "xxxxx.xxxx", "server_ipv6": "::", "server_port": 8080, "local_address": "127.0.0.1", "local_port": 1080, "password": "xxxxx.xxxx", "method": "none", "protocol": "auth_chain_a", "protocol_param": "", "obfs": "tls1.2_ticket_auth", "obfs_param": "", "speed_limit_per_con": 0, "speed_limit_per_user": 0, "timeout": 120, "udp_timeout": 60, "dns_ipv6": false, "connect_verbose_info": 0, "redirect": "", "fast_open": false }
```

## SSH

```bash
vim /etc/ssh/sshd_config
```

修改如下内容：

```bash
echo "PermitRootLogin yes" >> /etc/ssh/sshd_config echo "PasswordAuthentication yes" >> /etc/ssh/sshd_config
```

实际上具体修改内容如下：

```bash
#PermitRootLogin prohibit-password PermitRootLogin yes #PasswordAuthentication yes PasswordAuthentication yes
```

SSH 相关服务：

```bash
# 启动ssh /etc/init.d/ssh start # 设置为开机自启 update-rc.d ssh enable
```

## proxychains

终端命令行代理神器，科学上网，内网穿透渗透必备：

```bash
vim /etc/proxychains.conf
```

编辑配置文件，某尾修改自己的 socks5 本地代理地址很端口。我这里使用的是 `1086` 是因为 SSR 的 Mac 客户端代理的端口是 `1086`，其他系统下默认代理的端口为 `1080`

```bash
127.0.0.1 1086
```

验证下代理能否访问 Google

```bash
proxychains curl https://www.google.com.hk
```

## Git 代理的设置和取消

```bash
# 我这里习惯用1086端口 具体根据自己的配置来灵活设置 git config --global http.proxy 'socks5://127.0.0.1:1086' git config --global https.proxy 'socks5://127.0.0.1:1086' # 取消git代理 git config --global --unset http.proxy git config --global --unset https.proxy
```

## WPScan

WPScan 是一款免费的，用于非商业用途的黑盒 WordPress 漏洞扫描工具。因为国内特殊原因，更新漏洞库的时候需要挂代理才可以顺利更新。

```bash
# 版本查看 wpscan --version # 更新漏洞库 proxychains wpscan --update # 基本检测 wpscan --url http://xxx.com
```

## Firefox ESR 的汉化

Kali Linux 自带的浏览器，虽然系统语言已经是简体中文，但是这个浏览器依然是英文，强迫症可以执行一下命令手动安装汉化包：

```bash
apt update apt install firefox-esr-l10n-zh-cn
```

## Dash to Dock

Kali 2019.2 版本安装完成后发现自带的 Dash to Dock 非常的丑，这里手动调整下透明度，看着舒服一点：

![](https://cdn.wallleap.cn/img/pic/illustration/202408260923268.png?imageSlim)

## zsh 配置

```bash
# 安装zsh apt install zsh # 切换默认的shell为zsh= chsh -s /bin/zsh # 安装oh-my-zsh sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)" # 下载命令补全插件 git clone https://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions # zshrc配置文件中修改如下内容 plugins=(git zsh-autosuggestions) # 重新加载zsh配置 zsh
```

## nc 的配置

Kali 默认的 nc 版本是 netcat-traditional，虽然功能比较全，但是 2007 年 1 月的时候就停止更新了，这里建议再额外安装个 ncat

```bash
root@kali-linux:~# nc -h [v1.10-41.1]
```

nca t 的作者就是 nmap 的作者，在其他发行版的系统中一般情况下安装个 nmap 包的时候，`ncat` 的命令就可以正常使用来，但是 Kali Linux 默认就封装来 nmap，但是却没有 `ncat` 的命令，这里得我们手动安装一下。

```bash
apt update apt install ncat
```

安装完成后默认的 `nc` 命令实际上指向变成了 `ncat`，手动把 `nc` 命令换回 `netcat-traditioanl` 版本

```bash
update-alternatives --config nc
```

![](https://cdn.wallleap.cn/img/pic/illustration/202408260923269.png?imageSlim)

这样配置完成以后 `nc` 和 `ncat` 命令可以共存。

## Docker 的安装

Kali Linux 安装 Docker 用网上那个一键安装脚本貌似有点问题，这里单独记录一下，以便自己和其他网友使用：

```bash
# 添加Docker PGP密钥 curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add - # 配置docker apt源 我这里用的国内阿里云的docker下载源 echo 'deb https://mirrors.aliyun.com/docker-ce/linux/debian buster stable'> /etc/apt/sources.list.d/docker.list # 更新apt源 apt update # 如果之前安装了docker的话 这里得卸载旧版本docker apt remove docker docker-engine docker.io # 安装docker apt install docker-ce # 查看版本 docker version
```

Docker 安装完成后，还需要对其进行简单优化一下，后面才能用对舒服一点。

## Docker 更换国内镜像源

不替换源对话，docker pull 拉去镜像对速度实在太龟速了，如果你很佛系对话可以不进行更换

```bash
# 编辑这个文件，如果没有对话就创建这个文件 vim /etc/docker/daemon.json
```

内容如下：

```json
{ "registry-mirrors": [ "http://hub-mirror.c.163.com" ] }
```

这里我使用对是国内 163 网易源，其他源可以自行百度替换。
配置完成后重启服务才可以生效：

```bash
sudo systemctl daemon-reload sudo systemctl restart docker
```

## Kali 安装 docker compose

docker compose 神器，国内的 vulhubs 靶场就是用的 docker compose 规范，所以这里有必要安装一下。
首先来查看最新版本 [https://github.com/docker/compose/releases](https://github.com/docker/compose/releases)

我写这篇文章的时候目前是 `1.25.0-rc2` 版本，具体根据新版本的变化自行调整下面命令来安装：

```bash
# 下载docker-compose curl -L https://github.com/docker/compose/releases/download/1.25.0-rc2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose # 给docker-compose执行权限 chmod +x /usr/local/bin/docker-compose # 查看docker compose版本 root@kali-linux:~# docker-compose version docker-compose version 1.25.0-rc2, build 661ac20e docker-py version: 4.0.1 CPython version: 3.7.4 OpenSSL version: OpenSSL 1.1.0k 28 May 2019
```

## 支持一下

本文可能实际上也没有啥技术含量，但是写起来还是比较浪费时间的，在这个喧嚣浮躁的时代，个人博客越来越没有人看了，写博客感觉一直是用爱发电的状态。如果你恰巧财力雄厚，感觉本文对你有所帮助的话，可以考虑打赏一下本文，用以维持高昂的服务器运营费用（域名费用、服务器费用、CDN 费用等）

没想到文章加入打赏列表没几天 就有热心网友打赏了 于是国光我用 Bootstrap 重写了一个页面 用以感谢 支持我的朋友，详情请看 [打赏列表 | 国光](https://www.sqlsec.com/reward/)
