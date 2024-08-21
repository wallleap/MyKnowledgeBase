---
title: 一篇拒绝低级封装 axios 的文章
date: 2023-08-04T05:36:00+08:00
updated: 2024-08-21T10:32:44+08:00
dg-publish: false
aliases:
  - 一篇拒绝低级封装 axios 的文章
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
  - JS
  - web
url: //myblog.wallleap.cn/post/1
---

## 为什么要写这篇文章

***本文正在参加 [「金石计划 . 瓜分 6 万现金大奖」](https://juejin.cn/post/7162096952883019783 "https://juejin.cn/post/7162096952883019783")***

事前提醒：阅读本文存在不同想法时，可以在评论中表达，但请勿使用过激的措辞。

目前掘金上已经有很多关于 `axios` 封装的文章。自己在初次阅读这些文章中，见识到很多封装思路，但在付诸实践时一直有疑问：**这些看似高级的二次封装，是否会把 `axios` 的调用方式弄得更加复杂？** 优秀的二次封装，有以下特点：

1. **能改善原生框架上的不足**：明确原生框架的缺点，且在二次封装后能彻底杜绝这些缺点，与此同时不会引入新的缺点。
2. **保持原有的功能**：当进行二次封装时，新框架的 API 可能会更改原生框架的 API 的调用方式（例如传参方式），但我们要保证能通过新 API 调用原生 API 上的所有功能。
3. **理解成本低**：有原生框架使用经验的开发者在面对二次封装的框架和 API 时能迅速理解且上手。

但目前我见过，或者我接收过的项目里众多的 `axios` 二次封装中，并不具备上述原则，我们接下盘点一些常见的低级的二次封装的手法。

## 盘点那些低级的 `axios` 二次封装方式

## 1\. 对特定 method 封装成新的 API，却暴露极少的参数

例如以下代码：

```js
export const post = (url, data, params) => {
  return new Promise((resolve) => {
    axios
      .post(url, data, { params })
      .then((result) => {
        resolve([null, result.data]);
      })
      .catch((err) => {
        resolve([err, undefined]);
      });
  });
};
```

上面的代码中对 `method` 为 `post` 的请求方法进行封装，用于解决原生 API 中在处理报错时需要用 `try~catch` 包裹。但这种封装有一个缺点：整个 `post` 方法只暴露了 `url`,`data`,`params` 三个参数，通常这三个参数可以满足大多数简单请求。但是，如果我们遇到一个特殊的 `post` 接口，它的响应时间较慢，需要设置较长的超时时间，那上面的 `post` 方法就立马嗝屁了。

此时用原生的 `axios.post` 方法可以轻松搞定上述特殊场景，如下所示：

```js
// 针对此次请求把超时时间设置为15s
axios.post("/submit", form, { timeout: 15000 });
```

类似的特殊场景还有很多，例如：

1. 需要上传表单，表单中不仅含数据还有文件，那只能设置 `headers["Content-Type"]` 为 `"multipart/form-data"` 进行请求，如果要显示上传文件的进度条，则还要设置 `onUploadProgress` 属性。
2. 存在需要防止数据竞态的接口，那只能设置 `cancelToken` 或 `signal`。有人说可以在通过拦截器 `interceptors` 统一处理以避免竞态并发，对此我举个用以反对的场景：如果同一个页面中有两个或多个下拉框，两个下拉框都会调用同一个接口获取下拉选项，那你这个用拦截器实现的避免数据竞态的机制就会出现问题，因为会导致这些下拉框中只有一个请求不会被中断。

有些开发者会说不会出现这种接口，已经约定好的所有 `post` 接口只需这三种参数就行。对此我想反驳：一个有潜力的项目总会不断地加入更多的需求，如果你觉得你的项目是没有潜力的，那当我没说。但如果你不敢肯定你的项目之后是否会加入更多特性，不敢保证是否会遇到这类特殊场景，那请你在二次封装时，尽可能地保持与原生 `API` 对齐，以保证**原生 `API` 中一切能做到的，二次封装后的新 `API` 也能做到**。以避免在遇到上述的特殊情况时，你只能尴尬地修改新 `API`，而且还会出现为了兼容因而改得特别难看那种写法。

## 2\. 封装创建 `axios` 实例的方法，或者封装自定义 `axios` 类

例如以下代码：

```js
// 1. 封装创建`axios`实例的方法
const createAxiosByinterceptors = (config) => {
  const instance = axios.create({
    timeout: 1000,
    withCredentials: true,
    ...config,
  });

  instance.interceptors.request.use(xxx, xxx);
  instance.interceptors.response.use(xxx, xxx);
  return instance;
};

// 2. 封装自定义`axios`类
class Request {
  instance: AxiosInstance
  interceptorsObj?: RequestInterceptors

  constructor(config: RequestConfig) {
    this.instance = axios.create(config)
    this.interceptorsObj = config.interceptors

    this.instance.interceptors.request.use(
      this.interceptorsObj?.requestInterceptors,
      this.interceptorsObj?.requestInterceptorsCatch,
    )
    this.instance.interceptors.response.use(
      this.interceptorsObj?.responseInterceptors,
      this.interceptorsObj?.responseInterceptorsCatch,
    )
  }
}
```

上面的两种写法都是用于创建多个不同配置和不同拦截器的 `axios` 实例以应付多个场景。对此我想表明自己的观点：**一个前端项目中，除非存在多个第三方服务后端需要对接（例如七牛云这类视频云服务商），否则只能存在一个 `axios` 实例**。多个 `axios` 实例会增加代码理解成本，让参与或者接手项目的开发者花更多的时间去思考和接受每个 `axios` 实例的用途和场景，就好比一个项目多个 `Vuex` 或 `Redux` 一样鸡肋。

那么有开发者会问如果有相当数量的接口需要用到不同的配置和拦截器，那要怎么办？下面我来分**多个配置**和**多个拦截器**两种场景进行分析：

### 1\. 多个配置下的处理方式

如果有两种或以上不同的配置，这些配置各被一部分接口使用。那么就应该声明对应不同配置的常量，然后在调用 `axios` 时传入对应的配置常量，如下所示：

```js
// 声明含不同配置项的常量configA和configB
const configA = {
  // ....
};

const configB = {
  // ....
};

// 在需要这些配置的接口里把对应的常量传进去
axios.get("api1", configA);
axios.get("api2", configB);
```

对比起多个不同配置的 `axios` 实例，上述的写法更加直观，能让阅读代码的人直接看出区别。

### 2\. 多个拦截器下的处理方式

如果有两种或以上不同的拦截器，这些拦截器中各被一部分接口使用。那么，**我们可以把这些拦截器都挂载到全局唯一的 `axios` 实例上**，然后通过以下两种方式来让拦截器选择性执行：

**推荐**：在 `config` 中新加一个自定义属性以决定拦截器是否执行，代码如下所示：

调用请求时，写法如下所示：

```js
instance.get("/api", {
  //新增自定义参数 enableIcp 来决定是否执行拦截器
  enableIcp: true,
});
```

在拦截器中，我们这么编写逻辑

```js
// 请求拦截器写法
instance.interceptors.request.use(
  // onFulfilled写法
  (config: RequestConfig) => {
    // 从config取出enableIcp
    const { enableIcp } = config;
    if (enableIcp) {
      //...执行逻辑
    }
    return config;
  },

  // onRejected写法
  (error) => {
    // 从error中取出config配置
    const { config } = error;
    // 从config取出enableIcp
    const { enableIcp } = config;
    if (enableIcp) {
      //...执行逻辑
    }
    return error;
  }
);

// 响应拦截器写法
instance.interceptors.response.use(
  // onFulfilled写法
  (response) => {
    // 从response中取出config配置
    const { config } = response;
    // 从config取出enableIcp
    const { enableIcp } = config;
    if (enableIcp) {
      //...执行逻辑
    }
    return response;
  },

  // onRejected写法
  (error) => {
    // 从error中取出config配置
    const { config } = error;
    // 从config取出enableIcp
    const { enableIcp } = config;
    if (enableIcp) {
      //...执行逻辑
    }
    return error;
  }
);
```

通过以上写法，我们就可以通过 `config.enableIcp` 来决定所注册拦截器的拦截器是否执行。举一反三来说，我们可以通过往 `config` 塞自定义属性，同时在编写拦截器时配合，就可以完美的控制单个或多个拦截器的执行与否。

**次要推荐**：使用 `axios` 官方提供的 `runWhen` 属性来决定拦截器是否执行，**注意该属性只能决定请求拦截器的执行与否，不能决定响应拦截器的执行与否**。用法如下所示：

```js
function onGetCall(config) {
  return config.method === "get";
}

axios.interceptors.request.use(
  function (config) {
    config.headers.test = "special get headers";
    return config;
  },
  null,
  // onGetCall 的执行结果为 false 时，表示不执行该拦截器
  { runWhen: onGetCall }
);
```

关于 `runWhen` 更多用法可看 [axios#interceptors](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Faxios%2Faxios%23interceptors "https://github.com/axios/axios#interceptors")

## 本章总结

当我们进行二次封装时，切勿为了封装而封装，首先要分析原有框架的缺点，下面我们来分析一下 `axios` 目前有什么缺点。

## 盘点 `axios` 目前的缺点

## 1\. 不能智能推导 `params`

在 `axios` 的类型文件中，`config` 变量对应的类型 `AxiosRequestConfig` 如下所示：

```js
export interface AxiosRequestConfig<D = any> {
  url?: string;
  method?: Method | string;
  baseURL?: string;
  transformRequest?: AxiosRequestTransformer | AxiosRequestTransformer[];
  transformResponse?: AxiosResponseTransformer | AxiosResponseTransformer[];
  headers?: AxiosRequestHeaders;
  params?: any;
  paramsSerializer?: (params: any) => string;
  data?: D;
  timeout?: number;
  // ...其余属性省略
}
```

可看出我们可以通过泛型定义 `data` 的类型，但 `params` 被写死成 `any` 类型因此无法定义。

## 2\. 处理错误时需要用 `try~catch`

这个应该是很多 `axios` 二次封装都会解决的问题。当请求报错时，`axios` 会直接抛出错误。需要开发者用 `try~catch` 包裹着，如下所示：

```js
try {
  const response = await axios("/get");
} catch (err) {
  // ...处理错误的逻辑
}
```

如果每次都要用 `try~catch` 代码块去包裹调用接口的代码行，会很繁琐。

## 3\. 不支持路径参数替换

目前大多数后端暴露的接口格式都遵循 `RESTful` 风格，而 `RESTful` 风格的 `url` 中需要把参数值嵌到路径中，例如存在 `RESTful` 风格的接口，`url` 为 `/api/member/{member_id}`，用于获取成员的信息，调用中我们需要把 `member_id` 替换成实际值，如下所示：

```js
axios(`/api/member/${member_id}`);
```

如果把其封装成一个请求资源方法，就要额外暴露对应路径参数的形参。非常不美观，如下所示：

```js
function getMember(member_id, config) {
  axios(`/api/member/${member_id}`, config);
}
```

___

针对上述缺点，下面我分享一下自己精简的二次封装。

## 应该如何精简地进行二次封装

在本节中我会结合 `Typescript` 来展示如何精简地进行二次封装以解决上述 `axios` 的缺点。注意在这次封装中我不会写任何涉及到业务上的场景例如**鉴权登录**和**错误码映射**。下面先展示一下二次封装后的使用方式。

***本次二次封装后的所有代码可在 [enhance-axios-frame](https://link.juejin.cn/?target=https%3A%2F%2Fgitee.com%2FHitotsubashi%2Fenhance-axios-frame "https://gitee.com/Hitotsubashi/enhance-axios-frame") 中查看。***

## 使用方式以及效果

使用方式如下所示：

```js
apis[method][url](config);
```

`method` 对应接口的请求方法；`url` 为接口路径；`config` 则是 `AxiosConfig`，也就是配置。

返回结果的数据类型为:

```js
{
  // 当请求报错时，data为null值
  data: null | T;
  // 当请求报错时，err为AxiosError类型的错误实例
  err: AxiosError | null;
  // 当请求报错时，response为null值
  response: AxiosResponse<T> | null;
}
```

下面来展示一下使用效果：

 **支持 `url` 智能推导，且根据输入的 `url` 推导出需要的 `params`、`data`。在缺写或写错请求参数时，会出现 `ts` 错误提示**

举两个接口做例子：

- 路径为 `/register`，方法为 `post`，`data` 数据类型为 `{ username: string; password: string }`
- 路径为 `/password`，方法为 `put`，`data` 数据类型为 `{ password: string }`，`params` 数据类型为 `{ username: string }`

调用效果如下所示：

![](https://cdn.wallleap.cn/img/pic/illustration/202308041736533.gif)

![](https://cdn.wallleap.cn/img/pic/illustration/202308041736534.gif)

通过这种方式，我们无需再通过一个函数来执行请求接口逻辑，而是可以直接通过调用 `api` 来执行请求接口逻辑。如下所示：

```js
// ------------ 以前的方式 -------------
// 需要用一个 registerAccount 函数来包裹着请求代码行
function register(
  data: { username: string; password: string },
  config: AxiosConfig
) {
  return instance.post("/register", data, config);
}

const App = () => {
  const registerAccount = async (username, password) => {
    const response = await register({ username, password });
    //... 响应结束后处理逻辑
  };
  return <button onClick={registerAccount}>注册账号</button>;
};

// ------------ 现在的方式 -------------
const App = () => {
  const registerAccount = async (username, password) => {
    // 直接调用apis
    const response = await apis.post["/register"]({ username, password });
    //... 响应结束后处理逻辑
  };
  return <button onClick={registerAccount}>注册账号</button>;
};

```

以往我们如果想在组件里调用一个已写在前端代码里的接口，则需要先知道接口的 `url`(如上面的 `/register`)，再去通过 `url` 在前端代码里找到该接口对应的请求函数 (如上面的 `register`)。而如果用本文这种做法，我们只需要知道 `url` 就可以。

这么做还有一个好处是防止重复记录接口。

**支持返回结果的智能推导**

举一个接口为例子：

- 路径为 `/admin`，方法为 `get`，返回结果的数据类型为 `{admins: string[]}`

调用效果如下所示：

![](https://cdn.wallleap.cn/img/pic/illustration/202308041736535.gif)

**支持错误捕捉**，无需写 `try~catch` 包裹处理

调用时写法如下所示：

```js
const getAdmins = async () => {
 const { err, data } = await apis.get['/admins']();
 // 判断如果 err 不为空，则代表请求出错
 if (err) {
   //.. 处理错误的逻辑
   // 最后return跳出，避免执行下面的逻辑
   return
 };
 // 如果 err 为空，代表请求正常，此时需要用!强制声明 data 不为 null
 setAdmins(data!.admins);
};
```

**支持路径参数**，且路径参数也是会智能推导的

举一个接口为例子：

- 路径为 `/account/{username}`，方法为 `get`，需要 `username` 路径参数

写法如下所示：

```js
const getAccount = async () => {
  const { err, data } = await apis.get["/account/{username}"]({
    // config新增args属性，且在里面定义username的值。最终url会被替换为/account/123
    args: {
      username: "123",
    },
  });
  if (err) return;
  setAccount(data);
};
```

## 实现方式

先展示二次封装后的 API 层目录

![](https://cdn.wallleap.cn/img/pic/illustration/202308041736536.png)

我们先看 `/apis.index.ts` 的代码

```ts
import deleteApis from "./apis/delete";
import get from "./apis/get";
import post from "./apis/post";
import put from "./apis/put";

// 每一个属性中会包含同名的请求方法下所有接口的请求函数
const apis = {
  get,
  post,
  put,
  delete: deleteApis,
};

export default apis;
```

逻辑上很简单，只负责导出包含所有请求的 `apis` 对象。接下来看看 `/apis/get.ts`

```ts
import makeRequest from "../request";

export default {
  "/admins": makeRequest<{ admins: string[] }>({
    url: "/admins",
  }),
  "/delay": makeRequest({
    url: "/delay",
  }),
  "/500-error": makeRequest({
    url: "/500-error",
  }),
  // makeRequest用于生成支持智能推导，路径替换，捕获错误的请求函数
  // 其形参的类型为RequestConfig，该类型在继承AxiosConfig上加了些自定义属性,例如存放路径参数的属性args
  // makeRequest带有四个可选泛型，分别为：
  //  - Payload: 用于定义响应结果的数据类型，若没有则可定义为undefined，下面的变量也一样
  //  - Data：用于定义data的数据类型
  //  - Params：用于定义parmas的数据类型
  //  - Args：用于定义存放路径参数的属性args的数据类型
  "/account/{username}": makeRequest<
    { id: string; name: string; role: string },
    undefined,
    undefined,
    { username: string }
  >({
    url: "/account/{username}",
  }),
};
```

一切的重点在于 `makeRequest`，其作用我再注释里已经说了，就不再重复了。值得一提的是，我们在调用 `apis.get['xx'](config1)` 中的 `config1` 是配置，这里生成请求函数的 `makeRequest(config2)` 的 `config2` 也是配置，这两个配置在最后会合并在一起。这么设计的好处就是，如果有一个接口需要特殊配置，例如需要更长的 `timeout`，可以直接在 `makeRequest` 这里就加上 `timeout` 属性如下所示：

```js
{
  // 这是一个耗时较长的接口
  '/longtime':  makeRequest({
    url: '/longtime',
    // 设置超时时间
    timeout: 15000
  }),
}
```

这样我们每次在开发中调用 `apis.get['/longtime']` 时就不需要再定义 `timeout` 了。

额外说一种情况，如果请求里的 `body` 需要放入 `FormData` 类型的表单数据，则可以用下面的情况处理:

```ts
export default {
  "/register": makeRequest<null, { username: string; password: string }>({
    url: "/register",
    method,
    // 把Content-Type设为multipart/form-data后，axios内部会自动把{ username: string; password: string }对象转换为待同属性的FormData类型的变量
    headers: {
      "Content-Type": "multipart/form-data",
    },
  }),
};
```

关于上述详情可看 [axios#-automatic-serialization-to-formdata](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Faxios%2Faxios%23-automatic-serialization-to-formdata "https://github.com/axios/axios#-automatic-serialization-to-formdata")。

下面来看看定义 `makeRequest` 方法的 `/api/request/index.ts` 文件：

```ts
import urlArgs from "./interceptor/url-args";

const instance = axios.create({
  timeout: 10000,
  baseURL: "/api",
});
// 通过拦截器实现路径参数替换机制，之后会放出urlArgs代码
instance.interceptors.request.use(urlArgs.request.onFulfilled, undefined);

// 定义返回结果的数据类型
export interface ResultFormat<T = any> {
  data: null | T;
  err: AxiosError | null;
  response: AxiosResponse<T> | null;
}

// 重新定义RequestConfig，在AxiosRequestConfig基础上再加args数据
export interface RequestConfig extends AxiosRequestConfig {
  args?: Record<string, any>;
}

/**
 * 允许定义四个可选的泛型参数：
 *    Payload: 用于定义响应结果的数据类型
 *    Data：用于定义data的数据类型
 *    Params：用于定义parmas的数据类型
 *    Args：用于定义存放路径参数的属性args的数据类型
 */
// 这里的定义中重点处理上述四个泛型在缺省和定义下的四种不同情况
interface MakeRequest {
  <Payload = any>(config: RequestConfig): (
    requestConfig?: Partial<RequestConfig>
  ) => Promise<ResultFormat<Payload>>;

  <Payload, Data>(config: RequestConfig): (
    requestConfig: Partial<Omit<RequestConfig, "data">> & { data: Data }
  ) => Promise<ResultFormat<Payload>>;

  <Payload, Data, Params>(config: RequestConfig): (
    requestConfig: Partial<Omit<RequestConfig, "data" | "params">> &
      (Data extends undefined ? { data?: undefined } : { data: Data }) & {
        params: Params;
      }
  ) => Promise<ResultFormat<Payload>>;

  <Payload, Data, Params, Args>(config: RequestConfig): (
    requestConfig: Partial<Omit<RequestConfig, "data" | "params" | "args">> &
      (Data extends undefined ? { data?: undefined } : { data: Data }) &
      (Params extends undefined
        ? { params?: undefined }
        : { params: Params }) & {
        args: Args;
      }
  ) => Promise<ResultFormat<Payload>>;
}

const makeRequest: MakeRequest = <T>(config: RequestConfig) => {
  return async (requestConfig?: Partial<RequestConfig>) => {
    // 合并在service中定义的config和调用时从外部传入的config
    const mergedConfig: RequestConfig = {
      ...config,
      ...requestConfig,
      headers: {
        ...config.headers,
        ...requestConfig?.headers,
      },
    };
    // 统一处理返回类型
    try {
      const response: AxiosResponse<T, RequestConfig> =
        await instance.request<T>(mergedConfig);
      const { data } = response;
      return { err: null, data, response };
    } catch (err: any) {
      return { err, data: null, response: null };
    }
  };
};

export default makeRequest;
```

上面代码中重点在于 `MakeRequest` 类型中对泛型的处理，其余逻辑都很简单。

最后展示一下支持路径参数替换的拦截器 `urlArgs` 对应的代码：

```ts
const urlArgsHandler = {
  request: {
    onFulfilled: (config: AxiosRequestConfig) => {
      const { url, args } = config as RequestConfig;
      // 检查config中是否有args属性，没有则跳过以下代码逻辑
      if (args) {
        const lostParams: string[] = [];
        // 使用String.prototype.replace和正则表达式进行匹配替换
        const replacedUrl = url!.replace(/\{([^}]+)\}/g, (res, arg: string) => {
          if (!args[arg]) {
            lostParams.push(arg);
          }
          return args[arg] as string;
        });
        // 如果url存在未替换的路径参数，则会直接报错
        if (lostParams.length) {
          return Promise.reject(new Error("在args中找不到对应的路径参数"));
        }
        return { ...config, url: replacedUrl };
      }
      return config;
    },
  },
};
```

___

已上就是整个二次封装的过程了，如果有不懂的可以直接查看 [项目 enhance-axios-frame](https://link.juejin.cn/?target=https%3A%2F%2Fgitee.com%2FHitotsubashi%2Fenhance-axios-frame "https://gitee.com/Hitotsubashi/enhance-axios-frame") 里的代码或在评论区讨论。

## 进阶

我在之前的 [文章](https://juejin.cn/post/7131759714458664990 "https://juejin.cn/post/7131759714458664990") 里有介绍如何给 `axios` 附加更多高级功能，如果有兴趣的可以点击以下链接看看：

1. [反馈请求结果](https://juejin.cn/post/7131759714458664990#heading-15 "https://juejin.cn/post/7131759714458664990#heading-15")
2. [接口限流](https://juejin.cn/post/7131759714458664990#heading-17 "https://juejin.cn/post/7131759714458664990#heading-17")
3. [数据缓存](https://juejin.cn/post/7131759714458664990#heading-20 "https://juejin.cn/post/7131759714458664990#heading-20")
4. [错误自动重试](https://juejin.cn/post/7131759714458664990#heading-21 "https://juejin.cn/post/7131759714458664990#heading-21")

## 后记

这篇文章写到这里就结束了，如果觉得有用可以点赞收藏，如果有疑问可以直接在评论留言，欢迎交流 👏👏👏。
