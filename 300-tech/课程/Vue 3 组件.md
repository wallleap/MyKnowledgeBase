---
title: Vue 3 组件
date: 2023-08-15 13:49
updated: 2023-08-15 16:23
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - Vue 3 组件
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

## 组件注册

全局组件：可以在任何组件内使用

```html
<div id="app">
  全局组件在任何组件内都能使用
  <component-a></component-a>
  <component-b></component-b>
</div>
<script src="https://unpkg.com/vue@next"></script>
<script>
  const app = Vue.createApp({})
  app.component('component-a', {
    template: `
      <div>a</div>
      <component-b></component-b>
    `
  })
  app.component('ComponentB', {
    template: `<div>b</div>`
  })
  app.mount('#app')
</script>
```

局部组件：只能在该组件内使用

```html
<div id="app">
  组件内子组件只能在该组件内使用
  <component-a></component-a>
  <!-- component-b 只能在component-a内使用 -->
  <!-- <component-b></component-b> -->
</div>
<script src="https://unpkg.com/vue@next"></script>
<script>
  const ComponentB = {
    template: `<div>ComponentB</div>`
  }
  const ComponentA = {
    components: {
      // 'component-b': ComponentB,
      ComponentB // 简写
    },
    template: `
      <div>ComponentA</div>
      <component-b></component-b>
    `
  }

  Vue.createApp({
    components: { 
      ComponentA
    }
  }).mount('#app')
</script>
```

组件命名：

- kebab-case 短横线连接全小写的单词
	- 声明时使用 kebab-case，模板里必须用 kebaba-case
- PascalCase 多个首字母大写单词连接
	- 声明时使用驼峰写法，模板里两者都可以

## 父传子-props

子组件通过 props 接收数据

```html
<div id="app">
  <user v-bind="user"></user>
  <user v-bind:username="user.username" :id="[1,2,3]"></user>
</div>
<script src="https://unpkg.com/vue@next"></script>
<script>
  const User = {
    props: ['username', 'id'],
    template: `<div>{{username}}, {{id}}</div>`
  }

  Vue.createApp({
    components: { User },
    data() {
      return { 
        user: {
          username: 'hunger',
          id: 1
        }
    }
    }
  }).mount('#app')
</script>
```

几种 props 写法范例

```js
// 数组
props: ['name', 'other']
// 对象
props: {
  name: String, // 类型
  other: Number // Boolean, Array, Object, Function
}
// 更多配置
props: {
  name: {
    type: String,
    required: true,
    default: 'nickname'
  }
}
// 在 script setup 中直接使用 defineProps，不需要引入
const props = defineProps({
  msg: String
})
```

父组件中通过属性传递数据，如果要传入的不是字符串，则需要使用 `v-bind` 绑定

```html
<User name="luwang" />
<User :age="23" :calc="23+1" />
<User :show="true" />
```

如果传的是对象，可以整个传入，`v-bind` 后不接 `property`

```html
post: {
  id: 1,
  title: 'First Post'
}
<post-card v-bind="post" />
<!-- 这两个等价 -->
<post-card v-bind:id="post.id" v-bind:title="post.title" />
```

非 Prop 的 Attribute 将会自动加到子组件的根节点的属性中（例如 class、id、data-attr 等）

- 在子组件中可以通过 `$attrs`/`this.$attrs` 获取 attributes

```html
<div id="app">
  <user class="username" :data-user="username"></user>
</div>
<script src="https://unpkg.com/vue@next"></script>
<script>
  const User = {
    template: `<div>{{$attrs['data-user']}}</div>`,
    created() {
      console.log(this.$attrs['data-user'])
    }
  }

  Vue.createApp({
    components: { User },
    data() {
      return { username: 'hunger' }
    }
  }).mount('#app')
</script>
```

- 如果想在非根节点应用传递的 attribute，可以使用 `v-bind="$attrs"`

```html
<div id="app">
  <username class="username" :error="errorMsg" @input="onInput"></username>
</div>
<script src="https://unpkg.com/vue@next"></script>
<script>
  const Username = {
    props: ['error'],
    template: `
    <fieldset>
      <legend>用户名</legend>
      <input v-bind="$attrs">
      <div>{{error}}</div>
    </fieldset>
    `
  }

  Vue.createApp({
    components: { Username },
    data() {
      return {  errorMsg: '' }
    },
    methods: {
      onInput(e){
        this.errorMsg = e.target.value.length<6?"长度不够":""
      }
    }
  }).mount('#app')
</script>
```

## 子传父-自定义事件

- 子组件内触发事件用 `this.$emit('my-event')`
- 父组件使用子组件时绑定 `<child @my-event="doSomething"></child>`
- 事件推荐使用驼峰命名

