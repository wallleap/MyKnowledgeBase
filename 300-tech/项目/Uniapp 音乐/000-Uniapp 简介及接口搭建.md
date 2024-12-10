---
title: 000-Uniapp 简介及接口搭建
date: 2024-11-18T10:43:10+08:00
updated: 2024-11-22T16:07:04+08:00
dg-publish: false
---

uni-app 开源地址：<https://github.com/dcloudio/uni-app>

官网：<https://uniapp.dcloud.io>

`uni-app` 是一个使用 `Vue.js` 开发小程序、H5、App 的统一前端框架

## 创建项目

`uni-app` 支持通过 `vue-cli` 命令行、`HBuilderX` 可视化界面两种方式快速创建项目

### vue-cli 命令行

- [通过vue-cli命令行](https://uniapp.dcloud.net.cn/quickstart.html#_2-%E9%80%9A%E8%BF%87vue-cli%E5%91%BD%E4%BB%A4%E8%A1%8C)
- [在 VSCode 中开发](https://ask.dcloud.net.cn/article/36286)
- [在 WebStorm 中开发](https://ask.dcloud.net.cn/article/36307)

### HBuilderX 可视化

[官方IDE下载地址](https://www.dcloud.io/hbuilderx.html)

**新建项目**

点击菜单栏文件 - 新建 - 项目

**运行项目**

菜单栏 - 运行

## 后端

目前常见的音乐后端都是通过跨站请求伪造（CSRF）伪造请求头调用官方 API

- 网易云：<https://gitlab.com/Binaryify/neteasecloudmusicapi>
- 酷狗音乐：<https://github.com/MakcRe/KuGouMusicApi>
- QQ 音乐：
	- <https://github.com/rain120/qq-music-api>
	- <https://github.com/jsososo/QQMusicApi>
- 酷我音乐：<https://github.com/QiuYaohong/kuwoMusicApi>

编写 Dockerfile

```
FROM hub.vonne.me/node:lts-alpine

RUN apk add --no-cache tini

ENV NODE_ENV production

USER node

WORKDIR /app

COPY --chown=node:node . ./

RUN yarn --network-timeout=100000

EXPOSE 3000

CMD [ "/sbin/tini", "--", "node", "app.js" ]
```

在项目根目录运行

```sh
docker build -t musicapi .
docker run -p 30000:3000 musicapi
```

再进行反向代理通过域名访问即可，例如 <https://api.km.wallleap.cn>
