---
title: 浅谈 JavaScript 中的防抖和节流
date: 2023-08-04 16:07
updated: 2023-08-04 16:09
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - 浅谈 JavaScript 中的防抖和节流
rating: 1
tags:
  - JS
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

> 防抖和节流函数是工作中两种常用的前端性能优化函数，今天我就来总结一下什么是防抖和节流，并详细说明一下如何在工作中应用防抖和节流函数

## 什么是防抖和节流?

在 JavaScript 中，防抖（debounce）和节流（throttle）是用来限制函数执行频率的两种常见技术。

**防抖（debounce）** 是指在某个时间段内，只执行最后一次触发的函数调用。如果在这个时间段内再次触发该函数，会重新计时，直到等待时间结束才会执行函数。 这个技术通常用于处理频繁触发的事件，比如窗口大小调整、搜索框输入等。防抖可以避免函数执行过多次，以减少网络开销和性能负担。

**节流（throttle）** 是指在一段时间内限制函数的执行频率，保证一定时间内只执行一次函数调用。无论触发频率多高，都会在指定时间间隔内执行一次函数。 这个技术通常用于处理连续触发的事件，比如滚动事件、鼠标移动事件等。节流可以控制函数的执行频率，以减少资源消耗和提高性能。

### 手写一个防抖的工具函数

```js
function debounce(func, delay) {
  let timeoutId;
  
  return function() {
    const context = this;
    const args = arguments;
    
    clearTimeout(timeoutId);
    
    timeoutId = setTimeout(function() {
      func.apply(context, args);
    }, delay);
  };
}
```

#### 函数说明

1. 这个防抖函数接受两个参数：`func` 表示需要进行防抖的函数，`delay` 表示延迟的时间间隔（以毫秒为单位）
2. 函数内部使用了一个 `timeoutId` 变量来保存定时器的标识。当调用防抖函数返回的新函数时，会清除之前的定时器，并设置一个新的定时器。只有在延迟时间内没有再次调用该新函数时，才会触发最终的函数执行

#### 使用示例

该示例表示在全局滚动事件中使用防抖函数，每 200 毫秒内如果触发滚动事件，那么不会执行 `handleScroll()` 函数。 然后重新计时 200 毫秒，再次判断，直到最后一个 200 毫秒内没有触发滚动事件，才会执行 `handleScroll()` 函数

```js
function handleScroll() {
  console.log('Scrolled');
}
const debouncedScroll = debounce(handleScroll, 200);
window.addEventListener('scroll', debouncedScroll);
```

### 手写一个节流的工具函数

```js
function throttle(func, delay) {
  let timeoutId;
  let lastExecTime = 0;

  return function(...args) {
    const currentTime = Date.now();
    const remainingTime = delay - (currentTime - lastExecTime);

    clearTimeout(timeoutId);

    if (remainingTime <= 0) {
      func.apply(this, args);
      lastExecTime = currentTime;
    } else {
      timeoutId = setTimeout(() => {
        func.apply(this, args);
        lastExecTime = Date.now();
      }, remainingTime);
    }
  };
}
```

#### 函数说明

1. 这个节流函数接受两个参数：`func` 是要执行的函数，`delay` 是延迟时间（以毫秒为单位）
2. 它返回一个新的函数，该函数在调用时会根据指定的延迟时间来限制原始函数的执行频率

#### 使用示例

在全局滚动事件中使用节流函数，无论在滚动事件的监听过程中，触发了几次 `handleScroll()` 函数，都只会在每 200 毫秒内执行一次 `handleScroll()` 函数。

```js
function handleScroll() {
  console.log('Scrolled');
}
const throttledScroll = throttle(handleScroll, 200);
window.addEventListener('scroll', throttledScroll);
```

## 如何在工作中应用防抖和节流

> 防抖和节流主要应用于：搜索框输入事件监听、窗口大小调整事件监听、按钮点击事件监听、滚动事件监听、鼠标移动事件监听等等场景。

### 工作中哪些场景可以使用防抖函数？

