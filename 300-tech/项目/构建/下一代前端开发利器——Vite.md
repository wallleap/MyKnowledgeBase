---
title: 下一代前端开发利器——Vite
date: 2023-08-04T11:13:00+08:00
updated: 2024-08-21T10:32:45+08:00
dg-publish: false
aliases:
  - 下一代前端开发利器——Vite
author: Luwang
category: web
comments: true
cover: //cdn.wallleap.cn/img/post/1.jpg
description: 文章描述
image-auto-upload: true
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
rating: 1
source: #
tags:
  - Vite
  - web
url: //myblog.wallleap.cn/post/1
---

![](https://cdn.wallleap.cn/img/pic/illustration/202308041114320.png)

## 前言

Hi，大家好。今年将尝使用掘金持续更新一些文章，希望我能做到。

前段时间用 `Vue3` 搭建项目时看到同时推出的 `Vite`，只当它是一个新打包工具或者 `vue-cli` 的升级版，仍然选择了用 `Webpack` 构建项目。最近看了尤雨溪在 VueConf 上的演讲视频：[《Vue3 生态进展和计划》](https://link.juejin.cn/?target=https%3A%2F%2Fwww.yuque.com%2Fvueconf%2Fmkwv0c%2Fxqyxix "https://www.yuque.com/vueconf/mkwv0c/xqyxix")，感觉它确实解决了现阶段前端工程化的一些痛点，也能体会到尤雨溪对 `Vite` 的重视和大力推广的决心，再加上 `Vue` 本身的庞大用户基数，`Vite` 确实有可能成为下一代前端构建工具的突破口。

本文将讨论下 `Vite` 出现的背景，解决的痛点，核心功能的实现，存在的意义和预期的未来。`Vite` 本身并不复杂。中文官方文档非常清晰简洁，建议大家使用前仔细读下文档。

## 大纲

- 背景
- 什么是 Vite？
- 基本用法
- 实现原理
- 源码分析
- 优势与不足
- 与传统构建工具对比
- 兼容性
- 未来

## 背景

这里的背景介绍会从与 `Vite` 紧密相关的两个概念的发展史说起，一个是 `JavaScript` 的模块化标准，另一个是前端构建工具。

### 共存的模块化标准

为什么 `JavaScript` 会有多种共存的模块化标准？因为 js 在设计之初并没有模块化的概念，随着前端业务复杂度不断提高，模块化越来越受到开发者的重视，社区开始涌现多种模块化解决方案，它们相互借鉴，也争议不断，形成多个派系，从 `CommonJS` 开始，到 `ES6` 正式推出 `ES Modules` 规范结束，所有争论，终成历史，`ES Modules` 也成为前端重要的基础设施。

- **CommonJS**：现主要用于 Node.js（[Node@13.2.0](https://link.juejin.cn/?target=mailto%3ANode%4013.2.0 "mailto:Node@13.2.0") 开始支持直接使用 ES Module）
- **AMD**：`require.js` 依赖前置，市场存量不建议使用
- **CMD**：`sea.js` 就近执行，市场存量不建议使用
- **ES Module**：ES 语言规范，标准，趋势，未来 感兴模块化发展史可以看下当事人写的 [《前端模块化开发那点历史》@玉伯](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fseajs%2Fseajs%2Fissues%2F588 "https://github.com/seajs/seajs/issues/588")，而 `Vite` 的核心正是依靠浏览器对 ES Module 规范的实现。

### 发展中的构建工具

近些年前端工程化发展迅速，各种构建工具层出不穷，我按 npm 发版的时间列出的开发者比较熟知的一些构建工具，目前在前端构建工具中 `Webpack` 仍然占据统治地位，npm 周下载量达到两千多万次。

![](https://cdn.wallleap.cn/img/pic/illustration/202308041114321.png)

### 当前工程化痛点

现在常用的构建工具如 `Webpack`，主要是通过抓取 - 编译 - 构建整个应用的代码（也就是常说的打包过程），生成一份编译、优化后能良好兼容各个浏览器的的生产环境代码。在开发环境流程也基本相同，需要先将整个应用构建打包后，再把打包后的代码交给 `dev server`（开发服务器）。

`Webpack` 等构建工具的诞生给前端开发带来了极大的便利，但随着前端业务的复杂化，js 代码量呈指数增长，打包构建时间越来越久，`dev server`（开发服务器）性能遇到瓶颈：

- **缓慢的服务启动：** 大型项目中 `dev server` 启动时间达到几十秒甚至几分钟。
- **缓慢的 HMR 热更新：** 即使采用了 HMR 模式，其热更新速度也会随着应用规模的增长而显著下降，已达到性能瓶颈，无多少优化空间。

**缓慢的开发环境，大大降低了开发者的幸福感，在以上背景下 `Vite` 应运而生。**

## 什么是 Vite？

**基于 esbuild 与 Rollup，依靠浏览器自身 ESM 编译功能， 实现极致开发体验的新一代构建工具！**

### 概念

先介绍以下文中会经常提到的一些基础概念：

- **依赖：** 指开发不会变动的部分 (npm 包、UI 组件库)，esbuild 进行预构建。
- **源码：** 浏览器不能直接执行的开发环境代码 (.jsx、.css、.vue 等)，vite 只在浏览器请求相关源码的时候进行转换，以提供 ESM 源码。

### 开发环境

- 利用浏览器原生的 `ES Module` 编译能力，省略费时的编译环节，直给浏览器开发环境源码，`dev server` 只提供轻量服务。
- 浏览器执行 ESM 的 `import` 时，会向 `dev server` 发起该模块的 `ajax` 请求，服务器对源码做简单处理后返回给浏览器。
- `Vite` 中 HMR 是在原生 ESM 上执行的。当编辑一个文件时，Vite 只需要精确地使已编辑的模块失活，使得无论应用大小如何，HMR 始终能保持快速更新。
- 使用 `esbuild` 处理项目依赖，`esbuild` 使用 go 编写，比一般 `node.js` 编写的编译器快几个数量级。

### 生产环境

- 集成 `Rollup` 打包生产环境代码，依赖其成熟稳定的生态与更简洁的插件机制。

### 处理流程对比

`Webpack` 通过先将整个应用打包，再将打包后代码提供给 `dev server`，开发者才能开始开发。

![](https://cdn.wallleap.cn/img/pic/illustration/202308041114322.png)

`Vite` 直接将源码交给浏览器，实现 `dev server` 秒开，浏览器显示页面需要相关模块时，再向 `dev server` 发起请求，服务器简单处理后，将该模块返回给浏览器，实现真正意义的按需加载。 ![](https://cdn.wallleap.cn/img/pic/illustration/202308041114323.png)

## 基本用法

### 1. 创建 vite 项目

```sh
npm create vite@latest
```

### 2. 选取模板

`Vite` 内置 6 种常用模板与对应的 TS 版本，可满足前端大部分开发场景，可以点击下列表格中模板直接在 [StackBlitz](https://link.juejin.cn/?target=https%3A%2F%2Fvite.new%2F "https://vite.new/") 中在线试用，还有其他更多的 [社区维护模板](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fvitejs%2Fawesome-vite%23templates "https://github.com/vitejs/awesome-vite#templates") 可以使用。

| JavaScript | TypeScript |
| --- | --- |
| [vanilla](https://link.juejin.cn/?target=https%3A%2F%2Fvite.new%2Fvanilla "https://vite.new/vanilla") | [vanilla-ts](https://link.juejin.cn/?target=https%3A%2F%2Fvite.new%2Fvanilla-ts "https://vite.new/vanilla-ts") |
| [vue](https://link.juejin.cn/?target=https%3A%2F%2Fvite.new%2Fvue "https://vite.new/vue") | [vue-ts](https://link.juejin.cn/?target=https%3A%2F%2Fvite.new%2Fvue-ts "https://vite.new/vue-ts") |
| [react](https://link.juejin.cn/?target=https%3A%2F%2Fvite.new%2Freact "https://vite.new/react") | [react-ts](https://link.juejin.cn/?target=https%3A%2F%2Fvite.new%2Freact-ts "https://vite.new/react-ts") |
| [preact](https://link.juejin.cn/?target=https%3A%2F%2Fvite.new%2Fpreact "https://vite.new/preact") | [preact-ts](https://link.juejin.cn/?target=https%3A%2F%2Fvite.new%2Fpreact-ts "https://vite.new/preact-ts") |
| [lit](https://link.juejin.cn/?target=https%3A%2F%2Fvite.new%2Flit "https://vite.new/lit") | [lit-ts](https://link.juejin.cn/?target=https%3A%2F%2Fvite.new%2Flit-ts "https://vite.new/lit-ts") |
| [svelte](https://link.juejin.cn/?target=https%3A%2F%2Fvite.new%2Fsvelte "https://vite.new/svelte") | [svelte-ts](https://link.juejin.cn/?target=https%3A%2F%2Fvite.new%2Fsvelte-ts "https://vite.new/svelte-ts") |

### 3. 启动

```json
{
  "scripts": {
    "dev": "vite", // 启动开发服务器，别名：`vite dev`，`vite serve`
    "build": "vite build", // 为生产环境构建产物
    "preview": "vite preview" // 本地预览生产构建产物
  }
}
```

## 实现原理

### ESbuild 编译

`esbuild` 使用 go 编写，cpu 密集下更具性能优势，编译速度更快，以下摘自官网的构建速度对比：

**浏览器：“开始了吗？”**

**服务器：“已经结束了。”**

**开发者：“好快，好喜欢！！”**

![](https://cdn.wallleap.cn/img/pic/illustration/202308041114324.png)

### 依赖预构建

- **模块化兼容：** 如开头背景所写，现仍共存多种模块化标准代码，`Vite` 在预构建阶段将依赖中各种其他模块化规范 (CommonJS、UMD) 转换 成 ESM，以提供给浏览器。
- **性能优化：** npm 包中大量的 ESM 代码，大量的 `import` 请求，会造成网络拥塞。`Vite` 使用 `esbuild`，将有大量内部模块的 ESM 关系转换成单个模块，以减少 `import` 模块请求次数。

### 按需加载

- 服务器只在接受到 import 请求的时候，才会编译对应的文件，将 ESM 源码返回给浏览器，实现真正的按需加载。

### 缓存

- **HTTP 缓存：** 充分利用 `http` 缓存做优化，依赖（不会变动的代码）部分用 max-age,immutable **强缓存**，源码部分用 304**协商缓存**，提升页面打开速度。
- **文件系统缓存：** `Vite` 在预构建阶段，将构建后的依赖缓存到 `node_modules/.vite` ，相关配置更改时，或手动控制时才会重新构建，以提升预构建速度。

### 重写模块路径

浏览器 `import` 只能引入相对/绝对路径，而开发代码经常使用 `npm` 包名直接引入 `node_module` 中的模块，需要做路径转换后交给浏览器。

- `es-module-lexer` 扫描 import 语法
- `magic-string` 重写模块的引入路径

```js
// 开发代码
import { createApp } from 'vue'

// 转换后
import { createApp } from '/node_modules/vue/dist/vue.js'
```

## 源码分析

与 `Webpack-dev-server` 类似 `Vite` 同样使用 `WebSocket` 与客户端建立连接，实现热更新，源码实现基本可分为两部分，源码位置在:

- `vite/packages/vite/src/client` client（用于客户端）
- `vite/packages/vite/src/node` server（用于开发服务器）

client 代码会在启动服务时注入到客户端，用于客户端对于 `WebSocket` 消息的处理（如更新页面某个模块、刷新页面）；server 代码是服务端逻辑，用于处理代码的构建与页面模块的请求。

简单看了下源码（[vite@2.7.2](https://link.juejin.cn/?target=mailto%3Avite%402.7.2 "mailto:vite@2.7.2")），核心功能主要是以下几个方法（以下为源码截取，部分逻辑做了删减）：

命令行启动服务 `npm run dev` 后，源码执行 `cli.ts`，调用 `createServer` 方法，创建 http 服务，监听开发服务器端口。

```js
// 源码位置 vite/packages/vite/src/node/cli.ts
const { createServer } = await import('./server')
try {
    const server = await createServer({
        root,
        base: options.base,
        ...
    })
    if (!server.httpServer) {
        throw new Error('HTTP server not available')
    }
    await server.listen()
}
```

`createServer` 方法的执行做了很多工作，如整合配置项、创建 http 服务（早期通过 koa 创建）、创建 `WebSocket` 服务、创建源码的文件监听、插件执行、optimize 优化等。下面注释中标出。

```js
// 源码位置 vite/packages/vite/src/node/server/index.ts
export async function createServer(
    inlineConfig: InlineConfig = {}
): Promise<ViteDevServer> {
    // Vite 配置整合
    const config = await resolveConfig(inlineConfig, 'serve', 'development')
    const root = config.root
    const serverConfig = config.server

    // 创建http服务
    const httpServer = await resolveHttpServer(serverConfig, middlewares, httpsOptions)

    // 创建ws服务
    const ws = createWebSocketServer(httpServer, config, httpsOptions)

    // 创建watcher，设置代码文件监听
    const watcher = chokidar.watch(path.resolve(root), {
        ignored: [
            '**/node_modules/**',
            '**/.git/**',
            ...(Array.isArray(ignored) ? ignored : [ignored])
        ],
        ...watchOptions
    }) as FSWatcher

    // 创建server对象
    const server: ViteDevServer = {
        config,
        middlewares,
        httpServer,
        watcher,
        ws,
        moduleGraph,
        listen,
        ...
    }

    // 文件监听变动，websocket向前端通信
    watcher.on('change', async (file) => {
        ...
        handleHMRUpdate()
    })

    // 非常多的 middleware
    middlewares.use(...)
    
    // optimize
    const runOptimize = async () => {...}

    return server
}
```

使用 [chokidar](https://link.juejin.cn/?target=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2Fchokidar "https://www.npmjs.com/package/chokidar") 监听文件变化，绑定监听事件。

```js
// 源码位置 vite/packages/vite/src/node/server/index.ts
  const watcher = chokidar.watch(path.resolve(root), {
    ignored: [
      '**/node_modules/**',
      '**/.git/**',
      ...(Array.isArray(ignored) ? ignored : [ignored])
    ],
    ignoreInitial: true,
    ignorePermissionErrors: true,
    disableGlobbing: true,
    ...watchOptions
  }) as FSWatcher
```

通过 [ws](https://link.juejin.cn/?target=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2Fws "https://www.npmjs.com/package/ws") 来创建 `WebSocket` 服务，用于监听到文件变化时触发热更新，向客户端发送消息。

```js
// 源码位置 vite/packages/vite/src/node/server/ws.ts
export function createWebSocketServer(...){
    let wss: WebSocket
    const hmr = isObject(config.server.hmr) && config.server.hmr
    const wsServer = (hmr && hmr.server) || server

    if (wsServer) {
        wss = new WebSocket({ noServer: true })
        wsServer.on('upgrade', (req, socket, head) => {
            // 服务就绪
            if (req.headers['sec-websocket-protocol'] === HMR_HEADER) {
                wss.handleUpgrade(req, socket as Socket, head, (ws) => {
                    wss.emit('connection', ws, req)
                })
            }
        })
    } else {
        ...
    }
    // 服务准备就绪，就能在浏览器控制台看到熟悉的打印 [vite] connected.
    wss.on('connection', (socket) => {
        socket.send(JSON.stringify({ type: 'connected' }))
        ...
    })
    // 失败
    wss.on('error', (e: Error & { code: string }) => {
        ...
    })
    // 返回ws对象
    return {
        on: wss.on.bind(wss),
        off: wss.off.bind(wss),
        // 向客户端发送信息
        // 多个客户端同时触发
        send(payload: HMRPayload) {
            const stringified = JSON.stringify(payload)
            wss.clients.forEach((client) => {
                // readyState 1 means the connection is open
                client.send(stringified)
            })
        }
    }
}
```

在服务启动时会向浏览器注入代码，用于处理客户端接收到的 `WebSocket` 消息，如重新发起模块请求、刷新页面。

```js
//源码位置 vite/packages/vite/src/client/client.ts
async function handleMessage(payload: HMRPayload) {
  switch (payload.type) {
    case 'connected':
      console.log(`[vite] connected.`)
      break
    case 'update':
      notifyListeners('vite:beforeUpdate', payload)
      ...
      break
    case 'custom': {
      notifyListeners(payload.event as CustomEventName<any>, payload.data)
      ...
      break
    }
    case 'full-reload':
      notifyListeners('vite:beforeFullReload', payload)
      ...
      break
    case 'prune':
      notifyListeners('vite:beforePrune', payload)
      ...
      break
    case 'error': {
      notifyListeners('vite:error', payload)
      ...
      break
    }
    default: {
      const check: never = payload
      return check
    }
  }
}
```

## 优势

- 快！快！非常快！！
- 高度集成，开箱即用。
- 基于 ESM 急速热更新，无需打包编译。
- 基于 `esbuild` 的依赖预处理，比 `Webpack` 等 node 编写的编译器快几个数量级。
- 兼容 `Rollup` 庞大的插件机制，插件开发更简洁。
- 不与 `Vue` 绑定，支持 `React` 等其他框架，独立的构建工具。
- 内置 SSR 支持。
- 天然支持 TS。

## 不足

- `Vue` 仍为第一优先支持，量身定做的编译插件，对 `React` 的支持不如 `Vue` 强大。
- 虽然已经推出 2.0 正式版，已经可以用于正式线上生产，但目前市场上实践少。
- 生产环境集成 `Rollup` 打包，与开发环境最终执行的代码不一致。

## 与 webpack 对比

由于 `Vite` 主打的是开发环境的极致体验，生产环境集成 `Rollup`，这里的对比主要是 `Webpack-dev-server` 与 `Vite-dev-server` 的对比：

- 到目前很长时间以来 `Webpack` 在前端工程领域占统治地位，`Vite` 推出以来备受关注，社区活跃，GitHub star 数量激增，目前达到 37.4K

GitHub star40%25%21%14%GitHub starWebpackViteGlupRollup

- `Webpack` 配置丰富使用极为灵活但上手成本高，`Vite` 开箱即用配置高度集成
- `Webpack` 启动服务需打包构建，速度慢，`Vite` 免编译可秒开
- `Webpack` 热更新需打包构建，速度慢，`Vite` 毫秒响应
- `Webpack` 成熟稳定、资源丰富、大量实践案例，`Vite` 实践较少
- `Vite` 使用 `esbuild` 编译，构建速度比 `webpack` 快几个数量级

## 兼容性

- 默认目标浏览器是在 `script` 标签上支持原生 ESM 和 原生 ESM 动态导入
- 可使用官方插件 `@vitejs/plugin-legacy`，转义成传统版本和相对应的 `polyfill`

## 未来探索

- 传统构建工具性能已到瓶颈，主打开发体验的 `Vite`，可能会受到欢迎。
- 主流浏览器基本支持 ESM，ESM 将成为主流。
- `Vite` 在 `Vue3.0` 代替 `vue-cli`，作为官方脚手架，会大大提高使用量。
- `Vite2.0` 推出后，已可以在实际项目中使用 `Vite`。
- 如果觉得直接使用 `Vite` 太冒险，又确实有 `dev server` 速度慢的问题需要解决，可以尝试用 `Vite` 单独搭建一套 `dev server`

## 相关资源

### 官方插件

除了支持现有的 `Rollup` 插件系统外，官方提供了四个最关键的插件

- `@vitejs/plugin-vue` 提供 Vue3 单文件组件支持
- `@vitejs/plugin-vue-jsx` 提供 Vue3 JSX 支持（专用的 Babel 转换插件）
- `@vitejs/plugin-react` 提供完整的 React 支持
- `@vitejs/plugin-legacy` 为打包后的文件提供传统浏览器兼容性支持

### UI 组件库

- [Element UI](https://link.juejin.cn/?target=https%3A%2F%2Felement-plus.gitee.io%2Fzh-CN%2Fguide%2Fquickstart.html%23%25E6%258C%2589%25E9%259C%2580%25E5%25AF%25BC%25E5%2585%25A5 "https://element-plus.gitee.io/zh-CN/guide/quickstart.html#%E6%8C%89%E9%9C%80%E5%AF%BC%E5%85%A5")：支持 vite 引入

### 相关链接

- [Vite官网](https://link.juejin.cn/?target=https%3A%2F%2Fcn.vitejs.dev%2F "https://cn.vitejs.dev/")
- [Vue3 生态进展和计划-尤雨溪](https://link.juejin.cn/?target=https%3A%2F%2Fwww.yuque.com%2Fvueconf%2Fmkwv0c%2Fxqyxix "https://www.yuque.com/vueconf/mkwv0c/xqyxix")
- [Vite源码解析](https://link.juejin.cn/?target=http%3A%2F%2Fvite.ssr-fc.com%2F "http://vite.ssr-fc.com/")
- [Develop with Vite | Vite快速入门 - Anthony Fu • Vue北京聚会 Day 13](https://link.juejin.cn/?target=https%3A%2F%2Fwww.youtube.com%2Fwatch%3Fv%3Dxx8gEHet6n8 "https://www.youtube.com/watch?v=xx8gEHet6n8")
