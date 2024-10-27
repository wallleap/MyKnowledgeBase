---
title: jwt 是什么？怎么实现？
date: 2024-09-25T10:52:00+08:00
updated: 2024-10-25T09:33:55+08:00
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

## jwt 生成

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

登录后会得到 token，需要把这个 token 存在一个地方，例如 localStorage 中，之后每次请求都携带上这个 token（请求头中 `Authorazation: Bearer token`）

下面有具体实现代码

## 单点登录

一般用在单点登录模式上，多个子系统共享同一个用户体系

例如 SESSION + COOKIE 模式的

- 用户点击登录
- 在认证中心中创建一个键值对（例如 sid: 身份信息 键是一个唯一的 id，值为身份信息），**存储到 Session 中**。Session 中有这个键值对就是登录的，没有就是没登录或登录过期了
- 认证中心会返回一个响应，携带 set-Cookie 头，这样会把这个 sid 存储到本地的 Cookie 中
- 用户再去访问其他子系统的时候，都会携带这个 cookie，子系统会向认证中心查询，查到了就是有效的，可能会返回该用户的一些基本信息
- 子系统直到它登录了就能安心返回一些受保护的资源了

![](https://cdn.wallleap.cn/img/pic/illustration/202410241705867.png?imageSlim)

好处：只要把 Session 表中的键值对修改或删除就能让用户退出登录；缺点：需要内存/存储去保存 Session 表

TOKEN 模式

- 点击登录
- 在认证中心中生成 token，并不保存任何内容，将这个 token 返回给客户端，**客户端去存储**它（内存、localStorage 等能存储的地方）
- 每次请求子系统的时候都携带这个 token，子系统可以直接验证这个 token 是否有效和过期（服务器存储的是同一个密钥，能通过这个密钥进行加解密），有效就代表登录了
- 登录的用户就能安心返回受保护的资源

![](https://cdn.wallleap.cn/img/pic/illustration/202410241714290.png?imageSlim)

优点：服务器端不需要存储内容；缺点：很难集中控制，不能在认证中心让用户下线，只能让子系统各自维护

**双 TOKEN 模式**

- 登录的时候返回两个 Token
	- 登录的 token 过期时间设置短一点，让请求经常去认证中心
	- 刷新 token 的过期时间设置长一点，用刷新 token 获取登录 token
- 要让用户下线直接修改刷新 token

![](https://cdn.wallleap.cn/img/pic/illustration/202410241722698.png?imageSlim)

在中间有个认证服务器

- 登录的时候返回两个 token
	- 一个是判定登录的，时间很短就会过期 access_token
	- 一个用来续期上面那个登录的 token 的，过期时间设置长一点 refresh_token
- 只要刷新 token 没有过期就能无限续期

## 使用双 Token 实现无感 token 刷新

随便写一个后端，使用到 express，需要先安装 `pnpm add express`

- 为了简单，这里就不需要验证登录用户和进行加密
- token 就由固定字符串和当前时间组成，但请求登录和刷新接口的时候重新生成 token
- 有生成 token 的时候就在响应头上加 auth，有生成 refresh token 的时候就加上这个，这样浏览器端的时候就更好判断

```js
const express = require('express')
const app = express()
const port = 3000

let token = ''
const tokenExpiredTime = 1000 * 6
let refreshToken = ''
const refreshTokenExpiredTime = 1000 * 20


// 每个请求到来都打印一下
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`)
  next()
})

// 所有请求都加上 cors 相关响应头
app.use((req, res, next) => {
  res.setHeader('Access-Control-Allow-Origin', '*')
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization')
  next()
})

app.get('/', (req, res) => {
  res.send('Hello World!')
})

app.post('/api/login', (req, res) => {
  token = '1234567890=' + Date.now()
  refreshToken = '0987654321=' + Date.now()
  res.setHeader('Authorization', token)
  res.setHeader('Refresh-Token', refreshToken)
  res.json({
    code: 200001,
    msg: 'Login successful',
    data: {
      token,
      refreshToken
    }
  })
})

app.get('/api/user', (req, res) => {
  const bToken = req.headers.authorization?.replace('Bearer ', '')
  if (bToken === token) {
    console.log(Date.now() - token.split('=')[1], tokenExpiredTime)
    if (Date.now() - token.split('=')[1] > tokenExpiredTime) {
      return res.status(401).json({
        code: 401003,
        msg: 'Token expired',
        error: 'Token expired'
      })
    }
    res.json({
      code: 200002,
      msg: 'Get user info successful',
      data: {
        name: 'John Doe'
      }
    })
  } else {
    res.status(401).json({
      code: 401001,
      msg: 'Unauthorized',
      error: 'Token is invalid'
    })
  }
})