1. **用户输入：** 当用户在表单输入框中频繁输入时，可以使用防抖函数来延迟处理用户输入，避免频繁的请求或操作，提高性能和用户体验。
2. **搜索框：** 在搜索框中，当用户连续输入关键字时，可以使用防抖函数来延迟发送搜索请求，以避免请求过多。
3. **窗口调整：** 当窗口大小调整时，会触发 resize 事件，可以使用防抖函数来限制 resize 事件的触发次数，避免频繁执行调整相关的代码。
4. **按钮频繁点击：** 当按钮被频繁点击时，可以使用防抖函数来限制按钮点击的触发次数。

### 工作中哪些场景可以使用节流函数？

1. **用户输入：** 当用户在文本框中输入时，触发搜索功能。使用节流函数可以限制搜索请求的频率，以避免频繁的网络请求。例如，可以设置一个定时器，在用户输入后的一小段时间内不触发搜索请求，只在定时器结束后才进行搜索。
2. **无限加载：** 当用户滚动页面时，触发加载更多数据的操作。使用节流函数可以限制加载操作的频率，以提高页面的响应性能。例如，可以设置一个定时器，在用户滚动过程中只触发加载操作的最后一次滚动事件。
3. **按钮频繁点击：** 当用户频繁点击某个按钮时，触发某个操作。使用节流函数可以限制点击操作的频率，以避免重复操作或者混乱的界面状态。例如，可以设置一个定时器，在用户点击后的一小段时间内不触发重复操作。

### 如何使用 `loadsh.js` 工具库中的防抖和节流函数

实际开发过程中，我们的项目中可能会直接使用 `loadsh.js` 工具库，来避免重复造轮子，所以这里也特地说明一下如何使用 `loadsh.js` 工具库中的防抖和节流函数

安装 `loadsh.js` 工具库

```js
npm install lodash  
```

在项目中引入 `loadsh.js` 工具库，不同前端项目引入方式不同，请自行鉴别

```js
import { debounce, throttle } from 'loadsh';
```

使用防抖函数，该示例中，`submitData()` 函数被限制为每 1 秒只会执行最后一次，如果在等待时间内多次调用该函数，则会重置 1 秒的等待时间

```js
// 定义要延迟执行的函数
function submitData(data) {
  console.log('保存数据：', data);
}

// 使用debounce函数创建一个延迟执行的函数
const debouncedFn = _.debounce(submitData, 1000);

// 模拟连续触发保存数据的操作
// 等待1秒后，只会执行最后一次保存数据的操作
debouncedFn('数据1'); // 不会输出
debouncedFn('数据2'); // 不会输出
debouncedFn('数据3'); //  输出 —— 保存数据：数据3
```

使用节流函数，该示例中，`throttledFn()` 函数被限制为每秒只能执行一次，如果在等待时间内多次调用该函数，则不会执行

```js
// 定义要减少调用次数的函数
function fetchData(data) {
console.log('拉取数据:', data);
}

// 使用throttle函数创建一个定时执行的函数
const throttledFn = _.throttle(fetchData, 2000);

// 调用throttledFn函数
throttledFn('数据1'); // 输出 —— 拉取数据: 数据1

// 在1秒内多次调用throttledFn函数
throttledFn('数据2'); // 不会输出

// 2秒后再次调用throttledFn函数
setTimeout(() => {
  throttledFn('数据3'); // 输出 —— 拉取数据: 数据3
}, 1000);
```

我是 [fx67ll.com](https://link.juejin.cn/?target=https%3A%2F%2Ffx67ll.com "https://fx67ll.com")，如果您发现本文有什么错误，欢迎在评论区讨论指正，感谢您的阅读！

如果您喜欢这篇文章，欢迎访问我的 [本文github仓库地址](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Ffx67ll%2Ffx67llJs%2Fblob%2Fmaster%2Fjs-blog%2F2023%2F2023-08%2Fjs-debounce-throttle.md "https://github.com/fx67ll/fx67llJs/blob/master/js-blog/2023/2023-08/js-debounce-throttle.md")，为我点一颗 Star，Thanks~ :)

**转发请注明参考文章地址，非常感谢！！！**
