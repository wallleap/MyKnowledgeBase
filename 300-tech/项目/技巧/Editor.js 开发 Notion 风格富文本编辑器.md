---
title: Editor.js 开发 Notion 风格富文本编辑器
date: 2024-01-10 10:31
updated: 2024-01-10 10:31
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - Editor.js 开发 Notion 风格富文本编辑器
rating: 1
tags:
  - web
  - JavaScript
category: web
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: 
url: //myblog.wallleap.cn/post/1
---

## 前言

开发一个编辑博客内容的富文本编辑器，风格类似于 Notion 块级的编辑器，研究了 [editorjs](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fcodex-team%2Feditor.js "https://github.com/codex-team/editor.js") 开源项目，和它的风格接近，editorjs 界面简洁，操作个人感觉很高级，插件灵活易扩展，样式可编辑修改，开发成本也没有那么复杂

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5b160ee67e2345d9bbf4351547c5a7ae~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.gif)


## 介绍

Editor.js 亮点

- **用户界面友好**：Editor.js 提供了一个直观且易于使用的用户界面，能够轻松地创建和编辑文档

- **模块化编辑**： 使用块级元素（blocks）的方式组织文本，每个块负责处理不同类型的内容，易于添加新功能或定制编辑器，如段落、标题

- **可扩展性**： Editor.js 具有可插拔的 API 插件，可以方便地扩展功能，添加自定义的插件和内容

- **跨平台兼容性**： 编辑器输出简洁的 json 数据，可以解析转换嵌入到不同的平台和应用程序中，支持跨平台。

Editor.js 每一个 block 块时可编辑由 div `contenteditable` 属性实现，工作区由单独的块组成

如段落、标题、图像、列表、引号等。每个块都是由插件提供的独立内容可编辑元素，并由编辑器核心统一

## 开始安装

```shell
npm i @editorjs/editorjs --save
```

引入使用

```js
import EditorJS from '@editorjs/editorjs'; const editor = new EditorJS({ /** * Id of Element that should contain Editor instance */ holder: 'editorjs' });
```

编辑器提供一个默认的段落块，要使用标题、列表等块，需要安装对应的插件，在 [awesome-editorjs](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Feditor-js%2Fawesome-editorjs "https://github.com/editor-js/awesome-editorjs") 社区提供了丰富的插件

安装标题、列表插件

```js
import EditorJS from '@editorjs/editorjs'; import Header from '@editorjs/header'; import List from '@editorjs/list'; const editor = new EditorJS({ holder: 'editorjs', tools: { header: Header, list: List }, })
```

标题和列表可以支持配置，以 class 类作为插件，如开启行内工具 `inlineToolbar`，可配置 config 自定义配置、快捷键、和 svg 图标

```js
import EditorJS from '@editorjs/editorjs'; import Header from '@editorjs/header'; import List from '@editorjs/list'; const editor = new EditorJS({ holder: 'editorjs', tools: { header: { class: Header, // 标题 class 插件 inlineToolbar: ['link'], // 开启行内工具，只展示链接 config: { defaultLevel: 1, // 默认标题 // 默认创建的标题 levels: [1, 2, 3, 4] // 可转换的标题 }, toolbox: { icon: IconH1, // 可以修改图标，svg title: 'H1' } }, }, })
```

以修改标题配置为例

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/36b1cd18ecef4cd8a60424195ea89790~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.png#?w=1098&h=638&s=60742&e=png&b=ffffff)

以上配置 H1，默认创建 h1 标签，可以转换 h1-h4，行内工具只显示链接

## editor.js 运行流程

1、首先需要实例化 EditorJs 编辑器实例，`holderId` 是它挂载的 id dom 节点

2、如果希望它在实例化后做一些工作，它提供 `onReady` 方法，它是异步执行的

```js
var editor = new EditorJS({ // Other configuration properties onReady: () => { console.log('Editor.js is ready to work!') } });
```

onReady 是异步执行，可以单独在外通过 promise、async/await 执行

```js
var editor = new EditorJS(); editor.isReady .then(() => { console.log('Editor.js is ready to work!') }) .catch((reason) => { console.log(`Editor.js initialization failed because of ${reason}`) }); // 再或者 async/await 调用 var editor = new EditorJS(); try { await editor.isReady; console.log('Editor.js is ready to work!') } catch (reason) { console.log(`Editor.js initialization failed because of ${reason}`) }
```

3、想知道富文本编辑器的数据发现变化，监听 `onChange` 事件

```js
var editor = new EditorJS({ // Other configuration properties onChange: (api, event) => { console.log('Now I know that Editor\'s content changed!', event) } });
```

## 编辑器功能

### 国际化

editor.js 国际化是通过提数字字典映射实现的，并不是根据 i18n ，映射字段配置在 `i18n.messages` 字段，主要分四部分

