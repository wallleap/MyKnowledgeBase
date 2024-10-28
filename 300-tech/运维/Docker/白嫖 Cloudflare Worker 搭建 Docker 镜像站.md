---
title: 白嫖 Cloudflare Worker 搭建 Docker 镜像站
date: 2024-10-27T03:32:29+08:00
updated: 2024-10-27T03:42:46+08:00
dg-publish: false
---

项目地址：[cmliu/CF-Workers-docker.io: 这个项目是一个基于 Cloudflare Workers 的 Docker 镜像代理工具。它能够中转对 Docker 官方镜像仓库的请求，解决一些访问限制和加速访问的问题。 (github.com)](https://github.com/cmliu/CF-Workers-docker.io)

直接进入 <https://dash.cloudflare.com> 点左边的 workers 和 pages 进入概述

直接创建一个 worker，随意命名，部署之后点击编辑代码

把项目中的代码复制粘贴过去，然后保存

可以在自定义域中添加个域名，例如 [Docker Hub Search (vonne.me)](https://hub.vonne.me/)
