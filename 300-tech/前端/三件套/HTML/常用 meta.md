---
title: 常用 meta
date: 2023-05-03T10:07:07+08:00
updated: 2024-08-21T10:32:48+08:00
dg-publish: false
---

```html
<meta name="viewport" content="width=device-width,initial-scale=1,minimum-scale=1,maximum-scale=1,user-scalable=no,viewport-fit=cover">
```

## [#](https://xugaoyi.com/pages/8309a5b876fc95e3/#概要) 概要

meta 标签提供关于 HTML 文档的元数据。元数据不会显示在页面上，但是对于机器是可读的。它可用于浏览器（如何显示内容或重新加载页面），搜索引擎（关键词），或其他 web 服务。

**必要属性**

| 属性    | 值        | 描述                                   |
| ------- | --------- | -------------------------------------- |
| content | some text | 定义与 http-equiv 或 name 属性相关的元信息 |

**可选属性**

| 属性       | 值                                                           | 描述                                |
| ---------- | ------------------------------------------------------------ | ----------------------------------- |
| http-equiv | content-type / expire / refresh / set-cookie                 | 把 content 属性关联到 HTTP 头部。       |
| name       | author / description / keywords / generator / revised / others | 把 content 属性关联到一个 name。     |
| content    | some text                                                    | 定义用于翻译 content 属性值的格式。 |

## [#](https://xugaoyi.com/pages/8309a5b876fc95e3/#网页相关) 网页相关

- **申明编码**

```html
<meta charset='utf-8' />
```

- **优先使用 IE 最新版本和 Chrome**

```html
<!-- 关于X-UA-Compatible -->
<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" /> <!-- 推荐 -->

<meta http-equiv="X-UA-Compatible" content="IE=6" ><!-- 使用IE6 -->
<meta http-equiv="X-UA-Compatible" content="IE=7" ><!-- 使用IE7 -->
<meta http-equiv="X-UA-Compatible" content="IE=8" ><!-- 使用IE8 -->
```

