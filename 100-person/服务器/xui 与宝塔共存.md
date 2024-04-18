---
title: xui 与宝塔共存
date: 2023-11-15 15:36
updated: 2024-04-07 10:11
---

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
