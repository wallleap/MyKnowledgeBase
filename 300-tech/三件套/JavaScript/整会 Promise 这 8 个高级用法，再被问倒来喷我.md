---
title: 整会 Promise 这 8 个高级用法，再被问倒来喷我
date: 2023-08-04 14:38
updated: 2023-08-04 14:41
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - 整会 Promise 这 8 个高级用法，再被问倒来喷我
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

> 本文来自 alova 作者投稿，alova 可以让你编写更少的代码即可实现特定业务场景下的高效数据交互，在过去 3 个月对外发布以来在 github 上已收到了 1500+star，[alova官网](https://link.juejin.cn/?target=https%3A%2F%2Falova.js.org "https://alova.js.org")，感兴趣可以看看。

## 发现很多人还只会 promise 常规用法

在 js 项目中，promise 的使用应该是必不可少的，但我发现在同事和面试者中，很多中级或以上的前端都还停留在 `promiseInst.then()`、`promiseInst.catch()`、`Promise.all` 等常规用法，连 `async/await` 也只是知其然，而不知其所以然。

但其实，promise 还有很多巧妙的高级用法，也将一些高级用法在 alova 请求策略库内部大量运用。

现在，我把这些毫无保留地在这边分享给大家，看完你应该再也不会被问倒了，**最后还有压轴题哦**。

**觉得对你有帮助还请点赞收藏评论哦！**

## 1\. promise 数组串行执行

例如你有一组接口需要串行执行，首先你可能会想到使用 await

```js
const requestAry = [() => api.request1(), () => api.request2(), () => api.request3()];
for (const requestItem of requestAry) {
  await promiseItem();
}
```

如果使用 promise 的写法，那么你可以使用 then 函数来串联多个 promise，从而实现串行执行。

```js
const requestAry = [() => api.request1(), () => api.request2(), () => api.request3()];
const finallyPromise = requestAry.reduce(
    (currentPromise, nextRequest) => currentPromise.then(() => nextRequest()),
    Promise.resolve(); // 创建一个初始promise，用于链接数组内的promise
);

```

## 2\. 在 new Promise 作用域外更改状态

假设你有多个页面的一些功能需要先收集用户的信息才能允许使用，在点击使用某功能前先弹出信息收集的弹框，你会怎么实现呢？

以下是不同水平的前端同学的实现思路：

**初级前端**：我写一个模态框，然后复制粘贴到其他页面，效率很杠杠的！

**中级前端**：你这不便于维护，我们要单独封装一下这个组件，在需要的页面引入使用！

**高级前端**：封什么装什么封！！！写在所有页面都能调用的地方，一个方法调用岂不更好？

看看高级前端怎么实现的，以 vue3 为例来看看下面的示例。

```js
<!-- App.vue -->
<template>

  <!-- 以下是模态框组件 -->
  <div class="modal" v-show="visible">
    <div>
      用户姓名：<input v-model="info.name" />
    </div>
    <!-- 其他信息 -->
    <button @click="handleCancel">取消</button>
    <button @click="handleConfirm">提交</button>
  </div>

  <!-- 页面组件 -->
</template>

<script setup>
import { provide } from 'vue';

const visible = ref(false);
const info = reactive({
  name: ''
});
let resolveFn, rejectFn;

// 将信息收集函数函数传到下面
provide('getInfoByModal', () => {
  visible.value = true;
  return new Promise((resolve, reject) => {
    // 将两个函数赋值给外部，突破promise作用域
    resolveFn = resolve;
    rejectFn = reject;
  });
})

const handleConfirm = info => {
  resolveFn && resolveFn(info);
};
const handleCancel = () => {
  rejectFn && rejectFn(new Error('用户已取消'));
};
</script>
```

接下来直接调用 `getInfoByModal` 即可使用模态框，轻松获取用户填写的数据。

```js
<template>
  <button @click="handleClick">填写信息</button>
</template>

<script setup>
import { inject } from 'vue';

const getInfoByModal = inject('getInfoByModal');
const handleClick = async () => {
  // 调用后将显示模态框，用户点击确认后会将promise改为fullfilled状态，从而拿到用户信息
  const info = await getInfoByModal();
  await api.submitInfo(info);
}
</script>
```

这也是很多 UI 组件库中对常用组件的一种封装方式。

## 3\. async/await 的另类用法

很多人只知道在 `async函数` 调用时用 `await` 接收返回值，但不知道 `async函数` 其实就是一个返回 promise 的函数，例如下面两个函数是等价的：

```js
const fn1 = async () => 1;
const fn2 = () => Promise.resolve(1);

fn1(); // 也返回一个值为1的promise对象
```

而 `await` 在大部分情况下在后面接 promise 对象，并等待它成为 fullfilled 状态，因此下面的 fn1 函数等待也是等价的：

```js
await fn1();

const promiseInst = fn1();
await promiseInst;
```

**然而，await 还有一个鲜为人知的秘密**，当后面跟的是非 promise 对象的值时，它会将这个值使用 promise 对象包装，因此 await 后的代码一定是异步执行的。如下示例：

```js
Promise.resolve().then(() => {
  console.log(1);
});
await 2;
console.log(2);
// 打印顺序位：1  2
```

等价于

```js
Promise.resolve().then(() => {
  console.log(1);
});
Promise.resolve().then(() => {
  console.log(2);
});
```

## 4\. promise 实现请求共享

当一个请求已发出但还未响应时，又发起了相同请求，就会造成了请求浪费，此时我们就可以将第一个请求的响应共享给第二个请求。

```js
request('GET', '/test-api').then(response1 => {
  // ...
});
request('GET', '/test-api').then(response2 => {
  // ...
});
```

上面两个请求其实只会真正发出一次，并且同时收到相同的响应值。

那么，请求共享会有哪几个使用场景呢？我认为有以下三个：

1. 当一个页面同时渲染多个内部自获取数据的组件时；
2. 提交按钮未被禁用，用户连续点击了多次提交按钮；
3. 在预加载数据的情况下，还未完成预加载就进入了预加载页面；

这也是 alova 的高级功能之一，实现请求共享需要用到 promise 的缓存功能，即一个 promise 对象可以通过多次 await 获取到数据，简单的实现思路如下：

```js
const pendingPromises = {};
function request(type, url, data) {
  // 使用请求信息作为唯一的请求key，缓存正在请求的promise对象
  // 相同key的请求将复用promise
  const requestKey = JSON.stringify([type, url, data]);
  if (pendingPromises[requestKey]) {
    return pendingPromises[requestKey];
  }
  const fetchPromise = fetch(url, {
    method: type,
    data: JSON.stringify(data)
  })
  .then(response => response.json())
  .finally(() => {
    delete pendingPromises[requestKey];
  });
  return pendingPromises[requestKey] = fetchPromise;
}
```

## 5\. 同时调用 resolve 和 reject 会怎么样？

大家都知道 promise 分别有 `pending/fullfilled/rejected` 三种状态，但例如下面的示例中，promise 最终是什么状态？

```js
const promise = new Promise((resolve, reject) => {
  resolve();
  reject();
});
```

正确答案是 `fullfilled` 状态，我们只需要记住，promise 一旦从 `pending` 状态转到另一种状态，就不可再更改了，因此示例中先被转到了 `fullfilled` 状态，再调用 `reject()` 也就不会再更改为 `rejected` 状态了。

## 6\. 彻底理清 then/catch/finally 返回值

先总结成一句话，就是**以上三个函数都会返回一个新的 promise 包装对象，被包装的值为被执行的回调函数的返回值，回调函数抛出错误则会包装一个 rejected 状态的 promise。**，好像不是很好理解，我们来看看例子：

```js
// then函数
Promise.resolve().then(() => 1); // 返回值为 new Promise(resolve => resolve(1))
Promise.resolve().then(() => Promise.resolve(2)); // 返回 new Promise(resolve => resolve(Promise.resolve(2)))
Promise.resolve().then(() => {
  throw new Error('abc')
}); // 返回 new Promise(resolve => resolve(Promise.reject(new Error('abc'))))
Promise.reject().then(() => 1, () = 2); // 返回值为 new Promise(resolve => resolve(2))

// catch函数
Promise.reject().catch(() => 3); // 返回值为 new Promise(resolve => resolve(3))
Promise.resolve().catch(() => 4); // 返回值为 new Promise(resolve => resolve(调用catch的promise对象))

// finally函数
// 以下返回值均为 new Promise(resolve => resolve(调用finally的promise对象))
Promise.resolve().finally(() => {});
Promise.reject().finally(() => {});
```

## 7\. then 函数的第二个回调和 catch 回调有什么不同？

promise 的 then 的第二个回调函数和 catch 在请求出错时都会被触发，咋一看没什么区别啊，但其实，前者不能捕获当前 then 第一个回调函数中抛出的错误，但 catch 可以。

```js
Promise.resolve().then(
  () => {
    throw new Error('来自成功回调的错误');
  },
  () => {
    // 不会被执行
  }
).catch(reason => {
  console.log(reason.message); // 将打印出"来自成功回调的错误"
});
```

其原理也正如于上一点所言，catch 函数是在 then 函数返回的 rejected 状态的 promise 上调用的，自然也就可以捕获到它的错误。

## 8\. （压轴）promise 实现 koa2 洋葱中间件模型

koa2 框架引入了洋葱模型，可以让你的请求像剥洋葱一样，一层层进入再反向一层层出来，从而实现对请求统一的前后置处理。

![](https://cdn.wallleap.cn/img/pic/illustration/202308041439427.png)

我们来看一个简单的 koa2 洋葱模型：

```js
const app = new Koa();
app.use(async (ctx, next) => {
  console.log('a-start');
  await next();
  console.log('a-end');
});
app.use(async (ctx, next) => {
  console.log('b-start');
  await next();
  console.log('b-end');
});

app.listen(3000);
```

以上的输出为 `a-start -> b-start -> b-end -> a-end`，这么神奇的输出顺序是如何做到的呢，某人不才，使用了 20 行左右的代码简单实现了一番，如有与 koa 雷同，纯属巧合。

接下来我们分析一番

> **注意：以下内容对新手不太友好，请斟酌观看。**

首先将中间件函数先保存起来，并在 listen 函数中接收到请求后就调用洋葱模型的执行。

```js
function action(koaInstance, ctx) {
  // ...
}

class Koa {
  middlewares = [];
  use(mid) {
    this.middlewares.push(mid);
  }
  listen(port) {
    // 伪代码模拟接收请求
    http.on('request', ctx => {
      action(this, ctx);
    });
  }
}
```

在接收到请求后，先从第一个中间件开始串行执行 next 前的前置逻辑。

```js
// 开始启动中间件调用
function action(koaInstance, ctx) {
  let nextMiddlewareIndex = 1; // 标识下一个执行的中间件索引
  
  // 定义next函数
  function next() {
    // 剥洋葱前，调用next则调用下一个中间件函数
    const nextMiddleware = middlewares[nextMiddlewareIndex];
    if (nextMiddleware) {
      nextMiddlewareIndex++;
      nextMiddleware(ctx, next);
    }
  }
  // 从第一个中间件函数开始执行，并将ctx和next函数传入
  middlewares[0](ctx, next);
}
```

处理 next 之后的后置逻辑

```js
function action(koaInstance, ctx) {
  let nextMiddlewareIndex = 1;
  function next() {
    const nextMiddleware = middlewares[nextMiddlewareIndex];
    if (nextMiddleware) {
      nextMiddlewareIndex++;
      // 这边也添加了return，让中间件函数的执行用promise从后到前串联执行（这个return建议反复理解）
      return Promise.resolve(nextMiddleware(ctx, next));
    } else {
      // 当最后一个中间件的前置逻辑执行完后，返回fullfilled的promise开始执行next后的后置逻辑
      return Promise.resolve();
    }
  }
  middlewares[0](ctx, next);
}
```

到此，一个简单的洋葱模型就实现了。

## 结尾

各位看官们，要是觉得对你有用请帮忙点个赞或收藏，然后还有哪里不理解的或更高级的用法吗？请在评论区说出你的想法！

> **相关信息**
> 我是 alova 御用推广人🤘🏻🤘🏼🤘，alova 是一个轻量级的请求策略库，它针对不同请求场景分别提供了具有针对性的请求策略，来提升应用可用性、流畅性，降低服务端压力，让应用如智者一般具备卓越的策略思维。
> [alova官网](https://link.juejin.cn/?target=https%3A%2F%2Falova.js.org%2F "https://alova.js.org/")
> [alova Github地址](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Falovajs%2Falova "https://github.com/alovajs/alova")

## 往期热文

- [是时候该换掉你的axios了](https://juejin.cn/post/7213923957824979000 "https://juejin.cn/post/7213923957824979000")
- [（深度）开源框架/库的伟大与罪恶](https://juejin.cn/post/7215608036394729532 "https://juejin.cn/post/7215608036394729532")
- [10年老程序员告诉你，高级程序员应具备怎样的职业素质](https://juejin.cn/post/7262146559831212090 "https://juejin.cn/post/7262146559831212090")
