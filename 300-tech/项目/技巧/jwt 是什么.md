---
title: jwt 是什么
date: 2024-09-25T10:52:00+08:00
updated: 2024-10-14T05:09:26+08:00
dg-publish: false
---

JWT Json Web **Token**

一个字符串

## 没有 jwt

登录成功之后服务器会发送身份信息让浏览器保存，保存位置可以是 Cookie、LocalStorage 等能存储的地方，浏览器要请求的时候把信息读取出来，通过 HTTP 协议发送给服务器，服务器认证身份信息之后就表明是已经登录的

由于身份信息存储在浏览器，所以可以随便更改它（数据被**篡改**）

直接在浏览器或通过其他工具根据服务器规则添加这个信息（数据被**伪造**）

↓

服务器无法信任这个信息

↓

对信息进行签名 ← 算法 —— 信息 + 密钥

## jwt 雏形

算法有很多，常见的是 HS256，各种语言都有 crypto 这个库，提供了加密算法

例如：

```js
const crypto = require('crypto')

function sign(info, key) {
  const hmac = crypto.createHmac('sha256', key)
  hmac.update(info)
  return hmac.digest('hex')
}
const KEY = '1234556'
sign('username', KEY)  // 一个字符串
```

现在登录之后服务器根据密钥对信息进行加密得到签名，把信息和签名发送到浏览器，下次浏览器请求就把信息和签名携带上，如果有篡改的话服务器就能知道（把信息重新进行加密和签名进行对比）

只要密钥不泄露就是安全的

## jwt

给一个标准，整个信息如何组织

一个非常长的字符串，三部分组成，以 `.` 分割

- header ← BASE64 编码 - JSON 字符串 `{ "alg": "HS256", "type": "JWT" }`，固定写法，加密算法和类型
- payload ← BASE64 编码 - 传给浏览器的信息，例如身份信息/用户 id `{ "id": xxxx }`
- signature ← BASE64 - 签名 ← 密钥 - `header.payload`

```js
function jwt(info, key) {
  const header = {
    type: 'JWT',
    alg: 'HS256'
  }
  const headerStr = Buffer.from(JSON.stringify(header)).toString('base64')
  const payloadStr = Buffer.from(JSON.stringify(info)).toString('base64)
  const signStr = sign(headerStr + '.' + payloadStr, key)
  return headerStr + '.' + payloadStr + '.' + signStr
}
```

前两个部分是**明文**，最后一部分签名是用来验证是否被篡改的

payload 是明文有一个好处，浏览器端需要获取用户信息，可以不用发请求，直接从里面解码获取用户名等信息（**不要把重要信息，例如密码放到这里面**）；也可以单独再对这部分**进行加密再组成** jwt

## 浏览器端

登录后会得到 token，需要把这个 token 存在一个地方，例如 localStorage 中，之后每次请求都携带上这个 token（请求头中 `Authorazation: Bearer token`

## 双 Token

一般用在单点登录

在中间有个认证服务器
