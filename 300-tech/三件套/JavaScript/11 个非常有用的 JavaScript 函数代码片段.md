---
title: 11 个非常有用的 JavaScript 函数代码片段
date: 2023-08-03 15:22
updated: 2023-08-03 15:30
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - 11 个非常有用的 JavaScript 函数代码片段
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

JavaScript 是前端领域里功能强大的编程语言，它也是现代 Web 开发的主要语言之一。 作为一名开发人员，拥有一组方便的 JavaScript 函数片段可以提高您的工作效率，并使您能够编写更清晰、更高效的代码。 

在今天这篇文章中，我们将探讨一些非常有用的 JavaScript 函数片段，希望对您有用。

## 01、randomIntInRange

生成特定范围内的随机整数，是 JavaScript 应用程序中的常见需求。 `randomIntInRange` 函数允许您在给定的最小和最大范围内生成随机整数。

```js
function randomIntInRange(min, max) {
  return Math.floor(Math.random() * (max - min + 1)) + min;
}
```

通过利用 `Math.random` 函数返回 0（含）和 1（不含）之间的随机数，并应用适当的缩放和舍入，`randomIntInRange` 生成指定范围内的随机整数。

## 02、formatBytes

将文件大小从字节转换为人类可读的格式（例如，千字节、兆字节）是 Web 应用程序中的一项常见任务。 `formatBytes` 函数将给定数量的字节转换为人类可读的字符串表示形式。

```js
function formatBytes(bytes) {
  if (bytes === 0) {
    return '0 Bytes';
  }
  const k = 1024;
  const sizes = ['Bytes', 'KB', 'MB', 'GB', 'TB'];
  const i = Math.floor(Math.log(bytes) / Math.log(k));
  return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
}
```

`formatBytes` 函数通过将输入字节除以 1024 的幂来处理不同大小的字节。然后它根据计算出的大小从 sizes 数组中选择适当的单位。 使用 `toFixed(2)` 将结果值格式化为两位小数，并在其后附加单位。

## 03、formatDate

使用日期和时间是 Web 开发中的常见要求。 `formatDate` 函数提供了一种将 JavaScript Date 对象格式化为所需字符串表示形式的便捷方法。

```js
function formatDate(date, format) {
  const options = {
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  };
  return date.toLocaleDateString(format, options);
}
```

`formatDate` 函数将 `Date` 对象和格式字符串作为输入。 在此示例中，我们使用 `toLocaleDateString` 方法根据指定选项格式化日期。

通过为“年、月和日”指定选项，我们可以自定义结果字符串的格式。 您可以修改选项以满足您的特定格式要求。

## 04、capitalize

将字符串的首字母大写是一个简单的格式化任务，经常出现在 JavaScript 应用程序中。 `capitalize` 函数将给定字符串的第一个字母大写，同时保留字符串的其余部分。

```js
function capitalize(str) {
  return str.charAt(0).toUpperCase() + str.slice(1);
}
```

`capitalize` 函数使用 `charAt(0)` 提取字符串的第一个字符，使用 `toUpperCase()` 将其转换为大写，然后使用 `slice(1)` 将其与字符串的其余部分连接起来。 这导致原始字符串的大写版本。

## 05、scrollToTop

滚动到网页顶部是一种常见的交互方式，尤其是在存在长内容的场景中。 `scrollToTop` 函数将页面平滑地滚动到顶部位置。

```js
function scrollToTop() {
  window.scrollTo({
    top: 0,
    behavior: 'smooth'
  });
}
```

`scrollToTop` 函数利用窗口对象的 `scrollTo` 方法滚动到顶部位置。 通过将顶部值设置为 0 并将行为指定为“平滑”，页面可以平滑地滚动到顶部。

## 06、once

在某些情况下，您希望某个功能只执行一次，类似于您使用 `onload` 事件的方式。 `once` 函数确保给定函数只被调用一次，防止重复初始化或执行。

```js
function once(callback) {
  let executed = false;
  return function() {
    if (!executed) {
      executed = true;
      callback();
    }
  };
}
```

