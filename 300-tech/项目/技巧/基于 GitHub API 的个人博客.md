---
title: 基于 GitHub API 的个人博客
date: 2022-11-10T03:33:00+08:00
updated: 2024-08-21T10:32:45+08:00
dg-publish: false
aliases:
  - 搭建个人博客
author: Luwang
category: web
comments: true
cover: //cdn.wallleap.cn/img/pic.jpg
description: 文章描述
keywords:
  - 前端
  - GitHub
  - 博客
rating: 10
source: null
tags:
  - API
  - blog
url: null
---

记录一下搭建博客的过程

## 一、前言

之前用的是 [蝉时雨](https://chanshiyu.com/) 大佬的 [aurora](https://github.com/chanshiyucx/aurora) 主题，我就想着自己要不也做一个全套的（响应式的 web、小程序、后台页面），现在有空就研究一下。

### 1、REST API 指南

地址：<https://docs.github.com/cn/rest/guides/getting-started-with-the-rest-api>

[这里](https://docs.github.com/cn/rest/overview/endpoints-available-for-github-apps) 列出了所有 API，我们挑一些需要的就行

eg：需要获取某个仓库中所有议题

<kbd>Ctrl</kbd>+<kbd>F</kbd>查找 `issue`，猜测可能是这两个

![issues](https://cdn.wallleap.cn/img/pic/illustrtion/202211091504956.png)

点进去之后，发现 `POST` 是创建 issue 的，我们之后搭建后台的时候就可以使用这个接口，现在我们需要查询的话，就应该使用 `Get` 请求

![POST](https://cdn.wallleap.cn/img/pic/illustrtion/202211091504957.png)

可以看到参数有很多，我们先不管下面的，先把路径中的参数更改一下

![Get](https://cdn.wallleap.cn/img/pic/illustrtion/202211091504958.png)

- `owner` 是用户名，类型为 `string`，就比如我的是 `wallleap`
- `repo` 是仓库名，类型也为 ` string  `，我这里就直接用之前存放博客的仓库 `AuroraBlog`

所以拼合起来就是 `/repos/wallleap/AuroraBlog/issues`，加上 API 地址就是 `https://api.github.com/repos/wallleap/AuroraBlog/issues`，到 [ApiPost](https://www.apipost.cn/) 或 [Postman](https://www.postman.com/) 测试一下

![接口测试](https://cdn.wallleap.cn/img/pic/illustrtion/202211091504959.png)

数据返回成功，具体数据这里就不分析了

### 2、身份验证

> 未经身份验证的客户端每小时可以发出 60 个请求。 要每小时发出更多请求，我们需要进行*身份验证*。 事实上，使用 GitHub API 进行任何交互都需要 [身份验证](https://docs.github.com/cn/authentication)。

使用 GitHub API 进行身份验证的最简单和最佳的方式是 [通过 OAuth 令牌](https://docs.github.com/cn/rest/overview/other-authentication-methods#via-oauth-and-personal-access-tokens) 使用基本身份验证。 OAuth 令牌包括 [个人访问令牌](https://docs.github.com/cn/articles/creating-an-access-token-for-command-line-use)。

生成 token：

- 点击自己 GitHub 头像，点击 Settings
- 在侧边栏最下面一个 Developer settings 中随便填写一个名
- 过时选择永不过时（No expiration），勾选 repo
- 点击 Generate token 生成 token
- 复制 生成的 token，保存好（只会显示这一次，不记得需要重新生成）

![生成token](https://cdn.wallleap.cn/img/pic/illustrtion/202211091504960.jpeg)

身份验证：

token 放到请求头中（值为 token+ 空格 + 生成的 token 字符）

```
Authorization : token ghp_xxxxx1
```

在 ApiPost 中 Header 加上参数

![带请求头的](https://cdn.wallleap.cn/img/pic/illustrtion/202211091504961.png)

在右边可以生成代码

![生成代码功能](https://cdn.wallleap.cn/img/pic/illustrtion/202211091504962.png)
