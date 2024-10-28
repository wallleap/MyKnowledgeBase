---
title: 一个开箱即用、功能完善的 Express 项目
date: 2023-08-04T04:49:00+08:00
updated: 2024-08-21T10:32:30+08:00
dg-publish: false
aliases:
  - 一个开箱即用、功能完善的 Express 项目
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
  - Node
  - web
url: //myblog.wallleap.cn/post/1
---

## 前言

node.js 对前端来说无疑具有里程碑意义，在其越来越流行的今天，掌握 node.js 已经不仅仅是加分项，而是前端攻城师们必须要掌握的技能。而 express 以其快速、开放、极简的特性， 成为 node.js 最流行的框架，所以使用 express 进行 web 服务端的开发是个不错且可信赖的选择。但是 express 初始化后，并不马上就是一个开箱即用，各种功能完善的 web 服务端项目，例如：日志记录、错误捕获、mysql 连接、token 认证、websocket 等一系列常见的功能，需要开发者自己去安装插件进行配置完善功能，如果你对 web 服务端开发或者 express 框架不熟悉，那将是一项耗费巨大资源的工作。本文在 express 的初始架构上配置了日志记录、错误捕获、mysql 连接、token 认证、websocket 等一系列常见的功能，并且本文的项目已经在 github 开源，提供给大家学习、实际项目使用，希望能减轻大家的工作量，更高效完成工作，有更多时间提升自己的能力。

**辛苦整理良久，还望手动点赞鼓励~**

**博客 github 地址为**：[github.com/fengshi123/…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Ffengshi123%2Fblog "https://github.com/fengshi123/blog") **，汇总了作者的所有博客，也欢迎关注及 star ~**

**本项目 github 地址为**：[github.com/fengshi123/…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Ffengshi123%2Fexpress_project "https://github.com/fengshi123/express_project")

## 一、项目结构

### 1.1、基础环境

这个 express 服务端项目，相关运行环境配置如下：

| 工具名称 | 版本号 |
| --- | --- |
| node.js | 11.12.0 |
| npm | 6.7.0 |
| express | 4.16.0 |
| mysql | 5.7 |

### 1.2、运行项目

（1）安装插件，进入到 express\_project 目录，运行以下命令：

```sh
npm install
```

（2）开发环境启动，进入到 express\_project 目录，运行以下命令：

```sh
npm run dev
```

（3）正式环境启动，进入到 express\_project 目录，运行以下命令:

```sh
npm run start
```

### 1.3、项目目录

项目目录结构如下：

```
├─ bin                数据库初始化脚本
│  ├─ db      项目数据库安装，其中 setup.sh 为命令入口，初始化数据库
├─ common      共用方法、常量目录
├─ conf      数据库基本配置目录
├─ dao      代码主逻辑目录
├─ log                日志目录
├─ public      静态文件目录
├─ routes      url 路由配置
├─ .eslintrc.js      eslint 配置
├─ app.js      express 入口
├─ package.json      项目依赖包配置
├─ README.md      项目说明

```

## 二、常用功能介绍

### 2.1、结合 mysql

#### 2.1.1、安装配置 mysql

