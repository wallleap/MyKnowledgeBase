---
title: 使用Vuepress搭建基于Vue 3.2的组件库（保姆级教程）
date: 2022-11-10 15:29
updated: 2022-11-10 15:30
cover: //cdn.wallleap.cn/img/pic.jpg
author: Luwang
comments: true
aliases:
  - 使用Vuepress搭建组件库
rating: 10
tags:
  - web
  - 教程
  - 学习
  - 文档
category: web
keywords:
  - web
  - VuePress
  - 组件库
  - 文档
description: 文章描述
source: null
url: null
---

[![img](https://cdn.wallleap.cn/img/pic/illustrtion/202210091750513.webp)](https://juejin.cn/user/325111173888686)

[来蹭饭![lv-3](https://cdn.wallleap.cn/img/pic/illustrtion/202210091750607.png)](https://juejin.cn/user/325111173888686)

大家好我是来蹭饭，一个会点儿吉他和编曲，绞尽脑汁想傍个富婆的摸鱼大师。

今天跟大家分享使用`Vuepress2.0 + Vue3.2 + Ant`来搭建自己的组件库文档。

# 前言

众所周知`Vue`已经来到了**3.2**的版本，使用`<script setup>`的语法糖给开发者带来了极大的便利。源于对`Vue`高版本的支持，本次使用`Vuepress 2.0`开发组件库文档，需求如下：

- 组件库能直接引用或二次封装`Ant`的组件
- 组件库能显示组件代码供他人查阅

案例完成后，你将能看到这样婶儿的效果：

![1.png](https://cdn.wallleap.cn/img/pic/illustrtion/202210091750394.webp)

下面我将从**安装，页面搭建，组件的引入与封装**等章节跟大家分享组件库的搭建过程。废话少说，大家坐稳扶好，我们发车。

# 安装

## 初始化

Vuepress 2.0对Node的版本支持为`Node.js V12+`，各单位注意自行升级Node的版本。

安装非常简单，cd到项目目录下然后一条命令即可

- **步骤1：** 创建项目

```sql
yarn add -D vuepress@next
复制代码
```

安装完毕后我们会发现项目变成这样了：

![3.png](https://cdn.wallleap.cn/img/pic/illustrtion/202210091750145.webp)

纳尼？

![4.jpg](https://cdn.wallleap.cn/img/pic/illustrtion/202210091750304.webp)

怎么只有一个node_modules？这跟我们用cli梭出来的项目差异这么大吗？我反复确认了安装命令已经完成，才敢继续下面的操作。

- **步骤2：** 在`package.json`中添加如下命令

```json
{
  "scripts": {
    "docs:dev": "vuepress dev docs",
    "docs:build": "vuepress build docs"
  }
}
复制代码
```

- **步骤3：** 启动项目（完成页面搭建章节的**config.js配置**和**README.md配置**后启动）

```
yarn docs:dev
复制代码
```

## 项目结构

Vuepress是用来写文档的，所以它的所有文件都会被包裹在`docs`文件夹下（可以理解成Vuepress项目的`src`文件夹）。你可以在自己的Vuepress项目中单独创建`docs`也可以让它“寄生”在已经存在的项目下（与原项目的运行互不影响）：

![5.png](https://cdn.wallleap.cn/img/pic/illustrtion/202210091750318.webp)

本次案例我们以单独创建的Vuepress项目为例进行讲解。

经过我们的DIY后，docs文件夹初始化后的目录结构如下：

```arduino
├─ docs
│  ├─ .vuepress
│  │  └─ config
│  │  └─ public
│  │  └─ config.js
│  └─ README.md
├─ .gitignore
└─ package.json
复制代码
```

上述文件的作用分别是：

- `docs/.vuepress`: 存放全局的配置、组件、静态资源等。
- `docs/.vuepress/config`: 存放页眉导航栏和子页面侧栏的配置文件。
- `docs/.vuepress/public`: 静态资源目录。
- `docs/.vuepress/config.js`: 配置文件的入口文件。
- `docs/README.md`: 文档的入口页面。

至此安装步骤算是完成了，接下来进入**页面搭建**的环节。完成后，我们就可以启动项目查看自己的文档了。

# 页面搭建

> 写文档的过程就像写书，书本的每一页即为读者看到的内容。在Vuepress中`md`文件就是书的页面，也是对应路由的结果。接下来我们通过配置`config.js`文件，`md`文件，页眉导航栏`navbar`以及页面侧栏`sidebar`来完成页面搭建。

在开始配置前我们先把目录结构调整如下：

```arduino
├─ docs
│  ├─ .vuepress
│  │  └─ config
│  │  │  └─ navbar.js
│  │  │  └─ sidebar.js
│  │  └─ public
│  │  │  └─ img
│  │  └─ config.js
│  └─ README.md
├─ .gitignore
└─ package.json
复制代码
```

## config.js配置

themeConfig中logo寻址的路径默认是`public`文件夹，将对应的logo图片放入img文件夹中。

```scss
module.exports = {
  // 站点配置
  lang: 'zh-CN',
  title: 'UI-LIB',
  description: '组件库',
  head: [ // 注入到当前页面的 HTML <head> 中的标签
    ['link', {
      rel: 'icon',
      href: '/img/logo.png'
    }], // 增加一个自定义的 favicon(网页标签的图标)
  ],
  // 主题和它的配置
  theme: '@vuepress/theme-default',
  themeConfig: {
    logo: '/img/logo.png',
  }
}
复制代码
```

## 入口README.md配置

```yaml
---
home: true
heroImage: /img/logo.png
actionText: 快速上手 →
actionLink: /zh/`guide/
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

当完成上述配置后，通过`yarn docs:dev`运行项目，此时项目会新增`.cache`和`.temp`文件，不用管它们，直接看运行效果：

![6.png](https://cdn.wallleap.cn/img/pic/illustrtion/202210091750700.webp)

一个最基础的文档页面就诞生了！接下来我们进入**导航栏**和**侧栏**的配置。

## navbar.js 导航栏配置

导航栏的配置，是配置二级页面路由的过程（外链导航除外），我们希望导航栏实现这样的效果：

- 导航栏包含**组件，文档，工具箱**三个选项
- 点击**组件**，能路由到组件页面
- 鼠标经过**文档**时，弹出二级菜单选项，点击直接进入二级菜单页
- **工具箱**的展开内容是一些线上地址的链接，点击后跳转线上页面

![8.png](https://cdn.wallleap.cn/img/pic/illustrtion/202210091750022.webp)

接下来我们逐步实现上述需求：

- **步骤1：** 调整项目结构，在`docs`中添加`components`和`document`两个文件夹，具体内容如下：

![7.png](https://cdn.wallleap.cn/img/pic/illustrtion/202210091750948.webp)

- **步骤2：** 将navbar.js文件引入config.js中

```java
const navbar = require('./config/navbar')

module.exports = {
  // 站点配置
  lang: 'zh-CN',
  title: 'UI-LIB',
  description: '组件库',
  head: [ // 注入到当前页面的 HTML <head> 中的标签
    ['link', {
      rel: 'icon',
      href: '/img/logo.png'
    }], // 增加一个自定义的 favicon(网页标签的图标)
  ],
  // 主题和它的配置
  theme: '@vuepress/theme-default',
  themeConfig: {
    logo: '/img/logo.png',
    navbar,
    sidebarDepth: 2, // 侧边栏显示2级
  }
}
复制代码
```

- **步骤3：** 在`docs/.vuepress/config/navbar.js`中做出如下配置：

```scss
module.exports = [{    text: '组件',    link: '/components/'  },  {    text: '文档',    children: [{        text: '介绍',        link: '/document/introduction/'      },      {        text: '注意事项',        link: '/document/tips/'      },    ]
  },
  {
    text: '工具箱',
    children: [{
        text: '在线编辑',
        children: [{
          text: '图片压缩',
          link: 'https://tinypng.com/'
        }]
      },
      {
        text: '在线服务',
        children: [{
            text: '阿里云',
            link: 'https://www.aliyun.com/'
          },
          {
            text: '腾讯云',
            link: 'https://cloud.tencent.com/'
          }
        ]
      },
      {
        text: '博客指南',
        children: [{
            text: '掘金',
            link: 'https://juejin.im/'
          },
          {
            text: 'CSDN',
            link: 'https://blog.csdn.net/'
          }
        ]
      }
    ]
  }
]

复制代码
```

这里讲解一下各字段的含义

- **text：** 显示在导航栏上的文字信息

- **link：** 路由地址显示的页面（类似于router中的component）会自动寻址到指定目录下的`README.md`，把它作为主入口显示出来。`link`字段也可以配置线上地址，点击后跳转到线上页面。默认页面路由地址如下：

	| 文件的相对路径                 | 页面路由地址         |
	| ----------------------- | -------------- |
	| `/README.md`            | `/`            |
	| `/components/README.md` | `/components/` |

- **children：** 子路由的嵌套信息，可多层嵌套，上述示例已给出

以**工具箱**配置为例，效果如图所示，结合代码更好理解：

![9.png](https://cdn.wallleap.cn/img/pic/illustrtion/202210091750313.webp)

至此我们的导航栏配置工作完成。

**由此可见，导航栏上每一个能进入二级页面的导航，都要对应一个页面文件：**

| 导航栏选项     | 对应文件                    |
| --------- | ----------------------- |
| `组件`      | `components`            |
| `文档/介绍`   | `document/introduction` |
| `文档/注意事项` | `document/tips`         |

现在进入组件页面，里面还是一片空白。接下来我们通过配置`sidebar.js`，实现二级页面的侧栏切换功能。

![10.png](https://cdn.wallleap.cn/img/pic/illustrtion/202210091750699.webp)

## sidebar.js 侧栏配置

> 侧栏的承载的信息比较多，因此我们在`docs/.vuepress/config/sidebar.js`中配置整合后的侧栏信息。每个页面的侧栏交由该页面独立控制，以**components**页面为例进行侧栏配置。

- **步骤1：** config.js引入侧栏并使用

```javascript
const navbar = require('./config/navbar')
const sidebar = require('./config/sidebar')

module.exports = {
  // 站点配置
  lang: 'zh-CN',
  title: 'UI-LIB',
  description: '组件库',
  head: [ // 注入到当前页面的 HTML <head> 中的标签
    ['link', {
      rel: 'icon',
      href: '/img/logo.png'
    }], // 增加一个自定义的 favicon(网页标签的图标)
  ],
  // 主题和它的配置
  theme: '@vuepress/theme-default',
  themeConfig: {
    logo: '/img/logo.png',
    navbar,
    sidebar,
    sidebarDepth: 2, // 侧边栏显示2级
  }
}
复制代码
```

- **步骤2：** 在`config/sidebar.js`中导出配置

```java
module.exports = {
  '/components/': require('../../components/sidebar'),
}
复制代码
```

- **步骤3：** 配置components文件，结构修改如下：

	pages：当前页面下的子路由，目前新增了**Button**，**InputNumber**和**Slider**3个子页面

	sidebar.js：当前页面的侧栏配置

	![11.png](https://cdn.wallleap.cn/img/pic/illustrtion/202210091750610.webp)

- **步骤4：** 配置components文件中的sidebar.js：

```arduino
module.exports = [{
    text: '通用',
    collapsable: true,
    children: [{
      text: 'Button 按钮',
      link: '/components/pages/Button',
    }]
  }, {
    text: '数据录入',
    collapsable: true,
    children: [{
      text: 'Slider 滑动组件',
      link: '/components/pages/Slider',
    }]
  },
  {
    text: '数字输入框',
    collapsable: true,
    children: [{
      text: 'InputNumber 数字输入框',
      link: '/components/pages/InputNumber',
    }]
  },
]
复制代码
```

sidebar中的字段与navbar类似，这里不多做赘述，配置完成后就能正常点击components下的侧栏了。

![12.png](https://cdn.wallleap.cn/img/pic/illustrtion/202210091750105.webp)

至此页面搭建工作完成！接下来将`Ant`引入文档中，进一步完善我们的组件库。

# 组件的引入与封装

本章节需要解决以下两个问题：

- `Ant`引入Vuepress后如何注册并使用？
- 二次封装组件时，我们希望在`.vue`文件中操作，之后使用md文件引入`.vue`文件。如何让md文件正确识别`.vue`文件？

下面我将逐步讲解这两个问题的解决方式。

## 引入Ant

- **步骤1：** 修改`.vuepress`文件夹结构，新增`clientAppEnhance.js`文件（迁移特性Vuepress1中名称为enhanceApp.js），该文件的作用是增强客户端应用

```arduino
├─.vuepress
│  └─ config
│  │  └─ navbar.js
│  │  └─ sidebar.js
│  └─ public
│  │  └─ img
│  └─ clientAppEnhance.js
│  └─ config.js
复制代码
```

- **步骤2：** 安装`Ant`

> yarn add ant-design-vue@next

- **步骤3：** 配置`clientAppEnhance.js`文件

```javascript
import Antd from 'ant-design-vue'
import 'ant-design-vue/dist/antd.css'


import { defineClientAppEnhance } from '@vuepress/client'

export default defineClientAppEnhance(({ app, router, siteData }) => {
  app.use(Antd)
})
复制代码
```

到这里使用Ant的前置工作已经完成。

## 识别.vue文件

- **步骤1：** 安装`@vuepress/plugin-register-components`插件，在Vuepress1.0中，md文件能自动识别导出的`.vue`文件，Vuepress2.0中需要安装插件并做好配置

> yarn add @vuepress/plugin-register-components@next

- **步骤2：** `config.js`中配置修改如下：

```javascript
const {
  path
} = require('@vuepress/utils')
const navbar = require('./config/navbar')
const sidebar = require('./config/sidebar')

module.exports = {
  // 站点配置
  lang: 'zh-CN',
  title: 'UI-LIB',
  description: '组件库',
  head: [ // 注入到当前页面的 HTML <head> 中的标签
    ['link', {
      rel: 'icon',
      href: '/img/logo.png'
    }], // 增加一个自定义的 favicon(网页标签的图标)
  ],
  // 主题和它的配置
  theme: '@vuepress/theme-default',
  themeConfig: {
    logo: '/img/logo.png',
    navbar,
    sidebar,
    sidebarDepth: 2, // 侧边栏显示2级
  },
  plugins: [
    [
      '@vuepress/plugin-register-components',
      {
        componentsDir: path.resolve(__dirname, './components')
      }
    ],
  ]
}
复制代码
```

- **步骤3：** 在`.vuepress`文件夹中新建components文件夹，里面存放vue组件，文件结构如下：

```arduino
├─.vuepress
│  └─ components
│  │  └─ Button.vue
│  └─ config
│  │  └─ navbar.js
│  │  └─ sidebar.js
│  └─ public
│  │  └─ img
│  └─ clientAppEnhance.js
│  └─ config.js
复制代码
```

至此md文件就能无需引入即可自动识别`.vuepress/components/`下所有的vue组件了。继续完成下面的步骤，预览效果。

- **步骤4：** `Button.vue`文件添加Ant中Button的代码：

```css
<template>
  <a-button type="primary">Primary Button</a-button>
  <a-button>Default Button</a-button>
  <a-button type="dashed">Dashed Button</a-button>
  <a-button type="text">Text Button</a-button>
  <a-button type="link">Link Button</a-button>
</template>
复制代码
```

- **步骤5：** `docs/components/pages/Button.md`文件直接引入`Button.vue`：

```shell
常用的操作按钮。

## 基础用法

基础的按钮用法。


<Button />
复制代码
```

至此组件库初具雏形，我们看看效果：

![13.png](https://cdn.wallleap.cn/img/pic/illustrtion/202210091750428.webp)

现在还剩一些问题待解决：

- 如果组件业务逻辑不复杂，比如`Button.vue`中只有template代码，并不包含业务逻辑功能。这种情况下能否直接将`.vue`文件的内容写在`md`文件里？
- 如何在组件底部添加"显示代码"的按钮，供他人查阅组件代码？

我们通过本文最后的part来解决这些问题。

## 直接书写vue语法

上述问题可以通过安装`vuepress-plugin-demoblock-plus`插件，再配置config.js解决。

- **步骤1：** 安装插件。

> yarn add vuepress-plugin-demoblock-plus -D

- **步骤2：** 修改config.js中`plugins`的配置：

```css
plugins: [
    [
      '@vuepress/plugin-register-components',
      {
        componentsDir: path.resolve(__dirname, './components')
      }
    ],
    [      'vuepress-plugin-demoblock-plus'    ]
 ]
复制代码
```

- **步骤3：** 修改`docs/components/pages/Button.md`文件：

````bash
# Button 按钮

常用的操作按钮。

## 基础用法

基础的按钮用法。

:::demo 使用`type`、`plain`、`round`和`circle`属性来定义 Button 的样式。

```vue
<template>
  <a-button type="primary">Primary Button</a-button>
  <a-button>Default Button</a-button>
  <a-button type="dashed">Dashed Button</a-button>
  <a-button type="text">Text Button</a-button>
  <a-button type="link">Link Button</a-button>
</template>
```（避免转义，使用时去掉整个括号的内容）

:::
复制代码
````

我们看看预览效果：

![14.png](https://cdn.wallleap.cn/img/pic/illustrtion/202210091750678.webp)

到这一步组件库的基本功能就实现了。接下来我们简单封装一个自己的`InputNumber`组件。

## 展示二次封装的业务组件

- **步骤1：** 更改`.vuepress`文件夹结构：

```arduino
├─.vuepress
│  └─ components
│  │  └─ InputNumber.vue
│  └─ config
│  │  └─ navbar.js
│  │  └─ sidebar.js
│  └─ public
│  │  └─ img
│  └─ clientAppEnhance.js
│  └─ config.js
复制代码
```

- **步骤2：** `InputNumber.vue`文件添加如下代码：

```xml
<template>
  <a-input v-model:value="value" placeholder="Basic usage" />
</template>
<script setup>
import { ref, defineProps } from "vue";

const props = defineProps({
  defaultValue:String
})
const value = ref(props.defaultValue);

</script>
复制代码
```

- **步骤3：** 修改`docs/components/pages/InputNumber.md`文件：

````xml
# Input 输入

常用的操作按钮。

## 基础用法

基础的输入框用法。

:::demo 

```vue
<template>
  <InputNumber :defaultValue="defaultValue"/>
</template>
<script setup>
import { ref } from 'vue';

const defaultValue = ref('来蹭饭')
</script>
```（避免转义，使用时去掉整个括号的内容）

:::
复制代码
````

这样一个简单封装的组件就完成了，`InputNumber.vue`接收到了从`InputNumber.md`传入的参数，来看看最终的预览效果：

![16.png](https://cdn.wallleap.cn/img/pic/illustrtion/202210091750783.webp)

大功告成！通过上述方法我们就可以为所欲为地搭建自己的组件库了。

# 尾巴

读到这里你也不易，写到这里我也不易。不嫌麻烦的话欢迎点赞，评论，收藏，你们的支持是我创作的最大动力。
