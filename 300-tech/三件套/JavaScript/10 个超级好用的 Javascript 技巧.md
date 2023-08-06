---
title: 10 个超级好用的 Javascript 技巧
date: 2023-08-03 17:33
updated: 2023-08-03 17:38
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - 10 个超级好用的 Javascript 技巧
rating: 1
tags:
  - JS
  - web
category: web
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: //juejin.cn/post/7215497169469685815
url: //myblog.wallleap.cn/post/1
---

在实际的开发工作过程中，积累了一些常见又超级好用的 Javascript 技巧和代码片段，包括整理的其他大神的 JS 使用技巧，今天筛选了 10 个，以供大家参考。

## 动态加载 JS 文件

在一些特殊的场景下，特别是一些库和框架的开发中，我们有时会去动态的加载 JS 文件并执行，下面是利用 Promise 进行了简单的封装。

```js
function loadJS(files, done) {
  // 获取head标签
  const head = document.getElementsByTagName('head')[0];
  Promise.all(files.map(file => {
    return new Promise(resolve => {
      // 创建script标签并添加到head
      const s = document.createElement('script');
      s.type = "text/javascript";
      s.async = true;
      s.src = file;
      // 监听load事件，如果加载完成则resolve
      s.addEventListener('load', (e) => resolve(), false);
      head.appendChild(s);
    });
  })).then(done);  // 所有均完成，执行用户的回调事件
}

loadJS(["test1.js", "test2.js"], () => {
  // 用户的回调逻辑
});
```

上面代码核心有两点，一是利用 Promise 处理异步的逻辑，二是利用 script 标签进行 js 的加载并执行。

## 实现模板引擎

