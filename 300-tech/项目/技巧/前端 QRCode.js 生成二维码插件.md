---
title: 前端 QRCode.js 生成二维码插件
date: 2023-05-29T09:04:00+08:00
updated: 2024-08-21T10:32:45+08:00
dg-publish: false
aliases:
  - 前端 QRCode.js 生成二维码插件
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
source: //juejin.cn/post/6844903719255932936
tags:
  - 前端
  - 插件
url: //myblog.wallleap.cn/post/1
---

本文用于推荐一款很好用的二维码生成插件 QRCode.js，测试使用方便且简单。 其实官方就有很好的文档，这里只是做一个我工作的记录和总结。

- 目录
	- 介绍
	- 使用
		- 1.引入 js 文件
		- 2.定义承载二维码标签
		- 3.js 调用
		- 4.页面预览
	- 参数 API
		- QRCode 参数
		- Option 参数
		- API 接口
	- 实践
		- 生成二维码，微信 QQ 识别打开网页

## 介绍

- 这个插件主要使用 canvas 实现的。
- 原生代码不需要依赖 jquery，或者 zepto。
- 兼容性也很好，IE6~10, Chrome, Firefox, Safari, Opera, Mobile Safari, Android, Windows Mobile, ETC.
- [前端开发者仓库官网](https://link.juejin.cn?target=http%3A%2F%2Fcode.ciaoca.com%2Fjavascript%2Fqrcode%2F "http://code.ciaoca.com/javascript/qrcode/")
- [GitHub地址](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fdavidshimjs%2Fqrcodejs "https://github.com/davidshimjs/qrcodejs")

## 使用

### 1.引入 js 文件

```html
<script src="qrcode.js"></script>
```

### 2.定义承载二维码标签

```html
<div id="qrcode"></div>
```

### 3.js 调用

#### 简单调用

```js
new QRCode(document.getElementById('qrcode'), 'your content');
```

#### 设置参数调用

下面会有参数详解

```js
var qrcode = new QRCode('qrcode', {
	text: 'your content',
	width: 256,
	height: 256,
	colorDark: '#000000',
	colorLight: '#ffffff',
	correctLevel: QRCode.CorrectLevel.H
});
```

### 4.页面预览

这样就很简单的生成了一个二维码

![](https://cdn.wallleap.cn/img/pic/illustration/202305290910549.png)

## 参数 API

### QRCode 参数

```js
new QRCode(element, option)
```

| 名称 | 默认值 | 说明 |
| --- | --- | --- |
| element | \- | 承载二维码的 DOM 元素的 ID |
| option | \- | 参数设置 |

#### Option 参数

| 名称 | 默认值 | 说明 | 备注 |
| --- | --- | --- | --- |
| text | \- | 二维码承载的信息 | 如果是 string 那没有什么好说的。
如果是 url 的话，为了微信和 QQ 可以识别，连接中的中文使用 encodeURIComponent 进行编码 |
| width | 256 | 二维码宽度 | 单位像素（百分比不行） |
| height | 256 | 二维码高度 | 单位像素（百分比不行） |
| colorDark | '#000000' | 二维码前景色 | 英文\\十六进制\\rgb\\rgba\\transparent 都可以 |
| colorLight | '#ffffff' | 二维码背景色 | 英文\\十六进制\\rgb\\rgba\\transparent 都可以 |
| correctLevel | QRCode.CorrectLevel.L | 容错级别 | 由低到高

QRCode.CorrectLevel.L
QRCode.CorrectLevel.M
QRCode.CorrectLevel.Q
QRCode.CorrectLevel.H |

#### API 接口

| 名称 | 参数 | 说明 | 使用 |
| --- | --- | --- | --- |
| clear | \- | 清除二维码 | qrcode.clear() |
| makeCode | string | 替换二维码（参数里面是新的二维码内容） | qrcode.makeCode('new content') |

```js
var qrcode = new QRCode('qrcode',{
    'text':'content',
    'width':256,
    'height':256,
    'colorDark':'red',
    'colorLoght':'transparent',
    'correctLevel':QRCode.CorrectLevel.H
})


qrcode.clear();
qrcode.madkCode('new content');
```

## 实践

### 生成二维码，微信 QQ 识别打开网页

#### 需求

- 前端根据传的不同的参数，在页面生成一个二维码
- 由端分享到 QQ、QQ 空间、微信、朋友圈的时候，截屏成图片
- 长按图片，识别其中的二维码，打开网页链接。

#### 思路

- 和端交互的网页 a.html 后面加 query 参数，如：`http://www.test.html/a.html?code=123`
- a.html 中调用 QRCode.js 生成一个二维码，二维码中的信息是 `http://www.test.html/b.html?code=123`
- 分享出去的页面是截屏是 a.html 的，识别图中的二维码打开 b.html

#### 实现

由于很简单，所以就不贴代码了。

#### 注意

> 如果传的是 url，但是打开的时候只是一堆字符串让手动复制，那么说明 url 的地址不正确。 如果是微信，传的 url 的地址中有中文是可以识别的，但是在 QQ 中是不行的 所以其中的中文要进行 encodeURIComponent 编码，但是不要整体都编码，只是中文的部分编码即可。

@version1.0——2018-11-22——创建《前端 QRCode.js 生成二维码插件》

©burning\_ 韵七七

作者：顽皮的雪狐七七
链接：<https://juejin.cn/post/6844903719255932936>
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
