---
title: JS 中的文档注释
date: 2023-08-10 17:51
updated: 2023-08-11 11:20
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - JS 中的文档注释
rating: 1
tags:
  - blog
category: 分类
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: //www.jsdoc.com.cn
url: //myblog.wallleap.cn/post/1
---

写法：

```js
/**
* 函数名
* 函数解释功能
* @author 作者 <email@domail.com>
* @license MIT
* @param {类型} 参数1 参数注释
* @param {类型} 参数2 参数注释
* @return {类型} 返回值注释
* @example
* 代码用例 // 注释内容
```

类型有：`Number`、`String`、`Boolean`、`Object`、`'Value1' | 'Value2'`（枚举）

VSCode 中快捷添加：`/**` 出现提示后按 <kbd>Tab</kbd>

## 最简单的一句话描述

```js
/** This is a description of the foo function. */  
function foo() {  
}
```

## 构造函数

使用 `@constructor` 标记

```js
/**  
* Represents a book.  
* @constructor  
*/  
function Book(title, author) {  
}
```

如果有参数可以添加 `@param`

```js
/**  
* Represents a book.  
* @constructor  
* @param {string} title - The title of the book.  
* @param {string} author - The author of the book.  
*/  
function Book(title, author) {  
}
```

[你不知道的JSDoc - 掘金 (juejin.cn)](https://juejin.cn/post/7072685382323830821)

使用 JS Doc 包导出
