---
title: 使用 Cloudflare Workers 让 OpenAI API 绕过 GFW 且避免被封禁
date: 2023-06-10 22:45
updated: 2023-06-12 11:32
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - 使用 Cloudflare Workers 让 OpenAI API 绕过 GFW 且避免被封禁
rating: 1
tags:
  - blog
category: 分类
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: #
url: //myblog.wallleap.cn/post/1
---

> ℹ️ 近期因为 OpenAI 的风控有大量 API Key 或账号被封禁。因为 Cloudflare 勉强也算云服务商，而从云服务商请求 API 是再正常不过的操作，所以使用 Cloudfalre Worker 代理地址后，理论上也会降低被封禁的概率。

事实证明 ChatGPT 是足够火爆的，火爆到什么程度呢，其 API 一经推出便获得了 GFW 的认证。在 Twitter 上看到很多人都在为解决无法正常访问 OpenAI 的 API 而苦恼，最常见解决方案是使用一台服务器来进行反向代理，但这样又徒增了一些成本。因为之前在公司的业务上遇到过类似问题，当时老板找到了一个还不错的几乎零成本解决方案，试了一下现在仍然可以用来解决 OpenAI 的 API 无法访问的问题，所以在这里推荐给大家。

该方案的主要思路是使用 Cloudflare 的 Workers 来代理 OpenAI 的 API 地址，配合自己的域名即可在境内实现访问。因为 Cloudflare Workers 有每天免费 10 万次的请求额度，也有可以免费注册的域名，所以几乎可以说是零成本。而且该方法理论上支持所有被认证的网站，而不只是 OpenAI。

使用这个方案需要你有以下东西：

