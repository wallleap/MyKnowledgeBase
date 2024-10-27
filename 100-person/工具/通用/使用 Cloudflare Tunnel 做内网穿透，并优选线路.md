---
title: 使用 Cloudflare Tunnel 做内网穿透，并优选线路
date: 2024-10-25T09:59:28+08:00
updated: 2024-10-25T12:47:29+08:00
dg-publish: false
---

[没用公网IP？cloudflare优选IP，实现高速内网穿透 (youtube.com)](https://www.youtube.com/watch?v=uCgRb7KfANo)

准备工作：

- 两个域名例如
	- a.com 作为主域名
	- b.top 作为辅助域名
	- 都解析到 Cloudflare 上
- Papal 或 Visal 信用卡

## 内网穿透

点击主页的 [Zero Trust](https://one.dash.cloudflare.com/7a5fd825b42a2bdae43031e871070b90/onboarding?referrer=%2F7a5fd825b42a2bdae43031e871070b90%2Fhome)，随便输入个前缀，选择免费的，然后点击右边，接着**不要添加付款方式**，直接修改浏览器上方的网址回到 dash.cloudflare.com

重新点进 Zero Trust 就可以继续操作了

点击左侧菜单栏 Networks-Tunnels 进入页面，添加一个隧道，点左边的 Cloudflared，随便起个名字继续

上面根据系统选择下载安装，并运行命令

显示 `for cloudflared installed successfully` 就代表安装成功

之后随便启动一个网页，例如使用 `http-server`，确保内网能够进行访问，例如 `localhost:80`

继续在网页上，下划到最下面，点击 Next，开始配置域名

这里选 b.top 的，前缀随便，例如 origin，下面填内网服务，例如 `HTTP://localhost:80`，填完之后点 Save

直接访问 `https://origin.b.top` 应当能够访问服务

## 线路优选

点击那个 Tunnel App，点击 Edit，进入页面后点击 Public Hostname，点击添加一个 hostname

`www.a.com`，下面服务还是之前那个

配置自定义主机回退源，直接使用别人做好的，CNAME 解析过去

[CloudFlare公共优选Cname域名地址 - 宝塔迷 (baota.me)](https://www.baota.me/post-411.html)，在列表中选择一个域名，例如 `speed.marisalnc.com`

进入 `a.com` 的解析页面，删除 www 的解析记录

进入 `b.top` 的解析页面，在侧边栏 SSL/TLS 中点击自定义主机名

点击按钮，然后直接选择 Paypal，添加之后就能正常显示了

回退源中填写 `origin.b.top`，点击添加回退源

点击添加自定义主机名，填写 `www.a.com`，其他默认，点击添加，点击最右边三角，根据提示添加域名解析，等状态变成有效

下面的域名解析都只需要解析，不需要代理（去掉云朵）

- CNAME `cdn.b.top` → `speed.marisalnc.com`
- CNAME `www.a.com` → `cdn.b.top`

现在就可以通过下面链接进行访问

- 直连：`https://origin.b.top`
- 优选：`https://www.a.com`

尤其在被墙了之后更能显示出差距

![](https://cdn.wallleap.cn/img/pic/illustration/202410251050433.png?imageSlim)
