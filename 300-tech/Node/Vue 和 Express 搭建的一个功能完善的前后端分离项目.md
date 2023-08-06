---
title: Vue 和 Express 搭建的一个功能完善的前后端分离项目
date: 2023-08-04 16:56
updated: 2023-08-04 16:59
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - Vue 和 Express 搭建的一个功能完善的前后端分离项目
rating: 1
tags:
  - Vue
  - Node
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

## 前言

对于已有前端开发经验的朋友来说，深入学习 node.js 并且掌握它是一个必要的过程；对于即将毕业的软件开发学生来说，创造拥有自己的开源小产品是对大学最好的总结。

那么想和大家分享一个运用 Vue 框架写界面 +Express 框架写后端接口的这样一个开箱即用的前后端分离的项目，能更好更快的帮助大家学习 node+vue，并且也可在项目开发中使用，减轻大家的工作负担。

### 整理已久的文档和项目，还望大家手动点赞鼓励哈~~

### 因我之前是用博客记录自己的成长，我的博客地址为：[Miss\_hhl的博客](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2FMiss_hhl "https://blog.csdn.net/Miss_hhl")，汇总了我的所有博客，也欢迎大家的关注

### 本项目 github 地址为：[project项目](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fhhl-web%2Fproject "https://github.com/hhl-web/project")

## 一、项目结构

### 1.1、项目运行环境

node 版本：10.16.3

express 版本：4.16.1

npm 版本：6.9.0

vue 版本： ^2.6.10

@vue/cli 版本：^3.4.0

### 1.2、后端 Express 框架的项目目录

express-demo 文件夹下的目录如下：

```

|  app.js     express 入口
│  package-lock.json
│  package.json       项目依赖包配置
│
├─bin
│      www
|
|——common            公共方法目录
|   |——corsRequest.js
|   |——errorLog.js
|   |——loginFilter.js
|   |——request.log
│
|
|——config             配置文件
|   |——index.js
|     
|——log                日志文件夹
|  
├─public              静态文件目录
│  |——images
│  |——javascripts
│  └─stylesheets
│
├─routes               url路由配置
│      
│—sql                 数据库基本配置目录
|   |——db.js
|__
```

### 1.3、前端 Vue 框架的项目目

vue-demo 文件夹下的目录如下：

```
|——src                         源码目录
|  |——api                      请求文件
|  |——components               vue公共组件
|  |——pages                    页面组件文件
|     |——login                 登录页面
|  |——default.vue              默认组件展示区
|  |——router                   vue的路由管理文件
|  |——store                    vue的状态管理文件
|  |——App.vue                  页面入口文件
|  |—— main.js                 程序入口文件，加载各种公共组件
|
|—— public                     静态文件，比如一些图片，json数据等
|  |—— favicon.ico             图标文件
|  |—— index.html              入口页面
|—— vue.config.js              是一个可选的配置文件，包含了大部分的vue项目配置
|—— .babel.config.js           ES6语法编译配置
|—— .gitignore                 git上传需要忽略的文件格式
|—— README.md                  项目说明
|—— package.json               项目基本信息,包依赖信息等
```

## 二、如何在编辑器使用该项目

### 2.1、后端 Express 框架的使用

（1）安装插件，在安装插件之前要进入 express-demo 目录，运行命令行:

```
npm install
```

（2）开发环境启动，进入 express-demo 目录，运行以下命令：

```
npm start
```

（3）生产或测试环境，可在 config 文件夹配置路径，运行以下命令：

```
npm test   //测试环境
npm build  //生产环境
```

### 2.2、前端 Vue 框架的使用

（1）安装插件，在安装插件之前要进入 vue-demo 目录，运行命令行:

```
npm install
```

（2）开发环境启动，进入 vue-demo 目录，运行以下命令：

```
npm run serve
```

## 三、项目提供的功能点梳理

### 3.1、后端 Express 框架的功能点

#### 3.1.1、开发环境解决跨域配置

在 common 文件夹的 corsRequest.js 对跨域进行了配置，并且将它在 app.js 文件进行引用，这样客户端例如：单页面应用开发、移动应用等， 就可以跨域访问服务端对应的接口。

corsRequest.js 模块的代码如下：

