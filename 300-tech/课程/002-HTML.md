---
title: 002-HTML
date: 2023-02-06 21:36
updated: 2023-03-19 22:34
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - 002-HTML
rating: 1
tags:
  - HTML
category: web
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: null
url: //myblog.wallleap.cn/post/1
---

## class 与 id

```html
<div class="item">选项1</div>
<div class="item active">选项2</div>
<div id="avatar">头像</div>
```

- `id` 表示独⼀无⼆的“身份”，⼀个元素只能有⼀个 `id`，不同元素不能有相同 `id`
- `class` 表示“特征” 或者“共同点”，一个元素可以有多个不同 `class`，不同元素可以有相同 `class`
- 多个 `class` 用空格分隔
- 多用 `class`，只对某些特殊且重要元素用 `id`

## 属性

```html
<a id="copyright" class="link select" href="/" target="_blank" title="饥人谷">饥人谷</a>
```

- 全局属性
	- 所有标签都可以拥有， `id`、`class`、`title`、`style`... [了解更多](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Global_attributes)
- 自有属性
	- 仅自己可拥有，`<a>` 的 `href`、`target`；`<img>` 的 `src`、`alt` 等
- 自定义属性 `data-*`
	- 用户自定义，用于存储一些数据

## class起名

### 传统起名法

布局类

header, footer, container, main, content, aside, page, section

交互类

tips, alert, modal, pop, panel, tabs, accordion, slide, overlay

状态类

active, current, checked, hover, fail, success, warn, error, on, off

包裹类

wrapper, inner, box, block

大小类

s, m, l, xl, large, small

主次类

primary, secondary, sub, minor

样式类

text-large, color-primary, m-0, mt-10

结构类

hd, bd, ft, top, bottom, left, right, middle, col, row, grid, span

列表类

list, item, field

导航类

nav, prev, next, breadcrumb, forward, back, indicator, paging, first, last

### BEM起名法

`[block]__[element]--[modifier]`

- Block 我的容器是谁
- Element 我是谁
- Modifier 我是什么形态

```
范例：.nav__logo，.photo__img, .photo__img--framed.btn--secondary.hero__text--small, .hero__text--big
```

## 常用标签

### 按钮

```html
<button>确定</button>
<button disabled>禁用</button>
```

- `button` 有个 `type` 属性，属性值可以为 `button`、`submit`、`reset`，前者是默认的，后两者在表单中起到提交和重置的作用（可以通过点击事件实现）

### span、i、strong、em

```html
<div>
	<strong>饥</strong>人谷
</div>
<div>
	<span>饥</span>人谷
</div>
<div>
	<em>饥</em>人谷
</div>
<div><i>饥</i>人谷</div>
```

- 使用 `span` 表示标记普通的文本
- 使用 `i` 表示带有不同语义的文本
- 使用 `em` 表示强调或重读
- 使用 `strong` 表示重要性

一般经验：`div` 包含 `span` 包含 `i`

### figure、img、figcaption

```html
 <figure> 
	 <img    src="https://static.xiedaimala.com/xdml/image/8334f008-f5bb-41ea-9654-87bb65118b1e/MjAyMi05LTItMTEtNDItNC03Njk=.png"    alt="三个卡通人在写字看电脑看书">
	 <figcaption>饥人谷前端</figcaption>
 </figure> 
```

- `alt` 作用：
	1. 当图片不能正常显示的时候会显示 `alt` 中文字
	2. SEO，方便搜索引擎收录图片

### 表格

```html
<table>
	<thead>
		<tr><th>Items</th><th>Expenditure</th></tr>
	</thead>
	<tbody>
		<tr><td>Donats</td><td>3,000</td></tr>
		<tr><td>Stationery</td><td>18,000</td></tr>
	</tbody>
	<tfoot>
		<tr><td>Totals</td><td>21,000</td></tr>
	</tfoot>
</table>
```

- `table`、`thead`、`tbody`、`tfoot`（四部分）
- `th`、`tr`、`td`（单元格）

### 音视频

**video**

```html
<video controls>
	<source src="./a.webm" type="video/webm">
</video>
```

加上 `controls` 才会显示操控组件（播放、暂停等）

