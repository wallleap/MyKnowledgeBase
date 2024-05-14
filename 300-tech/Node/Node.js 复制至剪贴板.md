---
title: Node.js 复制至剪贴板
date: 2024-05-10 16:44
updated: 2024-05-10 16:45
---

拷贝是操作系统提供的一个能力，在不同操作系统下，指令集是不同的，如下：

```js
const { exec } = require('child_process')

// Windows
exec('clip').stdin.end('some text')

// Mac
exec('pbcopy').stdin.end('some text')

// Linux
exec('xclip').stdin.end('some text')
```

在社区上，也有提供了各平台通用的 NPM 包，比如：[copy-paste](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fxavi-%2Fnode-copy-paste)，使用也非常地简单。

```shell
$ npm i copy-paste
```

```js
const ncp = require('copy-paste')

ncp.copy('some text', function () {
  // complete...
})
```

> 需要注意的是，`copy()` 方法是异步的。