```js
function corsRequest(app){
app.all("*",(req,res,next)=>{
    //设置允许跨域的域名，*代表允许任意域名跨域
    res.header("Access-Control-Allow-Origin",'http://localhost:8081');
    //允许的header类型
    // res.header("Access-Control-Allow-Headers","X-Requested-With,Content-Type,content-type");
res.header('Access-Control-Allow-Headers:Origin,X-Requested-With,Authorization,Content-Type,Accept,Z-Key')
    //跨域允许的请求方式 
    res.header("Access-Control-Allow-Methods","DELETE,PUT,POST,GET,OPTIONS");
res.header("X-Powered-By","3.2.1");
res.header("Content-Type","application/json;charset=utf-8");
res.header("Access-Control-Allow-Credentials",true);
res.header("Cache-Control","no-store");
    if (req.method.toLowerCase() == 'options'){
res.send(200);  //让options尝试请求快速结束
}else{
next();
}
})
}
module.exports=corsRequest;
```

之后再 app.js 引用该模块,配置如下：

```js
let corsRequest=require('./common/corsRequest.js');
//跨域配置
corsRequest(app);
```

#### 3.1.2、保存用户信息——种 session 存 cookie

客户端通过登录请求成功之后，服务端保存用户信息，并且将信息通过 cookie 返给客户端这整个过程我把它认为是种 session 存 cookie；利用的是 cookie-parser 中间件和 express-session 中间件，如何使用它可详细看 [cookie-parser和express-session中间件使用](https://juejin.cn/post/6854573217093255182 "https://juejin.cn/post/6854573217093255182")，那么在该项目中的配置如下：

```js
//种session存cookie配置
app.use(cookieParser('123456'));
app.use(session({
    secret: '123456',
    resave: false,
    saveUninitialized: true
}));
```

#### 3.1.3、结合数据库 mysql

##### 安装配置 mysql

这里不具体介绍 mysql 的安装配置，具体操作可以参照配置文档，建议安装一个 mysql 数据库管理工具 **navicat for mysql** ，平时用来查看数据库数据增删改查的情况；

##### 数据库配置

在 sql/db.js 文件中配置 mysql 的基本信息，相关配置项如下，可以对应更改配置项，改成你自己的配置即可：

```js
//在sql目录下的db.js文件
let host=option.host||'127.0.0.1';     //数据库所在的服务器的ip地址
let user=option.user||'root';          //用户名
let password=option.password||'1qaz@WSX';     //密码
let database=option.database ||null           //你的数据库名
let db=mysql.createConnection({       
    host:host,       
    user:user,       
    password:password,       
    database:database 
});     //创建连接
```

#### 3.1.4、静态文件目录配置

```js
app.use(express.static(path.join(__dirname, 'public')));
```

#### 3.1.5、登录拦截器

客户端在每次请求时，服务端处理每个请求之前需要对用户身份进行校验，这样使得每个接口更具有安全性，在 common 文件夹下的 loginFilter.js 的相关代码逻辑如下：

```js
function loginFilter(app){
    app.use(function (req,res,next){
    if(!(req.session.auth_username && req.session.auth_password)){
    if(req.signedCookies.username && req.signedCookies.password){
    let {username,password}=req.signedCookies;
    req.session.auth_username=username;
    req.session.auth_password=password;//将cookie的值存在session里
    next();
    }else{
    let arr=req.url.split('/');
    let index=arr && arr.findIndex((item)=>{
    return (item.indexOf('login')!=-1 || item.indexOf('relogin')!=-1);
    });
    if(index!==-1){
    next();
    }else{
    return res.status(401).json({
    msg: '没有登入，请先登入'
     })
    }
    }
    }else{
    next();
    }
    })
}
module.exports=loginFilter;
```

之后在 app.js 进行引用该模块，配置如下：

```js
//登录拦截器
loginFilter(app);
```

#### 3.1.6、请求路由配置处理

在项目中按模块划分请求处理，我们在 routes 目录下分别新建每个模块路由配置，例如用户模块，则为 user.js 文件，我们在 app.js 主入口文件中引入每个模块的路由，以用户模块进行举例，相关代码逻辑如下：

```js
let user=require('./routes/user');
//路由配置
app.use('/api/user',user);
```

#### 3.1.7、日志处理

（1）通过 morgan 记录接口请求日志 morgan 是 express 默认的日志中间件，也可脱离 express，作为 node.js 的日志组件单独使用。详细学习可看 GitHub 库上的 morgan 模块。

项目在 app.js 文件中进行了以下配置：

```js
// 输出日志到目录
let accessLogStream = fs.createWriteStream(path.join(__dirname,'./log/request.log'), {flags:'a',encoding: 'utf8' }); 
app.use(logger('combined', { stream: accessLogStream }))
```

（2）通过 winston 记录错误日志 morgan 只能记录 http 请求的日志，所以还需要 winston 来记录其它想记录的日志，例如：访问数据库出错等。winston 中的每一个 logger 实例在不同的日志级别可以存在多个传输配置。详细学习可看 GitHub 库上的 winston 模块。

在项目中，在 common/errorLog.js 文件中进行了以下配置：

```js
const { createLogger, format, transports } = require('winston');
const { combine, timestamp, printf } = format;
const path = require('path');
const myFormat = printf(({ level, message, label, timestamp }) => {
  return `${timestamp}-${level}- ${message}`;
});
const option={
    file:{
        level:'error',
        filename: path.join(__dirname, '../log/error.log'),
        handleExceptions:true,
        json:true,
        maxsize:5242880,
        maxFiles:5,
        colorize:true,
    },
    console:{
        level:'debug',
        handleExceptions:true,
        json:false,
        colorize:true,
    }
}
const logger=createLogger({
    format: combine(
        timestamp(),
        myFormat
    ),
    transports:[
        new transports.File(option.file),
        new transports.Console(option.console),
    ],
    exitOnError:false,
});
logger.stream={
    write:function(message,encoding){
        logger.error(message)
    }
}
module.exports=logger;
```

之后在 app.js 使用 logger 模块，代码配置如下：

```js
let winston=require('./common/errorLog.js');
// error handler
app.use(function(err, req, res, next) {  
// set locals, only providing error in development 
res.locals.message = err.message;  
res.locals.error = req.app.get('env')=== 'development' ? err : {};  
winston.error(`${err.status || 500} - ${err.message} - ${req.originalUrl} -${req.method} - ${req.ip}`);  
// render the error page  
res.status(err.status || 500);  res.render('error');});
```

#### 3.1.8、文件上传处理

在项目中，使用 multiparty 模块实现文件的上传功能。包括如何将数据返回给前端，前端可通过 url 将图片显示。multiparty 模块具体用法可详细阅读 github，这里就不详细介绍。在文件夹 routes 的 upload.js 是实现文件上传的功能。

#### 3.1.9、文件操作处理

node 提供的内置文件操作模块 fs，fs 的功能强大。包括读写文件/文件夹，以及如何读流写流。在使用过程中可具体参考 node 官网，便于快速上手。

#### 3.1.10、环境变量统一配置

环境变量的统一管理配置可以更方便的维护项目，在 package.json 文件通过命令行给全局变量 process.env 添加新的属性,从而可配置项目环境变量：

```js
let obj=Object.create(null);
if (process.env.NODE_ENV='development'){
obj.baseUrl="http://localhost:3000";
} else if(process.env.NODE_ENV = 'test') {
console.log('当前是测试环境')
}else if(process.env.NODE_ENV='production'){
console.log('当前是生产环境');
}
module.exports=obj;
```

#### 3.1.11、自动更新

当启动项目时，更改项目中的任何一个文件代码，运用 nodemon 模块它将会自动帮你重新编译代码，节省了我们开发的时间。这个模块只要在 package.json 这个文件做一下配置即可。

### 3.2、前端 Vue 框架的功能点

#### 3.2.1、配置页面路由

vue 项目中，通过 vue-router 实现页面之间的跳转，也可以通过路由进行参数的收发，vue-router 的相关代码配置如下：

```js
import Router from 'vue-router';
import Vue from 'vue';
Vue.use(Router);
export default new Router({
mode:'hash',
routes:[
{
path:'/',
redirect:'/login',
component:()=>import('@/pages/login/index.vue')
},
]
})
```

#### 3.2.2、配置公共仓库

vue 项目中，通过 vuex 实现状态的统一管理，也可运用于多层嵌套组件的通信，以及如何持久化保存 state 状态值。vuex 的相关代码配置如下：

```js
import Vuex from 'vuex';
import Vue from 'vue';
import * as mutaionsType from './mutations-TYPE';
Vue.use(Vuex);
let persits=(store)=>{
let state;
if(state=sessionStorage.getItem('vuex-state')) store.replaceState(JSON.parse(state));
store.subscribe((mutations,state)=>{
sessionStorage.setItem('vuex-state',JSON.stringify(state));
})
}
export default new Vuex.Store({
    plugins:[persits],
state:{
cancelArray:[],//存放axios取消函数容器
},
mutations:{
[mutaionsType.clear_cancel]:(state,payload)=>{
state.cancelArray.forEach(fn=>{
fn.call(fn);
})
state.cancelArray=[];
},
[mutaionsType.filter_cancel]:(state,payload)=>{
let arr=state.cancelArray.filter(item=>!(item.url.includes(payload)));
state.cancelArray=[...arr];
},
[mutaionsType.push_cancel]:(state,payload)=>{
state.cancelArray.push(payload);
},
},
actions:{},
modules:{}
})
```

#### 3.2.3、二次封装 axios

通过运用 axios 来请求接口，在项目当中对 axios 进行了二次封装，主要的功能包括：实现 aixos 请求拦截器和响应拦截，请求前后 loading 的开启和关闭，利用发布订阅实现取消请求函数的配置。相关代码如下：

```js
import { baseURL } from './config.js'
import axios from 'axios'
import store from '@/store'
import { Loading, Message } from 'element-ui'
//每请求一次创建一个唯一的axios
class AjaxFetch {
  constructor() {
    this.config = {
      withCredentials: true,//跨域凭证
      responseType: 'json',
      baseURL: baseURL,
      timeout: 3000,
    }
    this.queue = {}
  }
  request(option) {
    //创建一个axios实例
    let config = {
      ...this.config,
      ...option,
    }
    let instance = axios.create()
    this.interceptors(instance,config.url)
    return instance(config)
  }
  interceptors(instance,url) {
    instance.interceptors.request.use(
      (config) => {
        let CancelToken = axios.CancelToken
        //设置取消函数
        config.cancelToken = new CancelToken((c) => {
          //c是一个函数
          store.commit('push_cancel', { fn: c, url:url }) //存放取消的函数实例
        })
        if (Object.keys(this.queue).length == 0) {
          this._loading = Loading.service({
            lock: true,
            text: 'Loading',
            spinner: 'el-icon-loading',
            background: 'rgba(0, 0, 0, 0.7)',
          })
        }
        this.queue[url] = url;
        return config;
      },
      (err) => {
        return Promise.reject(err)
      }
    )
    instance.interceptors.response.use(
      (response) => {
        let {data} = response;
        store.commit('filter_cancel',url) //存放取消的函数实例
        delete this.queue[url]
        if (Object.keys(this.queue).length == 0) {
          this._loading.close()
        }
        switch (data.code) {
          case 500:
            Message({
              type: 'error',
              message: data.msg,
            })
            break;
          case 401:
            Message({
              type: 'warning',
              message: data.msg,
            })
            break;
        }
        return data;
      },
      (err) => {
        delete this.queue[url];
        if (Object.keys(this.queue).length == 0) {
          this._loading.close();
        }
        return Promise.reject(err)
      }
    )
  }
}
export default new AjaxFetch()
```

#### 3.2.4、配置第三方库

在 vue 项目中，我们想直接通过 this 使用第三方库，可将第三方库直接挂载到 Vue 类的原型（prototype）上。如果是 vue 生态圈的模块，则直接通过选项注册在根组件上。

#### 3.2.5、使用路由守卫

在 vue 项目中，使用路由的前置守卫实现的功能主要包括：登录授权，切换页面将发布 axios 取消函数。当然你还可以编写更多的前置守卫业务代码。如下是 hook.js 的代码：

```js
import store from '@/store'
export default {
  permitterRouter: function(to, from, next) {
    let { username } = store.state;
    let flag=Object.keys(username).length;      //判断是否登录过的标识
    if(!flag){
        if(to.path.includes('/login')){
            next();
        }else{
            next('/login');
        }
    }else{
        if(to.path.includes('/login')){
            next('/home');
        }else{
            next();
        }
    }
  },
  cancelAjax: (to, from, next) => {
    store.commit('clear_cancel')
    next()
  },
}
```

那么写好 hook.js 在文件夹 route 下的 index.js 进行配置如下：

```js
//路由前置守卫
Object.values(hookRouter).forEach(hook=>{
    //使用bind可在hook函数获取this=>router
router.beforeEach(hook.bind(router))
})
```

## 四、总结

本文介绍了开源的前后端分离项目（开箱即用），完善了前后端的各种功能。希望能通过这篇文档对开源代码进行更直接的介绍，帮助使用者减轻工作量，更高效完成工作，有更多时间提升自己的能力。 辛苦整理了好久，还望手动点赞鼓励小琳同学~~

### 博客地址为：[Miss\_hhl的博客](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2FMiss_hhl "https://blog.csdn.net/Miss_hhl")，汇总了我所有的博客，欢迎大家关注~~

### 本项目 github 地址为：[project项目](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fhhl-web%2Fproject "https://github.com/hhl-web/project"),还望大家点个 star 鼓励~~（项目会持续更新迭代哦~希望大家持续关注哈）
