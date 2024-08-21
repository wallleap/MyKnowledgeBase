---
title: 如何使用 Web Worker 为 JS 创造多线程环境？
date: 2023-08-04T05:20:00+08:00
updated: 2024-08-21T10:33:29+08:00
dg-publish: false
aliases:
  - 如何使用 Web Worker 为 JS 创造多线程环境？
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

## Web Worker 是什么

我们都知道 JS 是单线程的，所有任务在一个线程上，一次只能做一件事。虽然可以通过 AJAX、定时器等可以实现 " 并行 "，但还是没有改变 JS 单线程的本质，把一些复杂的运算放在页面上执行，还是会导致很卡，甚至卡死

而 HTML5 标准中的 Web Worker 为 JS 创造多线程环境，允许主线程创建 Worker 线程并给它分配任务，而且在主线程执行任务的时候，worker 线程可以同时在后台执行它的任务，互不干扰

这让我们可以将一些复杂运算、高频输入的响应处理、大文件分片上传等放在 worker 线程处理，最后再返回给主线程。很大程度上缓解了主线程 UI 渲染阻塞的问题，页面就会很流畅

**使用 Web Worker 有几个注意点：**

- 同源限制：worker 线程运行的文件必须和主线程的脚本是**同源**的
- 文件限制：worker 线程不能打开本机的文件系统 (file://)，**只能加载网络文件**
- 其他限制：worker 线程**不能操作 DOM、不能使用 document、window、parent 对象和 alert、confirm 方法**

但可以使用**location、navigator，不过只能只读不能改写**；还可以使用**XMLHttpRequest 发送 AJAX 请求**；还可以使用**缓存**

## 怎么使用 Web Worker 呢？

分别看看在主线程的页面和 worker 线程中的方法和用法

### 主线程中的方法和使用

创建 worker 进程，直接 new 就完事儿了，而且主线程内和 worker 线程内都 `可以创建多个 worker 线程`

可以传两个参数，第一个是网络脚本文件的链接，不能是本地路径，第二个是配置对象，不是必填

假设引入了一个网络文件 worker1.js

```js
// 主线程
const worker = new Worker('http://worker1.js')
```

然后使用 `worker.postMessage()` 方法向 worker 线程发送数据，参数可以是各种数据类型，包括二进制。注意！发送过去的数据是传值，也就是拷贝关系而不是传址，所以 worker 线程内部无论怎么修改，都不会影响到主线程的数据

```js
worker.postMessage('这是发给worker线程的消息')
```

然后再通过 `worker.onmessage` 监听并接收 worker 线程处理完返回的数据

```js
worker.onmessage = function(event){
    const data = event.data // 这是返回的数据
}
```

如果 worker 线程中内部发生错误，会触发主线程的 `onerror` 事件

```js
worker.onerror( event => {
    console.log( 'worker 线程内部执行报错了', event )
})
// 或者
worker.onerrer = function( event ){
    console.log( 'worker 线程内部执行报错了', event )
}
// 或者
worker.addEventListener('error', event => {
    console.log( 'worker 线程内部执行报错了', event )
})
```

最后是关掉 worker 线程，因为 worker 线程创建成功就会始终运行，不会主动关闭，所以我们不用的时候要主动关掉，避免浪费资源

```js
worker.terminate() // 这样 worker 线程就关闭了
```

### worker 线程中的方法和使用

worker 线程内部执行上下文跟主线程的执行上下文 window 不一样，是一个叫 `WorkerGlobalScope` 的东西，我们可以用它或者 `self` 来访问全局对象

如果主线程创建 worker 线程的时候有传第二个参数，在 worker1.js 里面，也就是在 worker 线程里可以直接这样获取，比如区分多个 worker 线程的时候

```js
// 主线程
const worker = new Worker('http://worker1.js', { name: 'worker1' })

// worker1.js
self.name // worker1  
```

worker 线程里面，需要监听并接收主线程发送过来的数据。

```js
self.addEventListener('message', event => {
    const data = event.data // 接收到的数据 
    ...
    self.postMessage( '这是处理好的数据' ) // 将处理好的数据发送给主线程
}, false)
// 或者 不要 self 也可以
addEventListener('message', event => {
    const data = event.data // 接收到的数据 
    ...
    self.postMessage( '这是处理好的数据' ) // 将处理好的数据发送给主线程
}, false)
```

如果 worker 线程内部需要加载其他脚本，只能用 `importScripts()` 方法，路径还是只能网络文件，不能使用本地的

```js
importScripts('http://worker2.js', 'http://worker3.js') // 可以引入多个
```

worker 线程内部也可以使用 onerror 监听错误，和主线程使用方式一样。

worker 线程内部也可以关闭 worker 线程，方法和主线程不一样

```js
self.close() // 这样就从内部关闭了
```

### worker 线程能不能直接写在主线程的页面里

能！

既然它要接收一个网络地址 URL，就给它创建一个 URL

首先**给 script 标签起一个浏览器不认识的 type，然后将这个标签里的内容转成二进制对象，然后为这个对象生成 URL，再用这个 URL 创建 worker 线程**，如下

```html
<!DOCTYPE html>
    <body>
        <div>这是页面</div>
        <script id="worker" type="xxxxx">
            // worker 线程
            addEventListener('message', event => {
                console.log(event.data) // 收到了吗
                postMessage('收到了') // 发送给主线程
            })
        </script>
        <script type="text/javascript">
            // 主线程
            const blob = new Blob([ document.querySelector('#worker').textContent ])
            const url = window.URL.createObjectURL(blob)
            const worker = new Worker(url)
            worker.postMessage('收到了吗')
            worker.onmessage = event => {
                console.log(event.data) // 收到了
            }
        </script>
    </body>
</html>
```

### worker 线程轮询

```js
function createWorker(fn){
    const blob = new Blob([fn.toString()])
    const url = window.URL.createObjectURL(blob)
    const worker = new Worker(url)
    return worker
}

const webWorker = createWorker(function(){
    let cache;
    
    function compare(new, old){ ... }
    
    setInterval(()=>{
        fetch('/api/xxx').then(res=>{
            let data = res.data
            if(!compare(data, cache)){
                cache = data
                self.postMessage(data)
            }
        })
    },1000)
})

```

## 结语

点赞支持、手留余香、与有荣焉

感谢你能看到这里，加油哦！

## 参考

[\# Web Worker 使用教程](https://link.juejin.cn/?target=http%3A%2F%2Fwww.ruanyifeng.com%2Fblog%2F2018%2F07%2Fweb-worker.html "http://www.ruanyifeng.com/blog/2018/07/web-worker.html")