```html
<div id="app">
  <h1>{{username}}</h1>
  <user class="username" :user="username" @change-user="username=$event"></user>
</div>
<script src="https://unpkg.com/vue@next"></script>
<script>
  const User = {
    props: ['user'],
    events: ['change-user'],
    template: `<div>{{user}} <button @click="updateUser">update</button></div>`,
    methods: {
      updateUser() {
        this.$emit('change-user', this.user + '!')
      }
    }
  }

  Vue.createApp({
    components: { User },
    data() {
      return { username: 'hunger' }
    }
  }).mount('#app')
</script>
```

`v-model` 语法糖：

- `<comp v-model:foo="bar"></comp>` 等价于 `<comp :foo="bar" @update:foo="bar=$event"></comp>`
- 事件名是 `update:my-event`

```html
<div id="app">
  <h1>{{username}}</h1>
  <user v-model:user="username" v-model:id="id"></user>
  <user :user="username" @update:user="username=$event"></user>
</div>
<script src="https://unpkg.com/vue@next"></script>
<script>
  const User = {
    // events: ['update:user', 'update:id'] // 对于 v-model 可以不写 events
    props: ['user', 'id'],
    template: `<div>{{user}}  {{id}}<button @click="updateUser">update</button></div>`,
    methods: {
      updateUser() {
        this.$emit('update:user', this.user + '!')
        this.$emit('update:id', this.id++)
      }
    }
  }

  Vue.createApp({
    components: { User },
    data() {
      return { username: 'hunger', id: 100 }
    }
  }).mount('#app')
</script>
```

setup 语法糖

```vue
<!-- 父组件 -->
<script setup>
import Child from './child.vue'
const getMessage = (msg) => {
  console.log(msg)
}
</script>
<template>
  <Child @get-message="getMessage" />
</template>

<!-- 子组件 -->
<script setup>
const emit = defineEmits(['get-message'])
const sendMsg = () => {
  emit('get-message', '传递的消息')
}
</script>
<template>
  <button @click="sendMsg">Click</button>
</template>
```

## 插槽 slot

- 子组件的模板中预留一个空位（slot）
- 父组件使用子组件时可以在子组件标签内插入内容/组件，即向子组件内预留的空位插入
- 父组件往插槽插入的内容只能使用父组件实例的属性

```html
<div id="app">
  <x-button> <icon name="no"></icon> {{text}} </x-button>
</div>
<script src="https://unpkg.com/vue@next"></script>

<script>
  const Icon = {
    props: ['name'],
    template: `<span>{{type}}</span>`,
    computed: {
      type() { return this.name==='yes'?'✔':'✘' }
    }
  }
  const XButton = {
    template: `<button>
      <slot></slot>
    </button>`
  }
  Vue.createApp({
    components: { XButton, Icon },
    data() {
      return { text: '正确' }
    }
  }).mount('#app')
</script>
```

插槽默认值：可以在 slot 中写内容，如果父组件中加了内容则使用相应的内容，如果没有就用默认值

```html
<div id="app">
  <x-button type="big"> OK </x-button>
  <x-button type="small"></x-button>
  <x-button type="default"> {{type}} </x-button>
</div>
<script src="https://unpkg.com/vue@next"></script>
<script>
  const XButton = {
    props: ['type'],
    template: `<button :class="'btn-'+type">
      <slot>确定</slot>
    </button>`
  }
  Vue.createApp({
    components: { XButton },
  }).mount('#app')
</script>
<style>
  .btn-big {
    padding: 8px 16px;
  }
  .btn-small {
    padding: 2px 4px;
  }
</style>
```

具名插槽：给插槽起名字，父组件传的时候加上标识（`v-slot:xxx`、简写 `#xxx`）

```html
<div id="app">
  <layout>
    <template v-slot:header>
      <h1>页面header</h1>
    </template>
    <template #default>
      <p>页面content</p>
    </template>
    <template #footer>
      <div>页面footer</div>
    </template>
  </layout>
</div>
<script src="https://unpkg.com/vue@next"></script>
<script>
  const Layout = {
    template: `<div class="container">
      <header> <slot name="header"></slot> </header>
      <main> <slot></slot></main>
      <footer><slot name="footer"></slot></footer>
    </div>`
  }
  Vue.createApp({
    components: { Layout },
  }).mount('#app')
</script>
```

作用域插槽：子组件的 slot 上绑定，父组件上使用

```html
<div id="app">
  <news>👉hello world</news>
  <news v-slot="props">👉 {{props.item}}</news>
  <news v-slot="props">👉第{{props.index}}章 {{props.item}}</news>
  <news v-slot="{item, index}">✔第{{index}}章 {{item}}</news>
  <!-- <news v-slot="{ item }">✔ {{item}}</news>  -->
</div>
<script src="https://unpkg.com/vue@next"></script>
<script>
  const News = {
    data() { return { news: ['first news', 'second news'] } },
    template: `<ul>
      <li v-for="(item, index) in news">
        <slot :item="item" :index="index"></slot>  
      </li>
    </ul>`
  }
  Vue.createApp({
    components: { News },
  }).mount('#app')
</script>
```

