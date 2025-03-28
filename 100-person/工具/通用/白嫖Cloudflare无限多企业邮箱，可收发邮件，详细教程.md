---
title: 白嫖Cloudflare无限多企业邮箱，可收发邮件，详细教程
date: 2025-01-21T09:42:43+08:00
updated: 2025-01-21T09:45:53+08:00
dg-publish: false
source: https://linux.do/t/topic/123312
---

> 如何在赛博大善人 cloudflare 那里白嫖无限多个企业级邮箱，这些邮箱既可以收邮件，也可以发邮件。

可以用来接收网站验证码，注册账户的时候就可以很方便的注册一大堆小号。 还可以把他们当成临时邮箱，与人通信，避免暴露自己真实的邮箱，保护个人隐私。 Cloudflare 是一家提供 CDN、网络安全、DDos 防御和域名服务的公司。人称大善人，赛博活佛，爬爬虾之前介绍过 cloudflare 的 DNS，内网穿透等免费服务，今天重点来看如何白嫖邮箱，让你瞬间获得无限多个免费邮箱。

**文字版较简略，视频更详细：**

## 域名名称解析

使用 Cloudflare 的前提是要有一个域名。关于域名的购买，可以看一下这期视频。

有了域名以后，我们就把它托管到 Cloudflare 上。我们先登录一下 Cloudflare，没有账号就注册一个。右上角点击添加站点，输入你的域名，