下面示例用了极少的代码实现了动态的模板渲染引擎，不仅支持普通的动态变量的替换，还支持包含 for 循环，if 判断等的动态的 JS 语法逻辑，具体实现逻辑在我的另外一篇文章做了非常详详尽的说明，感兴趣的小伙伴戳此链接 [【造轮子系列】面试官问：你能手写一个模板引擎吗？](https://juejin.cn/post/7207697872706486328 "https://juejin.cn/post/7207697872706486328")

```js
// 这是包含了js代码的动态模板
var template = 
'My avorite sports:' + 
'<%if(this.showSports) {%>' +
    '<% for(var index in this.sports) {   %>' + 
    '<a><%this.sports[index]%></a>' +
    '<%}%>' +
'<%} else {%>' +
    '<p>none</p>' +
'<%}%>';
// 这是我们要拼接的函数字符串
const code = `with(obj) {
  var r=[];
  r.push("My avorite sports:");
  if(this.showSports) {
    for(var index in this.sports) {   
      r.push("<a>");
      r.push(this.sports[index]);
      r.push("</a>");
    }
  } else {
    r.push("<span>none</span>");
  }
  return r.join("");
}`
// 动态渲染的数据
const options = {
  sports: ["swimming", "basketball", "football"],
  showSports: true
}
// 构建可行的函数并传入参数，改变函数执行时this的指向
result = new Function("obj", code).apply(options, [options]);
console.log(result);

```

## 利用 reduce 进行数据结构的转换

有时候前端需要对后端传来的数据进行转换，以适配前端的业务逻辑，或者对组件的数据格式进行转换再传给后端进行处理，而 `reduce` 是一个非常强大的工具。

```js
const arr = [
    { classId: "1", name: "张三", age: 16 },
    { classId: "1", name: "李四", age: 15 },
    { classId: "2", name: "王五", age: 16 },
    { classId: "3", name: "赵六", age: 15 },
    { classId: "2", name: "孔七", age: 16 }
]; 

groupArrayByKey(arr, "classId"); 

function groupArrayByKey(arr = [], key) {
    return arr.reduce((t, v) => (!t[v[key]] && (t[v[key]] = []), t[v[key]].push(v), t), {})
}

```

很多很复杂的逻辑如果用 reduce 去处理，都非常的简洁。

## 添加默认值

有时候一个方法需要用户传入一个参数，通常情况下我们有两种处理方式，如果用户不传，我们通常会给一个默认值，亦或是用户必须要传一个参数，不传直接抛错。

```js
function double() {
    return value *2
}

// 不传的话给一个默认值0
function double(value = 0) {
    return value * 2
}

// 用户必须要传一个参数，不传参数就抛出一个错误

const required = () => {
    throw new Error("This function requires one parameter.")
}
function double(value = required()) {
    return value * 2
}

double(3) // 6
double() // throw Error
```

## 函数只执行一次

有些情况下我们有一些特殊的场景，某一个函数只允许执行一次，或者绑定的某一个方法只允许执行一次。

```js
export function once (fn) {
  // 利用闭包判断函数是否执行过
  let called = false
  return function () {
    if (!called) {
      called = true
      fn.apply(this, arguments)
    }
  }
}
```

## 实现 Curring

JavaScript 的柯里化是指将接受多个参数的函数转换为一系列只接受一个参数的函数的过程。这样可以更加灵活地使用函数，减少重复代码，并增加代码的可读性。

```js
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    } else {
      return function(...args2) {
        return curried.apply(this, args.concat(args2));
      };
    }
  };
}

function add(x, y) {
  return x + y;
}

const curriedAdd = curry(add);

console.log(curriedAdd(1)(2)); // 输出 3
console.log(curriedAdd(1, 2)); // 输出 3

```

通过柯里化，我们可以将一些常见的功能模块化，例如验证、缓存等等。这样可以提高代码的可维护性和可读性，减少出错的机会。

## 实现单例模式

JavaScript 的单例模式是一种常用的设计模式，它可以确保一个类只有一个实例，并提供对该实例的全局访问点，在 JS 中有广泛的应用场景，如购物车，缓存对象，全局的状态管理等等。

```js
let cache;
class A {
  // ...
}

function getInstance() {
  if (cache) return cache;
  return cache = new A();
}

const x = getInstance();
const y = getInstance();

console.log(x === y); // true
```

## 实现 CommonJs 规范

CommonJS 规范的核心思想是将每个文件都看作一个模块，每个模块都有自己的作用域，其中的变量、函数和对象都是私有的，不能被外部访问。要访问模块中的数据，必须通过导出（`exports`）和导入（`require`）的方式。

```js
// id：完整的文件名
const path = require('path');
const fs = require('fs');
function Module(id){
    // 用来唯一标识模块
    this.id = id; 
    // 用来导出模块的属性和方法
    this.exports = {}; 
}

function myRequire(filePath) {
    // 直接调用Module的静态方法进行文件的加载
    return Module._load(filePath);
}

Module._cache = {};
Module._load = function(filePath) {
    // 首先通过用户传入的filePath寻址文件的绝对路径
    // 因为再CommnJS中，模块的唯一标识是文件的绝对路径
    const realPath = Module._resoleveFilename(filePath);
    // 缓存优先，如果缓存中存在即直接返回模块的exports属性
    let cacheModule = Module._cache[realPath];
    if(cacheModule) return cacheModule.exports;
    // 如果第一次加载，需要new一个模块，参数是文件的绝对路径
    let module = new Module(realPath);
    // 调用模块的load方法去编译模块
    module.load(realPath);
    return module.exports;
}

// node文件暂不讨论
Module._extensions = {
   // 对js文件处理
  ".js": handleJS,
  // 对json文件处理
  ".json": handleJSON
}

function handleJSON(module) {
 // 如果是json文件，直接用fs.readFileSync进行读取，
 // 然后用JSON.parse进行转化，直接返回即可
  const json = fs.readFileSync(module.id, 'utf-8')
  module.exports = JSON.parse(json)
}

function handleJS(module) {
  const js = fs.readFileSync(module.id, 'utf-8')
  let fn = new Function('exports', 'myRequire', 'module', '__filename', '__dirname', js)
  let exports = module.exports;
  // 组装后的函数直接执行即可
  fn.call(exports, exports, myRequire, module,module.id,path.dirname(module.id))
}

Module._resolveFilename = function (filePath) {
  // 拼接绝对路径，然后去查找，存在即返回
  let absPath = path.resolve(__dirname, filePath);
  let exists = fs.existsSync(absPath);
  if (exists) return absPath;
  // 如果不存在，依次拼接.js,.json,.node进行尝试
  let keys = Object.keys(Module._extensions);
  for (let i = 0; i < keys.length; i++) {
    let currentPath = absPath + keys[i];
    if (fs.existsSync(currentPath)) return currentPath;
  }
};

Module.prototype.load = function(realPath) {
  // 获取文件扩展名，交由相对应的方法进行处理
  let extname = path.extname(realPath)
  Module._extensions[extname](this)
}
```

上面对 CommonJs 规范进行了简单的实现，核心解决了作用域的隔离，并提供了 `Myrequire` 方法进行方法和属性的加载，对于上面的实现，我专门有一篇文章进行了详细的说明，感兴趣的小伙伴戳此链接： [【造轮子系列】38行代码带你实现CommonJS规范](https://juejin.cn/post/7212503883263787064 "https://juejin.cn/post/7212503883263787064")

## 递归获取对象属性

如果让我挑选一个用的最广泛的设计模式，我会选观察者模式，如果让我挑一个我所遇到的最多的算法思维，那肯定是递归，递归通过将原始问题分割为结构相同的子问题，然后依次解决这些子问题，组合子问题的结果最终获得原问题的答案。

```js
const user = {
  info: {
    name: "张三",
    address: { home: "Shaanxi", company: "Xian" },
  },
};

// obj是获取属性的对象，path是路径，fallback是默认值
function get(obj, path, fallback) {
  const parts = path.split(".");
  const key = parts.shift();
  if (typeof obj[key] !== "undefined") {
    return parts.length > 0 ?
      get(obj[key], parts.join("."), fallback) :
      obj[key];
  }
  // 如果没有找到key返回fallback
  return fallback;
}

console.log(get(user, "info.name")); // 张三
console.log(get(user, "info.address.home")); // Shaanxi
console.log(get(user, "info.address.company")); // Xian
console.log(get(user, "info.address.abc", "fallback")); // fallback
```

**上面挑选了 10 个我认为比较有用的 JS 技巧，希望对大家有所帮助。**

## 系列文章

- [10个超级实用的reduce使用技巧](https://juejin.cn/post/7224043114360225847 "https://juejin.cn/post/7224043114360225847")
- [10个超级实用的Set、Map使用技巧](https://juejin.cn/post/7225425984312328252 "https://juejin.cn/post/7225425984312328252")
- [10个让你爱不释手的一行Javascript代码](https://juejin.cn/post/7230810119122190397 "https://juejin.cn/post/7230810119122190397")

## 我的更多前端资讯

欢迎大家技术交流 资料分享 摸鱼 求助皆可 —[链接](https://juejin.cn/pin/7225432788043120695 "https://juejin.cn/pin/7225432788043120695")