app.get('/api/refresh-token', (req, res) => {
  const bRefreshToken = req.headers.authorization.replace('Bearer ', '')
  if (bRefreshToken === refreshToken) {
    // 是否过期
    if (Date.now() - refreshToken.split('=')[1] > refreshTokenExpiredTime) {
      return res.status(401).json({
        code: 401003,
        msg: 'Refresh token expired',
        error: 'Refresh token expired'
      })
    }
    // 更新 token
    token = '1234567890=' + Date.now()
    res.setHeader('Authorization', token)
    res.json({
      code: 200003,
      msg: 'Refresh token successful',
      data: {
        token
      }
    })
  } else {
    res.status(401).json({
      code: 401002,
      msg: 'Unauthorized',
      error: 'Refresh token is invalid'
    })
  }
})

app.listen(port, () => {
  console.log(`Server is running, you can visit http://localhost:${port}`)
})
```

### 手动刷新

封装 `utils/request.js`

```js
import axios from 'axios'
import { getToken, setToken, getRefreshToken, setRefreshToken } from './auth'
import { ElMessage } from 'element-plus'

const BASE_URL = 'http://localhost:3000/api'
const TIMEOUT = 10000
// 不需要携带 token 的请求
const noTokenRequest = ['/login', '/register']
// 需要携带 refreshToken 的请求，如刷新 token
const refreshTokenRequest = ['/refresh-token']
// 错误码映射
const errorCodeMap = {
  401001: 'Token 不正确',
  401002: '刷新 Token 不正确',
  401003: 'Token 过期'
}

const ins = axios.create({
  baseURL: BASE_URL,
  timeout: TIMEOUT,
  headers: {
    'Content-Type': 'application/json'
  },
  validateStatus: function (status) {  // 应当让所有状态码都返回，由自己处理
    return status
  }
})

// 请求拦截器
ins.interceptors.request.use(config => {
  if (!noTokenRequest.includes(config.url)) {
    config.headers['Authorization'] = `Bearer ${getToken()}`
  }
  // 如果请求需要携带 refreshToken，则更新默认的 Authorization
  if (refreshTokenRequest.includes(config.url)) {
    config.headers['Authorization'] = `Bearer ${getRefreshToken()}`
  }
  return config
}, error => {
  return Promise.reject(error)
})

// 响应拦截器
ins.interceptors.response.use(response => {
  if (response.status >= 200 && response.status < 300) {
    console.log(response.headers)
    if (response.headers.authorization) {
      const token = response.headers.authorization.replace('Bearer ', '')
      setToken(token)  // 更新 token
      ins.defaults.headers.Authorization = `Bearer ${token}`
    }
    if (response.headers['refresh-token']) {
      const refreshToken = response.headers['refresh-token'].replace('Bearer ', '')
      setRefreshToken(refreshToken)
    }
  } else {
    ElMessage.error(errorCodeMap[response.data.code] || response.data.msg || '未知错误')
  }
  return response.data
}, error => {
  return Promise.reject(error)
})

export default ins
```

封装 `utils/auth.js`

```js
const TOKEN_KEY = 'token'
const REFRESH_TOKEN_KEY = 'refresh-token'

export function getToken() {
  return localStorage.getItem(TOKEN_KEY)
}

export function setToken(token) {
  localStorage.setItem(TOKEN_KEY, token)
}

export function getRefreshToken() {
  return localStorage.getItem(REFRESH_TOKEN_KEY)
}

export function setRefreshToken(token) {
  localStorage.setItem(REFRESH_TOKEN_KEY, token)
}

export function removeToken() {
  localStorage.removeItem(TOKEN_KEY)
}

export function removeRefreshToken() {
  localStorage.removeItem(REFRESH_TOKEN_KEY)
}
```

`api/login.js`

```js
import request from '../utils/request'

export const login = async (data) => request.post('/login', data)
```

`api/index.js`

```js
import request from '../utils/request'

export const getUser = () => request.get('/user')
```

`api/refreshToken.js`

```js
import request from '../utils/request'

export const refreshToken = () => request.get('/refresh-token')
```

`App.vue`

```vue
<script setup>
import { getUser } from './api'
import { refreshToken } from './api/refreshToken'
import { login } from './api/login'
import { ref } from 'vue'
import { ElButton, ElMessage } from 'element-plus'