**audio**

```html
<figure>
	<figcaption>Listen to the T-Rex:</figcaption>
	<audio controls src="./a.mp3"></audio>
</figure>
```

### 表单

用于收集用户输入并提交的数据

```html
<input type="text" name="username" value="jirengu"><input type="password" name="password" >
```

收集到的数据

```json
一般是 JSON 格式
{
  "username": "jirengu",
  "password": "123456"
}
```

常见的登录表单

```html
<form class="form" action="/login" method="post">
    <h2 class="form__title">登录</h2>
    <div class="form__input">
      <label for="username">用户名</label>
      <input id="username" class="form__username" name="username" type="text" required placeholder="输入用户名">
    </div>
    <div class="form__input">
      <label for="password">密码</label>
      <input id="password" class="form__password" name="password" type="password" autofocus placeholder="输入密码">
    </div>
 <form> 
```

- `form` 包裹着，点击 `submit` 标签可以直接提交（但是一般就直接使用 `input` 的事件做）
- `label` 可以用 `for` 属性、属性值为对应 `input` 的 `id` 值，或者直接使用 `label` 包裹着文字和 `input`，这样点击文字也能 `focus` 表单
- `id` 和 `name`
- `palceholder` 是占位符，当表单中没有内容时显示
- 其他简单的表单验证，例如 `required` 代表必填项

其他表单控件

```html
  <form class="form" action="/login" method="post">
    <div class="form__hobby">
      <input name="hobby" type="checkbox" value="music"> 音乐
      <input name="hobby" type="checkbox" value="football"> 足球
    </div>
    <div class="form__sex">
      <input name="sex" type="radio" value="female"> 女
      <input name="sex" type="radio" value="male"> 男
    </div>
    <div class="form__textarea">
      <textarea name="intro" cols="30" rows="10" placeholder="个人介绍"></textarea>
    </div>
  <form>
```

- 多选和单选以 `name` 来分组
- 文本框和文本域分别是自闭合标签和双标签，所以一个文本是通过 `value` 属性赋值，另一个是在标签内写

其他表单标签

```html
<select name="city">
  <option value="beijing">北京</option>
  <option value="shanghai" selected >上海</option>
  <option value="hangzhou" >杭州</option>
</select>
<input type="email" placeholder="email">
<input type="file">
<input type="date">
<input type="time" >
<input type="time" disabled>
<input type="range">
<input type="tel">
<input type="button" value="确定">
<input type="submit" value="提交">
<input type="reset" value="重置">
```

## 常见面试题

### HTML 标签

- 块级：`div`、`header`、`main`、`footer`、`section`、`article`、`nav`、`aside`、`h1`~`h6`、`p`、`ul`、`li`、`table`
- 行级：`span`、`i`、`strong`、`em`、`a`、`button`、`img`、`input`

### DTD 的作用

- `<!doctype html>` 放在页面开头
- 告诉浏览器以标准模式渲染页面，不加的话由浏览器自己决定怎么渲染

### HTML 5 是什么

- 是 HTML 4 的升级版本
- 新增了一些标签和属性（表单）

### HTML 语义化

1. 让人能看懂
2. 机器能看懂
3. 对残障人士友好

- 科学选择标签
- 科学命名
- 遵循一些编码规范

### meta viewport

让页面在移动端合理显示

###  src 和 href 的区别

`src` 和 `href` 都是**用来引用外部的资源**，它们的区别如下：

- `src`： 表示对**资源**的引用，它指向的内容会嵌入到当前标签所在的位置。`src` 会将其指向的资源下载并应⽤到⽂档内，如请求 JS 脚本、`img` 图片、`audio` 音频、`video` 视频等。当浏览器解析到该元素时，会暂停其他资源的下载和处理，直到将该资源加载、编译、执⾏完毕，所以⼀般 JS 脚本会放在页面底部。
- `href`： 表示**超文本引用**，它指向一些网络资源，建立和当前元素或本文档的链接关系。当浏览器识别到它他指向的⽂件时，就会并⾏下载资源，不会停⽌对当前⽂档的处理。 常用在 `a`、`link` 等标签上。
