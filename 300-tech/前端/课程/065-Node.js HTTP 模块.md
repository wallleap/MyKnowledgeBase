---
title: 065-Node.js HTTP 模块
date: 2024-05-06T09:48:00+08:00
updated: 2024-08-21T10:32:39+08:00
dg-publish: false
---

## HTTP 协议

## 一些工具

node-dev

- 当文件更新时自动重启的 node
- 避免每次改完代码都要重新运行的麻烦
- 不宜在生产环境使用

ts-node

- 让 node 支持直接运行 TypeScript 代码
- 不宜在生产环境使用

**ts-node-dev**

- 结合了上面两个工具功能
- 可以用 TypeScript 开发 Node.js 程序，且能自动重启 Node

curl 构建请求

- GET 请求：`curl -v url`
- POST 请求：`curl -v -d "name=Tom" url`
- 设置请求头：`-H 'Content-Type:application/json'`
- 设置动词：`-X PUT`
- JSON 请求：`curl -d '{"name": "Tom"}' -H 'Content-Type:application/json' url`

## 创建项目

```sh
mkdir node-server
cd node-server
yarn init -y
```

新建 `index.ts` 文件

```ts
const a: number = 1
const b: number = 2
console.log(a + b)
```

运行 `ts-node-dev index.ts`

安装 node 声明文件

```sh
yarn add -dev @types/node
```

编写 `index.ts`

```ts
// 1. 引入 http 模块
import * as http from "http";

// 2. 用 http 创建 server
const server = http.createServer();

// 3. 监听 server 的 request 事件
server.on('request', (request, response) => {
  console.log('有人请求了')
  response.end('hi')
})

// 4. 设置监听端口
server.listen(9999)

// 运行 ts-node-dev index.ts 之后命令行 curl -v http://localhost:9999 发请求
```

## server

`const server = http.createServer()` http.Server 实例，继承自 net.Server（error 事件和 address() 方法）

- 有一个 `request` 事件，在每次有请求时都会触发
- `server.listen()` 启动 HTTP 服务器监听连接

request 事件中 request 对象和 response 对象

`request.constructor` http.IncomingMessage 类

- 拥有 headers、method、url 等属性
- 从 stream.Readable 类继承了 data/end/error 事件
- 之所以不能直接拿到请求的消息体，是因为跟 TCP 有关

`response.constructor` http.ServerResponse 类

- 拥有 getHeader/setHeader/end/write 等方法
- 拥有 statusCode 属性，可读写
- 继承了 Stream，目前用不上

## 用 Node.js 获取请求内容

### GET 请求

- request.method 获取请求动词
- request.url 获取请求路径（含查询参数）
- request.header 获取请求头

get 消息一般没有消息体/请求体

```ts
server.on('request', (request, response) => {
  console.log('request.method', request.method)
  console.log('request.url', request.url)
  console.log('request.headers', request.headers)
  // response.setHeader('X-Name', 'Tom')
  // response.write('hello')
  response.end('hi') // response.statusCode = 404; response.end()
})
```

发送请求

```sh
$ curl -v http://127.0.0.1:9999
*   Trying 127.0.0.1:9999...
* Connected to 127.0.0.1 (127.0.0.1) port 9999 (#0)
> GET / HTTP/1.1
> Host: 127.0.0.1:9999
> User-Agent: curl/8.1.2
> Accept: */*
>
< HTTP/1.1 200 OK
< Date: Mon, 06 May 2024 03:33:00 GMT
< Connection: keep-alive
< Keep-Alive: timeout=5
< Content-Length: 2
<
* Connection #0 to host 127.0.0.1 left intact
hi
```

打印的内容

```
request.method GET
request.url /
request.headers { host: '127.0.0.1:9999', 'user-agent': 'curl/8.1.2', accept: '*/*' }
```

### POST 请求

- request.on('data', fn) 获取消息体
- request.on('end', fn) 拼接消息体

