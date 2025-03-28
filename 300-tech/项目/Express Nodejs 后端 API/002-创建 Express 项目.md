---
title: 002-创建 Express 项目
date: 2025-03-13T10:18:13+08:00
updated: 2025-03-13T11:14:36+08:00
dg-publish: false
---

Node.js 常用后端框架有 Express 和 Koa，由于 Express 出现更早一点，而且使用的人数较多、周下载量也比 Koa 多，所以直接选用 Express

- <https://www.npmjs.com/package/express>
- <https://www.npmjs.com/package/koa>

开发项目的时候一般不会从零开始搭建，可以直接使用脚手架 express-generator 生成项目

<https://www.npmjs.com/package/express-generator>

## 全局安装脚手架

```sh
npm install -g express-generator
```

接下来就可以使用 `express` 命令创建项目了

可用命令：

```sh
express -h
    --version        output the version number
-e, --ejs            add ejs engine support
    --pug            add pug engine support
    --hbs            add handlebars engine support
-H, --hogan          add hogan.js engine support
-v, --view <engine>  add view <engine> support (dust|ejs|hbs|hjs|jade|pug|twig|vash) (defaults to jade)
    --no-view        use static html instead of view engine
-c, --css <engine>   add stylesheet <engine> support (less|stylus|compass|sass) (defaults to plain css)
    --git            add .gitignore
-f, --force          force on non-empty directory
-h, --help           output usage information
```

## 创建项目

由于是纯 API 后端项目，所以不需要视图

```sh
express --no-view <项目名>
```

例如

```sh
express --no-view express-api
cd express-api
npm install
npm start
```

访问 <http://localhost:3000/> 就可以看到欢迎页面

## 返回 JSON

项目是不需要返回 HTML，直接返回 JSON 格式即可

修改文件 `routes/index.js`，把 `res.render()` 改成 `res.json()`

```js
var express = require('express');
var router = express.Router();

/* GET home page. */
router.get('/', function(req, res, next) {
  res.json({ message: 'Hello Express!' });
});

module.exports = router;
```

删除 `public` 目录

<kbd>Ctrl</kbd> + <kbd>c</kbd> 中断之前的命令，重新 `npm start` 启动项目，访问 <http://localhost:3000/>，页面显示

```json
{"message":"Hello Express!"}
```

## 安装 nodemon 热重载

由于 node 命令在修改代码之后并不会自动监听到修改，自动重启，需要每次手动中断运行，所以可以下载 nondemon

<https://www.npmjs.com/package/nodemon>

```sh
npm install --save-dev nodemon
# 或
pnpm add nodemon -D
```

修改 `package.json`，start 命令中 `node` 修改成 `nodemon`

```json
{
  "name": "express-api",
  "version": "0.0.0",
  "private": true,
  "scripts": {
    "start": "nodemon ./bin/www"
  },
  "dependencies": {
    "cookie-parser": "~1.4.4",
    "debug": "~2.6.9",
    "express": "~4.16.1",
    "morgan": "~1.9.1"
  },
  "devDependencies": {
    "nodemon": "3.1.9"
  }
}
```

现在重新运行 `npm start`，修改文件之后就只需要刷新页面就行了

## 项目结构

```sh
.
├── bin
│   └── www            # 启动项目的脚本
├── node_modules       # 项目依赖包
├── public             # 静态资源
│   ├── images
│   ├── index.html
│   ├── javascripts
│   └── stylesheets
│       └── style.css
├── .nvmrc             # nvm 配置文件
├── app.js             # 入口文件
├── routes             # 路由
│   ├── index.js
│   └── users.js
├── package.json       # 项目基本信息和依赖包信息
└── pnpm-lock.yaml     # 锁定安装依赖包时的版本号

12 directories, 8 files
```

其中之后需要重点修改的文件就是 `routes` 目录中的文件和 `app.js`

## 路由

查看 `routes/index.js`

使用 ES6 之后的语法，直接修改 `routes` 中文件和 `app.js`，查找替换 `var` 为 `const`

```js
const express = require('express');
const router = express.Router();  // 路由器

/*
** Get 请求，路径，要运行的函数
-- req 是 request 的缩写，通过这个参数可以获取用户通过 URL 或表单传递给我们的数据，例如 req.params、req.query、req.body
-- res 是 rsponse 的缩写，对用户的请求做出响应
-- next 在当前函数中对请求进行一些处理，再传递给下一个函数
*/
router.get('/', function(req, res, next) {
  res.json({ message: 'Hello Express!' });
});

// 导出
module.exports = router;
```
