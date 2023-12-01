---
title: 开箱即用的 axios 封装：Vue3+ts
date: 2023-08-07 09:41
updated: 2023-08-07 09:42
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - 开箱即用的 axios 封装：Vue3+ts
rating: 1
tags:
  - Vue
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

持续创作，加速成长！这是我参与「掘金日新计划 · 6 月更文挑战」的第 12 天，[点击查看活动详情](https://juejin.cn/post/7099702781094674468 "https://juejin.cn/post/7099702781094674468")

## 前言

Axios 多用于处理前端项目的 Ajax 请求，这里要注意区分 Axios 和 Ajax：Ajax 是一种技术统称，Axios 是第三方库。在使用的时候，我们可以直接使用 Axios 来发起请求，也可以封装后采用统一的接口发送请求。在前端项目中，应该大多数人都会选择封装一下 Axios，不仅可以节省代码，看起来更简洁；而且可以统一管理请求和响应。本文就以 Vue3+Typescript，封装一个”开箱即用“的 Axios。

## 开始

封装 axios 的好处我就不多说了，接下来直接上手。

首先我们来梳理一下思路：

- 想要封装 axios，肯定需要安装 axios 依赖
- 封装 axios，主要是通过拦截器分别处理 HTTP 请求和响应，并反馈 HTTP 请求结果
- 使用案例

### 安装依赖

安装 axios 依赖，安装 element-plus，用来反馈请求结果

```
npm i axios,element-plus
```

### 封装 axios

新建 `index.ts` 文件：

- 需要定义请求返回的数据格式，这个可以和服务端约定好数据格式
- 需要定义 axios 的配置信息，用于在创建 axios 实例时传入
- 请求拦截器，前端所有的接口请求都会先达到请求拦截器，我们可以在此添加请求头信息
- 响应拦截器，服务端返回的数据会先达到响应拦截器，我们可以处理服务端的响应信息。如果是报错，就处理常见的报错；如果是成功，就返回数据
- 封装常用的 get、put、post、delete 接口方法

```
import axios, {AxiosInstance, AxiosError, AxiosRequestConfig, AxiosResponse} from 'axios'
import {ElMessage} from 'element-plus'
// 数据返回的接口
// 定义请求响应参数，不含data
interface Result {
  code: number;
  msg: string
}

// 请求响应参数，包含data
interface ResultData<T = any> extends Result {
  data?: T;
}
const URL: string = ''
enum RequestEnums {
  TIMEOUT = 20000,
  OVERDUE = 600, // 登录失效
  FAIL = 999, // 请求失败
  SUCCESS = 200, // 请求成功
}
const config = {
  // 默认地址
  baseURL: URL as string,
  // 设置超时时间
  timeout: RequestEnums.TIMEOUT as number,
  // 跨域时候允许携带凭证
  withCredentials: true
}

class RequestHttp {
  // 定义成员变量并指定类型
  service: AxiosInstance;
  public constructor(config: AxiosRequestConfig) {
    // 实例化axios
    this.service = axios.create(config);

    /**
     * 请求拦截器
     * 客户端发送请求 -> [请求拦截器] -> 服务器
     * token校验(JWT) : 接受服务器返回的token,存储到vuex/pinia/本地储存当中
     */
    this.service.interceptors.request.use(
      (config: AxiosRequestConfig) => {
        const token = localStorage.getItem('token') || '';
        return {
          ...config,
          headers: {
            'x-access-token': token, // 请求头中携带token信息
          }
        }
      },
      (error: AxiosError) => {
        // 请求报错
        Promise.reject(error)
      }
    )

    /**
     * 响应拦截器
     * 服务器换返回信息 -> [拦截统一处理] -> 客户端JS获取到信息
     */
    this.service.interceptors.response.use(
      (response: AxiosResponse) => {
        const {data, config} = response; // 解构
        if (data.code === RequestEnums.OVERDUE) {
          // 登录信息失效，应跳转到登录页面，并清空本地的token
          localStorage.setItem('token', '');
          // router.replace({
          //   path: '/login'
          // })
          return Promise.reject(data);
        }
        // 全局错误信息拦截（防止下载文件得时候返回数据流，没有code，直接报错）
        if (data.code && data.code !== RequestEnums.SUCCESS) {
          ElMessage.error(data); // 此处也可以使用组件提示报错信息
          return Promise.reject(data)
        }
        return data;
      },
      (error: AxiosError) => {
        const {response} = error;
        if (response) {
          this.handleCode(response.status)
        }
        if (!window.navigator.onLine) {
          ElMessage.error('网络连接失败');
          // 可以跳转到错误页面，也可以不做操作
          // return router.replace({
          //   path: '/404'
          // });
        }
      }
    )
  }
  handleCode(code: number):void {
    switch(code) {
      case 401:
        ElMessage.error('登录失败，请重新登录');
        break;
      default:
        ElMessage.error('请求失败');
        break;
    }
  }

  // 常用方法封装
  get<T>(url: string, params?: object): Promise<ResultData<T>> {
    return this.service.get(url, {params});
  }
  post<T>(url: string, params?: object): Promise<ResultData<T>> {
    return this.service.post(url, params);
  }
  put<T>(url: string, params?: object): Promise<ResultData<T>> {
    return this.service.put(url, params);
  }
  delete<T>(url: string, params?: object): Promise<ResultData<T>> {
    return this.service.delete(url, {params});
  }
}

// 导出一个实例对象
export default new RequestHttp(config);
```

### 实际使用

在使用时，我们需要在 API 文档中导入 `index.ts`，会自动创建一个 axios 实例。我们在同目录下，新建一个 `login.ts`

```
import axios from './'
namespace Login {
  // 用户登录表单
  export interface LoginReqForm {
    username: string;
    password: string;
  }
  // 登录成功后返回的token
  export interface LoginResData {
    token: string;
  }
}
// 用户登录
export const login = (params: Login.LoginReqForm) => {
    // 返回的数据格式可以和服务端约定
    return axios.post<Login.LoginResData>('/user/login', params);
}
```

API 接口也定义好了，再来一个页面简单试试：

```
<script setup>
import { reactive } from 'vue';
import {login} from '@/api/login.js'
const loginForm = reactive({
  username: '',
  password: ''
})
const Login = async () => {
  const data = await login(loginForm)
  console.log(data);
}
</script>
<template>
  <input v-model="loginForm.username" />
  <input v-model="loginForm.password" type="password" />
  <button @click="Login">登录</button>
</template>
```

## 总结

本文按照 TypeScript 的风格封装了 axios，需要的朋友可以直接拿来使用，对自己来说也是一次学习的收获。封装 axios 并不难，重点是请求拦截器和响应拦截器，只是要注意 ts 的类型约束。
