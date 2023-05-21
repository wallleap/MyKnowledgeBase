---
title: VuePress 搭建组件库文档
date: 2022-11-10 15:20
updated: 2022-11-10 15:23
cover: //cdn.wallleap.cn/img/pic.jpg
author: Luwang
comments: true
aliases:
  - 文档系统
rating: 10
tags:
  - web
  - 文档
category: web
keywords:
  - web
  - VuePress
  - 组件库
  - 文档
description: 文章描述
source: //juejin.cn/user/1169536104532829
url: null
---

[![img](https://cdn.wallleap.cn/img/pic/illustrtion/202210091748716.webp)](https://juejin.cn/user/1169536104532829)

[65岁退休Coder![lv-4](https://cdn.wallleap.cn/img/pic/illustrtion/202210091748626.png)](https://juejin.cn/user/1169536104532829)

### 前言

前段时间弄了组件库 demo 以及私有 npm 包托管的一些内容，是时候需要一个在线文档了。也算是走了一个组件库开发的闭环，留下点内容，方便自己也希望能给小伙伴们一些帮助。

[基于VueCli开发vue组件库](https://juejin.cn/post/7057381682474647560)

[Verdaccio搭建私有npm仓库](https://juejin.cn/post/7057381682474647560)

文档方面本次同样使用 `vue` 相关技术栈 `vuepress` 。

> `vuepress` 基于 Vue 的静态网站生成器。它诞生的初衷就是为了支持 Vue 及其子项目的文档需求。
>
> so 用它真是在合适不过了。当然它也可以做一些其他的文档，比如个人博客等... ...

项目预览地址以及项目地址如下：

- [预览](https://link.juejin.cn/?target=https%3A%2F%2Fshuqingx.github.io%2Fvue-comp-test%2F)
- [源码](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FShuQingX%2Fvue-comp-test)

### 开始

1，创建 `comp-vuepress` 项目并执行 `npm init -y`。

2，安装 `VuePress` 为本地依赖。

```bash
npm install vuepress -D
复制代码
```

3，根目录下创建 `docs` 文件夹并在内部创建 `README.md` 文件，随意输入内容。

4，`package.json` 添加运行脚本以及打包脚本。

```json
"scripts": {
	"docs:dev": "vuepress dev docs",
	"docs:build": "vuepress build docs"
}
复制代码
```

5，运行 `npm run docs:dev` `vuepress` 会在 `http://localhost:8080` 运行服务。

![1.png](https://cdn.wallleap.cn/img/pic/illustrtion/202210091748836.webp)

### 基本配置

在文档目录下创建一个 `.vuepress` 的文件夹，所有 VuePress 相关的文件都将会被放在这里。你的项目结构可能是这样：

```go
.
├─ docs
│  ├─ README.md
│  └─ .vuepress
└─ package.json
复制代码
```

在 `.vuepress` 中创建 `config.js` 这是 VuePress 中必备的配置文件。应该导出一个对象，内容如下：

```js
module.exports = {
  title: 'comp-vuepress',
  description: 'comp 组件库文档。'
}
复制代码
```

### 导航栏

[导航栏](https://link.juejin.cn/?target=https%3A%2F%2Fvuepress.vuejs.org%2Fzh%2Ftheme%2Fdefault-theme-config.html%23%E5%AF%BC%E8%88%AA%E6%A0%8F) 配置主要就两种方式，一级导航和下拉列表导航。修改 `config.js`。

```js
module.exports = {
  // ... ...
  themeConfig: {
    nav: [
      // 一级导航
      { text: '指南', link: '/guide/' },
      // 下拉列表导航
      {
        text: '了解更多',
        items: [
          { text: 'github', link: 'https://github.com/ShuQingX/vue-comp-test', target: '_blank' },
          { text: 'preview', link: 'https://shuqingx.github.io/vue-comp-test/', target: '_blank' }
        ]
      }
    ]
    // 禁用导航，与上面的配置是互斥行为。
    // navbar: false
  }
};
复制代码
```

![2.png](https://cdn.wallleap.cn/img/pic/illustrtion/202210091748557.webp)

#### 小插曲（路由）

经过上面的配置相信会有一些小伙伴点击了 “ 指南 ” ，发现 404 了。主要是 `/guide/` 我们并没有配置，而 VuePress 遵循 **约定优于配置** 的原则。我们需要新建一个 guide 目录。

```lua
.
├─ docs
│   ├──	README.md
│   ├── guide
│   │   └── README.md
│   └── config.md
└─ package.json
复制代码
```

举个🌰：

- `/guide/README.md` 的查找规则为 `/guide/`
- `/guide.md` 的查找规则为 `/guide.html`

`README.md` 就像 JavaScript 中的 index.js 一样，有没有。

### 侧边栏

想要使 [侧边栏（Sidebar）](https://link.juejin.cn/?target=https%3A%2F%2Fvuepress.vuejs.org%2Fzh%2Ftheme%2Fdefault-theme-config.html%23%E4%BE%A7%E8%BE%B9%E6%A0%8F)生效，需要配置 `themeConfig.sidebar`，基本的配置，需要一个包含了多个链接的数组：

> 总体来说侧边栏大致分为两大类，一种根据标题生成，一种根据配置生成。根据配置生成的方式比较灵活，强烈建议反复查阅文档。

#### 配置生成侧边栏

1，首先在 `guide` 中创建 `Button.md` `Card.md` 文件。

```css
.
├─ docs
│   ├──	README.md
│   ├── guide
│   │   └── README.md
│   │   └── Button.md
│   │   └── Card.md
└─ package.json
复制代码
```

2，修改 `config.js` 文件

```js
module.exports = {
    themeConfig: {
        sidebar: {
          '/guide/': [
            ['', '介绍'], // '' 等价于 /guide/
            {
              title: '组件',
              collapsable: false,
              children: [
                ['../guide/Button.md', 'Button'],
                ['../guide/Card.md', 'Card']
              ]
            }
          ]
        }
    }
}
复制代码
```

![3.png](https://cdn.wallleap.cn/img/pic/illustrtion/202210091748664.webp)

#### 标题生成侧边栏

默认情况下，侧边栏会自动地显示由当前页面的标题（headers）组成的链接，并按照页面本身的结构进行嵌套，你可以通过 `themeConfig.sidebarDepth` 来修改它的行为。

- 默认的深度是 1，它将提取到 h2 的标题；
- 设置成 0 将会禁用标题（headers）链接；
- 最大的深度为 2，它将同时提取 h2 和 h3 标题。

很显然如果我们直接改配置文件，它就作用到了所有的文件中，此时可以使用 `YAML front matter` 来为某个页面重写此值：

```yaml
---
sidebarDepth: 2
---
复制代码
```

重写后随意加几个二三级标题，就会得到以下内容

![4.png](https://cdn.wallleap.cn/img/pic/illustrtion/202210091748092.webp)

### 首页

经过上面的折腾，至此我们的 `guide` 相关的侧边栏，导航栏都已经完成了。紧接着就搞一个和 `VuePress` 一样的首页吧。

默认主题的 [首页](https://link.juejin.cn/?target=https%3A%2F%2Fvuepress.vuejs.org%2Fzh%2Ftheme%2Fdefault-theme-config.html%23%E9%A6%96%E9%A1%B5) 是提供了一套默认的配置。 但是需要注意使用的前提。

- 根级的 `README.md` 中存在。
- `home: true` 是必须的。

```md
---
home: true
heroImage: /hero.png   # 注意这里可以使外链，也可以是静态资源
heroText: Hero 标题
tagline: Hero 副标题
actionText: 快速上手 →
actionLink: /guide/  # 这里是点击快速上手后的跳转链接。
features:
- title: 简洁至上
  details: 以 Markdown 为中心的项目结构，以最少的配置帮助你专注于写作。
- title: Vue驱动
  details: 享受 Vue + webpack 的开发体验，在 Markdown 中使用 Vue 组件，同时可以使用 Vue 来开发自定义主题。
- title: 高性能
  details: VuePress 为每个页面预渲染生成静态的 HTML，同时在页面被加载的时候，将作为 SPA 运行。
footer: MIT Licensed | Copyright © 2018-present Evan You
---
复制代码
```

#### 小插曲 (静态资源)

此时当你查看服务的时候，必然会出现图片没有显示的问题。随后把 `homeImage` 的值进行更改，但前提要在 `.vuepress` 中新建一个 `public` 文件夹存储咱们的静态资源。

此时项目的结构应该是下面这样：

```arduino
.
├─ docs
│  ├── .vuepress
│  │   ├── config.js
│  │   └── public
│  ├── guide
│  │   ├── Button.md
│  │   ├── Card.md
│  │   └── README.md
│  └─	README.md
└─ package.json
复制代码
```

此时如果你想使用 `public` 中的资源，可以直接引用，示例：`/hero.png`，当然此种方案并不适用静态资源较多的情况，因为它并不会被打包。当然还有 [另一种方案](https://link.juejin.cn/?target=https%3A%2F%2Fvuepress.vuejs.org%2Fzh%2Fguide%2Fassets.html%23%E7%9B%B8%E5%AF%B9%E8%B7%AF%E5%BE%84) ，这里不展示了。

![5.png](https://cdn.wallleap.cn/img/pic/illustrtion/202210091748231.webp)

#### 小插曲（样式）

此时我们发现图片显示的有点太大了，不够协调。并且我也不想要 Vue 的这种绿色，当然 `VuePress` 还是提供了方案的。

关于样式，主题颜色的修改，需要在 `.vuepress` 中新建 `styles` 文件夹。**注意** 内部的样式文件是 `stylus` 文件。

- 如果想要修改，添加一些样式可以 `styles` 中新建 `index.styl`。当然也支持导入其他的 css 文件。[参考](https://link.juejin.cn/?target=https%3A%2F%2Fvuepress.vuejs.org%2Fzh%2Fconfig%2F%23index-styl)
- 如果想要修改一些预设（内置的 css 变量）可以在 `styles` 中新建 `palette.styl`。**注意：** 可以修改以下变量

```stylus
// 颜色
$accentColor = #3eaf7c
$textColor = #2c3e50
$borderColor = #eaecef
$codeBgColor = #282c34
$arrowBgColor = #ccc
$badgeTipColor = #42b983
$badgeWarningColor = darken(#ffe564, 35%)
$badgeErrorColor = #DA5961

// 布局
$navbarHeight = 3.6rem
$sidebarWidth = 20rem
$contentWidth = 740px
$homePageWidth = 960px

// 响应式变化点
$MQNarrow = 959px
$MQMobile = 719px
$MQMobileNarrow = 419px
复制代码
```

此时项目的结构应该是下面这样：

```arduino
.
├─ docs
│  ├── .vuepress
│  │   ├── config.js
│  │   ├── styles
│  │   │	├── index.styl
│  │   │	└── palette.styl
│  │   └── public
│  ├── guide
│  │   ├── Button.md
│  │   ├── Card.md
│  │   └── README.md
│  └─	README.md
└─ package.json
复制代码
```

举个🌰：

- 假设我们想要修改图片的大小。

```stylus
.home .hero img
  width: 500px
复制代码
```

- 假设我们想要修改一些变量直接在 `palette.styl` 中进行覆盖即可。

### markdown 语法拓展

[语法拓展](https://link.juejin.cn/?target=https%3A%2F%2Fvuepress.vuejs.org%2Fzh%2Fguide%2Fmarkdown.html) 相信 markdown 的语法，大部分小伙伴都是了解一二的，这里是 `VuePress` 中的一些拓展语法，大部分语法都是比较简单的，这里重点看下文件导入。

#### 小插曲（components、md in Vue）

文件导入之前我们需要了解一个小知识：

- `docs/.vuepress/components` 该目录中的 Vue 组件将会被自动注册为全局组件。换句话说就是在这个文件夹中的 `*.vue` 是可以在 `*.md` 中当做组件被使用的。

```
components/demo-1.vue` 在使用时则是 `<demo-1></demo-1>
components/guide/demo1.vue` 在使用时则是 `<guide-demo1></guide-demo1>
```

- `*.md` 中也是会支持 `Vue` 的模板语法。原因是：Markdown 文件将首先被编译成 HTML，接着作为一个 Vue 组件传入 `vue-loader`

这意味着你可能会写出以下代码。

![6.png](https://cdn.wallleap.cn/img/pic/illustrtion/202210091748829.webp)

如果不想让它编译，仍然是使用 markdown 拓展语法。

```md
::: v-pre
{{ 1 + 2 }}
:::
复制代码
```

#### 代码块导入

假设我们此时要完善 `Button` 组件的文档。首先创建 `guide/button/demo1.js` 文件夹存放 demo ，接着随意输入一些代码。然后再 `Button.md` 中进行导入操作 导入的语法为 `<<< @/xxx/xx/x`

- `<<<` 特定语法
- `@` 的默认值为 `process.cwd()`

![7.png](https://cdn.wallleap.cn/img/pic/illustrtion/202210091748108.webp)

### 引入外部组件（应用级别配置）

**官方：** 由于 VuePress 是一个标准的 Vue 应用，你可以通过创建一个 `.vuepress/enhanceApp.js`文件来做一些应用级别的配置，当该文件存在的时候，会被导入到应用内部。`enhanceApp.js` 应该 `export default`一个钩子函数，并接受一个包含了一些应用级别属性的对象作为参数。你可以使用这个钩子来安装一些附加的 Vue 插件、注册全局组件，或者增加额外的路由钩子等。

```js
// 使用异步函数也是可以的
export default ({
  Vue, // VuePress 正在使用的 Vue 构造函数
  options, // 附加到根实例的一些选项
  router, // 当前应用的路由实例
  siteData, // 站点元数据
  isServer // 当前应用配置是处于 服务端渲染 或 客户端
}) => {
  // ...做一些其他的应用级别的优化
}
复制代码
```

上面的内容告诉我们可以使用插件，这是写组件库文档必不可少的，刚好符合需求，接下来就按照官方的提示改造 `enhanceApp.js` ：

文档就用 `element-ui` 作为示例（反正之前的组件库也是基于它来做的不冲突）。

```js
import ElementUI from 'element-ui';
import 'element-ui/lib/theme-chalk/index.css';

export default ({ Vue, options, router, siteData, isServer }) => {
  Vue.use(ElementUI);
};

复制代码
```

然后在 `Button.md` 中使用组件。

```html
<el-button type="primary" size="mini">按钮</el-button>
复制代码
```

那不出意外还是出意外了。

![10.png](https://cdn.wallleap.cn/img/pic/illustrtion/202210091748352.webp)

总体来说就是依赖不一致导致的问题，有兴趣的可以看下。[解决方案](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvuepress%2Fissues%2F2275) 如下：

```bash
npm install async-validator@1.11.5 
复制代码
```

![8.png](https://cdn.wallleap.cn/img/pic/illustrtion/202210091748744.webp)

### 插件

> [插件](https://link.juejin.cn/?target=https%3A%2F%2Fvuepress.vuejs.org%2Fzh%2Fplugin%2F%23%E6%A0%B7%E4%BE%8B) 可以使用官方或者社区的插件，也可以根据自己的需求开发适合自己的插件。[开发插件](https://link.juejin.cn/?target=https%3A%2F%2Fvuepress.vuejs.org%2Fzh%2Fplugin%2Fwriting-a-plugin.html)

安装

```bash
npm install -D @vuepress/plugin-back-to-top
复制代码
```

[插件使用](https://link.juejin.cn/?target=https%3A%2F%2Fvuepress.vuejs.org%2Fzh%2Fplugin%2Fusing-a-plugin.html)

```js
// config.js
module.exports = {
  // ... ...
  plugins: ['@vuepress/back-to-top']
}

复制代码
```

再推荐小伙伴们安装一个非官方插件，用于代码预览 `vuepress-plugin-demo-container` [如何使用？](https://link.juejin.cn/?target=https%3A%2F%2Fcalebman.github.io%2Fvuepress-plugin-demo-container%2Fzh%2F)

### 组件库文档编写

小伙伴去 element 的 [Button](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FElemeFE%2Felement%2Fblob%2Fdev%2Fexamples%2Fdocs%2Fzh-CN%2Fbutton.md%3Fplain%3D1) 把里面的文档复制下来放到咱们自己的 `Button.md` 中。

看到最后的预览图可能有些小伙伴会觉得按钮都有点挤，内容区域有点窄等等样式问题，`index.styl` `palette.styl` 帮你解决问题。

![QQ20220214-153524-HD.gif](https://cdn.wallleap.cn/img/pic/illustrtion/202210091748734.webp)

### 国际化

如果要实现国际化配置，项目结构应该是下面这样。 接着把 `en/guide/Button.md` 注入[英文文档](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FElemeFE%2Felement%2Fblob%2Fdev%2Fexamples%2Fdocs%2Fen-US%2Fbutton.md%3Fplain%3D1)。

```arduino
.
├─ docs
│  ├── .vuepress
│  │   ├── config.js
│  │   ├── styles
│  │   │	├── index.styl
│  │   │	└── palette.styl
│  │   └── public
│  ├── en
│  │   ├── guide
│  │   │   ├── Button.md
│  │   │   ├── Card.md
│  │   │   └── README.md
│  │   └─README.md
│  ├── guide
│  │   ├── Button.md
│  │   ├── Card.md
│  │   └── README.md
│  └─	README.md
└─ package.json
复制代码
```

在 `config.js` 中提供了 `locales` 选项：

**注意** `locales.title` `locales.description` 的优先级会高于 `title` `description`，同时页面的右上角会出现切换语言的下拉框。

```js
module.exports = {
  locales: {
    '/': {
      lang: 'zh-CN',
      title: 'VuePress',
      description: 'Vue 驱动的静态网站生成器'
    },
    // 键名是该语言所属的子路径
    // 作为特例，默认语言可以使用 '/' 作为其路径。
    '/en/': {
      lang: 'en-US', // 将会被设置为 <html> 的 lang 属性
      title: 'VuePress',
      description: 'Vue-powered Static Site Generator'
    }
  }
}
复制代码
```

然后我们还需要把 导航栏，侧边栏，以及文档的路由做出修改。这些操作都将在 `themeConfig.locales` 中进行。

**注意** 当配置此选项时，原有的 `themeConfig.nav` `themeConfig.sidebar` 更改为 `themeConfig.locales.nav` `themeConfig.locales.sidebar` 。

```js
modeule.exports = {
  themeConfig {
    locales: {
      '/': {
        selectText: '选择语言', // 页面右上角选择语言label
        lebel: '简体中文', // 页面右上角选择语言下拉框 value
	nav: [
          { text: '指南', link: '/guide/' },
          // ...
        ],
        sidebar: {
          '/guide/': [
            ['', '介绍'],
            {
              title: '组件',
              collapsable: false,
              children: [
                ['../guide/Button.md', 'Button'],
                ['../guide/Card.md', 'Card']
              ]
            }
          ]
        }
      },
      // 配置 英文时注意路由的更改
      '/en/': {
        selectText: 'Languages', // 页面右上角选择语言label
        lebel: 'English', // 页面右上角选择语言下拉框 value
	nav: [
          { text: 'guide', link: '/en/guide/' },
          // ...
        ],
        sidebar: {
          '/en/guide/': [
            ['', 'Guide'],
            {
              title: 'Components',
              collapsable: false,
              children: [
                ['../guide/Button.md', 'Button'],
                ['../guide/Card.md', 'Card']
              ]
            }
          ]
        }
      }
    }
  }
}
复制代码
```

![12.gif](https://cdn.wallleap.cn/img/pic/illustrtion/202210091748196.webp)

### 发布

#### 打包

忙活了大半天终于打包了。执行之前配置的打包命令 `npm run docs:build`。可能有些小伙伴打包环节会遇到以下错误。

![13.png](https://cdn.wallleap.cn/img/pic/illustrtion/202210091748382.webp)

原因就是 静态HTML 是通过 Node 渲染，咱们在注册 element 组件时机不太行。可以将 `enhanceApp.js` 修改成如下：

```js
import 'element-ui/lib/theme-chalk/index.css';

export default async ({ Vue, options, router, siteData, isServer }) => {
  if (!isServer) {
    await import('element-ui').then(ElementUI => {
      Vue.use(ElementUI);
    });
  }
};

复制代码
```

打包完成之后的文件在 `/docs/.vuepress/dist`

#### 发布到 githubPages

1，登录到 github 创建 Repository

![14.png](https://cdn.wallleap.cn/img/pic/illustrtion/202210091748993.webp)

2，在 `config.js` 中新增 `base` 选项：值为创建仓库时的 Repository name

```js
modeule.exports = {
  base: '/comp-vuepress/'
}
复制代码
```

3，根目录新建 `deploy.sh` 文件

```sh
#!/usr/bin/env sh

# 确保脚本抛出遇到的错误
set -e

# 生成静态文件
npm run docs:build

# 进入生成的文件夹
cd docs/.vuepress/dist

# 如果是发布到自定义域名
# echo 'www.example.com' > CNAME

git init
git add -A
git commit -m 'deploy'

# 如果发布到 https://<USERNAME>.github.io/<REPO> 这里做出对应的替换
# git push -f git@github.com:<USERNAME>/<REPO>.git master:gh-pages

cd -
复制代码
```

3，`package.json` 中新增一条脚本, 并执行

```json
"scripts": {
    "docs:deploy": "bash deploy.sh"
}
复制代码
```

4，随后去 github 查看会多出一个 gh-pages 的分支。然后点击 Setting 进行设置

![15.png](https://cdn.wallleap.cn/img/pic/illustrtion/202210091748429.webp)

5，随后等个两三分钟，刷新下页面，链接变成绿色即可点击啦！

![16.png](https://cdn.wallleap.cn/img/pic/illustrtion/202210091748559.webp)

分类：

[前端](https://juejin.cn/frontend)

标签：

[前端](https://juejin.cn/tag/前端)[Vue.js](https://juejin.cn/tag/Vue.js)
