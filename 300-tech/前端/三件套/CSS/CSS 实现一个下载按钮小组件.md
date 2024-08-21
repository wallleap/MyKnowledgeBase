---
title: CSS 实现一个下载按钮小组件
date: 2023-08-03T03:32:00+08:00
updated: 2024-08-21T10:32:48+08:00
dg-publish: false
aliases:
  - CSS 实现一个下载按钮小组件
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
  - CSS
  - web
url: //myblog.wallleap.cn/post/1
---

[【每日一练】01~100练大合集汇总 (qq.com)](https://mp.weixin.qq.com/s?__biz=MjM5MDA2MTI1MA==&mid=2649133984&idx=1&sn=0d274ff12af27e618423d822ed7119d2&chksm=be58b40d892f3d1b6d3b7a0f2e57cb16fc6e774282512517925d28ab01e5d009863f753c4590&scene=21#wechat_redirect)

以下是今天小组件练习的最终效果：

![](https://cdn.wallleap.cn/img/pic/illustration/202308031533125.gif)

HTML 代码：

```html
<!DOCTYPE html>
<html >
<head>
  <meta charset="UTF-8">
  <title>【每日一练】198—CSS实现一个下载按钮小组件</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/normalize/5.0.0/normalize.min.css">
  <link rel='stylesheet prefetch' href='https://fonts.googleapis.com/css?family=PT+Sans'>
      <link rel="stylesheet" href="css/style.css">
</head>

<body>
  <a class="Button" href="#0" data-tooltip="File size: 50MB">
    <span class="Button__textWrapper">
      <span class="Button__text">Download</span>
    <span class="Button__icon" aria-hidden="true"></span>
  </span>
</a> 
</body>
</html>
```

CSS 代码：

```css
*,
*:after,
*:before {
  box-sizing: border-box;
}

html {
  font-size: 16px;
}

body {
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  -webkit-box-pack: center;
      -ms-flex-pack: center;
          justify-content: center;
  -webkit-box-align: center;
      -ms-flex-align: center;
          align-items: center;
  font-size: 100%;
  font-family: 'PT Sans', sans-serif;
  background-color: #150811;
  height: 100vh;
}

a {
  text-decoration: none;
}

.Button__textWrapper, .Button__text, .Button__icon {
  position: absolute;
  width: 100%;
  height: 100%;
  left: 0;
}

.Button__text, .Button__icon {
  -webkit-transition: top 500ms;
  transition: top 500ms;
}

.Button {
  display: inline-block;
  position: relative;
  background-color: #0CBABA;
  color: #fff;
  font-size: 1.2rem;
  border-radius: 1000px;
  width: 200px;
  height: 60px;
  box-shadow: 0 2px 20px rgba(0, 0, 0, 0.7), inset 0 1px rgba(255, 255, 255, 0.3);
  text-align: center;
  -webkit-transition: background-color 500ms, -webkit-transform 100ms;
  transition: background-color 500ms, -webkit-transform 100ms;
  transition: background-color 500ms, transform 100ms;
  transition: background-color 500ms, transform 100ms, -webkit-transform 100ms;
}
.Button__textWrapper {
  overflow: hidden;
}
.Button__text {
  line-height: 60px;
  top: 0;
}
.Button__icon {
  top: 100%;
  background: url("https://cl.ly/0X3o100h2H0g/icon-download.svg") no-repeat center center;
}
.Button::before {
  content: attr(data-tooltip);
  width: 140px;
  height: 60px;
  background-color: #EEB868;
  font-size: 1rem;
  border-radius: .25em;
  line-height: 60px;
  bottom: 90px;
  left: calc(50% - 70px);
}
.Button::after {
  content: '';
  width: 0;
  height: 0;
  border: 10px solid transparent;
  border-top-color: #EEB868;
  left: calc(50% - 10px);
  bottom: 70px;
}
.Button::before, .Button::after {
  position: absolute;
  opacity: 0;
  -webkit-transition: all 500ms;
  transition: all 500ms;
  visibility: hidden;
}
.Button:hover {
  background-color: #01BAEF;
}
.Button:hover .Button__text {
  top: -100%;
}
.Button:hover .Button__icon {
  top: 0;
}
.Button:hover::before, .Button:hover::after {
  opacity: 1;
  visibility: visible;
}
.Button:hover::after {
  bottom: 60px;
}
.Button:hover::before {
  bottom: 80px;
}
.Button:active {
  -webkit-transform: translate(2px, 2px);
          transform: translate(2px, 2px);
}
```