- **浏览器内核控制**：国内浏览器很多都是双内核（webkit 和 Trident），webkit 内核高速浏览，IE 内核兼容网页和旧版网站。而添加 meta 标签的网站可以控制浏览器选择何种内核渲染。[参考文档(opens new window)](http://se.360.cn/v6/help/meta.html)

```html
默认用极速核(Chrome)：<meta name="renderer" content="webkit"> 
默认用ie兼容内核（IE6/7）：<meta name="renderer" content="ie-comp"> 
默认用ie标准内核（IE9/IE10/IE11/取决于用户的IE）：<meta name="renderer" content="ie-stand"> 
```

国内双核浏览器默认内核模式如下：

1. 搜狗高速浏览器、QQ 浏览器：IE 内核（兼容模式）
2. 360 极速浏览器、遨游浏览器：Webkit 内核（极速模式）

- **禁止浏览器从本地计算机的缓存中访问页面内容**：这样设定，访问者将无法脱机浏览。

```html
<meta http-equiv="Pragma" content="no-cache">
```

- **Windows 8**

```html
<meta name="msapplication-TileColor" content="#000"/> <!-- Windows 8 磁贴颜色 -->
<meta name="msapplication-TileImage" content="icon.png"/> <!-- Windows 8 磁贴图标 -->
```

- **站点适配**：主要用于 PC- 手机页的对应关系。

```html
<meta name="mobile-agent"content="format=[wml|xhtml|html5]; url=url">
<!--
[wml|xhtml|html5]根据手机页的协议语言，选择其中一种；
url="url" 后者代表当前PC页所对应的手机页URL，两者必须是一一对应关系。
 -->
```

- **转码申明**：用百度打开网页可能会对其进行转码（比如贴广告），避免转码可添加如下 meta。

```html
<meta http-equiv="Cache-Control" content="no-siteapp" />
```

## [#](https://xugaoyi.com/pages/8309a5b876fc95e3/#seo优化)SEO 优化

[参考文档(opens new window)](http://msdn.microsoft.com/zh-cn/library/ff724016)

- **页面关键词**，每个网页应具有描述该网页内容的一组唯一的关键字。 使用人们可能会搜索，并准确描述网页上所提供信息的描述性和代表性关键字及短语。标记内容太短，则搜索引擎可能不会认为这些内容相关。另外标记不应超过 874 个字符。

```html
<meta name="keywords" content="your tags" />
```

- **页面描述**，每个网页都应有一个不超过 150 个字符且能准确反映网页内容的描述标签。

```html
<meta name="description" content="150 words" />
```

- **搜索引擎索引方式**，robotterms 是一组使用逗号 (,) 分割的值，通常有如下几种取值：none，noindex，nofollow，all，index 和 follow。确保正确使用 nofollow 和 noindex 属性值。

```html
<meta name="robots" content="index,follow" />
<!--
    all：文件将被检索，且页面上的链接可以被查询；
    none：文件将不被检索，且页面上的链接不可以被查询；
    index：文件将被检索；
    follow：页面上的链接可以被查询；
    noindex：文件将不被检索；
    nofollow：页面上的链接不可以被查询。
 -->
```

- **页面重定向和刷新**：content 内的数字代表时间（秒），既多少时间后刷新。如果加 url,则会重定向到指定网页（搜索引擎能够自动检测，也很容易被引擎视作误导而受到惩罚）。

```html
<meta http-equiv="refresh" content="0;url=" />
```

- **其他**

```html
<meta name="author" content="author name" /> <!-- 定义网页作者 -->
<meta name="google" content="index,follow" />
<meta name="googlebot" content="index,follow" />
<meta name="verify" content="index,follow" />
```

## [#](https://xugaoyi.com/pages/8309a5b876fc95e3/#移动设备) 移动设备

- **viewport**：能优化移动浏览器的显示。如果不是响应式网站，不要使用 initial-scale 或者禁用缩放。

```html
<meta name="viewport" content="width=device-width,initial-scale=1.0,minimun-scale=1.0,maximum-scale=1.0,user-scalable=no"/>
<!--这是常用的移动meta设置-->
```

1. width：宽度（数值 / device-width）（范围从 200 到 10,000，默认为 980 像素）
2. height：高度（数值 / device-height）（范围从 223 到 10,000）
3. initial-scale：初始的缩放比例 （范围从>0 到 10）
4. minimum-scale：允许用户缩放到的最小比例
5. maximum-scale：允许用户缩放到的最大比例
6. user-scalable：用户是否可以手动缩 (no,yes)

**注意**，很多人使用 initial-scale=1 到非响应式网站上，这会让网站以 100% 宽度渲染，用户需要手动移动页面或者缩放。如果和 initial-scale=1 同时使用 user-scalable=no 或 maximum-scale=1，则用户将不能放大/缩小网页来看到全部的内容。

- **WebApp 全屏模式**：伪装 app，离线应用。

```html
<meta name="apple-mobile-web-app-capable" content="yes" /> <!-- 启用 WebApp 全屏模式 -->
```

- **主题颜色**

```html
<meta name="theme-color" content="#11a8cd">
```

![img](https://cdn.wallleap.cn/img/pic/illustrtion/202211091500541.jpeg)

- **隐藏状态栏/设置状态栏颜色**：只有在开启 WebApp 全屏模式时才生效。content 的值为 default | black | black-translucent 。

```html
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent" />
```

- **添加到主屏后的标题**

```html
<meta name="apple-mobile-web-app-title" content="标题">
```

- **忽略数字自动识别为电话号码**

```html
<meta content="telephone=no" name="format-detection" />
```

- **忽略识别邮箱**

```html
<meta content="email=no" name="format-detection" />
```

- **添加智能 App 广告条 Smart App Banner**：告诉浏览器这个网站对应的 app，并在页面上显示下载 banner(如下图)。[参考文档(opens new window)](https://developer.apple.com/library/ios/documentation/AppleApplications/Reference/SafariWebContent/PromotingAppswithAppBanners/PromotingAppswithAppBanners.html)

```html
<meta name="apple-itunes-app" content="app-id=myAppStoreID, affiliate-data=myAffiliateData, app-argument=myURL">
```

![img](https://cdn.wallleap.cn/img/pic/illustrtion/202211091500173.png)

- **其他** [参考文档(opens new window)](http://fex.baidu.com/blog/2014/10/html-head-tags/?qq-pf-to=pcqq.c2c)

```html
<!-- 针对手持设备优化，主要是针对一些老的不识别viewport的浏览器，比如黑莓 -->
<meta name="HandheldFriendly" content="true">
<!-- 微软的老式浏览器 -->
<meta name="MobileOptimized" content="320">
<!-- uc强制竖屏 -->
<meta name="screen-orientation" content="portrait">
<!-- QQ强制竖屏 -->
<meta name="x5-orientation" content="portrait">
<!-- UC强制全屏 -->
<meta name="full-screen" content="yes">
<!-- QQ强制全屏 -->
<meta name="x5-fullscreen" content="true">
<!-- UC应用模式 -->
<meta name="browsermode" content="application">
<!-- QQ应用模式 -->
<meta name="x5-page-mode" content="app">
<!-- windows phone 点击无高光 -->
<meta name="msapplication-tap-highlight" content="no">
```

## 图标显示

[Favicon Generator for perfect icons on all browsers (realfavicongenerator.net)](https://realfavicongenerator.net/)

```html
<link rel="apple-touch-icon" sizes="180x180" href="/apple-touch-icon.png">
<link rel="icon" type="image/png" sizes="32x32" href="/favicon-32x32.png">
<link rel="icon" type="image/png" sizes="192x192" href="/android-chrome-192x192.png">
<link rel="icon" type="image/png" sizes="16x16" href="/favicon-16x16.png">
<link rel="manifest" href="/site.webmanifest">
<link rel="mask-icon" href="/safari-pinned-tab.svg" color="#5bbad5">
<meta name="msapplication-TileColor" content="#da532c">
<meta name="theme-color" content="#ffffff">
```

微信缩略图

首先准备一张 300*300 的图片作为缩略图。

在 head 头部添加一个 div，包含 img 标签。

```Html
<div style="display:none;"><img src="/img/wechat.png" alt=""></div>
<meta property="og:image" content="https://*.***.***/*.png">
```

第一行 src、第二行 content 更改为图片地址即可，第二行的图片地址必须为完整地址。

## [#](https://xugaoyi.com/pages/8309a5b876fc95e3/#一个常用的移动端页面meta设置) 一个常用的移动端页面 meta 设置

```html
<meta charset="utf-8">
<meta http-equiv="X-UA-Compatible" content="IE=edge">
<meta name="viewport" content="width=device-width,initial-scale=1.0,minimun-scale=1.0,maximum-scale=1.0,user-scalable=no">
```