这里就不介绍 mysql 的安装配置，具体操作可以参照 [配置文档](https://link.juejin.cn/?target=)，建议安装一个 mysql 数据库管理工具 `navicat for mysql` ，平时用来查看数据库数据增删改查的情况；

#### 2.1.2、数据库初始化

我们在 bin/db 目录底下，已经配置编写好了数据库初始化脚本，其中 setup.sh 为编写好的命令入口，用于连接数据库、新建数据库等，主要逻辑已经进行注释如下：

```js
mysql -uroot -p123456 --default-character-set=utf8 <<EOF // 需要将帐号密码改成自己的
drop database if exists research; // 删除数据库
create database research character set utf8; // 新建数据库
use research; // 切换到 research 数据库，对应改成自己的
source init.sql; // 初始化 sql 建表 
EOF
cmd /k
```

我们可以在 init.sql 中插入每张表的初始数据，相关代码如下，至于表数据如何插入等简单逻辑这里就不再介绍，读者可以查看相关 sql 文件即会了解。

```sql
source t_user.sql;
source t_exam.sql;
source t_video.sql;
source t_app_version.sql;
source t_system.sql;
```

#### 2.1.3、数据库配置

我们在 conf/db.js 文件中配置 mysql 的基本信息，相关配置项如下，可以对应更改配置项，改成你自己的配置。

```js
module.exports = {
  mysql: {
    host: '127.0.0.1',
    user: 'root',
    password: '123456',
    database: 'research',
    port: 3306
  }
};
```

#### 2.1.4、数据库连接及操作

对数据进行基本的增、删、改、查操作，可以通过已经配置好的文件以及逻辑进行照搬，我们以增加一个用户举例进行简单介绍，在 dao/userDao.js 文件中：

```js
var conf = require('../conf/db'); // 引入数据库配置
var pool = mysql.createPool(conf.mysql); // 使用连接池

  add: function (req, res, next) {
    pool.getConnection(function (err, connection) {
      if (err) {
        logger.error(err);
        return;
      }
      var param = req.body;
      // 建立连接，向表中插入值
      connection.query(sql.insert, [param.uid, param.name, param.password, param.role,param.sex], function (err, result) {
        if (err) {
          logger.error(err);
        } else {
          result = {
            code: 0,
            msg: '增加成功'
          };
        }
        // 以json形式，把操作结果返回给前台页面
        common.jsonWrite(res, result);
        // 释放连接
        connection.release();
      });
    });
  },
```

### 2.2、日志功能

#### 2.2.1、morgan 记录请求日志

morgan 是 express 默认的日志中间件，也可以脱离 express，作为 node.js 的日志组件单独使用，morgan 的具体 api 可参考 [morgan 的 github 库](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fexpressjs%2Fmorgan "https://github.com/expressjs/morgan")；这里主要介绍我们项目中，帮你进行了哪些配置、实现了哪些功能。项目在 app.js 文件中进行了以下配置：

```js
const logger = require('morgan');
// 输出日志到目录
var accessLogStream = fs.createWriteStream(path.join(__dirname, '/log/request.log'), { flags: 'a', encoding: 'utf8' }); 
app.use(logger('combined', { stream: accessLogStream }));
```

我们已经配置好以上请求日志记录，这样每次 http 请求都会记录到 log/request.log 文件中。

#### 2.2.2、winston 记录错误日志

由于 morgan 只能记录 http 请求的日志，所以我们还需要 winston 来记录其它想记录的日志，例如：访问数据库出错等；Winston 是 node.js 上最流行的日志库之一。它被设计为一个简单通用的日志库，支持多种传输（一种传输实际上就是一种存储设备，例如日志存储在哪里）。winston 中的每一个 logger 实例在不同的日志级别可以存在多个传输配置；当然它也可以记录请求记录。winston 的具体 api 可参考 [winston 的 github 库](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fwinstonjs%2Fwinston "https://github.com/winstonjs/winston") ，我们这里不做详细的介绍。

这里主要介绍我们项目中，我们使用 winston 帮你进行了哪些配置、实现了哪些功能。项目在 common/logger.js 文件中进行了以下配置：

```js
var { createLogger, format, transports } = require('winston');
var { combine, timestamp, printf } = format;
var path = require('path');

var myFormat = printf(({ level, message, label, timestamp }) => {
  return `${timestamp} ${level}: ${message}`;
});

var logger = createLogger({
  level: 'error',
  format: combine(
    timestamp(),
    myFormat
  ),
  transports: [
    new (transports.Console)(),
    new (transports.File)({
      filename: path.join(__dirname, '../log/error.log')
    })
  ]
});

module.exports = logger;
```

通过以上 logger.js 文件配置，我们只需要在需要的地方引入该文件，然后调用 logger.error(err) 进行相应等级的日志记录，相关的错误日志就会记录到文件 log/error.log 中。

### 2.3、请求路由处理

我们在项目中按模块划分请求处理，我们在 routes 目录下分别新建每个模块路由配置，例如用户模块，则为 user.js 文件，我们在 app.js 主入口文件中引入每个模块的路由，以用户模块进行举例，相关代码逻辑如下：

```js
// 引入用户模块路由配置
var usersRouter = require('./routes/users');
// 使用用户模块的路由配置
app.use('/users', usersRouter);
```

其中 routes/users.js 文件中的路由配置如下：

```js
var express = require('express');
var router = express.Router();

// 增加用户
router.post('/addUser', function (req, res, next) {
  userDao.add(req, res, next);
});
// 获取全部用户
router.get('/queryAll', function (req, res, next) {
  userDao.queryAll(req, res, next);
});
// 删除用户
router.post('/deleteUser', function (req, res, next) {
  userDao.delete(req, res, next);
});
...
```

通过以上配置，这样客户端通过 /users 开头的请求路径，都会跳转到用户模块路由中进行处理，假设请求路径为 /users/addUser 则会进行增加用户的逻辑处理。通过这种规范和配置，这时如果增加一个新的模块，只需按照原有的规范，新增一个模块路由文件，进行相关的路由配置处理，不会影响到原有的模块路由配置，也不会出现路由冲突等。

### 2.4、token 认证

express-jwt 是 node.js 的一个中间件，他来验证指定 http 请求的 JsonWebTokens 的有效性，如果有效就将 jsonWebTokens 的值设置到 req.user 里面，然后路由到相应的 router。 此模块允许您使用 node.js 应用程序中的 JWT 令牌来验证 HTTP 请求。express-jwt 的具体 api 可参考 [express-jwt 的 github 库](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fauth0%2Fexpress-jwt "https://github.com/auth0/express-jwt") ，我们这里不做详细的介绍。

这里主要介绍我们项目中，我们使用 express-jwt 帮你进行了哪些配置、实现了哪些功能。项目在 app.js 文件中进行了以下 token 拦截配置，如果没有经过 token 认证的，会返回客户端 401 认证失败；

```js
var expressJWT = require('express-jwt');
// token 设置
app.use(expressJWT({
  secret: CONSTANT.SECRET_KEY
}).unless({
  // 除了以下这些 URL，其他的URL都需要验证
  path: ['/getToken',
    '/getToken/adminLogin',
    '/appVersion/upload',
    '/appVersion/download',
    /^\/public\/.*/,
    /^\/static\/.*/,
    /^\/user_disk\/.*/,
    /^\/user_video\/.*/
  ]
}));
```

我们可以选择在生成 token 时，将对应登录的用户 id 注入 token 中，如文件 dao/tokenDao.js 文件中所配置的；

```js
// uid 注入 token 中
ret = {
  code: 0,
  data: {
token: jsonWebToken.sign({
  uid: obj.uid
}, CONSTANT.SECRET_KEY, {
  expiresIn: 60 * 60 * 24
}),
  }
};
```

后续我们可以在请求中，通过请求信息 req.body.uid，获取得到先前注入 token 里面的用户 id 信息。

### 2.5、跨域配置

我们已经在项目的 app.js 中进行项目的跨域配置，这样一些客户端例如：单页面应用开发、移动应用等， 就可以跨域访问服务端对应的接口。我们在 app.js 文件中进行相关跨域配置：

```js
app.all('*', function (req, res, next) {
  res.header('Access-Control-Allow-Origin', '*');
  res.header('Access-Control-Allow-Headers', '*');
  res.header('Access-Control-Allow-Methods', '*');
  next();
});
```

### 2.6、静态目录配置

我们在项目中配置了静态目录，用于提供静态资源文件（图片、css 文件、js 文件等）的服务；传递一个包含静态资源的目录给 express.static 中间件用于提供静态资源。如下提供 public 目录下的图片、css 文件和 javascript 文件，当然你也可以根据自己的需要，通过类似配置，创建自己静态资源目录。

```js
app.use('/', express.static(path.join(__dirname, 'public')));
```

### 2.7、异常错误处理

如果我们如果没有对服务端程序进行处理，当服务端代码抛出异常时，会导致 node 进程退出，从而用户无法正常访问服务器，造成严重问题。我们在项目中配置使用 domain 模块，捕获 服务端程序中中抛出的异常；domain 主要的 API 有 domain.run 和 error 事件。简单的说，通过 domain.run 执行的函数中引发的异常都可以通过 domain 的 error 事件捕获。我们在 项目中配置使用 domain 的代码如下：

```js
var domain = require('domain');
// 处理没有捕获的异常，导致 node 退出
app.use(function (req, res, next) {  
  var reqDomain = domain.create();  
  reqDomain.on('error', function (err) {   
     res.status(err.status || 500);    
     res.render('error'); 
  }); 
  reqDomain.run(next);
});
```

### 2.8、自动重启

每次修改 js 文件，我们都需要重启服务器，这样修改的内容才会生效，但是每次重启比较麻烦，影响开发效果；所以我们在开发环境中引入 nodemon 插件，实现实时热更新，自动重启项目。所以如 1.3 章节所述，我们在开发环境中启动项目应该使用 npm run dev 命令，因为我们在 package.json 文件中配置了以下命令：

```json
  "scripts": {
    "dev": "nodemon ./app",
  },
```

### 2.9、pm2 使用

pm2 是 node 进程管理工具，可以利用它来简化很多 node 应用管理的繁琐任务，如性能监控、自动重启、负载均衡等，而且使用非常简单。 所以我们可以使用 pm2 来启动我们的服务端程序。这里我们不详细介绍 pm2，如果还不了解的同学可以查看 [这篇文档](https://link.juejin.cn/?target=https%3A%2F%2Fwww.cnblogs.com%2Fchyingp%2Fp%2Fpm2-documentation.html "https://www.cnblogs.com/chyingp/p/pm2-documentation.html")。

### 2.10、文件上传处理

服务端应用程序不可避免要处理文件上传的操作，我们在项目中为大家配置好 multiparty 插件，并提供相关的上传、重命名等操作，相关代码逻辑及注释在 dao/common.js 中。如果对 multiparty 插件还不了解的同学，可以参考 [multiparty 插件的 github 库](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fpillarjs%2Fmultiparty "https://github.com/pillarjs/multiparty") ，我们这里不做详细的介绍。

```js
var upload = function (path, req, res, next) {
  return new Promise(function (resolve, reject) {
    // 解析一个文件上传
    var form = new multiparty.Form();
    // 设置编辑
    form.encoding = 'utf-8';
    // 设置文件存储路径
    form.uploadDir = path;
    // 设置单文件大小限制
    form.maxFilesSize = 2000 * 1024 * 1024;
    var textObj = {};
    var imgObj = {};
    form.parse(req, function (err, fields, files) {
      if (err) {
        console.log(err);
      }
      Object.keys(fields).forEach(function (name) { // 文本
        textObj[name] = fields[name];
      });
      Object.keys(files).forEach(function (name) {
        if (files[name] && files[name][0] && files[name][0].originalFilename) {
          imgObj[name] = files[name];
          var newPath = unescape(path + '/' + files[name][0].originalFilename);
          var num = 1;
          var suffix = newPath.split('.').pop();
          var lastIndex = newPath.lastIndexOf('.');
          var tmpname = newPath.substring(0, lastIndex);
          while (fs.existsSync(newPath)) {
            newPath = tmpname + '_' + num + '.' + suffix;
            num++;
          }
          fs.renameSync(files[name][0].path, newPath);
          imgObj[name][0].path = newPath;
        }
      });
      resolve([imgObj, textObj])
    });
  });
};
```

### 2.11、文件/目录操作

服务端程序对服务器目录、文件进行操作，是频率比较高的操作，相对于 node.js 默认的 fs 文件操作模块，我们在项目中配置了 fs-extra 插件来对服务器目录、文件进行操作，fs-extra 模块是系统 fs 模块的扩展，提供了更多便利的 API，并继承了 fs 模块的 API ；例如我们在项目中使用 mkdir 方法来创建文件目录、ensureDir 方法来确认目录是否存在、remove 方法来删除文件、copy 方法来复制文件等，你可以在 dao/filesDao.js 文件中看到 fs-extra 丰富的操作方法。如果你还未接触过 fs-extra 插件，建议你可以查看 [它的 github 库](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fjprichardson%2Fnode-fs-extra "https://github.com/jprichardson/node-fs-extra")，有相关详细介绍。

### 2.12、配置 eslint 代码检查

为了让项目的代码风格保持良好且一致，故我们在项目中添加 eslint 来检查 js 代码规范；你可以在 .eslintignore 文件中配置哪些文件你不想通过 eslint 进行代码检查；你还可以在 .eslintrc.js 文件中配置你们团队的代码风格。

## 三、总结

本文介绍了作者开源的一个开箱即用，各种功能完善的 web 服务端项目的相关功能，例如：mysql 结合、日志记录、错误捕获、token 认证、跨域配置、自动重启、文件上传处理、eslint 配置 等一系列常见的功能，希望能通过源码开源和文章的相关介绍，帮助读者减轻工作量，更高效完成工作，有更多时间提升自己的能力。

**辛苦整理良久，还望手动点赞鼓励~**

**博客 github 地址为**：[github.com/fengshi123/…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Ffengshi123%2Fblog "https://github.com/fengshi123/blog") **，汇总了作者的所有博客，也欢迎关注及 star ~**

**本项目 github 地址为**：[github.com/fengshi123/…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Ffengshi123%2Fexpress_project "https://github.com/fengshi123/express_project")
