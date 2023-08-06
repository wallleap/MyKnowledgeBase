---
title: Vue 数据双向绑定
date: 2023-08-04 16:28
updated: 2023-08-04 16:31
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - Vue 数据双向绑定
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

我们从以下几点来了解：

1. 数据双向绑定帮我们解决了什么问题？
2. 数据双向绑定是什么？
3. 数据双向绑定的原理是什么？
4. 数据双向绑定如何实现的？

> 可以根据你目前的技术熟悉度，选择自己感兴趣的点。

如果，你已经很熟悉 vue 框架，那么可以看看我对双向绑定的实现方式，如果你有更好的方式，非常欢迎分享出来。 如果，你不熟悉 vue，则可以完整的看一遍，对于之后使用 vue 多少会有点帮助。

## 双向绑定帮我们解决了什么问题？

在分析它之前，我们可以先回答一个问题：为什么需要双向绑定呢？让我们把时间移到十年前，来看看前端行业。

### Vue 的初衷

![](https://cdn.wallleap.cn/img/pic/illustration/202308041628513.png)

尤雨溪在谷歌从事原型设计期间，当时他接触到了 AngularJS ，他发现此框架虽然有数据绑定的功能，但是过于臃肿。于是自己写一个轻量的版本，于是 Vuejs 诞生了，可以说 Vuejs 最初的版本借鉴了很多 AngularJS 的思想。

所以双向绑定技术，自然非是 Vue 独有的特性，而是一个公共的解决方案，它目前已经被广泛的框架运用。

那么，我们再把时间再往前推一推，看看 AngularJS 的诞生是为了解决什么问题！

### 单页应用的出现

虽然 AngularJS 早在 2009 年就出现了，但 single-page application (SPA) 则更早与 2003 年就开始被讨论，主要实践为在 Web 浏览器中使用 JavaScript 来显示和控制用户界面（UI），实现应用级的程序逻辑以及与 Web 服务器通信。

而为了实现 SPA ，在 09 年之前主流的解决方案还是基于 jQuery 这类库结合 Ajax 技术进行开发，因为当时的需求与交互还算简单，但应对丰富的需求，中大项目往往都非常臃肿和难以维护了。

### 解决的问题与好处

![](https://cdn.wallleap.cn/img/pic/illustration/202308041628515.png)

而 AngularJS 的诞生，带来了前端的模块化、语义化标签、依赖注入等等，以及我们要讲的“双向数据绑定”功能。直接解决的问题，就是项目的开发效率提升和维护成本降低。

带来的好处主要体现在：

- 项目质量：开启了前端的工程化的开发模式，以模块和规范化管理前端项目；
- 研发效率：减少了开发者的代码量，以框架为支持的各类 UI 框架横空出世；
- 团队管理：以及模块和组件的开发方式，让不同的角色成员可以独立专注于 UI 与业务逻辑；

至于数据绑定带来了什么？它就像框架的心脏，是框架诞生的基础。

## 数据双向绑定是什么？

### 从单向绑定

有没有想过，我们在做原生页面开发时，有涉及到“数据绑定“的概念吗？答案肯定是有的，比如下面的例子：

```xml
<p></p>
```

```js
const data = { value: 'hello' }
document.querySelector('p').innerText = data.value;
```

通过 JavaScript 控制 DOM 的展示，就是数据（Data）到模板（DOM）的绑定，这就是数据单向绑定。

### 到双向绑定

而双向绑定就是在这个基础上，又扩展了反向的绑定效果，就是模板到数据的绑定。

上面的例子扩展以下：

```xml
<input onkeyup="change(event)" />
<p></p>
```

```js
const data = { value: '' }
const change = e => {
    // 更新输入值
    data.value = e.target.value;
    // 且，同步值的展示
    document.querySelector('p').innerText = data.value
}
```

我们将与单向绑定的却别是，数据与模板是相互影响的，一方发生变化，另一方立即做出更新。在这个简单的例子中，我们认识了双向绑定，Vue 便是在此概念下进行模块化抽象封装。

### 双向绑定的构成

![](https://cdn.wallleap.cn/img/pic/illustration/202308041628516.png)

上面的例子中，我们看到双向绑定涉及到的“模板”和“数据”，这二者是双向绑定的核心要素，也是前端框架的核心技术点。

就拿 Vue3 升级举例：

- Virturl DOM，它就是模板操作和管理的升级；
- 数据绑定，它就是数据逻辑管理的升级；

现在我们认识了数据绑定，也知道了它的意义，接下来就是看看 Vue 框架是怎么实现它的。不过在分析实现之前，必然先弄清楚实现原理吧。

## 双向绑定的原理是什么？

### 认识双向绑定在框架中的作用

因为 Vue 是数据双向绑定的框架，而整个框架的由三个部分组成：

- 数据层（Model）：应用的数据及业务逻辑，为开发者编写的业务代码；
- 视图层（View）：应用的展示效果，各类 UI 组件，由 template 和 css 组成的代码；
- 业务逻辑层（ViewModel）：框架封装的核心，它负责将数据与视图关联起来；

而上面的这个分层的架构方案，可以用一个专业术语进行称呼：MVVM。（详细的 MVVM 知识不在讲解范围内，只需要清楚双向绑定在此架构中作用和位置即可）

这里的控制层的核心功能便是 “数据双向绑定” 。自然，我们只需弄懂它 是什么，便可以进一步了解数据绑定的原理。

### 理解 ViewModel

它的主要职责就是：

1. 数据变化后更新视图；
2. 视图变化后更新数据；

那么，就可以得出它主要由两个部分组成：

1. 监听器（Observer）：观察数据，做到时刻清楚数据的任何变化，然后通知视图更新；
2. 解析器（Compiler）：观察 UI，做到时刻清楚视图发生的一切交互，然后更新数据；

然后把二者组合起来，一个具有数据双向绑定的框架就诞生了。

## 双向绑定如何实现的？

### 实现监听器

确保它是一个独立的功能，它的任务就是监听数据的变化，并提供通知功能。

监听数据变化常见的方式有三种：

- 观察者模式（发布 + 订阅）；
- 数据劫持；
- 脏检查；

第三种，是 AngularJS 最早所提供的监听数据变化方式，而 Vue 是采用前两者的组合，那么让我们来看看它们是怎么被运用的。

#### 观察者模式

观察者模式是一种对象行为模式。它定义对象间的一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。

在观察者模式中，主导的是起通知作用的**发布者**，它发出通知时并不需要知道谁是它的观察者，**可以有任意数目的观察者**订阅并接收通知。

举例代码如下：

```js
/**
 * 发布者
 */
function Subject() {
  
  // 单个发布者的所有订阅者
  this.observers = [];
  
  // 添加一个订阅者
  this.attach = function(callback) {
    this.observers.push(callback);
  };
  
  // 通过所有的订阅者
  this.notify = function(value) {
    this.observers.forEach(callback => callback(value));
  };
}

/**
 * 订阅者
 */
function Observer(queue, key, callback) {
  queue[key].attach(callback);
}

// ====

// 手动更新数据
function setData(data, key, value) {
    data[key] = value;

    // 通知此值的所有订阅者，数据发生了更新
    messageQueue[key].notify(value);
}

// ====

// 消息队列
const messageQueue = {};

// 数据
const myData = { value: "" };

// 将每个数据属性添加可订阅的入口
for (let key in myData) {
  messageQueue[key] = new Subject();
}

// 订阅 value 值的变化
Observer(messageQueue, "value", value => {
  console.warn("value updated:", value);
});

// 更新数据
setData(myData, "value", "hello world.");
setData(myData, "value", 100);
setData(myData, "value", true);
```

可以在 codepen 上查看此运行效果（需要打开左下角的 console 查看输出，此时还不涉及 DOM）：[codepen.io/nachao/pen/…](https://link.juejin.cn/?target=https%3A%2F%2Fcodepen.io%2Fnachao%2Fpen%2FGRKBZKW "https://codepen.io/nachao/pen/GRKBZKW")

##### 消息队列（queue）

我们可以看到，单纯的订阅和发布功能，它们彼此是独立存在的，因此还需要一个消息队列来关联他们。

上面的例子中，消息队列是作为一个全局存储变量，而在框架中则是封装起来的，每个 `new Vue()` 都有独立的一个队列，在下面我们将具体演示。

#### 数据劫持

其实，就数据监听来说，观察者就已经满足了需求。但是，为什么和 Vue 不一样呢？因为 Vue 进行了优化，添加了数据劫持。

2009 年发布的 ECMAScript 5 中新增了一个 Object.definePropotype 的特性（具体使用不在此文章讲解范围内，请自行了解 [developer.mozilla.org/zh-CN/docs/…](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FObject%2FdefineProperty "https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty") ），能够定义对象属性的 getter 和 setter ，这可就厉害了，要知道 JavaScript 中一切皆对象。

那么我们的 `setData(myData, 'value', 100);` 就可以替换成 `myData.value = 100;` 的编写方式。从语法和使用上都变的更简单。

以下是变动后的代码：

```js
// 发布者
function Subject() {
  this.observers = [];
  this.attach = function(callback) {
    this.observers.push(callback);
  };
  this.notify = function(value) {
    this.observers.forEach(callback => callback(value));
  };
}

// 订阅者
function Observer(queue, key, callback) {
  queue[key].attach(callback);
}

// ====

// 数据拦截器
function Watcher(data, queue) {
  for (let key in data) {
    let value = data[key];
    Object.defineProperty(data, key, {
      enumerable: true,
      configurable: true,
      get: () => value,
      set: newValue => {
        value = newValue;

        // 通知此值的所有订阅者，数据发生了更新
        queue[key].notify(value);
      }
    });
  }
  return data;
}

// ====

// 消息队列
const messageQueue = {};

// 数据
const myData = Watcher({ value: "" }, messageQueue);

// 将每个数据属性都添加到观察者的消息队列中
for (let key in myData) {
  messageQueue[key] = new Subject();
}

// 订阅 value 值的变化
Observer(messageQueue, "value", value => {
  console.warn("value updated:", value);
});

// 更新数据
myData.value = "hello world.";
myData.value = 100;
myData.value = true;
```

可以在 codepen 上查看运行效果：[codepen.io/nachao/pen/…](https://link.juejin.cn/?target=https%3A%2F%2Fcodepen.io%2Fnachao%2Fpen%2FrNBrepN "https://codepen.io/nachao/pen/rNBrepN")

当然，ES2015 已经出来有点时间了，所以应尽用新的解决方案 Proxy（具体的使用请自行了解哦 [developer.mozilla.org/zh-CN/docs/…](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FProxy "https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy") ），讲到它主要是因为 Vue3 就采用了此方案。至于性能和具体的差异，之后有机会再单独分享，这里就不细说，让我们看看新的语法怎么写呢？

代码：

```js
// 发布者
function Subject() {
  this.observers = [];
  this.attach = function(callback) {
    this.observers.push(callback);
  };
  this.notify = function(value) {
    this.observers.forEach(callback => callback(value));
  };
}

// 订阅者
function Observer(queue, key, callback) {
  queue[key].attach(callback);
}

// ====

// 数据拦截器 - 代理方式
function ProxyWatcher(data, queue) {
  return new Proxy(data, {
    get: (target, key) => target[key],
    set(target, key, value) {
      target[key] = value;

      // 通知此值的所有订阅者，数据发生了更新
      queue[key].notify(value);
    }
  });
}

// ====

// 消息队列
const messageQueue = {};

// 数据
const myData = ProxyWatcher({ value: "" }, messageQueue);

// 将每个数据属性都添加到观察者的消息队列中
for (let key in myData) {
  messageQueue[key] = new Subject();
}

// 订阅 value 值的变化
Observer(messageQueue, "value", value => {
  console.warn("value updated:", value);
});

// 更新数据
myData.value = "hello world.";
myData.value = 100;
myData.value = true;
```

可以在 codepen 上查看运行效果：[codepen.io/nachao/pen/…](https://link.juejin.cn/?target=https%3A%2F%2Fcodepen.io%2Fnachao%2Fpen%2FbGbjpJz "https://codepen.io/nachao/pen/bGbjpJz")

以上，我们已经完成双向绑定中的数据层的功能，现在任何的数据变化，我们都可以及时的知道，并关联任何想要做的事情，比如更新视图。

接下来就是模板的操作了，也就是 Compile 模板解析功能。

### 模板解析（实现视图到数据的绑定）

对于 DOM 的操作，目前常见的方式有：

- 原生或者基于库的 DOM 操作；
- 将 DOM 转换为 Virtual DOM，然后进行对比与更新；
- 使用原生的 Web Component 技术；

以上三种的差异与性能等，之后有机会再单独分享。在 Vue 中使用的是 Virtual DOM 的方式，因为它比直接操作 DOM 所消耗的性能要少很多，也不存在 Web Component 的兼容性。

这三中方式的本质都是更新 DOM 的展示效果，只是方式不同而已，为了更简单的说明双向绑定的原理，我们就采用第一种方式。虚拟 DOM 是有很多独立的第三方库，如果有兴趣同学可以去研究哦。

#### 解析器与 DOM 操作

这里的主要任务是：

- 解析模板中所有的特定特性，例如：v-model、v-text、{{ }}语法等；
- 关联数据展示到 DOM；
- 关联事件绑定到 DOM；

因此，下面的功能只能需要满足：展示数据到模板上。

代码如下：

```xml
<div id="app">
  <input v-model="value" />
  <p v-text="value"></p>
</div>
```

```js
// 模板解析
function Compile(el, data) {

  // 关联自定义特性
  if (el.attributes) {
    [].forEach.call(el.attributes, attribute => {
      if (attribute.name.includes('v-')) {
        Update[attribute.name](el, data, attribute.value);
      }
    });
  }

  // 递归解析所有DOM
  [].forEach.call(el.childNodes, child => Compile(child, data));
}

// 自定义特性对应的事件
const Update = {
  "v-text"(el, data, key) {

    // 初始化DOM内容
    el.innerText = data[key];
  },
  "v-model"(input, data, key) {

    // 初始化Input默认值
    input.value = data[key];

    // 监听控件的输入事件，并更新数据
    input.addEventListener("keyup", e => {
      data[key] = e.target.value;
    });
  }
};

// ====

// 数据
const myData = { value: "hello world." };

// 解析
Compile(document.querySelector("#app"), myData);
```

可以在 codepen 上查看运行效果：[codepen.io/nachao/pen/…](https://link.juejin.cn/?target=https%3A%2F%2Fcodepen.io%2Fnachao%2Fpen%2FMWgBexm "https://codepen.io/nachao/pen/MWgBexm")

目前我们定义的 DOM 解析，并没有关联数据监听，让我们来完成它！

### 完整的双向绑定

代码如下：

```js
// 发布者
function Subject() {
  this.observers = [];
  this.attach = function(callback) {
    this.observers.push(callback);
  };
  this.notify = function(value) {
    this.observers.forEach(callback => callback(value));
  };
}

// 订阅者
function Observer(queue, key, callback) {
  queue[key].attach(callback);
}

// ====

// 数据拦截器 - 代理方式
function ProxyWatcher(data, queue) {
  return new Proxy(data, {
    get: (target, key) => target[key],
    set(target, key, value) {
      target[key] = value;

      // 通知此值的所有订阅者，数据发生了更新
      queue[key].notify(value);
    }
  });
}

// ====

// 模板解析
function Compile(el, data) {

  // 关联自定义特性
  if (el.attributes) {
    [].forEach.call(el.attributes, attribute => {
      if (attribute.name.includes('v-')) {
        Update[attribute.name](el, data, attribute.value);
      }
    });
  }

  // 递归解析所有DOM
  [].forEach.call(el.childNodes, child => Compile(child, data));
}

// 自定义特性对应的事件
const Update = {
  "v-text"(el, data, key) {

    // 初始化DOM内容
    el.innerText = data[key];

    // 创建一个数据的订阅，数据变化后更新展示内容
    Observer(messageQueue, key, value => {
        el.innerText = value;
    });
  },
  "v-model"(input, data, key) {

    // 初始化Input默认值
    input.value = data[key];

    // 监听控件的输入事件，并更新数据
    input.addEventListener("keyup", e => {
      data[key] = e.target.value;
    });

    // 创建一个订阅
    Observer(messageQueue, key, value => {
      input.value = value;
    });
  }
};

// ====

// 消息队列
const messageQueue = {};

// 数据
const myData = ProxyWatcher({ value: "hello world." }, messageQueue);

// 将每个数据属性都添加到观察者的消息队列中
for (let key in myData) {
    messageQueue[key] = new Subject();
}

// ====

// 解析+关联
Compile(document.querySelector("#app"), myData);
```

可以在 codepen 上查看运行效果：[codepen.io/nachao/pen/…](https://link.juejin.cn/?target=https%3A%2F%2Fcodepen.io%2Fnachao%2Fpen%2FWNeKxBx "https://codepen.io/nachao/pen/WNeKxBx")

如此，一个非常简单的 MVVM 功能就完成了。当然，它仅仅是为了讲解原理非编写的，如果要做成向 Vue 这样的成熟框架，就需要将各个核心封装成模块，进行更扩展性的定义。

如果再简单的封装一下，看上去就是一个极简的 Vue 了。[codepen.io/nachao/pen/…](https://link.juejin.cn/?target=https%3A%2F%2Fcodepen.io%2Fnachao%2Fpen%2FExYpgPp "https://codepen.io/nachao/pen/ExYpgPp")

代码如下：

```js
// 观察者功能
// 发布者
function Subject() {
  this.observers = [];
  this.attach = function(callback) {
    this.observers.push(callback);
  };
  this.notify = function(value) {
    this.observers.forEach(callback => callback(value));
  };
}
// 订阅者
function Observer(queue) {
  this.queue = queue
  this.add = function(key, callback) {
    this.queue[key].attach(callback);
  }
}

// ====

// 数据拦截器
// 监听数据更新 - 代理方式
function ProxyWatcher(data, queue) {
  return new Proxy(data, {
    get: (target, key) => target[key],
    set(target, key, value) {
      target[key] = value;

      // 通知此值的所有订阅者，数据发生了更新
      queue[key].notify(value);
    }
  });
}

// ====

// 模板解析
function Compile(el, vm) {

  // 关联自定义特性
  if (el.attributes) {
    [].forEach.call(el.attributes, attribute => {
      if (attribute.name.includes('v-')) {
        Update[attribute.name](el, vm.data, attribute.value, vm);
      }
    });
  }

  // 递归解析所有DOM
  [].forEach.call(el.childNodes, child => Compile(child, vm));

  return el
}

// 自定义特性对应的事件
const Update = {
  "v-text"(el, data, key, vm) {

    // 初始化DOM内容
    el.innerText = data[key];

    // 创建一个数据的订阅，数据变化后更新展示内容
    vm.observer.add(key, value => {
      el.innerText = value;
    });
  },
  "v-model"(input, data, key, vm) {

    // 初始化Input默认值
    input.value = data[key];

    // 创建一个订阅
    vm.observer.add(key, value => {
      input.value = value;
    });

    // 监听控件的输入事件，并更新数据
    input.addEventListener("keyup", e => {
      data[key] = e.target.value;
    });
  }
};

// ====

// 封装
function Vue({ el, data }) {

  // initProxy
  this.messageQueue = {};
  this.observer = new Observer(this.messageQueue)
  this.data = ProxyWatcher(data, this.messageQueue);

  // initState
  for (let key in myData) {
    this.messageQueue[key] = new Subject();
  }

  // initRender
  // initEvents
  this.el = Compile(el, this);
}

// ====

// 数据
const myData = { value: "hello world." };

// 实例
const vm = new Vue({
  el: document.querySelector("#app"),
  data: myData
});
```
