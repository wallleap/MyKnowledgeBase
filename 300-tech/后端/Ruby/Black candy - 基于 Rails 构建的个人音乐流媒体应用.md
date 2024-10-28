---
title: Black candy - 基于 Rails 构建的个人音乐流媒体应用
date: 2024-08-16T11:13:00+08:00
updated: 2024-08-21T10:32:31+08:00
dg-publish: false
source: //ruby-china.org/topics/38772
---

先放项目地址： [https://github.com/aidewoode/black_candy](https://github.com/aidewoode/black_candy)

这个项目最早脱胎于在去年的时候，之前的公司里组长叫我研究一下 stimulus 时我做的一个音乐播放器 demo。然后我发现在 Ruby 社区并没有这样一个开源的音乐流媒体应用，我唯一找到的是 [Beatstream](https://github.com/Darep/Beatstream) 但是作者早就没维护了。但反观其他一些社区都有其成熟的这样的应用，php 有 [koel](https://github.com/phanan/koel) ，python 有 [cherrymusic](https://github.com/devsnd/cherrymusic)，js 有 [mStream](https://github.com/IrosTheBeggar/mStream) 。所以我决定完善这个 demo，让 Ruby 社区也有这样一个音乐流媒体应用。然后我在接下来的一段时间里重新设计了样式，添加了功能，完善了代码。现在这个项目到了我认为的一个可用的地步了。所以分享一下关于这个项目的一些经验体会。

在这个项目里我尝试去构建一个更现代化的 Rails 应用。包括：

- 使用 webpacker 替换掉 asset pipeline
- 使用 stimulus 组织前端 js 代码
- 使用 docker 用于构建开发，测试环境和应用部署

## 关于 webpacker

我从 webpack 1.x 的时候就开始尝试使用 webpack，当时 webpack 简直让我觉得不可思议。不仅可以打包 js，也能打包其他资源，还能通过 loader 来处理前端代码。确实比同时期的什么 browserify，gulp 之类的工具强大不少。虽说是强大但是配置起来却恶心，不研究半天文档还真不知道该如何配置。

webpack 从 3.0 才开始简化配置，可能因为出现了 parcel 这样号称零配置的打包工具让 webpack 团队感到压力了吧。但是在 rails 里面集成 webpack 依旧麻烦，我之前都是通过 webpack 把打包后的资源放在 assets 下面，再通过 asset pipeline 引入。但这只是解决了如何引入的问题，其他的包括开发环境的 hot reload，部署时的 precompile，不同环境下的配置都还得额外处理。

webpacker 是真的解决了在 rails 项目中使用 webpack 的问题。如果之前熟悉前端那一套玩法，webpacker 真的做到了开箱即用，体验相当好。Rails 约定大于配置那一套在 webpacker 上得到了延伸。

## 关于 stimulus

我不讨论 stimulus 和 前端 js 框架的优劣，因为已经有太多文章去讲这个事了，而且各有各的适用场景。我想说的是在 rails 上使用 stimulus + turbolinks + sjr 是可以做到类似 SPA 的体验的。rails 这一套东西用来处理不是特别复杂交互是完全 hold 得住的。

stimulus 概念是相当简单的，不到半个小时文档就可以全部看完然后上手。虽说概念简单，但我觉得使用 stimulus 的门槛并不低，在 stimulus 中需要手动处理包括 DOM 的修改，事件的添加解绑，事件的代理等其他事，因为 stimulus 并不是一般的前端 js 框架，更像是一种组织 js 的代码的方式。很多操作还是需要用原生 js 去解决。

我用过很长一段时间的前端 js 框架，前端框架刚出来那会是很吸引我的，对于我来说是一种全新的前端开发方式。包括一系列基于 node 的构建工具，能感觉到前端开发变得越来越正规了。最近一两年我开始意思到，很多情况下我并不需要去使用前端 js 框架，很多的项目并没有那么复杂的交互，也没有严格的前后端分工。我开始想当初前端框架吸引我的到底是什么，是数据绑定？是单向数据流？是状态管理？我发现最主要的是前端框架提供给了我一个组织 js 代码的方式。在我还是个新人的时候，面对项目下凌乱的 js 的代码无所适从的时候，前度框架的出现让我仿佛看到了光。

虽然随着经验的积累，不用这些框架也能组织好前端 js 代码，但 stimulus 无疑让在 rails 下如何使用更 rails 的方式去组织前端 js 代码变得规范了，这也是我最喜欢 stimulus 的一点。

## 关于 docker

使用 docker 作为开发工具起源于我尝试在电脑上装两个不同版本的 mysql，因为有项目依赖特定的 mysql 版本。这次恶心的经历让我意识到为每个项目搭建相互隔离的开发环境的重要性。docker compose 让搭建开发环境异常容易，而且 dokcer-compose.yml 可以固化开发环境，即使在一台新电脑上，只要安装了 docker 和 docker compose，就可以一键处理好开发环境，对于项目新成员也一样。

我目前的做法是构建一个 base 镜像包含项目所有依赖，但不包括项目代码和类似于数据库，redis 之类的外部 service，那些通过 docker compose 来解决。然后这个 base 镜像就用于开发和测试环境。在 build 生产环境的镜像的时候再基于这个 base 镜像把代码加上构建镜像。

docker 目前唯一让我不爽的是，在 mac 下 docker 太慢了，而且是个遗留问题，具体可以查看这个 issue [File access in mounted volumes extremely slow · Issue #77 · docker/for-mac · GitHub](https://github.com/docker/for-mac/issues/77)。目前看来没什么进展，虽然可以通过 dokcer-sync 或者 nfs 之类的方式能提一下速。还是期待未来 docker 能拿出一个真正有效的办法吧或者 apple 给点力让 mac os 有原生的容器支持能力。最近 windows 放出 WSL2 后，docker 也宣布基于 WSL2 开发 windows 下的 docker [Docker ❤️ WSL 2 - The Future of Docker Desktop for Windows - Docker Engineering Blog](https://engineering.docker.com/2019/06/docker-hearts-wsl-2/)。估计之后在 windows 下使用 docker 的体验可以接近 linux 上了。哎，感觉又多了一个可以换 windows 的理由。

## 最后

欢迎大家试用反馈，Happy Hacking

---

Why we chose Turbolinks [https://changelog.com/posts/why-we-chose-turbolinks](https://changelog.com/posts/why-we-chose-turbolinks)

Changelog 不是一个 Rails 应用，主动引入了 Turbolinks 实现播放器的持久化，当时我就觉得 Rails 前端一套足够做流媒体网站的。

试了 demo，项目很酷 👍
