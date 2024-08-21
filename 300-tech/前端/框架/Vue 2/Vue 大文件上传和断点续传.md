---
title: Vue 大文件上传和断点续传
date: 2023-08-04T10:14:00+08:00
updated: 2024-08-21T10:33:30+08:00
dg-publish: false
aliases:
  - Vue 大文件上传和断点续传
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
  - Vue
  - web
url: //myblog.wallleap.cn/post/1
---

这是我参与更文挑战的第 9 天，活动详情查看： [更文挑战](https://juejin.cn/post/6967194882926444557?utm_campaign=30day&utm_medium=Ccenter&utm_source=20210528 "https://juejin.cn/post/6967194882926444557?utm_campaign=30day&utm_medium=Ccenter&utm_source=20210528")

> 本文是关于 Vue 大文件上传和断点续传的学习笔记，这里做个总结与分享，有不足之处还望斧正~

## 文件上传的 2 套方案

### 基于文件流（form-data）

element-ui 框架的上传组件，就是默认基于文件流的。

- 数据格式：form-data；
- 传递的数据： file 文件流信息；filename 文件名字

![](https://cdn.wallleap.cn/img/pic/illustration/202308041015155.png)

### 客户端把文件转换为 base64

通过 `fileRead.readAsDataURL(file)` 转为 base64 字符串后，要用 `encodeURIComponent` 编译再发送， 发送的数据经由 `qs.stringify` 处理，请求头添加 `"Content-Type": "application/x-www-form-urlencoded"`

## 大文件上传

首先借助 element-ui 搭建下页面。因为要自定义一个上传的实现，所以 `el-upload` 组件的 `auto-upload` 要设定为 `false`；`action` 为必选参数，此处可以不填值。

```vue
<template>
  <div id="app">
    <!-- 上传组件 -->
    <el-upload action drag :auto-upload="false" :show-file-list="false" :on-change="handleChange">
      <i class="el-icon-upload"></i>
      <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
      <div class="el-upload__tip" slot="tip">大小不超过 200M 的视频</div>
    </el-upload>

    <!-- 进度显示 -->
    <div class="progress-box">
      <span>上传进度：{{ percent.toFixed() }}%</span>
      <el-button type="primary" size="mini" @click="handleClickBtn">{{ upload | btnTextFilter}}</el-button>
    </div>

    <!-- 展示上传成功的视频 -->
    <div v-if="videoUrl">
      <video :src="videoUrl" controls />
    </div>
  </div>
</template>
```

### 获取到文件对象并转成 ArrayBuffer 对象

转成 ArrayBuffer 是因为后面要用 SparkMD5 这个库生成 hash 值，对文件进行命名

```js
async handleChange(file) {
  const fileObj = file.raw
  try{
    const buffer = await this.fileToBuffer(fileObj)
    console.log(buffer)
  }catch(e){
    console.log(e)
  }
}
```

打印 buffer 结果如下图

![](https://cdn.wallleap.cn/img/pic/illustration/202308041015156.png)

注意：before-upload 函数和 on-change 函数的参数都有 file，但是 on-change 中的 file 不是 File 对象，要获取 File 对象需要通过 file.raw。 这里用到 FileReader 类将 File 对象转 ArrayBuffer 对象，因为是异步过程，所以用 Promise 封装:

```js
// 将 File 对象转为 ArrayBuffer 
fileToBuffer(file) {
  return new Promise((resolve, reject) => {
    const fr = new FileReader()
    fr.onload = e => {
      resolve(e.target.result)
    }
    fr.readAsArrayBuffer(file)
    fr.onerror = () => {
      reject(new Error('转换文件格式发生错误'))
    }
  })
}
```

### 创建切片

可以通过固定大小或是固定数量把一个文件分成好几个部分，为了避免由于 js 使用的 IEEE754 二进制浮点数算术标准可能导致的误差，我决定用固定大小的方式对文件进行切割，设定每个切片的大小为 2M，即 2M = 2_1024KB = 2_1024\*1024B = 2097152B。切割文件用到的是 `Blob.slice()`

```js
// 将文件按固定大小（2M）进行切片，注意此处同时声明了多个常量
const chunkSize = 2097152,
  chunkList = [], // 保存所有切片的数组
  chunkListLength = Math.ceil(fileObj.size / chunkSize), // 计算总共多个切片
  suffix = /\.([0-9A-z]+)$/.exec(fileObj.name)[1] // 文件后缀名
  
// 根据文件内容生成 hash 值
const spark = new SparkMD5.ArrayBuffer()
spark.append(buffer)
const hash = spark.end()

// 生成切片，这里后端要求传递的参数为字节数据块（chunk）和每个数据块的文件名（fileName）
let curChunk = 0 // 切片时的初始位置
for (let i = 0; i < chunkListLength; i++) {
  const item = {
    chunk: fileObj.slice(curChunk, curChunk + chunkSize),
    fileName: `${hash}_${i}.${suffix}` // 文件名规则按照 hash_1.jpg 命名
  }
  curChunk += chunkSize
  chunkList.push(item)
}
console.log(chunkList)
```

选择一个文件后将打印得到诸如下图的结果

![](https://cdn.wallleap.cn/img/pic/illustration/202308041015157.png)

### 发送请求

发送请求可以是并行的或是串行的，这里选择串行发送。每个切片都新建一个请求，为了能实现断点续传，我们将请求封装到函数 `fn` 里，用一个数组 `requestList` 来保存请求集合，然后封装一个 `send` 函数，用于请求发送，这样一旦按下暂停键，可以方便的终止上传，代码如下：

```js
sendRequest() {
  const requestList = [] // 请求集合
  this.chunkList.forEach(item => {
    const fn = () => {
      const formData = new FormData()
      formData.append('chunk', item.chunk)
      formData.append('filename', item.fileName)
      return axios({
        url: '/single3',
        method: 'post',
        headers: { 'Content-Type': 'multipart/form-data' },
        data: formData
      }).then(res => {
        if (res.data.code === 0) { // 成功
          if (this.percentCount === 0) {
            this.percentCount = 100 / this.chunkList.length
          }
          this.percent += this.percentCount // 改变进度
        }
      })
    }
    requestList.push(fn)
  })
  
  let i = 0 // 记录发送的请求个数
  const send = async () => {
    // if ('暂停') return
    if (i >= requestList.length) {
      // 发送完毕
      return
    } 
    await requestList[i]()
    i++
    send()
  }
  send() // 发送请求
},
```

axios 部分也可以直接写成下面这种形式：

```js
axios.post('/single3', formData, {
  headers: { 'Content-Type': 'multipart/form-data' }
})
```

### 所有切片发送成功后

根据后端接口需要再发送一个 get 请求并把文件的 hash 值传给服务器，我们定义一个 `complete` 方法来实现，这里假定发送的为视频文件

```js
const complete = () => {
  axios({
    url: '/merge',
    method: 'get',
    params: { hash: this.hash }
  }).then(res => {
    if (res.data.code === 0) { // 请求发送成功
      this.videoUrl = res.data.path
    }
  })
}
```

这样就能在文件发送成功后在页面浏览到发送的视频了。

## 断点续传

首先是暂停按钮文字的处理，用了一个过滤器，如果 `upload` 值为 `true` 则显示“暂停”，否则显示“继续”：

```js
filters: {
  btnTextFilter(val) {
    return val ? '暂停' : '继续'
  }
}
```

当按下暂停按钮，触发 `handleClickBtn` 方法

```js
handleClickBtn() {
  this.upload = !this.upload
  // 如果不暂停则继续上传
  if (this.upload) this.sendRequest()
}
```

在发送切片的 `send` 方法的开头添加 `if (!this.upload) return`，这样只要 `upload` 这个变量为 `false` 就不会继续上传了。为了在暂停完后可以继续发送，需要在每次成功发送一个切片后将这个切片从 `chunkList` 数组里删除 `this.chunkList.splice(index, 1)`

## 代码汇总

```vue
<template>
  <div id="app">
    <!-- 上传组件 -->
    <el-upload action drag :auto-upload="false" :show-file-list="false" :on-change="handleChange">
      <i class="el-icon-upload"></i>
      <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
      <div class="el-upload__tip" slot="tip">大小不超过 200M 的视频</div>
    </el-upload>

    <!-- 进度显示 -->
    <div class="progress-box">
      <span>上传进度：{{ percent.toFixed() }}%</span>
      <el-button type="primary" size="mini" @click="handleClickBtn">{{ upload | btnTextFilter}}</el-button>
    </div>

    <!-- 展示上传成功的视频 -->
    <div v-if="videoUrl">
      <video :src="videoUrl" controls />
    </div>
  </div>
</template>

<script>
  import SparkMD5 from "spark-md5"
  import axios from "axios"
  
  export default {
    name: 'App3',
    filters: {
      btnTextFilter(val) {
        return val ? '暂停' : '继续'
      }
    },
    data() {
      return {
        percent: 0,
        videoUrl: '',
        upload: true,
        percentCount: 0
      }
    },
    methods: {
      async handleChange(file) {
        if (!file) return
        this.percent = 0
        this.videoUrl = ''
        // 获取文件并转成 ArrayBuffer 对象
        const fileObj = file.raw
        let buffer
        try {
          buffer = await this.fileToBuffer(fileObj)
        } catch (e) {
          console.log(e)
        }
        
        // 将文件按固定大小（2M）进行切片，注意此处同时声明了多个常量
        const chunkSize = 2097152,
          chunkList = [], // 保存所有切片的数组
          chunkListLength = Math.ceil(fileObj.size / chunkSize), // 计算总共多个切片
          suffix = /\.([0-9A-z]+)$/.exec(fileObj.name)[1] // 文件后缀名
          
        // 根据文件内容生成 hash 值
        const spark = new SparkMD5.ArrayBuffer()
        spark.append(buffer)
        const hash = spark.end()

        // 生成切片，这里后端要求传递的参数为字节数据块（chunk）和每个数据块的文件名（fileName）
        let curChunk = 0 // 切片时的初始位置
        for (let i = 0; i < chunkListLength; i++) {
          const item = {
            chunk: fileObj.slice(curChunk, curChunk + chunkSize),
            fileName: `${hash}_${i}.${suffix}` // 文件名规则按照 hash_1.jpg 命名
          }
          curChunk += chunkSize
          chunkList.push(item)
        }
        this.chunkList = chunkList // sendRequest 要用到
        this.hash = hash // sendRequest 要用到
        this.sendRequest()
      },
      
      // 发送请求
      sendRequest() {
        const requestList = [] // 请求集合
        this.chunkList.forEach((item, index) => {
          const fn = () => {
            const formData = new FormData()
            formData.append('chunk', item.chunk)
            formData.append('filename', item.fileName)
            return axios({
              url: '/single3',
              method: 'post',
              headers: { 'Content-Type': 'multipart/form-data' },
              data: formData
            }).then(res => {
              if (res.data.code === 0) { // 成功
                if (this.percentCount === 0) { // 避免上传成功后会删除切片改变 chunkList 的长度影响到 percentCount 的值
                  this.percentCount = 100 / this.chunkList.length
                }
                this.percent += this.percentCount // 改变进度
                this.chunkList.splice(index, 1) // 一旦上传成功就删除这一个 chunk，方便断点续传
              }
            })
          }
          requestList.push(fn)
        })
        
        let i = 0 // 记录发送的请求个数
        // 文件切片全部发送完毕后，需要请求 '/merge' 接口，把文件的 hash 传递给服务器
        const complete = () => {
          axios({
            url: '/merge',
            method: 'get',
            params: { hash: this.hash }
          }).then(res => {
            if (res.data.code === 0) { // 请求发送成功
              this.videoUrl = res.data.path
            }
          })
        }
        const send = async () => {
          if (!this.upload) return
          if (i >= requestList.length) {
            // 发送完毕
            complete()
            return
          } 
          await requestList[i]()
          i++
          send()
        }
        send() // 发送请求
      },
      
      // 按下暂停按钮
      handleClickBtn() {
        this.upload = !this.upload
        // 如果不暂停则继续上传
        if (this.upload) this.sendRequest()
      },
      
      // 将 File 对象转为 ArrayBuffer 
      fileToBuffer(file) {
        return new Promise((resolve, reject) => {
          const fr = new FileReader()
          fr.onload = e => {
            resolve(e.target.result)
          }
          fr.readAsArrayBuffer(file)
          fr.onerror = () => {
            reject(new Error('转换文件格式发生错误'))
          }
        })
      }
    }
  }
</script>

<style scoped>
  .progress-box {
    box-sizing: border-box;
    width: 360px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-top: 10px;
    padding: 8px 10px;
    background-color: #ecf5ff;
    font-size: 14px;
    border-radius: 4px;
  }
</style>
```

效果如下图：

![](https://cdn.wallleap.cn/img/pic/illustration/202308041015158.gif)

## One More Thing

### FormData

这里发送数据用到了 FormData，如果编码类型被设为 "multipart/form-data"，它会使用和表单一样的格式。

### FormData.append()

会添加一个新值到 `FormData` 对象内的一个已存在的键中，如果键不存在则会添加该键。该方法可以传递 3 个参数，`formData.append(name, value, filename)`，其中 `filename` 为可选参数，是传给服务器的文件名称， 当一个 Blob 或 File 被作为第二个参数的时候， Blob 对象的默认文件名是 "blob"。 File 对象的默认文件名是该文件的名称。