const tokenStr = ref('')
const refreshTokenStr = ref('')
const username = ref('')

const loginFn = async () => {
  const res = await login()
  if (res.code === 200001) {
    tokenStr.value = res.data.token
    refreshTokenStr.value = res.data.refreshToken
    ElMessage.success(res.msg || '登录成功')
  }
}

const getUserFn = async () => {
  const res = await getUser()
  if (res.code === 200002) {
    username.value = res.data.name
  }
}

const refreshTokenFn = async () => {
  const res = await refreshToken()
}
</script>

<template>
  <div>
    <h1>请求示例</h1>
    <ElButton @click="loginFn">登录</ElButton>
    <ElButton @click="getUserFn">获取用户信息</ElButton>
    <ElButton @click="refreshTokenFn">刷新 Token</ElButton>
    <div>token: {{ tokenStr }}</div>
    <div>refreshToken: {{ refreshTokenStr }}</div>
    <div>username: {{ username }}</div>
  </div>
</template>
```

> 目前实现的效果：
> 点击登录后能获取用户信息，当过期时间到了 401，接着点击刷新 token，能够继续获取用户信息，但 refreshToken 也过期了就都 401

### 自动刷新

`utils/request.js`

```js
import axios from 'axios'
import { getToken, setToken, getRefreshToken, setRefreshToken, removeToken, removeRefreshToken } from './auth'
import { refreshToken } from '../api/refreshToken'
import { ElMessage } from 'element-plus'

const BASE_URL = 'http://localhost:3000/api'
const TIMEOUT = 10000
// 不需要携带 token 的请求
const noTokenRequest = ['/login', '/register']
// 需要携带 refreshToken 的请求，如刷新 token
const refreshTokenRequest = ['/refresh-token']
// 错误码映射
const errorCodeMap = {
  401001: 'Token 不正确',
  401002: '刷新 Token 不正确',
  401003: 'Token 过期'
}

const ins = axios.create({
  baseURL: BASE_URL,
  timeout: TIMEOUT,
  headers: {
    'Content-Type': 'application/json'
  },
  validateStatus: function (status) {  // 应当让所有状态码都返回，由自己处理
    return status
  }
})

// 请求拦截器
ins.interceptors.request.use(config => {
  if (!noTokenRequest.includes(config.url)) {
    config.headers['Authorization'] = `Bearer ${getToken()}`
  }
  // 如果请求需要携带 refreshToken，则更新默认的 Authorization
  if (refreshTokenRequest.includes(config.url)) {
    config.headers['Authorization'] = `Bearer ${getRefreshToken()}`
  }
  return config
}, error => {
  return Promise.reject(error)
})

// 响应拦截器
ins.interceptors.response.use(async response => {
  if (response.status >= 200 && response.status < 300) {
    if (response.headers.authorization) {
      const token = response.headers.authorization.replace('Bearer ', '')
      setToken(token)  // 更新 token
      ins.defaults.headers.Authorization = `Bearer ${token}`
    }
    if (response.headers['refresh-token']) {
      const refreshToken = response.headers['refresh-token'].replace('Bearer ', '')
      setRefreshToken(refreshToken)
    }
  } else if (response.status === 401 && !refreshTokenRequest.includes(response.config.url)) {
    // 没有权限并且不是刷新 token 的请求
    // 刷新 token
    const res = await refreshToken()
    if (res.code === 200003) {
      // 刷新成功，重新请求
      response.config.headers['Authorization'] = `Bearer ${getToken()}`
      const res = await ins.request(response.config)
      return res
    } else {
      // 刷新失败，重新登录
      removeToken()
      removeRefreshToken()
      ElMessage.error('请重新登录')
      // window.location.href = '/login'
    }
  } else {
    ElMessage.error(errorCodeMap[response.data.code] || response.data.msg || '未知错误')
  }
  return response.data
}, error => {
  return Promise.reject(error)
})

export default ins
```

修改 `api/refreshToken.js`

```js
import request from '../utils/request'

// 防止 token 过期的时候，多次请求造成多次刷新 token
let promise = null

export const refreshToken = () => {
  if (promise) {
    return promise
  }
  promise = new Promise((resolve, reject) => {
    request.get('/refresh-token').then(res => {
      resolve(res)
    })
  })
  promise.finally(() => {
    promise = null
  })
  return promise
}
```

> 现在只要 token 过期了就会自动请求刷新 token 并重新发送请求，直到 refreshToken 也过期，跳转登录页面
