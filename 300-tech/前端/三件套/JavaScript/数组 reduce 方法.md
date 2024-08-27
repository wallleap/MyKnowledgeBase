---
title: 数组 reduce 方法
date: 2024-08-26T10:29:52+08:00
updated: 2024-08-26T10:34:05+08:00
dg-publish: false
---

```js
const callbackFn = (累加值, 当前值, 当前值的下标, 数组) => {}
reduce(callbackFn)
reduce(callbackFn, initialValue)
```

例子

```js
const array = [15, 16, 17, 18, 19];

function reducer(accumulator, currentValue, index) {
  const returns = accumulator + currentValue;
  console.log(
    `accumulator: ${accumulator}, currentValue: ${currentValue}, index: ${index}, returns: ${returns}`,
  );
  return returns;
}

array.reduce(reducer);
```