```ts
server.on('request', (request, response) => {
  console.log('request.method', request.method)
  console.log('request.url', request.url)
  console.log('request.headers', request.headers)
  
  const arr: Buffer[] = []
  request.on('data', (chunk: Buffer) => {
    arr.push(chunk)
  })
  request.on('end', () => {
    const body = Buffer.concat(arr).toString()
    console.log('body', body)
    response.end('hi')
  })
})
```

发送一个 POST 请求

```sh
$ curl -v -d "name=Tom"  http://127.0.0.1:9999
*   Trying 127.0.0.1:9999...
* Connected to 127.0.0.1 (127.0.0.1) port 9999 (#0)
> POST / HTTP/1.1
> Host: 127.0.0.1:9999
> User-Agent: curl/8.1.2
> Accept: */*
> Content-Length: 8
> Content-Type: application/x-www-form-urlencoded
>
< HTTP/1.1 200 OK
< Date: Mon, 06 May 2024 03:43:53 GMT
< Connection: keep-alive
< Keep-Alive: timeout=5
< Content-Length: 2
<
* Connection #0 to host 127.0.0.1 left intact
hi
```

打印内容

```
request.method POST
request.url /
request.headers {
  host: '127.0.0.1:9999',
  'user-agent': 'curl/8.1.2',
  accept: '*/*',
  'content-length': '8',
  'content-type': 'application/x-www-form-urlencoded'
}
body name=Tom
```

## 实现自己的静态服务器

### 根据 url 返回不同的文件

新建 `public` 目录，里面新建 `index.html`、`style.css`、`main.js` 三个文件

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport"
        content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
<h1>首页</h1>
<script src="main.js"></script>
</body>
</html>
```

style.css

```css
h1 {
  color: red;
}
```

main.js

```js
const h1El = document.querySelector('h1')

h1El.onclick = () => alert('点击了标题')
```

在 `index.ts` 中根据 request.url 的不同返回不同的内容

```ts
import * as http from "http";
import * as fs from "fs";

const p = require('path')

const server = http.createServer()
const publicDir = p.resolve(__dirname, 'public')

server.on('request', (request, response) => {
  const {method, url, headers} = request
  switch (url) {
    case '/':
    case '/index.html':
      fs.readFile(p.resolve(publicDir, 'index.html'), (err, data) => {
        if (err) throw err
        response.end(data.toString())
      })
      break
    case '/style.css':
      response.setHeader('Content-Type', 'text/css; charset=utf-8')
      fs.readFile(p.resolve(publicDir, 'style.css'), (err, data) => {
        if (err) throw err
        response.end(data.toString())
      })
      break
    case '/main.js':
      response.setHeader('Content-Type', 'text/javascript; charset=utf-8')
      fs.readFile(p.resolve(publicDir, 'main.js'), (err, data) => {
        if (err) throw err
        response.end(data.toString())
      })
      break
  }
})
server.listen(9999)
```

### 处理查询参数数

引入 url 模块，使用 `url.parse`

```ts
import * as url from "url";

const {method, url: path, headers} = request // 重命名 url 为 path
const {pathname, search} = url.parse(path ?? '')  

switch (pathname) {}
```

### 匹配任意文件并处理不存在的文件

```ts
// 1. 引入 http 模块
import * as http from "http";
import * as fs from "fs";
import * as url from "url";

const p = require('path')

const PORT = 9999

// 2. 用 http 创建 server
const server = http.createServer()
const publicDir = p.resolve(__dirname, 'public')

// 3. 监听 server 的 request 事件
server.on('request', (request, response) => {
  const {method, url: path, headers} = request // 重命名 url 为 path
  const curUrl = new URL(`http://localhost:${PORT}${path}`)
  // @ts-ignore
  const {pathname, search} = url.urlToHttpOptions(curUrl)
  let filename = pathname.substr(1)
  if(filename === '') filename = 'index.html'
  fs.readFile(p.resolve(publicDir, filename), (err, data) => {
    if (err) {
      if (err.errno === -4058 || err.errno === -4068) {
        response.statusCode = 404
        response.setHeader('Content-Type', 'text/html; charset=utf-8')
        response.end('你访问的文件不存在')
      } else {
        response.statusCode = 500
        response.setHeader('Content-Type', 'text/html; charset=utf-8')
        response.end('服务器繁忙，请稍候重试')
      }
    } else {
      response.end(data)
    }
  })
})