## Provide & Inject 爷孙数据传值

Provide Inject 适用与深度嵌套的组件，父组件可以为所有的子组件直接提供数据

![](https://cdn.wallleap.cn/img/pic/illustration/202308151606110.png)

```html
<div id="app">
  <toolbar></toolbar>
  <button @click="isDark=!isDark">切换</button>
</div>
<script src="https://unpkg.com/vue@next"></script>
<script>
  const ThemeButton = {
    inject: ['theme'],
    template: `
    <div :class="['button', theme.value]" ><slot></slot></div>
  `
  }
  const Toolbar = {
    components: { ThemeButton },
    inject: ['theme'],
    template: `<div :class="['toolbar', theme.value]">
      <theme-button>确定</theme-button>
    </div>`
  }

  Vue.createApp({
    data() {
      return { isDark: false }
    },
    //provide: { theme: 'dark'},
    // provide() {
    //   return { theme: this.isDark?'dark':'white' }
    // }, // 上面并不是响应式的，但是 theme 就是值，直接用 theme 就行，下面是对象，用 theme.value
    provide() {
      return { theme: Vue.computed(()=>this.isDark?'dark':'white') }
    },
    components: { Toolbar },
  }).mount('#app')
</script>
<style>
  .toolbar {
    border: 1px solid #333;
    color: #333;
    padding: 10px;
  }

  .toolbar.dark {
    background: #333;
    color: #fff;
  }

  .button {
    padding: 2px 4px;
    border-radius: 4px;
    border: 1px solid #ccc;
    display: inline-block;
  }

  .button.dark {
    background: #666;
    color: #fff;
  }
</style>
```

setup 语法糖

```ts
// 顶层组件提供
provide('key', 顶层组件中的数据)

// 底层组件获取
const msg = inject('key)
```

## 动态组件与 keep-alive

动态组件：当前不确定，共用一个组件

```html
<div id="app">
  <button v-for="tab in tabs" :key="tab" :class="{ active: currentTab === tab }"
    @click="currentTab = tab">
    {{ tab }}
  </button>
  <!-- 动态组件 -->
  <component :is="currentTab" class="tab"></component>
</div>
<script src="https://unpkg.com/vue@next"></script>
<script>
  const app = Vue.createApp({
    data() {
      return {
        currentTab: 'Tab1',
        tabs: ['Tab1', 'Tab2']
      }
    },
  })
  app.component('Tab1', {
    template: `<div>Tab1 content</div>`,
    created() { console.log('tab1 created') }
  })
  app.component('Tab2', {
    template: `<div>
      <input v-model="value" /> {{value}}
    </div>`,
    data() { return { value: 'hello' } },
    created() { console.log('tab2 created') }
  })
  app.mount('#app')
</script>
<style>
  .active { background: #e0e0e0; }
</style>
```

使用 keep-alive 包裹后会缓存，就不会再次创建了

- 组件第一次进入，钩子的触发 created → mounted → activated
- 退出时触发 deactivated
- 再次进入时只触发 activated

```html
<div id="app">
  <button v-for="tab in tabs" :key="tab" :class="{ active: currentTab === tab }"
    @click="currentTab = tab">
    {{ tab }}
  </button>
  <keep-alive>
    <component :is="currentTab" class="tab"></component>
  </keep-alive>
</div>
<script src="https://unpkg.com/vue@next"></script>
<script>
  const app = Vue.createApp({
    data() {
      return {
        currentTab: 'Tab1',
        tabs: ['Tab1', 'Tab2']
      }
    },
  })
  app.component('Tab1', {
    template: `<div>Tab1 content</div>`,
    created() { console.log('tab1 created') },
    activated() { console.log('tab1 activated') }
  })
  app.component('Tab2', {
    template: `<div>
      <input v-model="value" /> {{value}}
    </div>`,
    data() { return { value: 'hello' } },
    created() { console.log('tab2 created') },
    activated() { console.log('tab2 activated') }
  })
  app.mount('#app')
</script>
<style>
  .active { background: #e0e0e0; }
</style>
```

## 通过 ref 获取真实的 DOM 对象或者组件实例对象

```vue
<script setup>
import { onMounted, ref } from 'vue'
import Child from './child.vue'
const titleRef = ref(null) // 变量名和下面的 ref 属性值相同，在这里定义，但使用需要在 onMounted 中（等待 DOM 元素加载完）
onMounted(() => {
  console.log(titleRef.value)
})
</script>
<template>
  <h1 ref="titleRef">我是标题</h1>
  <Child ref="comRef" />
</template>

<!-- 如果需要暴露数据，可以在子组件中使用 define -->
const name = ref('test name')
defineExpose({
  name
})
```


