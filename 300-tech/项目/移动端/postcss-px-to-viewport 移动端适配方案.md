---
title: postcss-px-to-viewport 移动端适配方案
date: 2023-08-04 17:27
updated: 2023-08-04 17:28
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - postcss-px-to-viewport 移动端适配方案
rating: 1
tags:
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

github 地址：[postcss-px-to-viewport](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fevrone%2Fpostcss-px-to-viewport%2Fblob%2Fmaster%2FREADME.md "https://github.com/evrone/postcss-px-to-viewport/blob/master/README.md")

插件安装

```sh
npm install postcss-px-to-viewport --save-dev
```

参数配置 `postcss.config.js`

```js
 "postcss-px-to-viewport": {
  // options
  unitToConvert: "px", // 需要转换的单位
  viewportWidth: 750, // 设计稿的视口宽度
  unitPrecision: 5, // 单位转换后保留的精度
  propList: ["*"], // 能转换的vw属性列表
  viewportUnit: "vw", // 希望使用的视口单位
  fontViewportUnit: "vw", // 字体使用的视口单位
  selectorBlackList: [], // 需要忽略的css选择器
  minPixelValue: 1, // 设置最小的转换数值，如果为1，只有大于1的值才会被转换
  mediaQuery: false, // 媒体查询中是否需要转换单位
  replace: true, // 是否直接更换属性值
  exclude: [],
  landscape: false,
  landscapeUnit: "vw", // 横屏时使用的单位
  landscapeWidth: 568 // 横屏时使用的视口宽度
}
```

解决组件库冲突（以 vant 为例）

```js
selectorBlackList: ["van"], // 需要忽略的css选择器
```

尽情使用
