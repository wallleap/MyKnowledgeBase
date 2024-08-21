---
title: 这 9 种 Vue 技术你掌握了吗
date: 2023-08-04T04:18:00+08:00
updated: 2024-08-21T10:33:31+08:00
dg-publish: false
aliases:
  - 这 9 种 Vue 技术你掌握了吗
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

> 本文首发于公众号《前端全栈开发者》，第一时间阅读最新文章，会优先两天发表新文章。关注后私信回复：大礼包，送某网精品视频课程网盘资料，准能为你节省不少钱！

现在，Vue.js 已成为前端开发的热门框架。有很多工程师利用 Vue.js 的便利性和强大功能。但是，我们完成的某些解决方案可能未遵循最佳做法。好吧，让我们看一下那些必备的 Vue 技术。

## 1\. 函数组件

[函数组件](https://link.juejin.cn/?target=https%3A%2F%2Fvuejs.org%2Fv2%2Fguide%2Frender-function.html%23Functional-Components "https://vuejs.org/v2/guide/render-function.html#Functional-Components") 是无状态的，没有生命周期或方法，因此无法实例化

创建一个函数组件非常容易，你需要做的就是在 SFC 中添加一个 `functional: true` 属性，或者在模板中添加 `functional`。由于它像函数一样轻巧，没有实例引用，所以渲染性能提高了不少。

函数组件依赖于上下文，并随着其中给定的数据而突变。

```vue
<template functional>
  <div class="book">
    {{props.book.name}} {{props.book.price}}
  </div>
</template>

<script>
Vue.component('book', {
  functional: true,
  props: {
    book: {
      type: () => ({}),
      required: true
    }
  },
  render: function (createElement, context) {
    return createElement(
      'div',
      {
        attrs: {
          class: 'book'
        }
      },
      [context.props.book]
    )
  }
})
</script>
```

## 2.深层选择器

有时，你需要修改第三方组件的 CSS，这些都是 `scoped` 样式，移除 `scope` 或打开一个新的样式是不可能的。

现在，[深层选择器](https://link.juejin.cn/?target=https%3A%2F%2Fvue-loader.vuejs.org%2Fguide%2Fscoped-css.html%23child-component-root-elements "https://vue-loader.vuejs.org/guide/scoped-css.html#child-component-root-elements") `>>>` `/deep/` `::v-deep` 可以帮助你。

```vue
<style scoped>
>>> .scoped-third-party-class {
  color: gray;
}
</style>

<style scoped>
/deep/ .scoped-third-party-class {
  color: gray;
}
</style>

<style scoped>
::v-deep .scoped-third-party-class {
  color: gray;
}
</style>
```

## 3.高级“watcher”

### 立即执行

> 相关阅读：[Vue.js中 watch 的高级用法](https://juejin.cn/post/6844903600737484808 "https://juejin.cn/post/6844903600737484808")

当被监控的 prop 发生突变时，watch handler 就会触发。但有时，它是在组件被创建后才出现的。

是的，有一个简单的解决方案：在 `created` 的钩子中调用处理程序，但这看起来并不优雅，同时也增加了复杂性。

或者，你可以向观察者添加 `immediate` 属性：

```js
watch: {
  value: {
    handler: 'printValue',
      immediate: true
  }
},
methods : {
  printValue () {
    console.log(this.value)
  }
}
```

### 深度监听

有时，watch 属性是一个对象，但是其属性突变无法触发 wacher 处理程序。在这种情况下，为观察者添加 `deep:true` 可以使其属性的突变可检测到。

请注意，当对象具有多个层时，深层可能会导致一些严重的性能问题。最好考虑使用更扁平的数据结构。

```js
data () {
  return {
    value: {
      one: {
        two: {
          three: 3
        }
      }
    }
  }
},
watch: {
  value: {
    handler: 'printValue',
    deep: true
  }
},
methods : {
  printValue () {
    console.log(this.value)
  }
}
```

### 多个 handlers

实际上，watch 可以设置为数组，支持的类型为 String、Function、Object。触发后，注册的 watch 处理程序将被一一调用。

```js
watch: {
  value: [
    'printValue',
    function (val, oldVal) {
      console.log(val)
    },
    {
      handler: 'printValue',
      deep: true
    }
  ]
},
methods : {
  printValue () {
    console.log(this.value)
  }
}
```

### 订阅多个变量突变

`watcher` 不能监听多个变量，但我们可以将目标组合在一起作为一个新的 `computed`，并监视这个新的 " 变量 "。

```js
computed: {
  multipleValues () {
    return {
      value1: this.value1,
      value2: this.value2,
    }
  }
},
watch: {
  multipleValues (val, oldVal) {
    console.log(val)
  }
}
```

## 4.事件参数：$event

`$event` 是事件对象的一个特殊变量。它在某些场景下为复杂的功能提供了更多的可选参数。

### 原生事件

在原生事件中，该值与默认事件（DOM 事件或窗口事件）相同。

```js
<template>
  <input type="text" @input="handleInput('hello', $event)" />
</template>

<script>
export default {
  methods: {
    handleInput (val, e) {
      console.log(e.target.value) // hello
    }
  }
}
</script>
```

### 自定义事件

在自定义事件中，该值是从其子组件中捕获的值。

```vue
<!-- Child -->
<template>
  <input type="text" @input="$emit('custom-event', 'hello')" />
</template>

<!-- Parent -->
<template>
  <Child @custom-event="handleCustomevent" />
</template>

<script>
export default {
  methods: {
    handleCustomevent (value) {
      console.log(value) // hello
    }
  }
}
</script>
```

## 5.路由器参数解耦

我相信这是大多数人处理组件中路由器参数的方式：

```js
export default {
  methods: {
    getRouteParamsId() {
      return this.$route.params.id
    }
  }
}
```

在组件内部使用 `$route` 会对某个 URL 产生强耦合，这限制了组件的灵活性。

正确的解决方案是向路由器添加 props。

```js
const router = new VueRouter({
  routes: [{
    path: '/:id',
    component: Component,
    props: true
  }]
})
```

这样，组件可以直接从 props 获取 `params`。

```js
export default {
  props: ['id'],
  methods: {
    getParamsId() {
      return this.id
    }
  }
}
```

此外，你还可以传入函数以返回自定义 `props`。

```js
const router = new VueRouter({
  routes: [{
    path: '/:id',
    component: Component,
    props: router => ({ id: route.query.id })
  }]
})
```

## 6.自定义组件的双向绑定

> 允许自定义组件在使用 `v-model` 时自定义使 props 和 event。默认情况下，组件上的 v-model 使用 value 作为属性，Input 作为事件，但一些输入类型，如复选框和单选按钮可能希望使用 value 属性来实现不同的目的。在这种情况下，使用 model 选项可以避免冲突。

`v-model` 是众所周知的双向绑定。`input` 是默认的更新事件。可以通过 `$emit` 更新该值。唯一的限制是该组件需要 `<input>` 标记才能与 `value` 属性绑定。

```vue
<my-checkbox v-model="val"></my-checkbox>

<template>
  <input type="checkbox" :value="value" @input="handleInputChange(value)" />
</template>

<script>
export default {
  props: {
    value: {
      type: Boolean,
      default: false
    }
  },
  methods: {
    handleInputChange (val) {
      console.log(val)
    }
  }
}
</script>
```

双向绑定还有另一种解决方案，即 `sync` 修饰符。与 `v-model` 不同的是，它不需要你的组件有一个 `<input>` 标签并将值绑定到它上面。它仅触发 `update:<your_prop>` 通过事件系统对属性进行突变。

```vue
<custom-component :value.sync="value" />
```

## 7.组件生命周期 Hook

通常，你可以像这样监听子组件的生命周期（例如 `mounted`）

```vue
<!-- Child -->
<script>
export default {
  mounted () {
    this.$emit('onMounted')
  }
}
</script>

<!-- Parent -->
<template>
  <Child @onMounted="handleOnMounted" />
</template>
```

还有另一种简单的解决方案，你可以改用 `@hook:mount` 在 Vue 内部系统中使用。

```vue
<!-- Parent -->
<template>
  <Child @hook:mounted="handleOnMounted" />
</template>
```

## 8.事件监听 APIs

比如，在页面挂载时增加一个定时器，但销毁时需要清除定时器。这看起来不错。

坦白地说，只有在 `beforeDestroy` 中使用 `this.timer` 来获取计时器 ID 才有意义。并非刻薄，而是变量越少，性能越好。

```js
export default {
  data () {
    return {
      timer: null
    }
  },
  mounted () {
    this.timer = setInterval(() => {
      console.log(Date.now())
    }, 1000)
  },
  beforeDestroy () {
    clearInterval(this.timer)
  }
}
```

使其只能在生命周期钩子内访问。使用 `$once` 来放弃不必要的东西。

```js
export default {
  mounted () {
    let timer = null
    timer = setInterval(() => {
      console.log(Date.now())
    }, 1000)
    this.$once('hook:beforeDestroy', () => {
      clearInterval(timer)
    })
  }
}
```

## 9.以编程方式挂载组件

在某些情况下，以编程方式加载组件要优雅得多。例如，可以通过全局上下文 `$popup()` 或 `$modal.open()` 打开弹出窗口或模态窗口。

```js
import Vue from 'vue'
import Popup from './popup'

const PopupCtor = Vue.extend(Popup)

const PopupIns = new PopupCtr()

PopupIns.$mount()

document.body.append(PopupIns.$el)

Vue.prototype.$popup = Vue.$popup = function () {
  PopupIns.open()
}
```

[Element UI](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FElemeFE%2Felement%2Fblob%2Fdev%2Fpackages%2Fmessage-box%2Fsrc%2Fmain.js "https://github.com/ElemeFE/element/blob/dev/packages/message-box/src/main.js") 实现了结构良好的模式组件，该组件允许使用自定义 API 来控制实例的生命周期。该理论与我上面演示的几乎相同。

这是有关 Vue 2.x 的 9 种技术，希望在本文中你可以对使用框架有更好的了解。如果你认为本文很棒，请在其他社交网络上分享。

## 推荐阅读

- [2020年的12个Vue.js开发技巧和窍门](https://juejin.cn/post/6844904120499830792 "https://juejin.cn/post/6844904120499830792")
- [在Vue.js中编写更好的v-for循环的6种技巧](https://juejin.cn/post/6844904121825230856 "https://juejin.cn/post/6844904121825230856")
- [Vue.js中 watch 的高级用法](https://juejin.cn/post/6844903600737484808 "https://juejin.cn/post/6844903600737484808")
- [Vue3 Composition API如何替换Vue Mixins](https://juejin.cn/post/6844904136065056781 "https://juejin.cn/post/6844904136065056781")
- [Vue3 Composition API中的提取和重用逻辑](https://juejin.cn/post/6844904135188283406 "https://juejin.cn/post/6844904135188283406")
- [如何在Vue2与Vue3中构建相同的组件](https://juejin.cn/post/6844904136589180935 "https://juejin.cn/post/6844904136589180935")
- [Vue 3教程（适用于Vue 2用户）](https://juejin.cn/post/6844904176187605000 "https://juejin.cn/post/6844904176187605000")
- [Vue3中的Vue Router初探](https://juejin.cn/post/6844904150157754376 "https://juejin.cn/post/6844904150157754376")
- [Vue技巧 | 在Vue3中使元素在滚动视图时淡入](https://juejin.cn/post/6844904195389128717 "https://juejin.cn/post/6844904195389128717")
- [10+个很酷的Vue.js组件，模板和demo示例](https://juejin.cn/post/6844904183611523085 "https://juejin.cn/post/6844904183611523085")
- [思想实验：如何在Vue中使localStorage具有响应式？](https://juejin.cn/post/6850037271065722888 "https://juejin.cn/post/6850037271065722888")
