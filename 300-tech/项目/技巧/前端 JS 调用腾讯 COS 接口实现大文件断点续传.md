---
title: 前端 JS 调用腾讯 COS 接口实现大文件断点续传
date: 2024-03-20T10:24:00+08:00
updated: 2024-08-21T10:32:45+08:00
dg-publish: false
---

做大文件断点续传的功能，主要是为了提升用户体验。除了用户侧的体验外，实现的技术成本也是需要考虑的，比如服务器带宽、存储成本等，下面会介绍一种已经在生产环境中应用的企业级大文件断点续传实现方案。

![upload_gif.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a7bc95f809134ad2933f5ad2986782d1~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.gif#?w=669&h=191&s=34186&e=gif&f=52&b=fcfbfb)

## 为什么需要断点续传

对于普通的照片，或比较小的文件，直接调用上传接口就可以了，因为上传的很快，不需要额外处理。

但大文件上传所需要花费的时间较长，用户在上传等待的过程中可能会出现上传中断，中断后用户需要重新上传。假设用户花了 30 分钟，上传了一半，结果中断，又要重新传，这 30 分钟就浪费了，非常影响用户体验。

![0.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/68f61c0b2706451d9ccd5e2bb5ce3164~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.png#?w=680&h=146&s=24881&e=jpg&b=fefefe)

如上图，一般大文件的上传都会在前端显示上传进度，而且支持断点续传。这样用户可以在页面上看到上传进度，也不用担心上传中断后，又要重复上传的问题。

## 常规技术方案及其缺陷

一般实现大文件上传分两步：

1. 将大文件用 [Blob slice 方法](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fen-US%2Fdocs%2FWeb%2FAPI%2FBlob%2Fslice "https://developer.mozilla.org/en-US/docs/Web/API/Blob/slice")  切分为多个碎片逐一上传，后端接收到这些文件，并保存相应的进度到数据库。
2. 在下次相传相同的文件（md5 一致）时，读取对应的进度返回给前端，前端根据进度继续上传后续文件内容。

但实际在生产环境中使用时，会有一个问题，如下图，常规的流程

1. 前端上传文件后，调用我们自己的接口 /upload 上传文件到服务器
2. 服务器接收到文件后，一般不会存到自己的服务器，而是调用第三方平台提供的 API 存储大文件
3. 第三方 cdn/对象存储 平台返回文件上传进度或 url 信息

![1.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8d354650dbe243e282b8c94608d46321~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.png#?w=680&h=128&s=35313&e=jpg&b=fdfdfd)

这样就会有一个问题：

1. 服务器带宽有限，假设为 20M 带宽，上传大文件时间较长，会占用服务器带宽，影响其他接口响应速度
2. 本来上传文件时间就长，中间加了一层转发，时间会加长

理论上我们可以直接在前端调用第三方 cdn/存储 平台的 API 来进行上传，上传完成后把返回的 url 存储到数据库即可，省去中间环节，耗时会更短，还不占服务器带宽，不用担心服务器带宽被打满。

![2.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/71fff0e35abe4247bfd7747296858074~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.png#?w=533&h=275&s=53580&e=jpg&b=fdfdfd)

## 腾讯云 COS 对象存储 JS API

这里以腾讯云 COS 对象存储为例，它自身就支持断点续传，而且支持 js 直接调用 API 进行断点续传，可以很好的满足我们要求。[对象存储-SDK文档-JavaScript SDK 快速入门](https://link.juejin.cn/?target=https%3A%2F%2Fcloud.tencent.com%2Fdocument%2Fproduct%2F436%2F11459 "https://cloud.tencent.com/document/product/436/11459")

断点续传触发机制：在 COS 中如果文件上传了一半，下次上传该文件时（文件名 (key)+ 文件内容一致），会自动断点续传。可以在腾讯云控制台 - 文件碎片位置看具体的上传进度信息。

![3.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b638b6e31684471284f9e00622dc960d~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.png#?w=1915&h=607&s=320861&e=jpg&b=fdfdfd)

碎片详情

![4.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b2fa695eea594975afe7a8ffa50eccca~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.png#?w=680&h=620&s=74995&e=jpg&b=fefefe)

## demo 实现

腾讯云 COS 对象存储官方提供了基础的 JS SDK [cos-js-sdk-v5](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Ftencentyun%2Fcos-js-sdk-v5 "https://github.com/tencentyun/cos-js-sdk-v5")，主要包含两个部分

1. 鉴权
2. 前端上传

可以参照这个仓库的文档和代码实现一个最小化 demo，如下图

![5.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cc8dd2ed417a43c19799690161a3002f~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.png#?w=680&h=234&s=47400&e=jpg&b=fefdfd)

### 准备工作

首先需要在腾讯云后台创建存储桶，后面会把文件上传到对应的存储桶

1. 到 [COS对象存储控制台](https://link.juejin.cn/?target=https%3A%2F%2Fconsole.cloud.tencent.com%2Fcos "https://console.cloud.tencent.com/cos") 创建存储桶，得到 Bucket 和 Region（地域名称）
2. 到 [控制台密钥管理](https://link.juejin.cn/?target=https%3A%2F%2Fconsole.cloud.tencent.com%2Fcapi "https://console.cloud.tencent.com/capi") 获取您的项目 SecretId 和 SecretKey，用于上传接口鉴权

![6.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5a0b7878d64e45cfabf978c92ef64a5d~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.png#?w=653&h=430&s=65871&e=jpg&b=fef2f0)

### 鉴权接口

需要实现一个接口，用于鉴权，假设接口为：`http://127.0.0.1:3000/sts`, 参考文档：[STS 鉴权](https://link.juejin.cn/?target=https%3A%2F%2Fcloud.tencent.com%2Fdocument%2Fproduct%2F436%2F14048 "https://cloud.tencent.com/document/product/436/14048")

请求这个接口，需要返回 tmpSecretId, tmpSecretKey, sessionToken, 如下图

![7.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9ec4a72a25014ae59de3b46478452676~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.png#?w=689&h=325&s=119456&e=jpg&b=fefefe)

官方有提供现成的 node 实现，这里我们直接使用：下载 [cos-js-sdk-v5](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Ftencentyun%2Fcos-js-sdk-v5 "https://github.com/tencentyun/cos-js-sdk-v5")

```sh
# 进入 cos-js-sdk-v5 目录, 安装依赖
npm install
nodemon ./server/sts.js
```

会在 3000 端口开启一个接口服务，用 postman 请求 `http://127.0.0.1:3000/sts` 如果有正常返回上面截图的信息，就说明鉴权接口 ok

如果不成功，需要检查 server/sts.js 文件中的 SecretId、SecretKey 是否已配置成自己的信息，另外 `allowPrefix: '*'` 配置根据需要自己修改

### 前端 UI 逻辑实现

这里前端 demo 使用 vue3 纯 html 方式来实现，引入 COS 的 js 库文件 [cos-js-sdk-v5.js](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Ftencentyun%2Fcos-js-sdk-v5%2Fblob%2Fmaster%2Fdist%2Fcos-js-sdk-v5.js "https://github.com/tencentyun/cos-js-sdk-v5/blob/master/dist/cos-js-sdk-v5.js")，就可以在前端，用 js 调用 API 直传腾讯云 cos 对象存储。

#### 请求鉴权接口获取 cos 实例

上面的鉴权接口 ok 后，前端首先要请求鉴权接口，获取 cos 实例，在页面加载时调用 getCosInstance() 方法拿 cos 实例，具体代码如下

```js
async mounted() {
  this.getCosInstance();
},
methods: {
  // 初始化 cos 实例/鉴权
  getCosInstance() {
    var cos = new COS({
      getAuthorization: function (options, callback) {
        var url = "http://127.0.0.1:3000/sts"; // 鉴权接口地址
        var xhr = new XMLHttpRequest();
        xhr.open("GET", url, true);
        xhr.onload = function (e) {
          try {
            var data = JSON.parse(e.target.responseText);
            var credentials = data.credentials;
          } catch (e) {}
          if (!data || !credentials)
            return console.error("credentials invalid");
          callback({
            TmpSecretId: credentials.tmpSecretId,
            TmpSecretKey: credentials.tmpSecretKey,
            XCosSecurityToken: credentials.sessionToken,
            StartTime: data.startTime, // 时间戳，单位秒，如：1580000000，建议返回服务器时间作为签名的开始时间，避免用户浏览器本地时间偏差过大导致签名错误
            ExpiredTime: data.expiredTime, // 时间戳，单位秒，如：1580000900
          });
        };
        xhr.send();
      },
    });
    this.cos = cos;
  },
}
```

#### 上传文件

上传文件这里使用比较简单的 type 为 file 类型的 input 来进行上传，change 事件触发时，请求 cos 上传接口上传文件

![8-0.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ef3f7a059c194a92a66948f8c7007b2f~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.png#?w=228&h=56&s=6212&e=jpg&b=fdfcfc)

```html
<div id="app">
  <!-- 文件上传 -->
  <input id="file-selector" type="file" @change="fileChange" />
</div>
```

文件改变后调用 this.cos.uploadFile 上传，其中

1. onTaskReady 中可以获取到任务 id，可以暂停、继续、取消上传；onProgress 会不断触发上传进度信息，通过解析 progressData 可以在前端实时显示信息
2. 上传成功后，第二个参数回调函数中会返回文件的访问 url（Location 字段）
3. **这里的 Key （文件名）参数非常重要，是断点续传的关键**，如果 key 不一样，会当成两个文件，不会续传，key 一样，如果发现有碎片且文件内容一致，会自动续传

```js
fileChange(e) {
  var file = e.target.files[0];
  console.log(file);
  if (!file) return;
  // 上传文件
  this.cos.uploadFile(
    {
      Bucket: this.Bucket,
      Region: this.Region,
      Key: file.name,
      Body: file,
      SliceSize: 1024 * 1024, // 大于1mb才进行分块上传
      onTaskReady: (tid) => {
        this.progressInfo.taskId = tid;
        Object.assign(this.progressInfo, {
          taskId: tid,
          status: "uploading",
        });
      },
      onProgress: (progressData) => {
        console.log("上传中", JSON.stringify(progressData));
        let text = cosUploadUtils.getProgressText(progressData);
        Object.assign(this.progressInfo, {
          percent: Math.floor(progressData.percent * 100),
          text,
          name: file.name,
        });
      },
    },
    (err, data) => {
      console.log(err, data); 
      // 上传成功
      if (!err) {
        let { statusCode, Location, ETag, RequestId } = data;
        this.successList.push({
          name: this.progressInfo.name,
          url: Location,
        });
        // 初始化进度信息
        Object.assign(this.progressInfo, {
          percent: 0,
          text: "",
          name: "",
          taskId: "",
          status: "",
          url: "",
        });
      }
    }
  );
},
```

#### 进度信息

上面 onProgress 信息，接收到的内容如下图

![8-1.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0f2972972ab74d9b85c3d87d775e2c21~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.png#?w=680&h=225&s=111053&e=jpg&b=fefdfd)

需要转换成比较友好的提示信息

![9.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/055dba9b78574496bdfe1b5806571f44~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.png#?w=668&h=147&s=25128&e=jpg&b=fffefe)

这里我们使用 `cosUploadUtils.getProgressText()` 方法将内容做了简单的转换

```js
转换前
{"loaded":768212992,"total":2164933411,"speed":3502673.33,"percent":0.35}
转换后
上传进度: <span>768.2MB/2164.9MB</span> 当前速度: <span>3.5MB/s</span> 剩余时间: <span>6分39秒</span>
```

- 上传进度：已上传大小/总大小，byte 转 KB/MB
- 当前上传速度
- 计算剩余时间，显示分钟、秒

```js
/**
 * @description 根据进度信息生成提示文本
 * @param {*} progressInfo 进度信息  {"loaded":0,"total":302313472,"speed":0,"percent":0.12}
 * @returns 上传进度: 710.9MB/2164.9MB 当前速度: 2.9MB/s 剩余时间: 8分14秒
 */
function getProgressText(progressInfo, isPercent = false) {
  console.log("开始计算进度信息");
  const { loaded, total, speed, percent } = progressInfo;

  // 计算上传进度
  const getSizeInfo = (byteNumber) => {
    if (Number.isNaN(byteNumber)) {
      return "-";
    }
    const kbNumber = Math.round(byteNumber / 1000);
    return kbNumber < 1000
      ? `${kbNumber}KB`
      : `${(kbNumber / 1000).toFixed(1)}MB`;
  };
  const getLoadedInfo = () => {
    if (isPercent) {
      return `${Math.floor((percent || 0) * 100)}%`; // 12%
    } else {
      return `${getSizeInfo(loaded)}/${getSizeInfo(total)}`; // 123M/1024M
    }
  };

  // 当前速度
  const getSpeedInfo = () => {
    const speedKb = Math.round(speed / 1000);
    if (Number.isNaN(speedKb)) {
      return "-";
    }
    return speedKb > 500
      ? `${(speedKb / 1000).toFixed(1)}MB/s`
      : `${speedKb}KB/s`;
  };

  // 计算剩余时间
  const getRestTime = () => {
    const restSeconds = Math.round((total - loaded) / (speed || "-"));
    if (Number.isNaN(restSeconds)) {
      return "-";
    }
    const minutes = `${Math.floor(restSeconds / 60)}分${restSeconds % 60}秒`;
    return restSeconds > 60 ? minutes : `${restSeconds}秒`;
  };

  let info = "";

  try {
    info = `上传进度: <span>${getLoadedInfo()}</span> 当前速度: <span>${getSpeedInfo()}</span> 剩余时间: <span>${getRestTime()}</span>`;
  } catch (e) {
    console.error(e);
  }
  console.log("进度信息计算完成", info);
  return info;
}
```

#### 暂停继续取消

暂停、和继续上传分别使用 this.cos.restartTask / this.cos.pauseTask

> 这里需要注意：如果是 x 掉了文件，再重新选择文件继续上传，不能使用 cos.pausTask, 需要先 this.cos.cancelTask 取消任务，再继续。防止同一个文件 key 生成多个文件碎片，不能续传的问题

![10.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3bb930634502487e8c4f5324bba4c98b~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.png#?w=729&h=144&s=26229&e=jpg&b=fefdfd)

```js
// 开始或暂停 cos 下载任务
toggleUpload() {
    let { status, taskId } = this.progressInfo;
    if (status === "uploading") {
      // 暂停
      this.cos.pauseTask(taskId);
      this.progressInfo.status = "pause";
    } else if (status === "pause") {
      // 继续
      this.cos.restartTask(taskId);
      this.progressInfo.status = "uploading";
    }
},
// 组件卸载时时取消上传
beforeUnmount() {
    // 取消上传
    let { taskId } = this.progressInfo;
    this.cos.cancelTask(taskId);
},
```

### 完整代码

#### cos/index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>cos大文件断点续传</title>
    <script src="/cos/cos-js-sdk-v5.min.js"></script>
    <script src="/cos/cosUploadUtils.js"></script>
    <!-- Import style -->
    <link
      rel="stylesheet"
      href="//cdn.jsdelivr.net/npm/element-plus/dist/index.css"
    />
    <!-- Import Vue 3 -->
    <script src="//cdn.jsdelivr.net/npm/vue@3"></script>
    <!-- Import component library -->
    <script src="//cdn.jsdelivr.net/npm/element-plus"></script>
    <style>
      .progress {
        max-width: 680px;
        padding: 15px;
        margin: 16px 0;
        border: 1px solid #ccc;
        border-radius: 5px;
      }
      .progress-info {
        margin: 10px 0;
      }
      .progress-text span {
        font-weight: bold;
      }
    </style>
  </head>
  <body>
    <div id="app">
      <h3>前端JS调用腾讯COS接口实现大文件断点续传</h3>

      <!-- 文件上传 -->
      <input id="file-selector" type="file" @change="fileChange" />

      <!-- 已上传文件列表 -->
      <div v-if="successList.length">
        <pre>
            已上传：
            {{successList}}
        </pre>
      </div>

      <!-- 上传进度信息 -->
      <div class="progress" v-if="progressInfo.status">
        <div class="progress-info">
          {{progressInfo.name}}
          <el-progress :percentage="progressInfo.percent" />
          <el-button @click="toggleUpload" size="small">
            {{ progressInfo.status === 'pause' ? '继续上传' : '暂停'}}
          </el-button>
        </div>
        <div v-html="progressInfo.text" class="progress-text"></div>
      </div>
    </div>
    <script>
      // 在 #app 标签下渲染一个按钮组件
      const app = Vue.createApp({
        data() {
          return {
            cos: undefined, // 腾讯云 cos 操作实例
            Bucket: "testfunc-1322206765", // 存储桶名称，由bucketname-appid 组成，appid必须填入，可以在COS控制台查看存储桶名称。 https://console.cloud.tencent.com/cos5/bucket
            Region: "ap-guangzhou", // 存储桶Region可以在COS控制台指定存储桶的概览页查看 https://console.cloud.tencent.com/cos5/bucket/
            successList: [],
            progressInfo: {
              percent: 0, // 进度百分比
              text: "", // 进度信息文本
              name: "", // 正在上传中的文件名称
              taskId: "", // 上传任务 id，用于暂停/继续
              status: "", // 上传状态，用于暂停、继续按钮显示
            },
          };
        },
        async mounted() {
          this.getCosInstance();
        },
        methods: {
          // 初始化 cos 实例/鉴权
          getCosInstance() {
            var cos = new COS({
              getAuthorization: function (options, callback) {
                var url = "http://127.0.0.1:3000/sts"; // 这里替换成您的服务接口地址
                var xhr = new XMLHttpRequest();
                xhr.open("GET", url, true);
                xhr.onload = function (e) {
                  try {
                    var data = JSON.parse(e.target.responseText);
                    var credentials = data.credentials;
                  } catch (e) {}
                  if (!data || !credentials)
                    return console.error("credentials invalid");
                  callback({
                    TmpSecretId: credentials.tmpSecretId,
                    TmpSecretKey: credentials.tmpSecretKey,
                    XCosSecurityToken: credentials.sessionToken,
                    StartTime: data.startTime, // 时间戳，单位秒，如：1580000000，建议返回服务器时间作为签名的开始时间，避免用户浏览器本地时间偏差过大导致签名错误
                    ExpiredTime: data.expiredTime, // 时间戳，单位秒，如：1580000900
                  });
                };
                xhr.send();
              },
            });
            this.cos = cos;
          },
          // 开始或暂停 cos 下载任务
          toggleUpload() {
            let { status, taskId } = this.progressInfo;
            if (status === "uploading") {
              // 暂停
              this.cos.pauseTask(taskId);
              this.progressInfo.status = "pause";
            } else if (status === "pause") {
              // 继续
              this.cos.restartTask(taskId);
              this.progressInfo.status = "uploading";
            }
          },
          // 组件卸载时时取消上传
          beforeUnmount() {
            // 取消上传
            let { taskId } = this.progressInfo;
            this.cos.cancelTask(taskId);
          },
          fileChange(e) {
            var file = e.target.files[0];
            console.log(file);
            if (!file) return;
            // 上传文件
            this.cos.uploadFile(
              {
                Bucket: this.Bucket,
                Region: this.Region,
                Key: file.name,
                Body: file,
                SliceSize: 1024 * 1024, // 大于1mb才进行分块上传
                onTaskReady: (tid) => {
                  this.progressInfo.taskId = tid;
                  Object.assign(this.progressInfo, {
                    taskId: tid,
                    status: "uploading",
                  });
                },
                onProgress: (progressData) => {
                  console.log("上传中", JSON.stringify(progressData));
                  let text = cosUploadUtils.getProgressText(progressData);
                  Object.assign(this.progressInfo, {
                    percent: Math.floor(progressData.percent * 100),
                    text,
                    name: file.name,
                  });
                },
              },
              (err, data) => {
                console.log(err, data); //
                if (!err) {
                  let { statusCode, Location, ETag, RequestId } = data;
                  this.successList.push({
                    name: this.progressInfo.name,
                    url: Location,
                  });
                  // 初始化进度信息
                  Object.assign(this.progressInfo, {
                    percent: 0,
                    text: "",
                    name: "",
                    taskId: "",
                    status: "",
                    url: "",
                  });
                }
              }
            );
            // cos.pauseTask(taskId);
          },
        },
      });
      app.use(ElementPlus);
      app.mount("#app");
    </script>
  </body>
</html>
```

#### cos/cosUploadUtils.js

```js
const cosUploadUtils = {
  getProgressText,
};

/**
 * @description 根据进度信息生成提示文本
 * @param {*} progressInfo 进度信息  {"loaded":0,"total":302313472,"speed":0,"percent":0.12}
 * @returns
 */
function getProgressText(progressInfo, isPercent = false) {
  console.log("开始计算进度信息");
  const { loaded, total, speed, percent } = progressInfo;

  // 计算上传进度
  const getSizeInfo = (byteNumber) => {
    if (Number.isNaN(byteNumber)) {
      return "-";
    }
    const kbNumber = Math.round(byteNumber / 1000);
    return kbNumber < 1000
      ? `${kbNumber}KB`
      : `${(kbNumber / 1000).toFixed(1)}MB`;
  };
  const getLoadedInfo = () => {
    if (isPercent) {
      return `${Math.floor((percent || 0) * 100)}%`; // 12%
    } else {
      return `${getSizeInfo(loaded)}/${getSizeInfo(total)}`; // 123M/1024M
    }
  };

  // 当前速度
  const getSpeedInfo = () => {
    const speedKb = Math.round(speed / 1000);
    if (Number.isNaN(speedKb)) {
      return "-";
    }
    return speedKb > 500
      ? `${(speedKb / 1000).toFixed(1)}MB/s`
      : `${speedKb}KB/s`;
  };

  // 计算剩余时间
  const getRestTime = () => {
    const restSeconds = Math.round((total - loaded) / (speed || "-"));
    if (Number.isNaN(restSeconds)) {
      return "-";
    }
    const minutes = `${Math.floor(restSeconds / 60)}分${restSeconds % 60}秒`;
    return restSeconds > 60 ? minutes : `${restSeconds}秒`;
  };

  let info = "";

  try {
    info = `上传进度: <span>${getLoadedInfo()}</span> 当前速度: <span>${getSpeedInfo()}</span> 剩余时间: <span>${getRestTime()}</span>`;
  } catch (e) {
    console.error(e);
  }
  console.log("进度信息计算完成", info);
  return info;
}
```

## 一些问题

### 无感知断点续传与有感知的用户交互取舍

在上面的方案中，基本没有做啥特殊的断点续传处理，但它确实是无感知的断点续传

如下图，我们上传了一半，刷新页面，再次选择相同文件上传，会从 800M 左右的地方继续上传。腾讯云 cos 提供的 jssdk + 碎片机制自带断点续传功能

![无感知断点续传动图.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ac1fc9ef17fb47a9ba020d6bc2bf8245~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.gif#?w=771&h=342&s=123531&e=gif&f=58&b=fcfbfb)

这种是无感知的，另外还有这一种有感知的，就是

1. 存储上传了一半的进度信息到数据库
2. 下次进来查看是否有上传一半的，点击继续，可以继续上传

但这里有个问题，**这种继续上传，还是要重新选择文件**，要增加一个交互，并且还需存做上传了一半的交互信息展示比较麻烦，建议使用无感知方式

### 文件名一致会覆盖之前的文件问题

上面的例子中，我们直接用文件名作为 key（文件唯一标识，存储桶存放路径），是为了方便看效果

真实的场景，需要在文件名后面拼接 时间戳 + 随机数，防止文件 key 一致覆盖之前已经上传好的文件内容。

这样又会衍生出一个文件，就是文件 key 每次都不一样，怎么断点续传？

这里我们的解决方法是

1. 将历史上传中的文件 key 存到 localStorage，用文件 md5 标识
2. 如果下次选择文件，获取 md5，去 localStorage 里面找，有没有 key，有就直接使用这个相同的 key，让他断点续传

### 页面异常退出拦截

虽然上面做了断点续传，但如果用户在上传过程中，刷新了页面，还是需要在离开前给一个提示比较好

这里使用系统自带的拦截方法，这种方法只会在有表单操作或有交互的时候，浏览器会默认拦截，但不能保证 100% 拦截，有些场景不会拦截：比如页面没有交互，页面内部跳转（比如点击了页面中的某个菜单）

```js
async mounted() {
  window.addEventListener("beforeunload", (event) => {
    event.preventDefault();
    event.returnValue = true;
  });
},
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d7dca0a340484ee093fccd6aece8e200~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.png#?w=675&h=270&s=25672&e=png&b=252525)

### iOS 手机端相同视频每次 md5 不一致问题

上面我们提到为了防止文件覆盖问题，用了文件时间戳 +md5 来判断是否是同一文件，再使用相同的 key 来保证断点续传。

但 iOS 选择视频上传时，每次都会压缩视频，且相同的视频每次的 md5 还不一样，这样每次都会被当成新文件，无法使用断点续传，暂时无解。

## 总结

以上，我们先是介绍了为什么需要做断点续传的功能，然后又分析了常规的技术实现方案以及其缺陷。再引出我们这次的主题：前端直接使用 js 将文件传到 cos 对象存储。

后面又从 0 到 1 实现了一个断点续传的 demo，列举了一些可能会存在的问题，希望对大家有帮助，Thanks!
