---
title: Worker 调研
date: 2024-08-22T02:20:16+08:00
updated: 2024-08-22T02:20:22+08:00
dg-publish: false
---

### what（worker是什么？）

> **背景**

JavaScript 是一门单线程的语言，也就是说所有的任务只能在同一个线程上执行，同一个时间内只能执行一个任务，这就造成了一个问题，当线程中任务过多时，就会导致任务的阻塞。而随着现在无论是硬件的发展（多核 CPU），还是前端系统（需求复杂，功能众多）的发展，这都会造成众多不便，而 Worker 就是为了解决这些问题而来的。

> **概述**

Worker 可以创建一条独立于主线程之外的另一线程。通过 Worker 创建的这个线程运行在后台，主线程可以将一些任务分配给这一独立线程去执行，两者互不干扰，当在 Worker 线程中的任务执行完毕后可以通过 message 来向主线程发送执行结果。

![](https://cdn.wallleap.cn/img/pic/illustration/202408221420355.png?imageSlim)

因为 Worker 是独立运行的线程，所以它在执行时全局上下文是与平常使用的全局 window 对象不同的，根据 Worker 全局上下文的不同又可以分为两种：

[Worker](https://developer.mozilla.org/zh-CN/docs/Web/API/Worker)：专用 Worker，仅供当前执行脚本的页面使用，当这个页面关闭时，当前 worker 也会随之关闭，对应的上下文为 DedicatedWorkerGlobalScope

[sharedWorker](https://developer.mozilla.org/zh-CN/docs/Web/API/SharedWorker)：共享 Worker，可以供多个页面共同使用，当所有关联页面关闭时，当前 worker 才会被关闭。对应的上下文为 SharedWorkerGlobalScope

> **注意点**

1. 同源限制
分配给 Worker 线程运行的脚本文件，必须与主线程的脚本文件同源。

2. API 访问限制
上面已经提到了 Worker 线程所在的全局对象，与主线程不一样，所以无法读取主线程上的一些对象和函数。这里来简单列出几种常见的

- 不可用：
	- window
	- document
	- alert()
	- confirm()
- 可用：
	- navigator
	- location
	- XMLHttpRequest
	- fetch

[更详细的API访问权限](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Workers_API/Functions_and_classes_available_to_workers#apis_available_in_workers)

1. 文件限制
worker 无法访问本地的资源，所传入的 URL 必须要是网络资源。所以对于本地的脚本可以将其转换成 Blob URL 使用

##  why（为什么要使用？）

Worker 是可以分担主线程上一些计算量大，耗费时间长的任务, 如大数据量的组织、操作、统计、频繁请求等

## how（怎么用？）

1. 创建一个 index.html 文件

```html
  <script type="module">
    // 引入worker脚本
    import workerScript from './worker.js'
    // 创建worker线程，参数为脚本的url
    // 这里需要注意一下，这里的url是Blob URL，具体原因在上文已经说过
    const worker = new Worker(workerScript);
    // 主线程和子线程进行通信
    worker.postMessage({
      msg: '主线程的msg',
      url: 'http://'
    })
    // 获取子线程发送来的消息
    worker.onmessage = (msg) => {
      const { data } = msg
      document.body.innerHTML = data
    }
    // 监听worker的错误事件
    worker.onmessageerror = (error) => console.log(error)
  </script>
```

1. 创建一个 worker.js 文件

```js
// 需要在worker中执行的脚本
const workerScript = () => {
  onmessage = (msg) => {
    if (msg.data) {
      // 接收到主线程的消息之后，与主线程进行通信
      postMessage(`Worker's message comes from ${msg.data}`);
    }
    if(msg.url) {
      // 发送网络请求
      const response = fetch('https://www.fastmock.site/mock/3410de68d1e03849d328e0b0651c4f1f/api/api/services/app/TransactionBill/GetTransactionBillInfo', {
        method: "post",
        body: `yes = 1`,
      }).then((res) => res.json);
      postMessage(`network is ${response}`);
    }
  };
};

// 将上述执行脚本转换为字符串
let code = workerScript.toString();
// 对字符串进行分割，取到onmessage部分的函数声明
code = code.substring(code.indexOf("{") + 1, code.lastIndexOf("}"));
// 将字符串转换为Blob URL
const blob = new Blob([code], { type: "application/javascript" });
const blobUrl = URL.createObjectURL(blob);
// 导出生成的Blob URL
export default blobUrl;
```

**newwork 里面如果请求前面带了一个小齿轮, 就证明这个请求是在 worker 线程中进行请求的**
