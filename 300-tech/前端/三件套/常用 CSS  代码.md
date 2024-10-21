---
title: 常用 CSS  代码
date: 2023-07-26T05:00:00+08:00
updated: 2024-08-21T10:32:36+08:00
dg-publish: false
aliases:
  - 常用 CSS  代码
author: Luwang
category: 分类
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
  - blog
url: //myblog.wallleap.cn/post/1
---

## 系统字体

使用 Font.css —— 跨平台中文字体解决方案

[Fonts.css (zenozeng.github.io)](https://zenozeng.github.io/fonts.css/)

[github.com/zenozeng/fonts.css](https://github.com/zenozeng/fonts.css)

```css
--font-emoji: 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol',
  'Noto Color Emoji';

--font-sans: 'Inter', -apple-system, 'HarmonyOS Sans SC', 'Source Han Sans SC', 'Source Han Sans',
  'Noto Sans SC', BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen', 'Ubuntu', 'Cantarell',
  'Fira Sans', 'Droid Sans', 'Helvetica Neue', sans-serif, var(--font-emoji);

--font-serif: 'Source Han Serif SC', 'Source Han Serif', 'Noto Serif SC',
  'Noto Serif', Georgia, 'Times New Roman', Times, 'Droid Serif', serif, var(--font-emoji);

--font-mono: 'JetBrains Mono', 'Fira Mono', Consolas, 'Courier New',
  'Droid Sans Mono', var(--font-sans);
```

### 黑体

```css
/* Apple System, Noto Sans, Helvetica Neue, Helvetica, Nimbus Sans L, Arial, Liberation Sans, PingFang, 冬青黑, 思源黑体, 微软雅黑, 文泉驿微米黑, 文泉驿正黑, 华文黑体, 中易黑体, 文泉驿点阵正黑 */
font-family: -apple-system, "Noto Sans", "Helvetica Neue", Helvetica, "Nimbus Sans L", Arial, "Liberation Sans", "PingFang SC", "Hiragino Sans GB", "Noto Sans CJK SC", "Source Han Sans SC", "Source Han Sans CN", "Microsoft YaHei", "Wenquanyi Micro Hei", "WenQuanYi Zen Hei", "ST Heiti", SimHei, "WenQuanYi Zen Hei Sharp", sans-serif;
```

### 楷体

```css
/* Baskerville, Georgia, Liberation Serif, Kaiti SC, 华文楷体, AR PL UKai, 文鼎ＰＬ简中楷, 楷体, 標楷體, 全字库正楷体 */
font-family: Baskerville, Georgia, "Liberation Serif", "Kaiti SC", STKaiti, "AR PL UKai CN", "AR PL UKai HK", "AR PL UKai TW", "AR PL UKai TW MBE", "AR PL KaitiM GB", KaiTi, KaiTi_GB2312, DFKai-SB, "TW\-Kai", serif;
```

### 宋体

```css
/* Georgia, Nimbus Roman No9, Songti SC, 思源宋体, 华文宋体, AR PL New Sung, 文鼎ＰＬ简报宋, 新宋体, 中易宋体, 全字库正宋体, 文泉驿点阵宋, AR PL UMing, 新细明体, 细明体 */
font-family: Georgia, "Nimbus Roman No9 L", "Songti SC", "Noto Serif CJK SC", "Source Han Serif SC", "Source Han Serif CN", STSong, "AR PL New Sung", "AR PL SungtiL GB", NSimSun, SimSun, "TW\-Sung", "WenQuanYi Bitmap Song", "AR PL UMing CN", "AR PL UMing HK", "AR PL UMing TW", "AR PL UMing TW MBE", PMingLiU, MingLiU, serif;
```

### 仿宋

```css
/* Baskerville, Times New Roman, Liberation Serif, 华文仿宋, 仿宋, CWTEX仿宋体 */
font-family: Baskerville, "Times New Roman", "Liberation Serif", STFangsong, FangSong, FangSong_GB2312, "CWTEX\-F", serif;
```

## 伪元素实现 Tooltip

```html
<style>
span::after {
  content: attr(data-tooltip);
  position: absolute;
  left: 50%;
  top: calc(100% + 0.5rem);
  padding: 0.25rem 0.75rem;
  display: flex;
  border-radius: 0.25rem;
  color: #ffffff;
  background-color: #181818;
  white-space: nowrap;
  transform: translateX(-50%);
}
</style>
<span data-tooltip="Hello"> CSS Tooltip </span>
```

## 文本处理

### 单行文本溢出省略号

```css
p {
  white-space: nowrap;
  text-overflow: ellipsis;
  overflow: hidden;
}
```

记得限制好外部容器的宽度。

### 多行文本溢出省略号

```css
p {
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 3; // 显示的行数
  -webkit-box-orient: vertical;
}
```

### 文本两端对齐

```css
text-align: justify;
text-justify: auto;
overflow-wrap: break-word;
```

如果屏幕比较窄而且长单词多，就不要用这个。

## CSS mask 和 text

```css
background-image: linear-gradient(to bottom, #ec428c, #32d1d3); // 背景线性渐变
color: #ffffff; // 兜底颜色，防止文字裁剪不生效
background-clip: text;
-webkit-background-clip: text; // 背景被裁减为文字的前景色
-webkit-text-fill-color: transparent; // 文字填充为透明，优先级比color高。
```

webkit-mask

```css
-webkit-mask: linear-gradient(to bottom, transparent, #000);
```

## Clip path

```css
clip-path: circle(0% at 50% 50%)
```

## shape-outside

```css
float: left;
shape-outside: circle(0% at 50% 50%);
```

## 文字交融

```css
.parent {
  filter: contrast(30);
}
.child {
  /* 过渡下面两个属性的值 */
  letter-spacing: -4em;
  filter: blur(5px);
}
```

