---
title: Vue 状态仓库持久化
date: 2024-11-25T11:10:30+08:00
updated: 2024-11-25T11:32:13+08:00
dg-publish: false
---

目前 Vue 主要使用的状态管理库有

- Vuex
- Pinia

都可以通过插件实现，有对应的现成的插件可以下载使用

## Vuex

```js
import { createStore } from 'vuex'
import counter from './modules/couter'
import text from './modules/text'

const KEY = 'VUEX_STORE'
const persistPlugin = (store) => {
  // 存储
  window.addEventListener('beforeunload', () => {
    localStorage.setItem(KEY, JSON.stringify(store.state))
  })
  // 读取
  try {
    const localState = localStorage.getItem(KEY)
    if (localState) {
      store.replaceState(JSON.parse(localState))
    }
  } catch{
    console.log('本地存储数据异常')
  }
}

const store = createStore({
  modules: {
    counter,
    text
  },
  plugins: [persistPlugin]
})

export default store
```

## Pinia

```js
// persistPlugin.js
const KEY_PREFIX = 'PINIA_STORE_'
export default function(context) {
  const { store } = context
  const KEY = KEY_PREFIX + store.$id
  // 存
  window.addEventListener('beforeunload', () => {
    localStorage.setItem(KEY, JSON.stringify(store.$state))
  })
  // 读
  try {
    const localData = localStorage.getItem(KEY)
    if (localData) {
      store.$patch(JSON.parse(localData))
    }
  } catch{
    console.log('本地存储数据异常')
  }
}
```

`main.js`

```js
import { createApp } from 'vue'
import App from './App.vue'
import persistPlugin from './store/persistPlugin'
import { createPinia } from 'pinia'

const pinia = createPinia()
pinia.use(persistPlugin)
const app = createApp(App)
app.use(pinia)
app.mount('#app')
```

`store/index.js`

```js
import { defineStore } from 'pinia'
import { ref } from 'vue'

export const useCounterStore = defineStore('counter', () => {
  const count = ref(0)
  return { count }
})

export const useTextStore = defineStore('text', () => {
  const text = ref('Hello Pinia')
  return { text }
})
```