- 一个没有被 GFW 认证的域名（~~没有的话也可以到 [https://www.freenom.com](https://www.freenom.com/) 免费注册一个，~~ ⚠️ 据推友提醒，freenom 已暂停新用户注册，但相信对于大家来说注册域名不是啥大问题）
- 一个 Cloudflare 账号（当然也可以现注册）

⚠️ 请不要直接使用本教程示例中的地址，因为随时会被关闭。也不要使用任何其他人搭建的不受信任的地址，因为有 api key 被盗取的可能。

## 太长不看

1. 新建一个 Cloudflare Worker
2. 将 [https://gist.github.com/noobnooc/d0407b5fb81cff9d36f981170b99d4e6](https://gist.github.com/noobnooc/d0407b5fb81cff9d36f981170b99d4e6) 里的代码粘贴到 Worker 中并部署
3. 给 Worker 绑定一个没有被 GFW 认证的域名
4. 使用自己的域名代替 `api.openai.com`

如果具体步骤有问题，可以参考下面的详细版教程。

## 🆕 将域名 NS 转到 Cloudflare

如果域名已经托管在 Cloudflare 的忽略这一步即可。

⚠️ 经评论区指出，Cloudflare Workers 的域名绑定仅支持托管在 Cloudflare 上的域名。由于本人常年是把域名托管在 Cloudflare 的没有注意到这一点，所以得先将域名的 NS 转到 Cloudflare，如果介意将域名转到 Cloudflare 的话，可以考虑使用 nginx 反代、Docker 容器等其他方法 🥲。

没有 Cloudflare 账号的话可以注册一个，具体注册细节就不多说了。注册或登录到 Cloudflare 的管理界面后，点击侧边栏的 “Websites” ，然后点击 “Add a Site” 按钮准备将域名转到 Cloudflare：

![image](https://cdn.wallleap.cn/img/pic/illustration/202306102247788.png?imageMogr2/format/webp/interlace/1/quality/80%7Cwatermark/1/image/aHR0cDovL3dhbGxsZWFwLTEyNTkwODQzMzAuY29zLmFwLWd1YW5nemhvdS5teXFjbG91ZC5jb20vaW1nL3dhdGVybWFyazEucG5n/gravity/southeast/dx/16/dy/16/scatype/3/spcent/10/dissolve/90%7Cwatermark/3/type/3/text/bHV3YW5nMHdhbGxsZWFw)

在 “Enter your site (example.com)” 处输入要转入的域名后，点击 “Add Site”：

![image](https://cdn.wallleap.cn/img/pic/illustration/202306102248540.png?imageMogr2/format/webp/interlace/1/quality/80%7Cwatermark/1/image/aHR0cDovL3dhbGxsZWFwLTEyNTkwODQzMzAuY29zLmFwLWd1YW5nemhvdS5teXFjbG91ZC5jb20vaW1nL3dhdGVybWFyazEucG5n/gravity/southeast/dx/16/dy/16/scatype/3/spcent/10/dissolve/90%7Cwatermark/3/type/3/text/bHV3YW5nMHdhbGxsZWFw)

根据 Cloudflare 的提示，在域名注册商处将 NS 修改到 Cloudflare 指定的地址，等待域名解析成功后，即可进行后续操作。

## 创建一个 Cloudflare Worker

登录到 Cloudflare 的管理界面后，点击侧边栏的 “Workers” 选项，然后点击 “Create a Service” 创建一个 Worker。

![image](https://cdn.wallleap.cn/img/pic/illustration/202306102248136.png?imageMogr2/format/webp/interlace/1/quality/80%7Cwatermark/1/image/aHR0cDovL3dhbGxsZWFwLTEyNTkwODQzMzAuY29zLmFwLWd1YW5nemhvdS5teXFjbG91ZC5jb20vaW1nL3dhdGVybWFyazEucG5n/gravity/southeast/dx/16/dy/16/scatype/3/spcent/10/dissolve/90%7Cwatermark/3/type/3/text/bHV3YW5nMHdhbGxsZWFw)

然后在创建界面中输入 “Service name” 后点击 “Create Service” 按钮新建 Worker。“Select a starter” 项先不用管。

![image](https://cdn.wallleap.cn/img/pic/illustration/202306102249403.png?imageMogr2/format/webp/interlace/1/quality/80%7Cwatermark/1/image/aHR0cDovL3dhbGxsZWFwLTEyNTkwODQzMzAuY29zLmFwLWd1YW5nemhvdS5teXFjbG91ZC5jb20vaW1nL3dhdGVybWFyazEucG5n/gravity/southeast/dx/16/dy/16/scatype/3/spcent/10/dissolve/90%7Cwatermark/3/type/3/text/bHV3YW5nMHdhbGxsZWFw)

至此 Cloudflare 的 Worker 便创建好了，下面开始修改 Worker 的代码，使其能代理 OpenAI 的 API。

## 修改 Cloudflare Worker 的代码

在 Worker 的管理界面，点击右上角的 “Quick Edit” 按钮编辑代码 Worker 的代码。

![image](https://cdn.wallleap.cn/img/pic/illustration/202306102250234.png)

在左侧的代码编辑器中，删除现有的所有代码，然后复制粘贴以下内容到代码编辑器：

```js
export default {
  async fetch(request) {
    const url = new URL(request.url);
    url.host = 'api.openai.com';
    return fetch(url, { headers: request.headers, method: request.method, body: request.body });
  },
};
```

最后点击编辑器右下角的 “Save and deploy” 按钮部署该代码，在弹出的对话框中继续选择 “Save and deploy” 确认部署。

![image](https://cdn.wallleap.cn/img/pic/illustration/202306102251780.png)

至此，便可以使用该 worker 的地址来代替 OpenAI 的 API 地址了。比如想要请求 ChatGPT 的 API 时，把官方文档中的 `https://api.openai.com/v1/chat/completions` 替换成 `https://openai.workers.dev` 即可（注意这个地址并不存在，是需要换成自己刚刚创建的 Worker 的地址）。

但是你可能会发现，这样做了依然还是没有解决问题，因为 Cloudflare Workers 的 `workers.dev` 域名也是被 GFW 认证过的🥲。但是好在只是认证了 `workers.dev` 域名，而 ip 还是幸存的状态，所以我们可以给 Worker 绑定一个自己的域名。

## 绑定域名

在 Cloudflare Workers 的管理界面中，点击 “Triggers” 选项卡，然后点击 “Custom Domians” 中的 “Add Custom Domain” 按钮以绑定域名。

![image](https://cdn.wallleap.cn/img/pic/illustration/202306102251543.png)

输入域名后点击 “Add Custom Domain”，~~根据提示修改域名的 DNS 记录。因为我的域名是托管在 Cloudflare 上的，所以无需手动更改 DNS 记录，如果域名没有托管在 Cloudfalre 上，可以根据相关提示自行配置。~~ ⚠️ 据评论区提示，目前只支持 NS 托管在 Cloudflare 上的域名，如果不介意，可以点击 Cloudflare 侧边栏的 “Websites”，然后点击 “Add a Site” 按钮，根据提示将域名的 NS 记录指定到 Cloudflare。

![image](https://cdn.wallleap.cn/img/pic/illustration/202306102252745.png)

至此便大功告成。等待片刻，应该就可以通过你自己的域名来代替 OpenAI 的 API 地址了，比如在本文的例子中，想要请求 ChatGPT 的 API ，即是把官方 API 地址 `https://api.openai.com/v1/chat/completions` 换为我自己的域名 `https://openai.nooc.ink/v1/chat/completions` ，其他参数均参照官方示例即可。由于 Cloudflare 有每天免费 10 万次的请求额度，所以轻度使用基本是零成本的。⚠️ 注意请不要使用我这里的 `openai.nooc.ink`，因为随时可能会被我关闭🤪

![image](https://cdn.wallleap.cn/img/pic/illustration/202306102252814.png)

🥰 如果本文对你有帮助，欢迎 follow 我的 [Twitter](https://twitter.com/noobnooc) 和 [GitHub](https://github.com/noobnooc) 。

🤖 也欢迎关注我发布的开源项目 [OhMyGPT](https://github.com/noobnooc/ohmygpt.git) ，仅需几步即可发布属于自己的基于 ChatGPT API 的 Web 应用。

📱如果你不知道拿 OpenAI API 来干嘛，又或者想找一个 ChatGPT 的第三方客户端，不妨试试我新开发的 [AssisChat](https://apps.apple.com/us/app/assischat/id6446092669)。
