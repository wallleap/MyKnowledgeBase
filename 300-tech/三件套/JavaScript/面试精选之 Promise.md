---
title: 面试精选之 Promise
date: 2023-08-04 14:44
updated: 2023-08-04 14:54
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - 面试精选之 Promise
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

> 前端面试过程中，基本都会问到 Promise，如果你足够幸运，面试官问的比较浅，仅仅问 Promise 的使用方式，那么恭喜你。事实上，大多数人并没有那么幸运。所以，我们要准备好九浅一深的知识。

不知道读者有没有想过，为什么那么多面试官都喜欢问 Promise？可以思考一下哦~

## 常见 Promise 面试题

我们看一些 Promise 的常见面试问法，由浅至深。

- 1、了解 Promise 吗？
- 2、Promise 解决的痛点是什么？
- 3、Promise 解决的痛点还有其他方法可以解决吗？如果有，请列举。
- 4、Promise 如何使用？
- 5、Promise 常用的方法有哪些？它们的作用是什么？
- 6、Promise 在事件循环中的执行过程是怎样的？
- 7、Promise 的业界实现都有哪些？
- 8、能不能手写一个 Promise 的 polyfill。

这些问题，如果你都能 hold 住，那么面试官基本认可你了。带着上面这些问题，我们往下看。

## Promise 出现的原因

在 Promise 出现以前，我们处理一个异步网络请求，大概是这样：

```js
// 请求 代表 一个异步网络调用。
// 请求结果 代表网络请求的响应。
请求1(function(请求结果1){
    处理请求结果1
})
```

看起来还不错。

但是，需求变化了，我们需要根据第一个网络请求的结果，再去执行第二个网络请求，代码大概如下：

```js
请求1(function(请求结果1){
    请求2(function(请求结果2){
        处理请求结果2
    })
})
```

看起来也不复杂。

但是。。需求是永无止境的，于是乎出现了如下的代码：

```js
请求1(function(请求结果1){
    请求2(function(请求结果2){
        请求3(function(请求结果3){
            请求4(function(请求结果4){
                请求5(function(请求结果5){
                    请求6(function(请求结果3){
                        ...
                    })
                })
            })
        })
    })
})
```

这回傻眼了。。。 臭名昭著的 **回调地狱** 现身了。

更糟糕的是，我们基本上还要对每次请求的结果进行一些处理，代码会更加臃肿，在一个团队中，代码 review 以及后续的维护将会是一个很痛苦的过程。

回调地狱带来的负面作用有以下几点：

- 代码臃肿。
- 可读性差。
- 耦合度过高，可维护性差。
- 代码复用性差。
- 容易滋生 bug。
- 只能在回调里处理异常。

出现了问题，自然就会有人去想办法。这时，就有人思考了，能不能用一种更加友好的代码组织方式，解决异步嵌套的问题。

```js
let 请求结果1 = 请求1();
let 请求结果2 = 请求2(请求结果1); 
let 请求结果3 = 请求3(请求结果2); 
let 请求结果4 = 请求2(请求结果3); 
let 请求结果5 = 请求3(请求结果4); 
```

类似上面这种同步的写法。 于是 Promise 规范诞生了，并且在业界有了很多实现来解决回调地狱的痛点。比如业界著名的 **Q** 和 **bluebird**，**bluebird** 甚至号称运行最快的类库。

> 看官们看到这里，对于上面的问题 2 和问题 7 ，心中是否有了答案呢。^\_^

## 什么是 Promise

Promise 是异步编程的一种解决方案，比传统的异步解决方案【回调函数】和【事件】更合理、更强大。现已被 ES6 纳入进规范中。

### 代码书写比较

还是使用上面的网络请求例子，我们看下 Promise 的常规写法：

```js
new Promise(请求1)
    .then(请求2(请求结果1))
    .then(请求3(请求结果2))
    .then(请求4(请求结果3))
    .then(请求5(请求结果4))
    .catch(处理异常(异常信息))
```

比较一下这种写法和上面的回调式的写法。我们不难发现，Promise 的写法更为直观，并且能够在外层捕获异步函数的异常信息。

### API

Promise 的常用 API 如下：

- Promise.resolve(value)