// 4. 设置监听端口
server.listen(PORT)
```

### 处理非 GET 请求

```ts
if (method !== 'GET') {
  response.statusCode = 405
  response.end()
  return
}
```

### 添加缓存选项

```ts
let cacheAge = 3600 * 24 * 365
response.setHeader('Cache-Control', `public, max-age=${cacheAge}`)
```

### 响应内容启用 gzip

### 业界范例

http-server、node-static

## 制作一个命令行翻译工具

```sh
mkdir fanyi
cd fanyi
yarn init -y
```

安装依赖

```sh
yarn add --dev @types/node
yarn add commander
```

创建 `src/cli.ts`

```ts
import fs from 'fs'

const {Command} = require('commander')
const pkg = require('../package.json')

const program = new Command()

program
  .version(pkg.version)
  .name('fy')
  .usage('<english>')

program.parse(process.argv)
```

修改内容

```ts
import fs from 'fs'
import {translate} from "./main"

const {Command} = require('commander')
const pkg = require('../package.json')

const program = new Command()

program
  .version(pkg.version)
  .name('fy')
  .usage('<English>')
  .argument('<English>', 'English to translate')
  .action((English: string) => {
    console.log('English:', English);
    translate(English)
  })

program.parse(process.argv)
```

`translate` 是从 `main.ts` 中导入的

```ts
export const translate = (word: string) => {
  console.log(word)
}
```

### 找一个翻译接口

例如：

`https://clients5.google.com/translate_a/t?client=dict-chrome-ex&sl=auto&tl=zh-cn&q=English`

### 进行请求

```ts
import * as https from "https"
import * as querystring from "querystring"

export const translate = (word: string) => {
  let tl: 'zh-cn' | 'en' = 'en'
  if (/[a-zA-Z]/.test(word[0])) {
    tl = 'zh-cn'
  }
  // https://clients5.google.com/translate_a/t
  // ?client=dict-chrome-ex&sl=auto&tl=zh-cn&q=word
  const query: string = querystring.stringify({
    client: "dict-chrome-ex",
    sl: "auto",
    tl,
    q: word
  })

  const options = {
    hostname: 'clients5.google.com',
    port: 443,
    path: '/translate_a/t?' + query,
    method: 'GET',
  }

  const request = https.request(options, (response) => {
    let chunks: Buffer[] = []
    response.on('data', (chunk: Buffer) => {
      chunks.push(chunk)
    })
    response.on('end', () => {
      const string = Buffer.concat(chunks).toString()
      const object = JSON.parse(string)
      console.log(object[0][0])
    })
  })
  request.on('error', (e) => {
    console.error(e)
  })
  request.end()
}
```

### 发布到 npm

```sh
yarn global add typescript
tsc -V
tsc --init
```

修改 `tsconfig.json` 中这项

```json
{
  "outDir": "dist/"
}
```

执行

```sh
tsc
```

修改 `package.json`

```json
{
  "name": "fanyi-node-cli",
  "version": "0.0.1",
  "description": "A cli tool to translate word",
  "bin": {
    "fy": "dist/cli.js"
  },
  "files": [
    "dist/*.js"
  ],
  "main": "index.js",
  "license": "MIT",
  "dependencies": {
    "commander": "^12.0.0"
  },
  "devDependencies": {
    "@types/node": "^20.12.8"
  }
}
```

参考之前的添加 bangshell 和可执行权限

运行命令上传

```sh
npm adduser
npm publish
pnpm add -g fanyi-node-cli@0.0.1
```
