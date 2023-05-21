---
title: WebPack
date: 2023-05-15 09:30
updated: 2023-05-15 09:51
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - WebPack 的使用
rating: 1
tags:
  - web
category: web
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: null
url: //myblog.wallleap.cn/post/1
---

模块化之后，前端面临一个更大的问题——用户要下载太多文件，由此需要通过前端自动化来解决这个问题

![文件太多](https://cdn.wallleap.cn/img/pic/illustration/202305150932230.png)

## 文件太多怎么办

**前端自动化（借助 Node.js）**

- Concat - 拼接，把所有文件拼接成一个大文件
- Minify - 压缩，删除注释、空白、分号等以减小代码体积
- Uglify - 混淆，把局部变量 username 改为 a 以减小代码体积
- Treeshaking - 摇树，不打包没用到的导出变量
- Code Split - 分割，让可以推后加载的文件单独下载以加快首屏渲染

或者**优化网络**、让后端将 HTTP/1.1 升级为 **HTTP/2**、让用户将**浏览器升级到最新**

## 如何实现自动化

选择有很多：

- Browserify（已过时）
- Webpack（市场占有率高）
- Rollup（现在流行）
- Vite（很受欢迎）
- Parcel（零配置）
- create-react-app（老旧）
- `@vue/cli` （Vue 上个时代的自动化工具）

之后将以 Webpack 为例，来入门前端工程化

## 为什么选 Webpack

- 功能强大
	- 几乎满足了前端各种变态的要求
- 使用广泛
	- 大中小型公司都在用
- 生态繁荣
	- Webpack 周边插件应有尽有
- 缺点
	- 配置复杂，需要熟悉英文
	- 运行速度慢，需要多等待
	- 质量参差不齐，有很多坑

## 查看 Webpack 文档

<https://webpack.js.org/>

- 版本选择
	- 不同版本功能可能不同
- 安装
	- 全局安装
	- 局部安装 + scripts/npx
- Cli
	- 命令行的使用
- 概念
	- Entry、Output、Loader、Plugin、Mode
- 文档的作用
	- 先通看一遍，然后用的时候搜索关键词

DOCUMENTATION
webpack5
- API
- Concepts
- Configuration
- Guides
- Loaders
- Migrate
- Plugins

## 初始化一个 Webpack 项目

事先下载好 NVM，并通过 NVM 下载好 Node18（最新偶数版本）

下载好 pnpm

开始创建一个 Webpack 项目

```sh
mkdir webpack-demo-001
cd webpack-demo-001
pnpm init
pnpm config set save-prefix=""
pnpm install webpack webpack-cli -D
```

如何运行 webpack

```sh
npx webpack
./node_modules/.bin/webpack
```

init 的时候会创建 `package.json`，安装好插件之后 `devDependencies` 开发依赖中会默认加上

```json
{
  "name": "my-webpack-001",
  "version": "0.0.1",
  "description": "My webpack project",
-  "main": "src/index.js",
+  "private": true
  "scripts": {
    "build": "webpack",
    "test": "echo \"Error: no test specified\" && exit 1",
    "preview": "http-server . -c-1"
  },
  "keywords": [],
  "author": "luwang",
  "license": "ISC",
  "devDependencies": {
    "http-server": "14.1.1",
    "webpack": "5.82.1",
    "webpack-cli": "5.1.1"
  }
}
```

可以在 `"scripts"` 中加上 `"build": "webpack"`，这样运行 `pnpm run build` 的时候会自动找到 webpack 的位置，并运行

## 创建 Webpack 配置文件

在根目录下创建  `webpack.config.js` 文件

```js
const path = require('path')
module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js',
  }
}
```

CommonJS  的语法，导出了一个对象

配置了入口 entry 和 出口 output，入口默认是 `src/index.js`，出口路径使用到了 NodeJS 的 `path` 变量，需要导入（默认出口为 `dist/main.js`）

配置好之后，在 `src/index.js` 中随便写点内容，然后执行 `pnpm run build`

```sh
> my-webpack-001@0.0.1 build /Users/luwang/Workspace/code/my-webpack-001
> webpack

asset bundle.js 29 bytes [emitted] [minimized] (name: main)
./src/index.js 28 bytes [built] [code generated]

WARNING in configuration
The 'mode' option has not been set, webpack will fallback to 'production' for this value.
Set 'mode' option to 'development' or 'production' to enable defaults for each environment.
You can also set it to 'none' to disable any default behavior. Learn more: https://webpack.js.org/configuration/mode/

webpack 5.82.1 compiled with 1 warning in 115 ms
```

有个警告，可以加上 `mode` 选项，值可以为 `development` 或 `production`

```sh
npx webpack --mode production
./node_modules/.bin/webpack --mode production
# 多一个 -- 代表是里面的配置
pnpm run build -- --mode production
# 或者直接在 package.json 的配置项里加好 "build": "webpack --mode production"
pnpm run build
```

或者到配置文件 `webpack.config.js` 中配置

```js
module.exports = {
  mode: 'production'
}
```

这样也只用运行 `pnpm run build` 就可以了

- production 生产模式，代码给用户用，尽量优化、压缩、混淆
- development 开发模式，代码给开发者用，需要边写边预览，尽量方便开发、调试

## 开始任务驱动学习法

### index.html、main.js

把生成的 main.js 到 index.html 中引入

```html
<script src="./dist/main.js"></script>
```

下载插件

```sh
pnpm install http-server -D
```

修改配置文件，添加 `"preview": "http-server . -c-1"` 脚本，并运行命令预览

```sh
pnpm run preview
```

### 边开发，边预览

使用 `npx webpack --watch` 命令或 `npx webpack watch`

```json
{
  "scripts": {
    "watch": "webpack --watch",
  },
}
```

运行

```sh
pnpm run watch
# 另一个终端运行
pnpm run preview
```

### 自动运行 server

安装依赖，并运行命令

```sh
pnpm install webpack-dev-server -D
npx webpack serve
```

配置 package.json 的脚本

```json
{
  "scripts": {
    "serve": "webpack serve",
  },
}
```

运行

```sh
pnpm run serve
```

前端开发默认端口 8080，后端 3000

可以加参数

```sh
npx webpack serve --open
```

会帮忙做以下事情
- watch：监听文件的变化
- build：文件一变就重新打包
- server：开启 http server
- open：自动打开浏览器页面

### 让 index.html 可以访问

上次运行 serve 命令输出了

```sh
> my-webpack-001@0.0.1 serve /Users/luwang/Workspace/code/my-webpack-001
> webpack serve --open

<i> [webpack-dev-server] Project is running at:
<i> [webpack-dev-server] Loopback: http://localhost:8080/
<i> [webpack-dev-server] On Your Network (IPv4): http://10.1.209.63:8080/
<i> [webpack-dev-server] On Your Network (IPv6): http://[fe80::1]:8080/
<i> [webpack-dev-server] Content not from webpack is served from '/Users/luwang/Workspace/code/my-webpack-001/public' directory
<i> [webpack-dev-middleware] wait until bundle finished: /
asset main.js 122 KiB [emitted] [minimized] (name: main) 1 related asset
orphan modules 31.7 KiB [orphan] 13 modules
runtime modules 27.3 KiB 12 modules
cacheable modules 176 KiB
```

所以需要把 `index.html` 放到 `public` 目录下

移动完成之后重新运行

```sh
pnpm run serve
```

但是在 build 之后发现并没有生成 HTML

需要安装插件

[HtmlWebpackPlugin](https://webpack.js.org/plugins/html-webpack-plugin/)

```sh
pnpm add -D html-webpack-plugin
```

修改配置文件

```js
const HtmlWebpackPlugin = require('html-webpack-plugin')
module.exports = {
  mode: 'production',
  plugins: [
    plugins: [
    new HtmlWebpackPlugin({
      title: 'Webpack Demo',
      filename: 'index.html',
      template: './public/index.html',
      favicon: "./public/favicon.ico",
      inject: 'body',
    }),
  ]
}
```

这样 build 的时候就会自动生成 index.html 和 favicon.ico 了

### 引入第三方库

例如 axios、Vue、React

先安装

```sh
pnpm install axios
pnpm install vue@3
pnpm install react react-dom
```

(查看信息和版本 `pnpm info vue`、`pnpm info vue versions`)

导入

```js
// index.js
import axios from 'axios'
import * as vue, { createApp } from 'vue'
import React from 'react'
import ReactDom from 'react-dom'
```

`console.log()` 一下，看下导入是否是需要的东西

总结：
**安装依赖**：
- 如果是开发时才会用的，就用命令 `pnpm install -D xxx@x.x.x`
- 如果是用户浏览器上用的，就用 `pnpm install xxx@x.x.x`

**引入**
- `import x from 'xxx'`
- `import {x} from 'xxx'`
- `import x as y from 'xxx'`

### 使用 CSS

在 `index.js` 中引入新建的 `style.css`

```js
import './style.css'
```

运行 serve，提示需要 loader

搜索 webpack css loader

[css-loader | webpack](https://webpack.js.org/loaders/css-loader/)

按照操作，发现有两个 loader，所以先添加

```sh
pnpm add -D css-loader style-loader
```

然后修改配置文件 webpack.config.js

```js
module.exports = {
  mode: 'development',
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: ["style-loader", "css-loader"],
      },
    ],
  },
}
```

会将 CSS 先转化成 JS 字符串，再创建一个 style 标签，把 JS 字符串放到 style 标签中

### 将 CSS 单独打包

打包的时候就会在 JS 中

使用插件 [MiniCssExtractPlugin](https://webpack.js.org/plugins/mini-css-extract-plugin/) 提取 CSS 文件

安装插件

```sh
pnpm add -D mini-css-extract-plugin
```

只有在打包的时候需要提取 CSS

拆分配置文件 webpack.config.prod.js

```js
const MiniCssExtractPlugin = require("mini-css-extract-plugin")

module.exports = {
  mode: 'development',
  plugins: [new MiniCssExtractPlugin()],
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: ["style-loader", "css-loader"],
      },
      {
        test: /\.css$/i,
        use: [MiniCssExtractPlugin.loader, "css-loader"],
      },
    ],
  },
}
```

webpack.config.dev.js

```js
module.exports = {
  mode: 'development',
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: ["style-loader", "css-loader"],
      },
    ],
  },
}
```

修改 package.json 脚本

```json
{
"scripts": {
    "serve": "webpack serve -c webpack.config.dev.js --open",
    "build": "webpack -c webpack.config.prod.js"
  },
}
```

打包和开发的时候用不同的配置文件

### 加入一个 JPG/PNG/GIF 图片

在 `src/images` 中加入一张 png 图片，然后到 `index.js` 中引入它

```js
import logo from './images/webpack-logo.png'
```

运行 serve，发现没有 loader 能够加载它，搜索 webpack png loader

使用的是 [file-loader](https://v4.webpack.js.org/loaders/file-loader/)

先安装

```sh
pnpm add -D file-loader
```

在配置文件中的 rules 数组中多加一个

```js
       {
        test: /\.(png|jpe?g|gif)$/i,
        use: [
          {
            loader: 'file-loader',
          },
        ],
      },
```

运行 serve，发现打印出来是个 URL，所以自己创建一个 img 标签，设置 src 为这个 logo，插入到页面中

总结：
loader load a file（加载一类文件）
plugin extend webpack's ability（扩展 webpack 功能）

### 按需加载某些模块

```js
import('./a.js').then(()=>{},()=>{})
```

使用 `import()`，等用户点击或触发的时候才加载，打包的时候会单独打包为一个文件

## 总结

- Webpack 是个空架子，需要自己下载 loader 或 plugins 去配置
- 不需要重点去学，可以给自己设定一个任务，然后去官方文档或 GitHub 搜索，配置解决这个任务
- 开发模式 mode 是 development，用 serve/dev（也会 build，但是是在内存中）
- 生产模式 mode 是 production，用 build，文件必打包必加后缀