> 类方法，该方法返回一个以 value 值解析后的 Promise 对象 1、如果这个值是个 thenable（即带有 then 方法），返回的 Promise 对象会“跟随”这个 thenable 的对象，采用它的最终状态（指 resolved/rejected/pending/settled）
> 2、如果传入的 value 本身就是 Promise 对象，则该对象作为 Promise.resolve 方法的返回值返回。
> 3、其他情况以该值为成功状态返回一个 Promise 对象。

上面是 resolve 方法的解释，传入不同类型的 value 值，返回结果也有区别。这个 API 比较重要，建议大家通过练习一些小例子，并且配合上面的解释来熟悉它。如下几个小例子：

```js
//如果传入的 value 本身就是 Promise 对象，则该对象作为 Promise.resolve 方法的返回值返回。  
function fn(resolve){
    setTimeout(function(){
        resolve(123);
    },3000);
}
let p0 = new Promise(fn);
let p1 = Promise.resolve(p0);
// 返回为true，返回的 Promise 即是 入参的 Promise 对象。
console.log(p0 === p1);
```

传入 thenable 对象，返回 Promise 对象跟随 thenable 对象的最终状态。

> ES6 Promises 里提到了 Thenable 这个概念，简单来说它就是一个非常类似 Promise 的东西。最简单的例子就是 jQuery.ajax，它的返回值就是 thenable 对象。但是要谨记，并不是只要实现了 then 方法就一定能作为 Promise 对象来使用。

```js
//如果传入的 value 本身就是 thenable 对象，返回的 promise 对象会跟随 thenable 对象的状态。
let promise = Promise.resolve($.ajax('/test/test.json'));// => promise对象
promise.then(function(value){
   console.log(value);
});
```

返回一个状态已变成 resolved 的 Promise 对象。

```js
let p1 = Promise.resolve(123); 
//打印p1 可以看到p1是一个状态置为resolved的Promise对象
console.log(p1)
```

- Promise.reject

> 类方法，且与 resolve 唯一的不同是，返回的 promise 对象的状态为 rejected。

- Promise.prototype.then

> 实例方法，为 Promise 注册回调函数，函数形式：fn(vlaue){}，value 是上一个任务的返回结果，then 中的函数一定要 return 一个结果或者一个新的 Promise 对象，才可以让之后的 then 回调接收。

- Promise.prototype.catch

> 实例方法，捕获异常，函数形式：fn(err){}, err 是 catch 注册 之前的回调抛出的异常信息。

- Promise.race

> 类方法，多个 Promise 任务同时执行，返回最先执行结束的 Promise 任务的结果，不管这个 Promise 结果是成功还是失败。 。

- Promise.all

> 类方法，多个 Promise 任务同时执行。
> 如果全部成功执行，则以数组的方式返回所有 Promise 任务的执行结果。 如果有一个 Promise 任务 rejected，则只返回 rejected 任务的结果。

- ...

以上几种便是 Promise 的常用 API，掌握了这些，我们便可以熟练使用 Promise 了。

> 一定要多练习，熟练掌握，否则一知半解的理解在面试时捉襟见肘。

### 如何理解 Promise

为了便于理解 Promise，大家除了要多加练习以外，最好的方式是能够将 Promise 的机制与现实生活中的例子联系起来，这样才能真正得到消化。

我们可以把 Promise 比作一个保姆，家里的一连串的事情，你只需要吩咐给他，他就能帮你做，你就可以去做其他事情了。

比如，作为一家之主的我，某一天要出门办事，但是我还要买菜做饭送到老婆单位（请理解我在家里的地位。）

出门办的事情很重要，买菜做饭也重要。。但我自己只能做一件事。

这时我就可以把买菜做饭的事情交给保姆，我会告诉她：

- 你先去超市买菜。
- 用超市买回来的菜做饭。
- 将做好的饭菜送到老婆单位。
- 送到单位后打电话告诉我。

我们知道，上面三步都是需要消耗时间的，我们可以理解为三个异步任务。利用 Promise 的写法来书写这个操作：