[![](https://cdn.wallleap.cn/img/pic/illustration/20250121094335050.jpeg?imageSlim)image1125×506 56.7 KB](https://linux.do/uploads/default/original/3X/a/5/a5d86393a5d9c3b9892bb0591d9e431580258649.jpeg)

然后点击继续。这里有一些付费的，我们都不要，直接找下面这个免费的，点击继续。

[![](https://cdn.wallleap.cn/img/pic/illustration/20250121094335052.png?imageSlim)image981×283 37.2 KB](https://linux.do/uploads/default/original/3X/3/9/39aa27287f7ef1a448ace9cadabdd368fe9d5bc3.png)

一路下一步。这步是重点，更改名称服务器。找到这两个名称服务器。

[![](https://cdn.wallleap.cn/img/pic/illustration/20250121094335053.png?imageSlim)image919×298 61.7 KB](https://linux.do/uploads/default/original/3X/f/f/ffd252067082dd99e5ddb521b3e3f81c6c102570.png)

我们需要去刚才买域名的网站更改一下，这里有一个 Name Server。就把这里改一下，把原来的这 3 条全部删掉，然后填上刚才网站给我分配的这两个，点击保存。

[![](https://cdn.wallleap.cn/img/pic/illustration/20250121094335054.png?imageSlim)image591×456 57.3 KB](https://linux.do/uploads/default/original/3X/9/f/9fd59e000a1c9fc394d5ac75657783a91b176e42.png)

我们再回到 Cloudflare，点击立即检查，然后点击继续。这里什么都不用动，直接开始使用就可以了。

左上角点击 Cloudflare 的图标，回到首页，域名这里打上一个对勾，那就可以开始直接使用了。

[![](https://cdn.wallleap.cn/img/pic/illustration/20250121094335055.png?imageSlim)image1252×471 85.3 KB](https://linux.do/uploads/default/original/3X/0/2/02ea77da08fd6217759e277d33a81be060d06848.png)

## 收邮件

[这里我以我自己的域名tech-shrimp.com](http://xn--tech-shrimp-2u0ri05cgtm89vy8oba5435g9i1bgi6coti.com/)，也就是技术爬爬虾为例进行演示。我们先点击这个域名，进来以后选择左侧电子邮件，选择电子邮件路由。

第一次进来以后，会有一个新手设置，点击取消新手设置，点击启用电子邮件路由。

[![](https://cdn.wallleap.cn/img/pic/illustration/20250121094335056.jpeg?imageSlim)image1125×612 72.1 KB](https://linux.do/uploads/default/original/3X/8/1/81e9fcc2f934cea6be6c0f317c679e2ffa890897.jpeg)

这里面有 4 个 DNS 值，我们点击添加记录并启用，Cloudflare 就会自动添加上这 4 个 DNS 记录。

[![](https://cdn.wallleap.cn/img/pic/illustration/20250121094335057.jpeg?imageSlim)image1125×735 90.4 KB](https://linux.do/uploads/default/original/3X/a/d/adae45eb5c9425a7c8a5ba960acd09dfa1a5e043.jpeg)

弄好以后，我们再点击路由规则，这里有一个 catch-all 地址，我们点击右侧的编辑，

[![](https://cdn.wallleap.cn/img/pic/illustration/20250121094335058.png?imageSlim)image1125×308 62.9 KB](https://linux.do/uploads/default/original/3X/9/6/96c9e1aca257cedd5a0c4ae8b093877ca41e88d1.png)

点击发送到电子邮件。这里填写一个自己的电子邮箱，然后点击保存。

[![](https://cdn.wallleap.cn/img/pic/illustration/20250121094335059.png?imageSlim)image1125×618 87.8 KB](https://linux.do/uploads/default/original/3X/b/4/b4c2f51c2a6528a6547787f901e34b44cec2eadb.png)

保存好以后，这里会出现一个待验证。

[![](https://cdn.wallleap.cn/img/pic/illustration/20250121094335060.png?imageSlim)image1125×359 95.5 KB](https://linux.do/uploads/default/original/3X/7/b/7b70469481711b80562941327414b89fad55d489.png)

Cloudflare 会发送一封验证邮件到这个地址，我们来接收一下。刷新一下页面，打上一个对勾，那就配置完成了。

[![](https://cdn.wallleap.cn/img/pic/illustration/20250121094335061.png?imageSlim)image1125×288 80.4 KB](https://linux.do/uploads/default/original/3X/0/b/0bc7edbfc4681bc98afb4fe6507b631ab7cbfa44.png)

**这个配置的意思是所有发送到 tech-shrimp.com 域名下面的邮件都会自动转到 163 的邮箱** 。这样就实现了无限多个邮箱可以收邮件。

## 发邮件

使用邮箱发送邮件，我使用的服务是**[resend.com](http://resend.com/)** ，这也是一个免费的服务。没有账号的话就自己注册一个。

选择左侧的 API Keys，点击 create API Key，名字随便填一个，下面两个保持默认，点击添加。

[![](https://cdn.wallleap.cn/img/pic/illustration/20250121094335062.png?imageSlim)image1125×482 83.6 KB](https://linux.do/uploads/default/original/3X/f/7/f7a904bf0975bd8142bd082dd4d0af2a50ad51d5.png)

[![](https://cdn.wallleap.cn/img/pic/illustration/20250121094335063.png?imageSlim)image1125×852 74.4 KB](https://linux.do/uploads/default/original/3X/f/b/fb13cb69e6e482f33eaf249bd9005d27880630ff.png)

就会获得一个 API Key，把它保存好。这里有一个域名，我们需要把域名也添加到这个网站上，就是刚才托管到 Cloudflare 的那个域名。

[![](https://cdn.wallleap.cn/img/pic/illustration/20250121094335064.png?imageSlim)image1125×478 58.8 KB](https://linux.do/uploads/default/original/3X/e/d/eda1acee6dd3081099f87eee7ac6c1bb4648a41d.png)

**好，这一步是重点** 。这里给了 3 个 DNS 记录，我们需要把这个记录填回 Cloudflare

[![](https://cdn.wallleap.cn/img/pic/illustration/20250121094335065.jpeg?imageSlim)image1125×538 69.9 KB](https://linux.do/uploads/default/original/3X/8/c/8c5bbfd5d4641ae6d08d4c98b473e3f2adcc97fc.jpeg)

回到 cloudflare

[![](https://cdn.wallleap.cn/img/pic/illustration/20250121094335066.jpeg?imageSlim)image1125×477 79.1 KB](https://linux.do/uploads/default/original/3X/5/a/5a74826849494b308fe3f40a18a841e9415c6792.jpeg)

[![](https://cdn.wallleap.cn/img/pic/illustration/20250121094335067.jpeg?imageSlim)image1125×418 79.5 KB](https://linux.do/uploads/default/original/3X/1/1/11542f552c274d2d3c85a65229c5c317ca5190b3.jpeg)

我们回到 resend，有一个按钮 verify DNS records，我们点击一下，它检查我们刚才的配置是否正确。等校验完，页面标成绿色，那就配置完成了。

[![](https://cdn.wallleap.cn/img/pic/illustration/20250121094335068.jpeg?imageSlim)image1125×653 70.9 KB](https://linux.do/uploads/default/original/3X/4/7/47e6e97a806db768e7e54731ddae6d9b38f22fef.jpeg)

### 使用 Python 发邮件

```csharp
# 先安装依赖
# pip install resend
import resend
# 这里换成自己的resend API Key
resend.api_key = "re_xxxxxxxxxxxxxxxxxxxxxxx"

params: resend.Emails.SendParams = {
# 发件人可以是自己域名下的任何一个人
"from": "sdkjfskdhf@你的域名",
"to": ["tech-shrimp@outlook.com"],
"subject": "hi",
"html": "<strong>hello, world!</strong>"
}

email = resend.Emails.send(params)
print(email)
```

### 使用 cURL 发邮件

Authorization 换成自己的 resend API Key

发件人可以是自己域名下的任何一个人

````bash
curl -X POST 'https://api.resend.com/emails' \
-H 'Authorization: Bearer re_VxeNCEn1_6w4bYF93xQKgKGFYRxNK2D3J' \
-H 'Content-Type: application/json' \
-d 
```{
"from": "Acme <onboarding@你的域名>",
"to": ["tech-shrimp@outlook.com"],
"subject": "hello world",
"text": "it works!"
}'
````