要使用 `once` 函数，请将所需函数作为回调传递。 然后可以调用返回的函数，它将确保回调仅在第一次调用时执行。

## 07、truncateString

有时，您可能需要将字符串截断到特定长度并在末尾添加省略号 (...) 以指示截断。 `truncateString` 函数将给定的字符串截断为指定的最大长度，并在必要时附加省略号。

```js
function truncateString(str, maxLength) {
  if (str.length <= maxLength) {
    return str;
  }
  return str.slice(0, maxLength) + '...';
}
```

`truncateString` 函数检查字符串的长度是否小于或等于指定的 `maxLength`。 如果是这样，它将按原样返回原始字符串。 否则，它使用 slice 提取字符串的一部分，从开头到 `maxLength` 并附加省略号以指示截断。

## 08、isNative

在某些情况下，了解给定功能是否是本机功能至关重要，尤其是在决定是否覆盖它时。 `isNative` 函数允许您确定一个函数是否是原生的或者它是否已经在 JavaScript 中实现。

```js
function isNative(fn) {
  return /\[native code\]/.test(fn.toString());
}
```

`isNative` 函数利用正则表达式来检查函数的字符串表示形式。 如果函数的源代码包含短语“\[native code\]”，则表示该函数是浏览器原生的，而不是在 JavaScript 中实现的

## 09、debouncePromise

有时您可能需要消除异步操作，例如进行 API 调用或处理用户输入。 `debouncePromise` 函数提供了一种去抖动 Promise 的方法，确保只执行最后一次调用。

```js
function debouncePromise(fn, delay) {
  let timeoutId;
  return function (...args) {
    return new Promise((resolve, reject) => {
      clearTimeout(timeoutId);
      timeoutId = setTimeout(async () => {
        try {
          const result = await fn(...args);
          resolve(result);
        } catch (error) {
          reject(error);
        }
      }, delay);
    });
  };
}
```

`debouncePromise` 函数采用基于 Promise 的函数 (fn) 和延迟作为参数。 它返回一个将原始函数包装在 Promise 中的去抖动函数。 该函数使用 `setTimeout` 对调用进行去抖动，确保在指定延迟后仅执行最后一次调用。 它根据包装函数的结果解决或拒绝承诺。

## 10、memoize

memoize 化是一种用于缓存昂贵函数调用的结果并为后续调用检索它们的技术。 `memoize` 函数提供了一种通用方法来记忆任何具有不同参数的函数。

```js
function memoize(fn) {
  const cache = new Map();
  return function (...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      return cache.get(key);
    }
    const result = fn(...args);
    cache.set(key, result);
    return result;
  };
}
```

`memoize` 函数使用 Map 对象创建缓存。 它返回一个包装函数，检查当前参数的结果是否存在于缓存中。 如果是，它会检索并返回缓存的结果。 否则，它将使用参数调用原始函数，将结果存储在缓存中并返回。 使用相同参数的后续调用将检索缓存的结果，而不是重新计算它。

## 11、insertRule

在处理动态和大量使用 AJAX 的网站时，将样式应用于多个元素可能效率低下且麻烦。 `insertRule` 函数提供了一种更有效的方法，它允许您为选择器定义样式，类似于您在样式表中的做法。

```js
function insertRule(selector, style) {
  const styleSheet = document.styleSheets[0];
  if (styleSheet.insertRule) {
    styleSheet.insertRule(`${selector} { ${style} }`, 0);
  } else if (styleSheet.addRule) {
    styleSheet.addRule(selector, style, 0);
  }
}
```

要使用 `insertRule` 函数，请提供所需的选择器和样式作为参数。 该函数将在文档中找到的第一个样式表中插入一个新规则，确保指定的样式应用于与选择器匹配的所有元素。

## 结论

请记住定制这些片段以适合您的项目需求和编码风格。 对它们进行试验、组合，并在它们的基础上进行构建，以创建更强大的自定义功能。

最后，感谢您的阅读，希望对您有所帮助！ 
