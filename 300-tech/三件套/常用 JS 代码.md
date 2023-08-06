---
title: 常用 JS 代码
date: 2023-08-01 14:52
updated: 2023-08-01 14:52
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - 常用 JS 代码
rating: 1
tags:
  - blog
category: 分类
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: #
url: //myblog.wallleap.cn/post/1
---

## 获取 DOM 节点

```js
document.documentEl // 获取根标签
document.body // 获取 body 标签
document.querySelector(el) // 获取选择器为 el 的元素
document.querySelectorAll(el) // 获取选择器为 el 的元素数组
```

## 操作 DOM 元素

```js
EL.append(el)
/* <p data-test-a='AreYouOK'>test</p> */
document.querySelector('p').dataset['testA']
document.querySelector('p').setAttribute('')
```

## CSS 样式

```gist
34086cbddd6e3a2072aadfd109193a02
```

