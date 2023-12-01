---
title: 001-使用 Vue-cli 创建项目
date: 2023-08-15 17:56
updated: 2023-08-15 17:56
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - 001-使用 Vue-cli 创建项目
rating: 1
tags:
  - Vue
  - web
category: web
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: #
url: //myblog.wallleap.cn/post/1
---

单文件组件：一个文件代表一个组件

使用 webpack 进行配置

使用 Vue-cli 脚手架搭建项目，快速创建 Vue 项目模板，包含一些预定义的配置

安装

```sh
npm install -g @vue/cli
vue -V
```

创建项目

```sh
vue create demo01
```

可以自选设置

启动

```sh
# 启动
cd demo01
npm run serve
npm run build
```

目录结构

```
demo01/
|babel.config.js
|package.json
|src/ → 源码
|-- main.js → 项目入口
|-- App.vue
|-- assets/
|-- components/
|-- |-- HelloWorld.vue
|public/
|-- index.html
```

main.js 入口

```js
import { createApp } from 'Vue'
import App from './App.vue'

createApp(App)
  .mount('#app')
```

index.html

```html
...
<div id="app"></div>
...
```

单文件组件：一个文件是一个组件，后缀是 `.vue`

模板放在 `<template>` 里，JS 放 `<script>`，CSS 放 `<style>` 里

App.vue

```vue
<script>
import HelloWorld from './components/HelloWorld.vue'

export default {
  name: 'App',
  components: {
    HelloWorld
  }
}
</script>

<template>
  <img src="./assets/logo.png" alt="logo" />
  <HelloWorld msg="Welcome" />
</template>

<style>
  img {
    object-fit: cover;
  }
</style>
```

HelloWorld.vue

```vue
<script>
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  }
}
</script>

<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
  </div>
</template>

<style>
.hello {
  color: #333;
}

.hello h1 {
  font-size: 2rem;
}
</style>
```

引入 sass

需要安装相应版本（和 webpack 版本匹配的）的 sass-loader 和 sass/node-sass（两者选一个，sass 安装更快一点）

```sh
npm install --save-dev sass-loader@7 sass
```

使用

```vue
<style lang="scss">
.hello {
  $title-size: 2rem;
  $title-color: #333;
  color: $title-color;
  h1 {
    font-size: $title-size;
  }
}
</style>
```

在 `style` 标签上加 `scoped` 属性，这样写的样式只对当前模板生效

原理：编译后模版会增加随机的自定义属性，编译后的 CSS 选择器上也会加上该随机属性（例如 `.hello[data-v-f3f9eg8]`）

CLI 服务是构建于 webpack he webpack-dev-server 之上的，包含了

- 加载其他 CLI 插件的核心服务
- 一个针对绝大部分应用优化过的内部的 webpack 配置
- 项目内部的 `vue-cli-service` 命令，提供 `serve`、`build` 和 `inspect` 命令

运行 `@vue/cli-service` 可以

- 到包里找到 bin 下的文件运行
- 使用 npm script 脚本运行（`npm run serve`）
- 使用 npx 运行（`npx vue-cli-service serve`）

运行 `npm run serve -- --help` 可以查看后面可跟的参数等

额外配置：项目根目录下新建 `vue.config.js`

![](https://cdn.wallleap.cn/img/pic/illustration/202308161035533.png)
![webpack 配置](https://cdn.wallleap.cn/img/pic/illustration/202308161041317.png)
![](https://cdn.wallleap.cn/img/pic/illustration/202308161043095.png)