- ui：是编辑器内部核心的 UI 界面内容，如转换、点击添加等
- toolNames：块或行内块的名称，如锻炼、标题、列表、斜体
- tools: 插件自定义内部的命名
- blockTunes：块级转换，删除、上移、下移

```js
const editor = new EditorJS({ ... i18n: { messages: { ui: { blockTunes: { toggler: { 'Click to tune': '点击转换' } }, inlineToolbar: { converter: { 'Convert to': '转换' } }, toolbar: { toolbox: { Add: '工具栏添加' } }, popover: { Filter: '过滤', 'Nothing found': '找不到' } }, toolNames: { Text: '段落', Bold: '加粗', Italic: '斜体', }, tools: { paragraph: { 'Press Tab': '输入内容' }, }, blockTunes: { delete: { Delete: '删除' }, moveUp: { 'Move up': '上移' }, moveDown: { 'Move down': '下移' }, } } ... });
```

### 自动聚焦

第一次打开，自动聚焦到编辑器

```js
const editor = new EditorJS({ // other configuration options /** * Enable autofocus */ autofocus: true })
```

### 展位符 placeholder

编辑时，内容为空的展位符提示

```js
const editor = new EditorJS({ ... placeholder: 'Press Tab' ... });
```

### 只读模式

readOnly 只读模式，场景可以根据权限判断是否有编辑权限

```js
const editor = new EditorJS({ // ... readOnly: true, // ... });
```

可以通过 调用 API 进行模式切换

```js
const editor = new EditorJS(); editor.readOnly.toggle();
```

### 内联工具栏顺序

使用 `inlineToolbar` 属性开启内联工具，可排序

```js
const editor = new EditorJS({ // ... inlineToolbar: ['link', 'marker', 'bold', 'italic'], } });
```

### Tunes 块连接

Editor.js 每一个块是独立的，也可以通过配置实现块连接

通过 editor.js 编写自己的连接块

```js
const editor = new EditorJS({ tools: { myTune: MyTune }, tunes: ['myTune'] });
```

然后,在特定的块中，开启块的连接

```js
const editor = new EditorJS({ tools: { myTune: MyTune, blockTool: { class: MyBlockTool, tunes: ['myTune'] } } });
```

实现这个功能的插件，如块的左中右对齐，[editorjs-alignment-blocktune](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fkaaaaaaaaaaai%2Feditorjs-alignment-blocktune "https://github.com/kaaaaaaaaaaai/editorjs-alignment-blocktune") 插件

### 保存数据

editor.js 输出干净的 json 数据，然后保存到后台，用于数据回显

```js
const editor = new EditorJS(); editor.save().then((outputData) => { console.log('Article data: ', outputData) }).catch((error) => { console.log('Saving failed: ', error) });
```

outputData 输出的数据格式，它是一个 json 对象，数据存储到 `blocks` 数组中

- 对象 type 表现不同的块类型，如标题、段落、列表 type 分别表示为 header、paragraph、list
- 数据是存储到 `data` 对象中，文本是在 text 字段表示，标题等级用 level

```json
{ "time": 1550476186479, "blocks": [ { "id": "oUq2g_tl8y", "type": "header", "data": { "text": "Editor.js", "level": 2 } }, { "id": "zbGZFPM-iI", "type": "paragraph", "data": { "text": "dddd." } }, { "id": "XV87kJS_H1", "type": "list", "data": { "style": "unordered", "items": [ "It is a block-styled editor", "It returns clean data output in JSON", "Designed to be extendable and pluggable with a simple API" ] } } ], "version": "2.8.1" }
```

## 总结

从 `Editor.js` 的设计是有很多地方值得借鉴

- 它是使用 TS 的面向对象编程的，每个块的功能都定义的很清新，代码解偶
- 用实例化对象来做数据隔离，并通过设计 API 通知外部的变化，如 onReady、onChange，在编辑器销毁提供 destroy 方法释放资源，防止内存溢出，方法的设计考虑的很全面
- 通过配置增强扩展，如自动聚焦、占位符、国际化，来扩展编辑器的功能
- 特别是它的插件设计机制，分为块级、行内工具等，每个块提供约定的 api，短短几百行就可以编写一个自己的插件，文档易读

如果真正应用到线上产品，还是有很多地方需要打磨

- 工具栏功能需要加强，特别是**链接**工具，当聚焦到输入框，选中范围发生变化，要决解工具栏直接的切换问题
- 表格的功能，如拖拽、合并等高级功能
- 图片、按钮元素，在撤销、重做、回车键删除等的流畅体验操作
- 兼容移动端、小程序，需要编写一套解析规则
- ……

目前它的 `start` 数达到了 2.5+w，相信它将来会变得越来越强大，是一个不错的开源学习项目
