---
title: Reflect 调用对象的基本方法
date: 2024-10-30T04:33:10+08:00
updated: 2024-10-30T16:42:57+08:00
dg-publish: false
---

之前是通过语法去操作对象，本质还是会调用对象的基本方法（Object internal methods）去实现（除了调用基本方法还会执行其他额外代码），ES6 中就把基本方法暴露出来了

| Internal method         | Corresponding trap                                           |
| :---------------------- | :----------------------------------------------------------- |
| `[[GetPrototypeOf]]`    | [`getPrototypeOf()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy/getPrototypeOf) |
| `[[SetPrototypeOf]]`    | [`setPrototypeOf()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy/setPrototypeOf) |
| `[[IsExtensible]]`      | [`isExtensible()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy/isExtensible) |
| `[[PreventExtensions]]` | [`preventExtensions()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy/preventExtensions) |
| `[[GetOwnProperty]]`    | [`getOwnPropertyDescriptor()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy/getOwnPropertyDescriptor) |
| `[[DefineOwnProperty]]` | [`defineProperty()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy/defineProperty) |
| `[[HasProperty]]`       | [`has()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy/has) |
| `[[Get]]`               | [`get()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy/get) |
| `[[Set]]`               | [`set()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy/set) |
| `[[Delete]]`            | [`deleteProperty()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy/deleteProperty) |
| `[[OwnPropertyKeys]]`   | [`ownKeys()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy/ownKeys) |

Function objects also have the following internal methods:

| Internal method | Corresponding trap                                                                                                      |
| :-------------- | :---------------------------------------------------------------------------------------------------------------------- |
| `[[Call]]`      | [`apply()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy/apply)         |
| `[[Construct]]` | [`construct()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy/construct) |

例如

```js
const obj = {}
obj.a = 3
// 上面会触发 Set 的执行

// 可以这样写
Reflect.set(obj, 'a', 3)
```

尤其是 get（读属性）的时候，get 的第三个参数指定 this，默认会封装成自己

和 Proxy 配合使用
