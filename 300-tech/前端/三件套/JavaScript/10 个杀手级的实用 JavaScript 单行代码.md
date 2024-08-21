---
title: 10 个杀手级的实用 JavaScript 单行代码
date: 2023-08-03T05:25:00+08:00
updated: 2024-08-21T10:32:48+08:00
dg-publish: false
aliases:
  - 10 个杀手级的实用 JavaScript 单行代码
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
  - JS
  - web
url: //myblog.wallleap.cn/post/1
---

JavaScript 是一门简单而复杂的语言，简单是因为他有很多框架库可以使用，复杂也是因为它有太多的框架库可以选择。

很多时候，我们不知道如何使用，但是，在实际开发中，我们经常用的东西真的不多，在前端领域里，如果 CSS 能够实现的效果，其实，很多时候不鼓励用 JavaScript 来实现。

但是，如果有时候，我们用一行代码就可以实现的话，我们还是可以考虑一下使用 JavaScript 来完成需求任务。

因此，今天这篇文章，我想与大家分享 11 个实用的 JavaScript 单行代码技巧，也是每个前端开发人员都应该学会使用 的技巧，使用这些 javascript 单行技巧，可以帮助我们提高生产力和技能。

我们不多说，现在开始吧。

## 排序数组

使用 `sort` 方法对数组进行排序非常容易。

```js
const number = [2,6,3,7,8,4,0];number.sort(); 
// expected output: [0,2,3,4,6,7,8]
```

## 检查数组中的值

很多时候我们需要在 `includes` 方法的帮助下检查值是否存在于数组中。

```js
const array1 = [1, 2, 3];console.log(array1.includes(2));
// expected output: true
```

## 过滤数组

```js
const words = ['spray', 'limit', 'elite', 'exuberant', 'destruction', 'present'];

const result = words.filter(word => word.length > 6);console.log(result);
// expected output: Array ["exuberant", "destruction", "present"]
```

## 从数组中查找元素

如果你只需要一个元素，但你在数组中得到了很多元素，不用担心 JavaScript 有 `find` 方法。

```js
const array1 = [5, 12, 8, 130, 44];const found = array1.find(element => element > 10);console.log(found);
// expected output: 12
```

## 查找数组中任意元素的索引

要查找数组中元素的索引，您可以简单地使用 `indexOf` 方法。

```js
const beasts = ['ant', 'bison', 'camel', 'duck', 'bison'];console.log(beasts.indexOf('bison'));
// expected output: 1
```

## 将数组转换为字符串

```js
const elements = ['Fire', 'Air', 'Water'];console.log(elements.join(", "));
// expected output: "Fire, Air, Water"
```

## 查找数字是偶数还是奇数

很容易找出给定的数字是偶数还是奇数。

```js
const isEven = num => num % 2 === 0;
// or
const isEven = num => !(n & 1);
```

## 删除数组中的所有重复值

删除数组中所有重复值的一种非常简单的方法。

```js
const setArray = arr => [...new Set(arr)];const arr = [1,2,3,4,5,1,3,4,5,2,6];
setArray(arr);
// expected output: [1,2,3,4,5,6]
```

## 合并多个数组的不同方式

```js
// merge but don't remove duplications
const merge = (a, b) => a.concat(b);
// or 
const merge = (a, b) => [...a, ...b]; // merge with remove duplications
const merge = (a, b) => [...new Set(a.concat(b))];
// or 
const merge = (a, b) => [...new Set([...a, ...b])];
```

## 滚动到页面顶部

有很多方法可以将页面滚动到顶部。

```js
const goToTop = () => window.scrollTo(0,0, "smooth");
// or
const scrollToTop = (element) => element.scrollIntoView({behavior: "smooth", block: "start"}); // scroll to bottom of the page
const scrollToBottom = () => window.scrollTo(0, document.body.scrollHeight);
```

## 复制到剪贴板

在 web 应用程序中，复制到剪贴板因其对用户的便利性而迅速流行起来。

```js
const copyToClipboard = text => (navigator.clipboard?.writeText ?? Promise.reject)(text);
```

## 总结

以上就是我今天与你分享的全部内容，希望对你有用，如果你觉得不错的话，请记得点赞我，关注我，并将这篇文章分享给你的小伙伴，也许能够帮助到他，与你一起学习进步。

最后，感谢你的阅读，编程愉快！