```js
function 买菜(resolve，reject) {
    setTimeout(function(){
        resolve(['西红柿'、'鸡蛋'、'油菜']);
    },3000)
}
function 做饭(resolve, reject){
    setTimeout(function(){
        //对做好的饭进行下一步处理。
        resolve ({
            主食: '米饭',
            菜: ['西红柿炒鸡蛋'、'清炒油菜']
        })
    },3000) 
}
function 送饭(resolve，reject){
    //对送饭的结果进行下一步处理
    resolve('老婆的么么哒');
}
function 电话通知我(){
    //电话通知我后的下一步处理
    给保姆加100块钱奖金;
}
```

好了，现在我整理好了四个任务，这时我需要告诉保姆，让他按照这个任务列表去做。这个过程是必不可少的，因为如果不告诉保姆，保姆不知道需要做这些事情。（我这个保姆比较懒）

```js
// 告诉保姆帮我做几件连贯的事情，先去超市买菜
new Promise(买菜)
  //用买好的菜做饭
  .then((买好的菜)=>{
      return new Promise(做饭);
  })
  //把做好的饭送到老婆公司
  .then((做好的饭)=>{
      return new Promise(送饭);
  })
  //送完饭后打电话通知我
  .then((送饭结果)=>{
      电话通知我();
  })
```

至此，我通知了保姆要做这些事情，然后我就可以放心地去办我的事情。

> 请一定要谨记：如果我们的后续任务是异步任务的话，必须 return 一个 新的 promise 对象。
> 如果后续任务是同步任务，只需 return 一个结果即可。
> 我们上面举的例子，除了**电话通知我**是一个同步任务，其余的都是异步任务，异步任务 return 的是 promise 对象。

除此之外，一定谨记，一个 Promise 对象有三个状态，并且状态一旦改变，便不能再被更改为其他状态。

- pending，异步任务正在进行。
- resolved (也可以叫 fulfilled)，异步任务执行成功。
- rejected，异步任务执行失败。

### Promise 的使用总结

Promise 这么多概念，初学者很难一下子消化掉，那么我们可以采取强制记忆法，强迫自己去记住使用过程。

首先初始化一个 Promise 对象，可以通过两种方式创建， 这两种方式都会返回一个 Promise 对象。

1、new Promise(fn)

2、Promise.resolve(fn)

然后调用上一步返回的 promise 对象的 then 方法，注册回调函数。

then 中的回调函数可以有一个参数，也可以不带参数。如果 then 中的回调函数依赖上一步的返回结果，那么要带上参数。比如

```js
new Promise(fn)
  .then(fn1(value）{
    //处理value
  })
```

最后注册 catch 异常处理函数，处理前面回调中可能抛出的异常。

通常按照这三个步骤，你就能够应对绝大部分的异步处理场景。用熟之后，再去研究 Promise 各个函数更深层次的原理以及使用方式即可。

看到这里之后，我们便能回答上面的问题 4 和问题 5 了。

## Promsie 与事件循环

Promise 在初始化时，传入的函数是同步执行的，然后注册 then 回调。注册完之后，继续往下执行同步代码，在这之前，then 中回调不会执行。同步代码块执行完毕后，才会在事件循环中检测是否有可用的 promise 回调，如果有，那么执行，如果没有，继续下一个事件循环。

关于 Promise 在事件循环中还有一个 微任务的概念（microtask），感兴趣的话可以看我这篇关于 nodejs 时间循环的文章 [剖析nodejs的事件循环](https://juejin.cn/post/6844903621444763662 "https://juejin.cn/post/6844903621444763662")，虽然和浏览器端有些不同，但是 Promise 微任务的执行时机相差不大。

## Promise 的升级

ES6 出现了 generator 以及 async/await 语法，使异步处理更加接近同步代码写法，可读性更好，同时异常捕获和同步代码的书写趋于一致。上面的列子可以写成这样：

```js
(async ()=>{
    let 蔬菜 = await 买菜();
    let 饭菜 = await 做饭(蔬菜);
    let 送饭结果 = await 送饭(饭菜);
    let 通知结果 = await 通知我(送饭结果);
})();
```

是不是更清晰了有没有。需要记住的是，async/await 也是基于 Promise 实现的，所以，我们仍然有必要深入理解 Promise 的用法。

## 结语

相信各位看官看到这里也累了，同时限于篇幅，本文不再对手动实现 Promise 进行讲解了，留待下一篇文章~

上面的内容希望读者多做练习，吃透 Promise 的使用与原理，让面试更加从容。
