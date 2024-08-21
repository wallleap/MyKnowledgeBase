---
title: 010-axios 拦截器及访问控制
date: 2023-06-25T08:10:00+08:00
updated: 2024-08-21T10:33:31+08:00
dg-publish: false
aliases:
  - 010-axios 请求拦截器
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
  - 项目
url: //myblog.wallleap.cn/post/1
---

正常接口方法里，在函数里请求 headers 函数需要写多个，要修改的话需要修改多处，故我们需要在请求拦截器统一携带

![](https://cdn.wallleap.cn/img/pic/illustration/202306252013982.png)

请求拦截器：前 → 后

响应拦截器：后 → 前

## 定义请求拦截器

在 `utils/request.js` 中，使用自定义 axios 函数绑定请求拦截器，判断有 token 再携带到请求头上（**API 每次调用 request 都会先走这个请求拦截器**）

```js
import store from '@/store'
// 定义请求拦截器
myAxios.interceptors.request.use(function (config) {
  // 在请求前会触发一次
  // 为请求头挂载 Authorization 字段
  config.headers.Authorization = store.state.token
  return config  // config 配置对象（要请求的后端参数都在这个对象上），它返回给 axios 内源码
}, function (error) {
  // 请求发起前的代码，如果有异常报错，会直接记录在这里
  return Promise.reject(error)  // 返回一个拒绝状态的 Promise 对象
})
```

进行完善

```js
if(store.state.token) {  // 有 token 才添加 headers
  config.headers.Authorization = store.state.token
}
```

删除 `src/api/index.js` 中 `header` 内容

```js
/**
 * 获取用户信息接口
 * @returns Promise对象
 */
export const getUsersInfoAPI = () => {
  return request({
    url: '/my/userinfo',
  })
}

/**
 * 获取侧边栏策单列表接口
 * @returns Promise对象
 */
export const getMenusAPI = () => {
  return request({
    url: '/my/menus',
  })
}
```

## 权限 - 控制访问

没有登录就不能看一些内容（不能访问正常页面）

在路由守卫中进行判断

- 浏览器第一次打开项目页面，会触发一次全局前置路由守卫
- 直接修改页面路径也会触发
- 强制 `next()` 切换路由地址，也会触发，且会递归死循环（栈溢出）

判断登录与否：有 token 就是登录了，没有 token 就没登录

由于直接 `next('/login')` 也会触发路由守卫，需要加个判断（新增一个不需要重定向的白名单）

```js
// 白名单，不需要重定向
const whiteList = ['/login', '/reg']

// 全局前置路由守卫
router.beforeEach((to, from, next) => {
  // 不论有无登录，都可以直接前往白名单中的页面
  if (whiteList.includes(to.path)) {
    return next()
  }
  const token = store.state.token
  if (token) {  // 登录了
    if (!store.state.userInfo.username) {
      store.dispatch('getUserInfoActions')
    }
    next()
  } else {  // 没有登录回到登录页
    next('/login')
  }
})```

## 定义响应拦截器

有 token，但是过期了

1. 前端是无法判断 token 是否过期了的，所以当某次发请求把 token 带给后台做验证的时候
2. 后台发现 token 过期了，则会返回响应状态码 401
3. 但是不能确定在哪个请求会 401，所以要用统一的响应拦截器做判断
4. 在 `src/utils/request.js` 中，给自定义 axios 函数添加响应拦截器

```js
// 定义响应拦截器
myAxios.interceptors.response.use(function (response) {
  // 响应状态码为 2xx 或 3xx 时触发成功的回调，形参中的 response 是“成功的结果”
  return response
}, function (error) {
  // 响应状态码是 4xx、5xx 时触发失败的回调，形参中的 error 是“失败的结果”
  return Promise.reject(error)  // 在逻辑页面使用 try-catch 或 catch 能够捕获
})
```

完善，进行自动退出（删除 Vuex 中内容，跳转到登录页，显示一个提示信息）

```js
import store from '@/store'
import router from '@/router'
import { Message } from 'element-ui'
// 定义响应拦截器
myAxios.interceptors.response.use(function (response) {
  return response
}, function (error) {
  if (error.response.status === 401) {
    store.commit('updateToken', '')
    store.commit('updateUserInfo', {})
    router.push('/login')
    Message.error('登录状态无效，请重新登录')
  }
  return Promise.reject(error)
})
```
