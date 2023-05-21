# web前端知识体系

# 前言

我最近几年一直关注“知识体系”这个话题。经历了很久的积累的总结，前段时间刚刚写完这份知识体系目录。但后面还会再继续补充，也能会做出调整。因为内容较多，我就做了一个开源的网站 [what-is-fe.gitee.io](https://what-is-fe.gitee.io/) ，读者可以去那里查阅。

# 什么是知识体系

- 完善的知识范围，包含了[前端工程师](https://class.imooc.com/sale/fe2020)常用的所有知识点
- 合理的结构化，便于理解和记忆

# 初衷

Web 前端知识体系很重要，而当今网络上又没有一个靠谱的，只能自己去写。一为自己总结输出，二为分享帮助他人。

- 你并不是学不动了，而是学太多迷路了 —— 知识体系就是一张地图
- 你是否能看到当前的热门技术/框架，而看不到整体？—— 知识体系给你上帝视角
  - 找准知识体系
  - 刻意训练
  - 及时反馈

# 目录

以下是大纲目录，查看详情可点击每个的链接。还未写的部分，会继续更新，请戳 [**what-is-fe.gitee.io**](https://what-is-fe.gitee.io/)

## 相关科目

- 数学思维
  - 抽象
  - 建模
  - 流程严谨、死板无情
- 英语
  - 国外问答网站（如 stackoverflow.com ）
  - github
  - 英文文档
  - 报错信息

## 编程基础

- 计算机基础
  - [计算机组成原理](https://what-is-fe.gitee.io/01-编程基础/01-计算机基础/01-计算机组成原理.html)
  - [操作系统](https://what-is-fe.gitee.io/01-编程基础/01-计算机基础/02-操作系统.html)
  - [汇编语言](https://what-is-fe.gitee.io/01-编程基础/01-计算机基础/03-汇编语言.html)
  - [编译原理](https://what-is-fe.gitee.io/01-编程基础/01-计算机基础/04-编译原理.html)
  - 计算机网络
  - 编码
- 数据结构
  - 结构化与非结构化
  - 逻辑结构
  - 线性结构（数组、链表、栈、队列）
  - 树
  - 优先级队列
  - 词典
  - 图
- 算法
  - 复杂度
  - 算法思想（二分、贪心、动态规划）
  - 递归思想（爆栈）
  - 常用算法（查找、排序、树遍历、最短路径）
  - leetcode 刷题
- 编程模式
  - 面向对象 OOP
    - 类和对象
    - 三要素（继承、封装、多态）
    - UML 类图
  - 面向切面编程 AOP
  - 函数式编程 FP
    - 高级函数
    - 纯函数
    - curry
    - compose
- 设计
  - SOLID 设计原则
  - 常见设计模式
    - 工厂模式
    - 单例模式
    - 观察者
    - 发布订阅
    - 代理
    - 装饰器，等等……
  - MVC
  - MVVM

## 语法和 API

- 业界标准

  - [W3C](http://www.w3.org/TR/)
  - ECMA-262

- XML

- HTML

  - 基础标签
  - 媒体标签
  - 图形标签
    - SVG
    - Canvas

- CSS

  - 理解“流” - CSS 的设计核心
  - 基础的图文样式
  - 布局
    - `inline` `block` 和 `inline-block`
    - 盒模型
    - margin 相关
    - flex
    - float 布局
    - BFC
  - 定位
    - relative
    - absolute
    - fixed
    - 定位上下文
  - 响应式
    - 关于 viewport
    - rem
    - vw vh
  - css3
    - 渐变
    - 动画
  - 字体 iconfont
  - 模块化
    - BEM
    - css-in-js ，等等……

- ES 语法

  - 变量，表达式，函数等基础语法
  - 作用域
    - 自由变量
    - 闭包
    - this
  - class 和继承
  - 原型
  - 异步编程
    - promise
    - async/await
    - 宏任务和微任务
    - event loop 模型
  - Map 和 Set
  - Proxy 和 Reflect
  - 装饰器 Decorator
  - 常用 API
    - 字符串
    - 数组
    - 对象
    - 正则表达式
    - 日期处理
  - 异常处理
  - 模块化
    - ES6 Module（可对比 AMD、CMD、CommonJS）

- Typescript

   

  语法

  - 类型
    - 变量类型
    - 函数参数类型
    - 函数返回值类型
  - 自定义类型
  - 接口
  - 泛型
  - 枚举
  - 函数重载
  - 命名空间

- JS Web API

  - DOM 操作
    - DOM 结构
    - DOM 操作
    - DOM 操作的性能考虑
  - DOM 事件
    - 事件绑定
    - 冒泡模型
    - 事件代理（委托）
  - BOM 操作
    - window
    - location（获取 url）
    - navigator（获取 UA）
    - history（前端路由）
    - screen
  - 通讯
    - postMessage
    - BroadcastChannel
  - ajax
    - XMLHttpRequest
    - Fetch
    - 浏览器同源策略
    - 跨域请求
      - cors
      - jsonp
    - cookie
  - 存储
    - localStorage
    - sessionStorage
    - indexedDB
  - webworker

- JSON 格式

- WebAssembly

- Web RTC

## 网络和通讯

- TCP/IP（了解即可）
  - 什么是 IP
  - 7 层模型
  - 3 次握手
- url 格式
- DNS 解析（域名到 IP）
- http
  - method（Restful API）
  - 状态码
  - Request Header
  - Response Header（其中会有 gzip 压缩）
  - cookie
  - 缓存策略
    - 强制缓存
    - 协商缓存
- https
  - 如何加密传输？
  - 购买证书
- websocket
- 登录方案
  - session
  - jwt
- 鉴权方案
  - oauth2
  - sso
- 文件上传
- 相关术语
  - PV
  - UV
  - QPS ，等等……
- CDN（专门提供静态文件服务，需要知道）

## 开发流程

- 代码版本管理 git
  - 常用命令
  - 多人协作开发
  - github gitlab coding.net 等
- 软件包管理
  - npm
  - package.json
  - yarn
- 技术基建
  - UI 组件库，业务组件库
  - 公共 SDK
- mock
  - charles mock
  - mock.js
  - 在线 mock server
- 调试和抓包
  - debugger
  - charles 和 fiddler
    - 抓包
    - 配置代理（如微信能力的测试，需要线上域名）
  - chrome 开发者工具
  - 深入：内存泄漏的排查
- 单元测试
  - jest 等工具
  - Vue React 框架的测试工具
- CI/CD
  - 概念
  - github actions
  - travis ，等等
- linux 基本应用
  - ssh
  - ssh key 信任
  - 常用命令
  - scp 远程拷贝
  - vim 编辑器
- 文档
  - wiki 平台
  - 在线 office / 知识库
  - markdown 格式
- 研发流程
  - 需求评审和 UI 设计
  - 技术方案设计
  - 开发过程：编码、调试、单元测试、自测、写注释和文档、代码走查等
  - 联调
  - 提测
  - 上线
    - 全量
    - 小流量（灰度）
    - ABTest
  - DevOps 概念

## 前端工程化

前端工程化，也是开发流程的一部分。内容较多，因此单独拿出来讲解。

- 规范化
  - eslint 编码规范（结合 pre-commit）
  - git 规范：branch 命名规范，commit 规范
  - jsdoc 注释规范
- 模块化
  - 代码分模块组织（依赖于 js css 模块化语法和 webpack 等工具的支持）
  - 组件化
- 自动化
  - 脚手架（如 create-react-app vue-cli）
    - yeoman
  - 监听文件变化，自动构建
  - CI/CD 自动单元测试，自动提测、上线等
  - 自动兼容（如 postcss babel）
- webpack
  - 基础配置
  - 常用 loader
  - 常用 plugin
  - 代码拆分、公共代码抽离（产出多 chunk）
  - 性能优化
- babel
  - babel-polyfill
  - babel-runtime
  - corejs
- rollup

## 运行和监控

- 浏览器和 webview
- 页面加载和渲染：从输入 url 到页面显示的整个过程
  - 加载过程
  - 渲染过程
  - 重绘/重排
- js 运行机制
  - js 引擎（ 如 V8 ）
  - 内存机制
  - 垃圾回收机制
- 性能优化
  - 优化加载速度
  - 优化渲染速度
  - 优化操作体验（如节流、防抖）
  - Chrome Performance API 性能监控
- 安全
  - XSS
  - CSRF
  - DDOS 攻击
  - 密码加密存储
- 监控和统计
  - 错误监控、报警
  - 性能监控
  - 统计上报（小流量统计）
  - APM 监控平台
- 客户端能力
  - jsbridge（如微信 jssdk）
  - js 调起 app

## 前端框架

- 框架基础
  - SPA 和 MPA （单页应用和多页应用）
  - 前端路由
  - MVVM
  - 组件化
  - 虚拟 DOM 和 diff 算法
- Vue
  - vue-cli
  - 使用
    - 组件
    - 模板
    - 指令
    - 组件属性
    - 组件生命周期
    - 高级使用（如动态组件、异步组件、slot 等等）
    - 性能优化
  - 原理
    - 响应式（ Object.defineProperty ）
    - 模板编译
    - 虚拟 DOM
  - 周边
    - vuex
    - vue-router
    - UI 框架
      - elementUI
      - view-design（之前叫 iview）
      - ant-design-vue
  - vue3（新版本，尚未发布）
- React
  - create-react-app
  - 使用
    - JSX 语法
    - 组件和属性
    - state 和 setState
    - 组件生命周期
    - 高级使用（如 Context、高阶组件、render-prop 等）
    - 性能优化（如 shouldComponentUpdate PureComponent ）
  - 原理
    - JSX 语法糖本质
    - 合成事件机制
    - batchUpdate 机制
    - 事物机制
    - 组件渲染流程
    - fiber
  - 周边
    - redux
    - mobX
    - react-router
    - UI 框架
      - ant-design
  - Hooks
- Angular

## 常用工具和插件

- lodash - 工具函数
- echarts - 统计图表
- jquery 或 zepto - DOM 操作
- axios - ajax
- date-fns 或 moment - 日期时间格式
- css reset 库
- caniuse.com 浏览器兼容性
- （待继续补充……）

## 技术广度

- 服务端
  - nodejs
    - 基本 API（如 http、fs 等）
    - commonjs 模块化
    - 框架
      - express
      - koa
        - 中间件
        - 洋葱圈模型
      - egg
    - 调试
  - SSR
    - 服务端模板，如 ejs artTemplate 等
    - nuxt.js （Vue SSR）
    - next.js（React SSR）
  - 常用软件
    - nginx（反向代理、负载均衡）
    - 数据库
      - redis
      - mysql
      - mongodb
    - docker
    - 日志分析
  - serverless
  - Deno
- 小程序
- PWA
- 跨端（如 RN Weex）
- 客户端 electron

## 其他重要的事儿

- 持续学习（有输入有输出）
- 定期反思总结，低头做事抬头看路
- 时间管理，todo 管理


作者：双越
链接：http://www.imooc.com/article/details/id/309313
来源：慕课网
本文原创发布于慕课网 ，转载请注明出处，谢谢合作