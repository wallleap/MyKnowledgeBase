---
title: AList 干倒闭百度网盘、阿里云盘，告别乱冲会员
date: 2024-08-01T09:18:00+08:00
updated: 2024-10-25T03:31:16+08:00
dg-publish: false
---

[干倒闭百度网盘、阿里云盘。告别乱冲会员 – 程序员哈利 (hali.life)](https://hali.life/?p=590)

## 简介

使用本方法，可以在自己本地电脑搭建一个功能不亚于百度网盘的网盘工具，并且无需域名、内网穿透等复杂技术，就可以分享给其他人。**==擦边博主分享付费用户福利必备工具==**。

## 第一步、下载所需工具

工具 exe 和我编写的脚本：[**==alist安装包_哈利整合脚本==**](https://pan.quark.cn/s/b37410d1b4f4)

如果熟悉文档可以直接去开源仓库下载：

[Releases · alist-org/alist (github.com)](https://github.com/alist-org/alist/releases)

## 第二步、本地安装 AList

只需运行第一步中下载文件中的 “==1 运行 alist 网盘.bat==” 脚本

运行后，在浏览器中打开 [==http://localhost:5244==](http://localhost:5244/) 就看可以打开网盘

忘记密码 请运行：“==2 修改网盘 admin 密码为 123456 (保证 alist 网盘运行中，再运行此脚本).bat==”

后台运行 alist 网盘：“==3 将 alist 网盘变成后台服务.bat==”

写在 alist 网盘：“==4 将 alist 服务卸载.bat==”

查看文档：[手动安装 | AList文档 (nn.ci)](https://alist.nn.ci/zh/guide/install/manual.html)

手动安装或者 Docker 安装

mac 有一个可以设置守护进程的

使用任意方式编辑 `~/Library/LaunchAgents/ci.nn.alist.plist` 并添加如下内容，修改 `path_alist` 为 AList 所在的路径，`path/to/working/dir` 为 AList 的工作路径

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
    <dict>
        <key>Label</key>
        <string>ci.nn.alist</string>
        <key>KeepAlive</key>
        <true/>
        <key>ProcessType</key>
        <string>Background</string>
        <key>RunAtLoad</key>
        <true/>
        <key>WorkingDirectory</key>
        <string>path/to/working/dir</string>
        <key>ProgramArguments</key>
        <array>
            <string>path_alist/alist</string>
            <string>server</string>
        </array>
    </dict>
</plist>
```

然后，执行 `launchctl load ~/Library/LaunchAgents/ci.nn.alist.plist` 加载配置（加 `-w` 强制），现在你可以使用这些命令来管理程序：

- 开启: `launchctl start ~/Library/LaunchAgents/ci.nn.alist.plist`
- 关闭: `launchctl stop ~/Library/LaunchAgents/ci.nn.alist.plist`
- 卸载配置: `launchctl unload ~/Library/LaunchAgents/ci.nn.alist.plist`

## 第三步、使用 ipv6 让别人访问

需要两个前置条件：

1、自己的宽带和路由器支持 ipv6

2、对方的宽带和路由器支持 ipv6

修改 “控制面板\所有控制面板项\网络连接” => 选择网卡，开启 ipv6

进入路由器控制台，开启 ipv6，设置 Native 模式。

使用 ipconfig 查询自己的 ipv6 地址，例如：`2408:8220:134:84fc::ccb`

别人访问你的 alist 网盘只需在浏览器中输入：`http://[2408:8220:134:84fc::ccb]:5244`

## 没有 IPv6

内网穿透

[[使用 Cloudflare Tunnel 做内网穿透，并优选线路]]

已经安装过 cloudflared，直接进入那个 Tunnel，编辑

新建 public hostname

- `alist.vonne.me` → `HTTP localhost:5244`

把相关 TXT 记录解析好

之前已经配置好了 `cdn.wallleap.eu.org` CNAME 到了公共优选域名

那直接在 vonne.me 中删除 alist 解析记录，添加 CNAME 就行

- CNAME `alist.vonne.me` → `cdn.wallleap.eu.org`

现在就可以通过下面链接进行访问

- 优选：`https://alist.vonne.me`

如果还需要直连的话，可以直接在 cloudflared 中多添加一条域名指向同一个服务
