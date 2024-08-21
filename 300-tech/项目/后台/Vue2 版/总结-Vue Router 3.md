---
title: 总结-Vue Router 3
date: 2023-07-20T07:40:00+08:00
updated: 2024-08-21T10:33:32+08:00
dg-publish: false
aliases:
  - 总结-Vue Router 3
author: Luwang
category: web
comments: true
cover: //cdn.wallleap.cn/img/pic.jpg
description: 文章描述
image-auto-upload: true
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
rating: 10
source: #
tags:
  - 项目
url: #
---

简介：Vue.js 官方的路由管理器

功能：

- 嵌套的路由/视图表
- 模块化的、基于组件的路由配置
- 路由参数、查询、通配符
- 基于 Vue.js 过渡系统的视图过渡效果
- 细粒度的导航控制
- 带有自动激活的 CSS class 的链接
- HTML5 历史模式或 hash 模式，在 IE9 中自动降级
- 自定义的滚动条行为

仓库及示例：[vuejs/vue-router](https://github.com/vuejs/vue-router/)

## 安装

在项目中运行命令

```sh
pnpm add vue-router
```

新建 `router/index.js` 文件

```js
import Vue from 'vue'
import VueRouter from 'vue-router' // 引入

Vue.use(VueRouter) // 明确地安装路由功能

const router = new VueRouter({ 
  // 配置项
})

export default router
```

在 `main.js` 中引入

```js
import Vue from 'vue'
import router from './router'
import App from './app.vue'

new Vue({
  router,
  render: h => h(App), // App 是组件
}).$mount('#app')
```

## 简单使用

将组件 (components) **映射**到路由 (routes)，然后告诉 Vue Router 在哪里**渲染**它们

- 入口 `<router-link to="/xxx">Go xxx</router-link>`
	- 使用 `router-link` 组件来导航
	- 通过传入 `to` 属性指定跳转链接，可以是字符串、对象（命名路由、带参数等）
	- 默认会被渲染成 `a` 标签，推荐使用 router-link 而不是直接使用 a
	- `v-slot` 作用域插槽，一般自定义 NavLink 组件的使用使用
	- `replace` 是否直接替换当前路由，不会留 history 记录
	- `append` 是否在当前路径前添加基路径
	- `tag="li"` 渲染成其他标签，它还是会监听点击，触发导航
	- `active-class="active"` 自定义激活时的类名，默认值是 `"router-link-active"`
	- `exact` 是否完全匹配才激活类名
	- `event` 触发导航的事件，值为字符串或一个包含字符串的数组，默认值是 `'click'`
	- `exact-active-class` 完全匹配时激活的类名，默认值是 `"router-link-exact-active"`
	- `aria-current-value` 默认值是 `"page"`，枚举类型
- 路由出口 `<router-view></router-view>`
	- 路由匹配到的组件渲染到这里
	- 配合 `transition` 和 `keep-alive` 使用，同时出现时应该 `transition>keep-alive>router-view`
	- `name="xxx"` 设置了 `name` 之后会渲染对应的路由配置中 `components` 下面的同名组件
- 路由实例 `const router = new VueRouter({})`
	- 可以传入 `routes` 路由，每个路由需要映射一个组件
	- 配置项 `path` 必传，`component` 是引入的组件，`name` 设置了为命名路由，`components` 命名视图组件，`redirect` 重定向，`props`，`alias` 设置别名，`children` 嵌套路由，`beforeEnter` 路由内导航守卫，`meta` 路由元信息，`caseSensitive` 匹配规则是否大小写敏感，`pathToRegexOptions` 编译的正则选项
	- 还有其他的参数
	- `mode` 可选值 `hash | history | abstract`
	- `base` 应用的基路径，默认是 `"/"`
	- `linkActiveClass` 设置全局默认激活类名
	- `linkExactActiveClass` 设置全局默认的全局匹配激活类名
	- `scrollBehavior` 滚动行为
	- `parseQuery/stringifyQuery` 提供自定义查询字符串的解析/反解析函数
	- `fallback` 在不支持 `history.pushState` 的浏览器下是否回退到 `hash` 模式，默认为 `true`
	- 实例属性
	- `router.app` 配置了 `router` 的 Vue 根实例
	- `router.mode` 路由使用的模式
	- `router.currentRoute` 当前路由对应的路由信息对象
	- `router.START_LOCATION` 路由开始的地方（初始导航）
	- 实例方法
	- `router.beforeEach`、`router.beforeResolve`（前两个必须调用 `next()`）、`router.afterEach`
	- `router.push`、`router.replace`（前两个既支持 Promise 写法，也支持后两个参数回调）、`router.go`、`router.back`、`router.forward`
	- `router.getMatchedComponents` 返回目标位置或当前路由匹配的组件数组
	- `router.resolve` 解析目标位置
	- `router.addRoute` 添加一条新的路由规则，如果多传一个参数，则添加一条路由规则记录作为现有路由的子路由，第一个参数为字符串
	- `router.getRoutes` 获取所有活跃的路由记录列表
	- `router.onReady`、`router.onError`
- 在 `main.js` 中创建和挂载根实例，这样全局都可以使用路由功能
	- 在任何组件中可以通过 `this.$router` 访问路由器（路由实例）
	- 在组件中通过路由对象 `this.$route` 访问当前路由（在 watch `$route` 的回调内、`router.match(location)`、导航守卫|`scrollBehavior` 的参数 `to` 和 `from` 都是路由对象）
	- 路由对象的属性
	- `$route.path` 当前路由的路径（绝对路径）
	- `$route.params` 路由参数，没有则是空对象
	- `$route.query` 查询参数，没有则是空对象
	- `$route.hash` 当前路由的 hash 值，没有则是空字符串
	- `$route.fullPath` 包含查询参数和 hash 的完整路径
	- `$route.matched` 包含当前路由的所有嵌套路径片段的路由记录，是个数组
	- `$route.name` 当前路由的名称
	- `$route.redirectedForm` 如果存在重定向，则为重定向来源的路由的名字
	- 组件中还可以使用的配置选项
	- 组件内守卫 `beforeRouteEnter`、`beforeRouteUpdate`、`beforeRouteLeave`
- `router-link` 对应的路由匹配成功，会自动加上 class 属性值 `.router-link-active`
	- `router-link-exact-active`：当路由链接的路径与当前路由的路径完全匹配时，会添加 `router-link-exact-active` 类名。这意味着只有当路由链接的路径与当前路由的路径完全相同时，才会应用该类名。
	- `router-link-active`：当路由链接的路径是当前路由路径的子集时，会添加 `router-link-active` 类名。这意味着当路由链接的路径是当前路由路径的一部分时，都会应用该类名。

例如：

- `main.js` 中（除了上面那种拆分成模块，也可以直接在 `main.js` 中写）

```js
import Vue from 'vue'
// 0. 如果使用模块化机制编程，导入Vue和VueRouter，要调用 Vue.use(VueRouter)
import VueRouter from 'vue-router'
Vue.use(VueRouter)

// 1. 定义 (路由) 组件
const Foo = { template: '<div>foo</div>' }
const Bar = { template: '<div>bar</div>' }
// 可以从其他文件 import 进来，或者动态导入（项目中下面这条应该和其他 import 放一起）
// import Home from './views/home/index.vue'

// 2. 定义路由
const routes = [
  // 每个路由应该映射一个组件
  { path: '/foo', component: Foo },
  // component 可以是通过 Vue.extend() 创建的组件构造器或者知识一个组件配置对象
  { path: '/bar', component: Bar }
  // { path: '/home', component: Home }
  // 动态导入的
  { path: '/home', component: import('./views/home/index.vue') }
  // 还可以是嵌套路由
]

// 3. 创建 router 实例
const router = new VueRouter({
  routes // 传入路由配置 routes: routes 简写
})

// 4. 创建和挂载根实例
const app = new Vue({
  router // 上面传入了 routes 配置项，可以让整个应用都有路由功能
}).$mount('#app')
```

- `app.vue` 中

```vue
<template>
  <div id="app">
    <h1>Hello App!</h1>
    <p>
      <!-- 使用 router-link 组件来导航 -->
      <router-link to="/foo">Go to Foo</router-link>
      <router-link to="/bar">Go to Bar</router-link>
    </p>
    <!-- 路由出口 路由匹配到的组件将渲染在这里 -->
    <router-view></router-view>
  </div>
</template>

<script>
export default {
  computed: {
    username() {
      // this.$route 访问当前路由
      return this.$route.params.username
    }
  },
  methods: {
    goBack() {
      // 组件中通过 this.$router 访问路由器，就是上面的 router 实例
      window.history.length > 1 ? this.$router.go(-1) : this.$router.push('/')
    }
  }
}
</script>

<style>
.router-link-active {
  color: red;
}
</style>
```

## 动态路由匹配

在 `vue-router` 的路由路径中使用“动态路径参数”(dynamic segment) ，来达到把某种模式匹配到的所有路由，全都映射到同个组件

例如：

```js
const User = {
  template: `<div>User {{ $route.params.id }}</div>`
}

const router = new VueRouter({
  routes: [
    {
      path: '/user/:id',
      component: User,
    }
  ]
})
```

`/user/foo` 和 `/user/bar` 使用的都是一个组件 `User`，只是路径不一样

- 路径参数使用冒号 `:` 标记
- 当匹配到一个路由时，参数值会被设置到 `this.$route.params`，可以在每个组件内使用

可以在一个路由中设置多段“路径参数”，对应的值都会设置到 `$route.params` 中。例如：

|模式|匹配路径|$route.params|
|---|---|---|
|/user/:username|/user/evan|`{ username: 'evan' }`|
|/user/:username/post/:post_id|/user/evan/post/123|`{ username: 'evan', post_id: '123' }`|

除了 `$route.params` 外，`$route` 对象还提供了其它有用的信息，例如，`$route.query` (如果 URL 中有查询参数)、`$route.hash` 等等

### 响应路由参数的变化 -watch 路由对象或导航守卫

当使用路由参数时，例如从 `/user/foo` 导航到 `/user/bar`

- **原来的组件实例会被复用**。因为两个路由都渲染同个组件，比起销毁再创建，复用则显得更加高效。
- **不过，这也意味着组件的生命周期钩子不会再被调用**。

路由跳转之后一般都需要再次请求新的数据，所以需要对路由参数的变化作出响应

1、简单地 watch (监测变化) `$route` 对象

```js
const User = {
  template: '...',
  watch: {
    $route(to, from) {
      // 对路由变化作出响应...，例如请求数据、滚动到顶部等
    }
  }
}
```

2、使用 `beforeRouteUpdate` 导航守卫

```js
const User = {
  template: '...',
  beforeRouteUpdate(to, from, next) {
    // react to route changes...
    // don't forget to call next()
    next()
  }
}
```

### 捕获所有路由

常规参数只会匹配被 `/` 分隔的 URL 片段中的字符。如果想匹配**任意路径**，我们可以使用通配符 (`*`)

```js
{
  // 会匹配所有路径
  path: '*'
}
{
  // 会匹配以 `/user-` 开头的任意路径
  path: '/user-*'
}
```

当使用*通配符*路由时，请确保路由的顺序是正确的，也就是说含有*通配符*的路由应该放在最后

路由 `{ path: '*' }` 通常用于客户端 404 错误，可以搭配之后的重定向使用

```js
{
  path: '*',
  redirect: '/404'
}
```

当使用一个*通配符*时，`$route.params` 内会自动添加一个名为 `pathMatch` 参数

它包含了 URL 通过*通配符*被匹配的部分

```js
// 给出一个路由 { path: '/user-*' }
this.$router.push('/user-admin')
this.$route.params.pathMatch // 'admin'
// 给出一个路由 { path: '*' }
this.$router.push('/non-existing')
this.$route.params.pathMatch // '/non-existing'
```

### 高级匹配模式

可以使用正则

[文档](https://github.com/pillarjs/path-to-regexp/tree/v1.7.0#parameters)

### 匹配优先级 - 先定义的优先级高

有时候，同一个路径可以匹配多个路由，此时，匹配的优先级就按照路由的定义顺序：路由定义得越早，优先级就越高。

## 嵌套路由

**以 `/` 开头的嵌套路径会被当作根路径，使用嵌套组件而无须设置嵌套的路径**

**嵌套路由**使用 `children` 定义，里面的配置项和外层的一样（所以可以嵌套多层），路由出口需要在外层的组件中添加

例如：

在 `VueRouter` 的参数中使用 `children` 配置

```js
const router = new VueRouter({
  routes: [
    {
      path: '/user/:id',
      component: User,
      children: [
        { path: '', component: UserHome } // 下面都没匹配到的时候会渲染 UserHome
        {
          // 当 /user/:id/profile 匹配成功，
          // UserProfile 会被渲染在 User 的 <router-view> 中
          path: 'profile',
          component: UserProfile
        },
        {
          // 当 /user/:id/posts 匹配成功
          // UserPosts 会被渲染在 User 的 <router-view> 中
          path: 'posts',
          component: UserPosts
        }
      ]
    }
  ]
})
```

在 `User` 组件的模板添加一个 `<router-view>`

```js
const User = {
  template: `
    <div class="user">
      <h2>User {{ $route.params.id }}</h2>
      <router-view></router-view> // 这个不是 / 或 app 下的
    </div>
  `
}
```

## 编程式的导航 - 使用 router.push 等

除了使用 `<router-link>` 创建 a 标签来定义导航链接（声明式）

还可以借助 router 的实例方法，通过编写代码来实现（编程式）

**在 Vue 实例内部，你可以通过 `$router` 访问路由实例**，例如在组件中 `this.$router.push('home')`、`this.$route.replace('home')`、`this.$router.go(-1)`

分别是像 history 栈添加记录的前往页面（点击返回键能回去）、替换页面（不留记录）、在记录中向前或向后

- `router.push(location, onComplete?, onAbort?)`
	- `<router-link :to="location">`
	- location 可以是字符串路径或描述地址的对象
		- `router.push('home')` 字符串
		- `router.push({ path: 'home' })` 对象
		- `router.push({ name: 'user', params: { userId: '123' }})` 命名路由
		- `router.push({ path: 'register', query: { plan: 'private' }})` 带查询参数 `/register?plan=private`
- `router.replace(location, onComplete?, onAbort?)`
	- `<router-link :to="location" replace>`
	- 和上面的很像，唯一不同的是不会向 history 添加新纪录
- `router.go(n)`
	- 在 history 记录中向前或后退 n 步
	- `router.go(-1)` 后退一步记录，等同于 `history.back()`
	- `router.go(1)` 前进一步，等同于 `history.forward()`
	- `router.go(100)` 如果 history 记录不够，就是失败

`onComplete` 和 `onAbort` 回调会在导航成功完成或终止的时候进行调用

- 导航成功完成：在所有的异步钩子被解析之后
- 终止：导航到相同的路由、或在当前导航完成之前导航到另一个不同的路由
	- 如果目的地和当前路由相同，只有参数发生了改变 (比如从一个用户资料到另一个 `/users/1` -> `/users/2`)，你需要使用 [`beforeRouteUpdate`](https://v3.router.vuejs.org/zh/guide/essentials/dynamic-matching.html#%E5%93%8D%E5%BA%94%E8%B7%AF%E7%94%B1%E5%8F%82%E6%95%B0%E7%9A%84%E5%8F%98%E5%8C%96) 来响应这个变化 (比如抓取用户信息)

可以在 Vue Router 中操作 history

- 效仿 `window.history` API 的  [`window.history.pushState`、 `window.history.replaceState` 和 `window.history.go`](https://developer.mozilla.org/en-US/docs/Web/API/History)
- Vue Router 的导航方法 (`push`、 `replace`、 `go`) 在各类路由模式 (`history`、 `hash` 和 `abstract`) 下表现一致

## 命名路由 - 有 name 的 route

有时候，通过一个名称来标识一个路由显得更方便一些，特别是在链接一个路由，或者是执行一些跳转的时候

可以在创建 Router 实例的时候，在 `routes` 配置中给某个路由设置名称：

```js
const router = new VueRouter({
  routes: [
    {
      path: '/user/:userId',
      name: 'user', // 这里，就是给路由一个名称标识
      component: User
    }
  ]
})
```

要链接到一个命名路由，可以给 `router-link` 的 `to` 属性传一个对象：

```js
<router-link :to="{ name: 'user', params: { userId: 123 }}">User</router-link>
```

这跟代码调用 `router.push()` 是一回事：

```js
router.push({ name: 'user', params: { userId: 123 } })
```

这两种方式都会把路由导航到 `/user/123` 路径

## 命名视图 - 有 name 的 router-view

有时候想同时 (同级) 展示多个视图，而不是嵌套展示，例如创建一个布局，有 `sidebar` (侧导航) 和 `main` (主内容) 两个视图，这个时候命名视图就派上用场了

可以在界面中拥有多个单独命名的视图，而不是只有一个单独的出口

如果 `router-view` 没有设置名字，那么默认为 `default`

```js
<router-view class="view one"></router-view>
<router-view class="view two" name="a"></router-view>
<router-view class="view three" name="b"></router-view>
```

一个视图使用一个组件渲染，因此对于同个路由，多个视图就需要多个组件，确保正确使用 `components` 配置 (带上 s)：

```js
const router = new VueRouter({
  routes: [
    {
      path: '/',
      components: {
        default: Foo,
        a: Bar,
        b: Baz
      }
    }
  ]
})
```

它也是可以被嵌套的——嵌套命名视图

```html
<!-- UserSettings.vue -->
<div>
  <h1>User Settings</h1>
  <NavBar/>
  <router-view/>
  <router-view name="helper"/>
</div>
```

路由配置

```js
{
  path: '/settings',
  // 你也可以在顶级路由就配置命名视图
  component: UserSettings,
  children: [{
    path: 'emails',
    component: UserEmailsSubscriptions // /settings/emails 显示
  }, {
    path: 'profile',
    components: { // 两个都在 /settrings/profile 下显示
      default: UserProfile,
      helper: UserProfilePreview // 这个
    }
  }]
}
```

## 重定向 -redirect

```js
const router = new VueRouter({
  routes: [
    { path: '/a', redirect: '/b' }, // 从 /a 重定向到 /b
    { path: '/a', redirect: { name: 'foo' }}, // 可以是命名路由
    { path: '/redirect-with-params/:id', redirect: '/with-params/:id' }, // 带参数
    { path: '/foobar', component: Foobar, caseSensitive: true }, // 大小写敏感
    { path: '/FooBar', component: FooBar, pathToRegexpOptions: { sensitive: true }}, 
    { path: '/a', redirect: to => { // 还可以是个方法，注意传入的只有 to
      // 方法接收 目标路由 作为参数
      // return 重定向的 字符串路径/路径对象
      const { hash, params, query } = to
        if (query.to === 'foo') {
          return { path: '/foo', query: null }
        }
        if (hash === '#baz') {
          return { name: 'baz', hash: '' }
        }
        if (params.id) {
          return '/with-params/:id'
        } else {
          return '/bar'
        }
    }},
    { path: '*', redirect: '/' }
  ]
})
```

## 别名 - 实际名称是不变的

**`/a` 的别名是 `/b`，意味着，当用户访问 `/b` 时，URL 会保持为 `/b`，但是路由匹配则为 `/a`，就像用户访问 `/a` 一样。**

```js
const router = new VueRouter({
  routes: [
    { path: '/a', component: A, alias: '/b' },
    { path: 'baz', component: Baz, alias: ['/baz', 'baz-alias'] },
  ]
})
```

可以自由地将 UI 结构映射到任意的 URL，而不是受限于配置的嵌套路由结构

## 路由组件传参

在组件中使用 `$route` 会使之与其对应路由形成高度耦合，从而使组件只能在某些特定的 URL 上使用，限制了其灵活性

```js
const User = {
  template: '<div>User {{ $route.params.id }}</div>'
}
const router = new VueRouter({
  routes: [{ path: '/user/:id', component: User }]
})
```

使用 `props` 将组件和路由解耦

```js
const User = {
  props: ['id'],
  template: '<div>User {{ id }}</div>'
}
const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User, props: true },

    // 对于包含命名视图的路由，你必须分别为每个命名视图添加 `props` 选项：
    {
      path: '/user/:id',
      components: { default: User, sidebar: Sidebar },
      props: { default: true, sidebar: false }
    }
  ]
})
```

- 布尔模式
	- 如果 `props` 被设置为 `true`，`route.params` 将会被设置为组件属性
- 对象模式
	- 如果 `props` 是一个对象，它会被按原样设置为组件属性。当 `props` 是静态的时候有用。

```js
const router = new VueRouter({
  routes: [
    {
      path: '/promotion/from-newsletter',
      component: Promotion,
      props: { newsletterPopup: false }
    }
  ]
})
```

- 函数模式
	- 你可以创建一个函数返回 `props`。这样你便可以将参数转换成另一种类型，将静态值与基于路由的值结合等等。

```js
const router = new VueRouter({
  routes: [
    {
      path: '/search',
      component: SearchUser,
      props: route => ({ query: route.query.q })
    }
  ]
})
```

URL `/search?q=vue` 会将 `{query: 'vue'}` 作为属性传递给 `SearchUser` 组件。

请尽可能保持 `props` 函数为无状态的，因为它只会在路由发生变化时起作用。如果你需要状态来定义 `props`，请使用包装组件，这样 Vue 才可以对状态变化做出反应。

```js
function dynamicPropsFn (route) {
  const now = new Date()
  return {
    name: (now.getFullYear() + parseInt(route.params.years)) + '!'
  }
}

const router = new VueRouter({
  routes: [
    { path: '/', component: Hello }, // No props, no nothing
    { path: '/hello/:name', component: Hello, props: true }, // Pass route.params to props
    { path: '/static', component: Hello, props: { name: 'world' }}, // static values
    { path: '/dynamic/:years', component: Hello, props: dynamicPropsFn }, // custom logic for mapping between route and props
    { path: '/attrs', component: Hello, props: { name: 'attrs' }}
  ]
})
```

## 其他模式 - 默认 hash、可修改为 history

`vue-router` 默认 hash 模式 —— 使用 URL 的 hash 来模拟一个完整的 URL（链接中有井号），于是当 URL 改变时，页面不会重新加载。

如果不想要很丑的 hash，我们可以用路由的 **history 模式**（URL 就像正常的 url，没有井号），这种模式充分利用 `history.pushState` API 来完成 URL 跳转而无须重新加载页面。

```js
const router = new VueRouter({
  mode: 'history',
  routes: [...]
})
```

单页客户端应用，需要后台配置支持

如果后台没有正确的配置，当用户在浏览器直接访问就会返回 404

如果 URL 匹配不到任何静态资源，则应该返回同一个 `index.html` 页面，这个页面就是你 app 依赖的页面

给个 Node.js 的例子

```js
const http = require('http')
const fs = require('fs')
const httpPort = 80

http.createServer((req, res) => {
  fs.readFile('index.html', 'utf-8', (err, content) => {
    if (err) {
      console.log('We cannot open "index.html" file.')
    }

    res.writeHead(200, {
      'Content-Type': 'text/html; charset=utf-8'
    })

    res.end(content)
  })
}).listen(httpPort, () => {
  console.log('Server listening on: http://localhost:%s', httpPort)
})
```

这么做之后所有不匹配的页面都会返回 `index.html` 文件，不会返回 404 页面，故需要先处理

```js
const router = new VueRouter({
  mode: 'history',
  routes: [
    { path: '*', component: NotFoundComponent }
  ]
})
```

或者，如果你使用 Node.js 服务器，你可以用服务端路由匹配到来的 URL，并在没有匹配到路由的时候返回 404，以实现回退

## 导航守卫 - 路由改变的钩子

> “导航”表示路由正在发生改变

1. `vue-router` 提供的导航守卫主要用来通过跳转或取消的方式守卫导航
2. **参数或查询的改变并不会触发进入/离开的导航守卫**，可以通过 watch `$route` 对象或使用 `beforeRouteUpdate` 的组件内守卫

分类

- 全局前置守卫：
	- 使用 `router.beforeEach` 注册一个全局前置守卫
	- 三个参数
		- **`to: Route`**: 即将要进入的目标 [路由对象](https://v3.router.vuejs.org/zh/api/#%E8%B7%AF%E7%94%B1%E5%AF%B9%E8%B1%A1)
		- **`from: Route`**: 当前导航正要离开的路由
		- **`next: Function`**: 一定要调用该方法来 **resolve** 这个钩子。执行效果依赖 `next` 方法的调用参数。
		- **`next()`**: 进行管道中的下一个钩子。如果全部钩子执行完了，则导航的状态就是 **confirmed** (确认的)。
			- **`next(false)`**: 中断当前的导航。如果浏览器的 URL 改变了 (可能是用户手动或者浏览器后退按钮)，那么 URL 地址会重置到 `from` 路由对应的地址。
			- **`next('/')` 或者 `next({ path: '/' })`**: 跳转到一个不同的地址。当前的导航被中断，然后进行一个新的导航。你可以向 `next` 传递任意位置对象，且允许设置诸如 `replace: true`、`name: 'home'` 之类的选项以及任何用在 [`router-link` 的 `to` prop](https://v3.router.vuejs.org/zh/api/#to) 或 [`router.push`](https://v3.router.vuejs.org/zh/api/#router-push) 中的选项。
			- **`next(error)`**: (2.4.0+) 如果传入 `next` 的参数是一个 `Error` 实例，则导航会被终止且该错误会被传递给 [`router.onError()`](https://v3.router.vuejs.org/zh/api/#router-onerror) 注册过的回调。

```js
const router = new VueRouter({ ... })

router.beforeEach((to, from, next) => {
  // ...
})
```

**确保 `next` 函数在任何给定的导航守卫中都被严格调用一次。它可以出现多于一次，但是只能在所有的逻辑路径都不重叠的情况下，否则钩子永远都不会被解析或报错**

```js
router.beforeEach((to, from, next) => {
  if (to.name !== 'Login' && !isAuthenticated) next({ name: 'Login' })
  else next()
})
```

- 全局解析守卫
	- 用 `router.beforeResolve` 注册一个全局守卫
	- 和上面的相似
	- 区别是在导航被确认之前，**同时在所有组件内守卫和异步路由组件被解析之后**，解析守卫就被调用
- 全局后置钩子
	- `router.afterEach` 注册全局后置钩子
	- 这些钩子不会接受 `next` 函数也不会改变导航本身

```js
router.afterEach((to, from) => {
  // ...
})
```

- 路由独享的守卫
	- 在路由配置上直接定义 `beforeEnter` 守卫

```js
const router = new VueRouter({
  routes: [
    {
      path: '/foo',
      component: Foo,
      beforeEnter: (to, from, next) => {
        // ...
      }
    }
  ]
})
```

- 组件内的守卫
	- 可以在组件内直接定义 `beforeRouteEnter`、`beforeRouteUpdate`、`beforeRouteLeave`

```js
const Foo = {
  template: `...`,
  beforeRouteEnter(to, from, next) {
    // 在渲染该组件的对应路由被 confirm 前调用
    // 不！能！获取组件实例 `this`
    // 因为当守卫执行前，组件实例还没被创建
  },
  beforeRouteUpdate(to, from, next) {
    // 在当前路由改变，但是该组件被复用时调用
    // 举例来说，对于一个带有动态参数的路径 /foo/:id，在 /foo/1 和 /foo/2 之间跳转的时候，
    // 由于会渲染同样的 Foo 组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用。
    // 可以访问组件实例 `this`
  },
  beforeRouteLeave(to, from, next) {
    // 导航离开该组件的对应路由时调用
    // 可以访问组件实例 `this`
  }
}
```

`beforeRouteEnter` 守卫 **不能** 访问 `this`，因为守卫在导航确认前被调用，因此即将登场的新组件还没被创建

可以通过传一个回调给 `next` 来访问组件实例。在导航被确认的时候执行回调，并且把组件实例作为回调方法的参数，其他的 `this` 是可用的，所以不支持传递回调

```js
beforeRouteEnter (to, from, next) {
  next(vm => {
    // 通过 `vm` 访问组件实例
  })
}
```

这个离开守卫通常用来禁止用户在还未保存修改前突然离开。该导航可以通过 `next(false)` 来取消。

```js
beforeRouteLeave (to, from, next) {
  const answer = window.confirm('Do you really want to leave? you have unsaved changes!')
  if (answer) {
    next()
  } else {
    next(false)
  }
}
```

完整的导航解析流程

1. 导航被触发。
2. 在失活的组件里调用 `beforeRouteLeave` 守卫。
3. 调用全局的 `beforeEach` 守卫。
4. 在重用的组件里调用 `beforeRouteUpdate` 守卫 (2.2+)。
5. 在路由配置里调用 `beforeEnter`。
6. 解析异步路由组件。
7. 在被激活的组件里调用 `beforeRouteEnter`。
8. 调用全局的 `beforeResolve` 守卫 (2.5+)。
9. 导航被确认。
10. 调用全局的 `afterEach` 钩子。
11. 触发 DOM 更新。
12. 调用 `beforeRouteEnter` 守卫中传给 `next` 的回调函数，创建好的组件实例会作为回调函数的参数传入。

## 路由元信息 -meta

定义路由的时候可以配置 `meta` 字段

```js
{
  path: 'bar',
  component: Bar,
  // a meta field
  meta: { requiresAuth: true }
}
```

称呼 `routes` 配置中的每个路由对象为 **路由记录**

一个路由匹配到的所有路由记录会暴露为 `$route` 对象 (还有在导航守卫中的路由对象) 的 `$route.matched` 数组

遍历 `$route.matched` 来检查路由记录中的 `meta` 字段

```js
router.beforeEach((to, from, next) => {
  if (to.matched.some(record => record.meta.requiresAuth)) {
    // this route requires auth, check if logged in
    // if not, redirect to login page.
    if (!auth.loggedIn()) {
      next({
        path: '/login',
        query: { redirect: to.fullPath }
      })
    } else {
      next()
    }
  } else {
    next() // 确保一定要调用 next()
  }
})
```

## 过渡动效 - 还是 Vue 中的 transition

`<router-view>` 是基本的动态组件，所以我们可以用 `<transition>` 组件给它添加一些过渡效果

给所有路由设置一样的过渡效果

```html
<transition>
  <router-view></router-view>
</transition>
```

每个路由组件有各自的过渡效果，设置不一样的 name

```js
const Foo = {
  template: `
    <transition name="slide">
      <div class="foo">...</div>
    </transition>
  `
}

const Bar = {
  template: `
    <transition name="fade">
      <div class="bar">...</div>
    </transition>
  `
}
```

基于当前路由与目标路由的变化关系，动态设置过渡效果

```js
<!-- 使用动态的 transition name -->
<transition :name="transitionName">
  <router-view></router-view>
</transition>

// 接着在父组件内
// watch $route 决定使用哪种过渡
watch: {
  '$route' (to, from) {
    const toDepth = to.path.split('/').length
    const fromDepth = from.path.split('/').length
    this.transitionName = toDepth < fromDepth ? 'slide-right' : 'slide-left'
  }
}
```

需要自己设置好样式

除了 path 其他的一样都可以使用，例如元信息

## 后端数据获取

有时候，进入某个路由后，需要从服务器获取数据

- **导航完成之后获取**：先完成导航，然后在接下来的组件生命周期钩子中获取数据。在数据获取期间显示“加载中”之类的指示。

马上导航和渲染组件，然后在组件的 `created` 钩子中获取数据

有机会在数据获取期间展示一个 loading 状态，还可以在不同视图间展示不同的 loading 状态

假设我们有一个 `Post` 组件，需要基于 `$route.params.id` 获取文章数据：

```vue
<template>
  <div class="post">
    <div v-if="loading" class="loading">
      Loading...
    </div>

    <div v-if="error" class="error">
      {{ error }}
    </div>

    <div v-if="post" class="content">
      <h2>{{ post.title }}</h2>
      <p>{{ post.body }}</p>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      loading: false,
      post: null,
      error: null
    }
  },
  created () {
    // 组件创建完后获取数据，
    // 此时 data 已经被 observed 了
    this.fetchData()
  },
  watch: {
    // 如果路由有变化，会再次执行该方法
    '$route': 'fetchData'
  },
  methods: {
    fetchData () {
      this.error = this.post = null
      this.loading = true
      // replace getPost with your data fetching util / API wrapper
      getPost(this.$route.params.id, (err, post) => {
        this.loading = false
        if (err) {
          this.error = err.toString()
        } else {
          this.post = post
        }
      })
    }
  }
}
</script>
```

- **导航完成之前获取**：导航完成前，在路由进入的守卫中获取数据，在数据获取成功后执行导航。

可以在接下来的组件的 `beforeRouteEnter` 守卫中获取数据，当数据获取成功后只调用 `next` 方法

```js
export default {
  data () {
    return {
      post: null,
      error: null
    }
  },
  beforeRouteEnter (to, from, next) {
    getPost(to.params.id, (err, post) => {
      next(vm => vm.setData(err, post))
    })
  },
  // 路由改变前，组件就已经渲染完了
  // 逻辑稍稍不同
  beforeRouteUpdate (to, from, next) {
    this.post = null
    getPost(to.params.id, (err, post) => {
      this.setData(err, post)
      next()
    })
  },
  methods: {
    setData (err, post) {
      if (err) {
        this.error = err.toString()
      } else {
        this.post = post
      }
    }
  }
}
```

在为后面的视图获取数据时，用户会停留在当前的界面，因此建议在数据获取期间，显示一些进度条或者别的指示。如果数据获取失败，同样有必要展示一些全局的错误提醒。

## 滚动行为

使用前端路由，当切换到新路由时，想要页面滚到顶部，或者是保持原先的滚动位置

**注意: 这个功能只在支持 `history.pushState` 的浏览器中可用。**

当创建一个 Router 实例，你可以提供一个 `scrollBehavior` 方法：

```js
const router = new VueRouter({
  routes: [...],
  scrollBehavior (to, from, savedPosition) {
    // return 期望滚动到哪个的位置
  }
})
```

`scrollBehavior` 方法接收 `to` 和 `from` 路由对象。第三个参数 `savedPosition` 当且仅当 `popstate` 导航 (通过浏览器的 前进/后退 按钮触发) 时才可用。

```js
scrollBehavior (to, from, savedPosition) {
  if (savedPosition) {
    return savedPosition // return 期望滚动到哪个的位置
  } else {
    return { x: 0, y: 0 }
  }
}
```

返回滚动位置的对象信息

- `{ x: number, y: number }`
- `{ selector: string, offset? : { x: number, y: number }}`

如果返回一个 falsy 或者是一个空对象，那么不会发生滚动

模拟“滚动到锚点”的行为：

```js
scrollBehavior (to, from, savedPosition) {
  if (to.hash) {
    return {
      selector: to.hash
    }
  }
}
```

由于它接收 to 和 from，因此需要更细致地控制滚动行为，可以进行判断

也可以返回一个 Promise 来得出预期的位置描述（异步滚动）

```js
scrollBehavior (to, from, savedPosition) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve({ x: 0, y: 0 })
    }, 500)
  })
}
```

平滑滚动

```js
scrollBehavior (to, from, savedPosition) {
  if (to.hash) {
    return {
      selector: to.hash,
      behavior: 'smooth',
    }
  }
}
```

## 路由懒加载 - 分包减少打包体积

结合 Vue 的 [异步组件 (opens new window)](https://cn.vuejs.org/v2/guide/components-dynamic-async.html#%E5%BC%82%E6%AD%A5%E7%BB%84%E4%BB%B6) 和 Webpack 的 [代码分割功能 (opens new window)](https://doc.webpack-china.org/guides/code-splitting-async/#require-ensure-/)，轻松实现路由组件的懒加载

这两步不用看

首先，可以将异步组件定义为返回一个 Promise 的工厂函数 (该函数返回的 Promise 应该 resolve 组件本身)：

```js
const Foo = () =>
  Promise.resolve({
    /* 组件定义对象 */
  })
```

第二，在 Webpack 2 中，我们可以使用 [动态 import (opens new window)](https://github.com/tc39/proposal-dynamic-import) 语法来定义代码分块点 (split point)：

```js
import('./Foo.vue') // 返回 Promise
```

> Babel 需要添加 [`syntax-dynamic-import` (opens new window)](https://babeljs.io/docs/plugins/syntax-dynamic-import/) 插件

**定义**一个能够被 Webpack 自动代码分割的异步组件

```js
const Foo = () => import('./Foo.vue')

const router = new VueRouter({
  routes: [{
    path: '/foo',
    component: Foo // 或者直接写这 component: () => import('./Foo.vue')
  }]
})
```

把组件按组分块，有时候我们想把某个路由下的所有组件都打包在同个异步块 (chunk) 中，只需要使用命名 chunk（特殊的注释语法）

```js
const Foo = () => import(/* webpackChunkName: "group-foo" */ './Foo.vue')
const Bar = () => import(/* webpackChunkName: "group-foo" */ './Bar.vue')
const Baz = () => import(/* webpackChunkName: "group-foo" */ './Baz.vue')
```

Webpack 会将任何一个异步模块与相同的块名称组合到相同的异步块中。

## 导航故障

> *导航故障*，或者叫*导航失败*，表示一次失败的导航，原文叫 navigation failures，本文统一采用*导航故障*

当使用 `router-link` 组件时，Vue Router 会自动调用 `router.push` 来触发一次导航。 虽然大多数链接的预期行为是将用户导航到一个新页面，但也有少数情况下用户将留在同一页面上：

- 用户已经位于他们正在尝试导航到的页面
- 一个 [导航守卫](https://v3.router.vuejs.org/zh/guide/advanced/navigation-guards.html) 通过调用 `next(false)` 中断了这次导航
- 一个 [导航守卫](https://v3.router.vuejs.org/zh/guide/advanced/navigation-guards.html) 抛出了一个错误，或者调用了 `next(new Error())`

当使用 `router-link` 组件时，**这些失败都不会打印出错误**。然而，如果你使用 `router.push` 或者 `router.replace` 的话，可能会在控制台看到一条 *"Uncaught (in promise) Error"* 这样的错误，后面跟着一条更具体的消息。让我们来了解一下如何区分 _ 导航故障 _。

在 v3.2.0 中，可以通过使用 `router.push` 的两个可选的回调函数：`onComplete` 和 `onAbort` 来暴露 _ 导航故障 _。从版本 3.1.0 开始，`router.push` 和 `router.replace` 在没有提供 `onComplete`/`onAbort` 回调的情况下会返回一个 *Promise*。这个 *Promise* 的 resolve 和 reject 将分别用来代替 `onComplete` 和 `onAbort` 的调用。

导航故障是一个 `Error` 实例，附带了一些额外的属性。要检查一个错误是否来自于路由器，可以使用 `isNavigationFailure` 函数：

```js
import VueRouter from 'vue-router'
const { isNavigationFailure, NavigationFailureType } = VueRouter

// 正在尝试访问 admin 页面
router.push('/admin').catch(failure => {
  if (isNavigationFailure(failure, NavigationFailureType.redirected)) {
    // 向用户显示一个小通知
    showToast('Login in order to access the admin panel')
  }
})
```

`NavigationFailureType` 可以帮助开发者来区分不同类型的 _ 导航故障 _。有四种不同的类型：

- `redirected`：在导航守卫中调用了 `next(newLocation)` 重定向到了其他地方。
- `aborted`：在导航守卫中调用了 `next(false)` 中断了本次导航。
- `cancelled`：在当前导航还没有完成之前又有了一个新的导航。比如，在等待导航守卫的过程中又调用了 `router.push`。
- `duplicated`：导航被阻止，因为我们已经在目标位置了。

所有的导航故障都会有 `to` 和 `from` 属性，分别用来表达这次失败的导航的目标位置和当前位置。

```js
// 正在尝试访问 admin 页面
router.push('/admin').catch(failure => {
  if (isNavigationFailure(failure, NavigationFailureType.redirected)) {
    failure.to.path // '/admin'
    failure.from.path // '/'
  }
})
```
