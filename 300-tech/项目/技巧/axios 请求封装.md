---
title: axios 请求封装
date: 2024-11-07T12:57:06+08:00
updated: 2024-11-07T16:22:52+08:00
dg-publish: false
---

安装 axios 之后可以直接使用，但一般会封装一层

约定后端返回的**资源** API 数据格式为（特殊接口除外）

```json
{
  "code": 0,
  "msg": "请求成功",
  "error": "",
  "data": [
    { "id": 1, "name": "TOM" }
  ]
}
```

- code：Number 业务状态码
- msg: String 描述信息
- error: String | Object 错误信息
- data: Object 返回的资源数据

例如

- 200001 'success' 操作成功
- 200204 数据异常
- 200216 帐号已停用
- 200217 节点下有子节点，不可以删除
- 200219 库存不足
- 401001 用户未登录
- 401002 登录过期
- 401010 用户名或密码错误
- 401011 验证码错误
- 999001 网络有问题

还可以

```
#1000～1999 区间表示参数错误
#2000～2999 区间表示用户错误
#3000～3999 区间表示接口异常
```

总之自己定好一套规范

`utils/auth.js`

```js
const ACCESS_TOKEN = 'access_token'

const getToken = () => {
  return localStorage.getItem(ACCESS_TOKEN)
}
```

`utils/request.js`

```js
import axios from 'axios'

const baseURL = 'http://localhost:3000'
// 和后端约定好的 code 值（业务状态码），0 为成功
const errorCode = {
  401001: (res) => {  // invalid token
    // $message.error(res?.msg || 'Token 错误，请重新登录')
    logout()
  },
  401002: (res) => {  // token expired
    // $message.error(res?.msg || '登录过期，请重新登录')
    logout()
  },
  403001: (res) => {
    // $message.error(res?.msg || '没有权限')
  },
}
// HTTP 状态码
const httpErrorCode = {
  400: () => {
    // $message.error('请求参数缺失或格式不正确')
  },
  404: () => {
    // $message.error('接口不存在')
  },
  405: () => {
    // $message.error('错误的请求方式')
  },
  410: () => {
    // $message.error('请求的资源已经被永久删除')
  },
  429: () => {
    // $message.error('请求太过频繁，请稍候再试')
  },
  500: () => {
    // $message.error('服务器错误')
  },
  502: () => {
    // $message.error('网关错误')
  },
  503: () => {
    // $message.error('服务不可用')
  },
  504: () => {
    // $message.error('网关超时')
  },
}

// 创建实例
const instance = axios.create({
  baseURL,
})

// 请求拦截器
instance.interceptors.request.use(
  config => {
    const token = getToken()
    if (!token) {
      // 没有 token，退出登录 logout()
    } else {
      // 有 token 就携带 token
      config.headers.Authorization = `Bearer ${token}`
    }
    // 其他修改 header 或 config 的操作
    return config
  },
  error => {
    // 请求错误处理
    return Promise.reject(error)
  }
)

// 响应拦截器
instance.interceptors.response.use(
  response => {
    // 没有 code 的接口
    if (response.data instanceof Blob) return response.data
    const { code } = response.data
    if (code === 0) {
      // 成功，使用 .then，返回的是数据
      return response.data.data
    }
    // 其他，使用 .catch
    errorCode[code]?.(response)
    return Promise.reject(response)
  },
  error => {
    const { status } = error
    httpErrorCode[status]?.()
    return Promise.reject(error)
  }
)

export function request(url, config = {}) {
  return instance.get(url, config)
}

export default instance
```

例如：

```js
import { ref } from 'vue'
import { request } from '@/utils/request.js'

const users = ref([])

const getSuccessData = () => {
  request('/api/users')
    .then(res => {
      users.value = res  // [{ id: 0, name: 'Tom' }]
    })
    .catch(err => {
      console.log('请求失败', err)
    })
}

const getTokenExpiredData = () => {
  request('/api/token-expired')
    .then(() => {
      console.log('测试退出登录')
    })
}

const getFile = () => {
  request('/api/download', {
    responseType: 'blob'
  }).then(res => {
    // 下载文件 res
  })
}

const request404 = () => {
  request('/api/404').then(() => {
      console.log('测试404')
    })
}
```
