---
title: Vue-Vue-Mixin数据管理(model层)
date: 2024-08-22T02:18:28+08:00
updated: 2024-08-22T02:18:47+08:00
dg-publish: false
---

## 背景

我们都习惯性的将接口请求，数据逻辑处理放在 vue 文件中，比如最常见的 分页表格数据 ， 基础表单显示，每一个页面中都声明了非常多的 total ， pageSize ， currentPage ，tableData 等字段，如何减少**反复 CV， 提高公共逻辑复用性** ？

## 首先要明白

1. Vuex 是状态管理，是为了解决跨组件之间数据共享问题的，一个组件的数据变化会映射到使用这个数据的其他组件当中
2. vuex 的状态存储是响应式的, 当应用遇到多个组件共享状态时候，即：多个视图依赖于同一个状态，不同视图的行为需要变更同一状态。
3. vuex 一般会在全局生效, 依赖他的组件都会更新

> 如果对混入的页面生效，可以对 mixin 进行了包装，抽象成为了一个 model 层，这个 model 层的作用主要是处理菜单级页面的异步数据的流向打造的，视图页面数据在 .vue 中声明， 后端数据 在 model.js 中

**基本的 model.js**

```js
export default {
  namespace: 'Home',
  state: {
    move: {},
    a: 1,
    b: 2
  },
  actions: {
    /**
     *
     * @param { Object } state
     * @param { Object } payload
     * @param { function } set
     */
    async setUser (state, { payload, set }) {
      const data = await test()
      await set(state, 'move', data)
      return state.move
    }
  }
}

const test = () => {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve({
        user: 'wangly',
        age: 21
      })
    }, 2000)
  })
}


```

**实现方式:**

使用 vue.observableAPI 和 mixins，mixins 主要是去做初始化和销毁

使用 createModelComponent 会在组件挂载 useData 属性，访问 model 里的 state 和 useDispatch 方法，访问 model 里的方法

### createModelComponent.js

```js
/**
 * model层专注数据处理
 * 抽离model要实现的目标
 * 1.抽离model层业务
 * 2.数据共享
 */

import Vue from 'vue'

/**
 * 是否是被监听的对象
 * @param {object} state
 * @returns {boolean}
 */
function isObservable(state) {
  return hasOwnProperty(state, '__ob__')
}

/**
 * useData计算属性
 * @param {object} model
 * @returns {object}
 */
function computedUseData(model) {
  return {
    get: () => model.state,
    set() {
      throw ('do not set useData')
    }
  }
}

/**
 * 对象本身是否该属性
 * @param {object} obj
 * @param {string} property
 * @returns {boolean}
 */
function hasOwnProperty(obj, property) {
  return Object.hasOwnProperty.call(obj, property)
}

/**
 * 分发model层actions事件
 * @param {object} model
 * @returns {function}
 */
function useDispatch(model) {
  return function (actionName, payload) {
    if (!hasOwnProperty(model, actionName)) {
      throw (`${actionName} not defined`)
    }

    if (typeof model[actionName] !== 'function') {
      throw (`${actionName} not function`)
    }

    return model[actionName](payload)
  }
}

/**
 * 深拷贝
 * @param {object} obj
 * @param {map} hash
 */
function deepClone(obj, hash = new WeakMap()) {
  if (obj === null) return obj
  if (obj instanceof Date) return new Date(obj)
  if (obj instanceof RegExp) return new RegExp(obj)

  // 可能是对象或者普通的值  如果是函数的话是不需要深拷贝
  if (typeof obj !== 'object') return obj

  // 是对象的话就要进行深拷贝
  if (hash.get(obj)) return hash.get(obj)
  let cloneObj = new obj.constructor()

  // 找到的是所属类原型上的constructor,而原型上的 constructor指向的是当前类本身
  hash.set(obj, cloneObj)
  for (let key in obj) {
    if (obj.hasOwnProperty(key)) {
      // 实现一个递归拷贝
      cloneObj[key] = deepClone(obj[key], hash)
    }
  }

  return cloneObj
}

/**
 * 在组件生命周期里初始化 model
 * @param {objec} model
 * @returns {object}
 */
function initModelMixin(model) {
  return {
    beforeCreate() {
      if (model.state && !isObservable(model.state)) {
        const state = Vue.observable(model.state)
        model.state = state
      }
    },
    destroyed() {
      model.state = deepClone(model.$state)
    }
  }
}

/**
 * 创建model层组件
 * 组件挂载useData属性访问model层的state
 * @param {object} component 组件
 * @param {object} model 组件的model层
 * @returns {object}
 */
export function createModelComponent(component, model = {}) {
  let newComponent = { ...component }

  if (model.state) {
    if (!model.$state) {
      model.$state = deepClone(model.state)
    }

    newComponent.computed = {
      ...component.computed,
      useData: computedUseData(model)
    }
  }

  newComponent.methods = {
    ...component.methods,
    useDispatch: useDispatch(model)
  }

  const mixins = component.mixins ? component.mixins : []
  newComponent.mixins = [initModelMixin(model), ...mixins]

  return newComponent
}
```

### 使用

```vue
<template>
  <div class="home">
    {{ useData.userName }}
    <img alt="Vue logo" src="../assets/logo.png" />

    <button @click="handleClick">actions</button>

    <button @click="handleClick3">actions3</button>

    <HelloWorld msg="Welcome to Your Vue.js App" />
  </div>
</template>
<script>
import HelloWorld from '@/components/HelloWorld.vue'
import { createModelComponent } from '@/libs/createModelComponent'
import homeModel from './homeModel'

export default createModelComponent(
  {
    name: 'Home',
    components: {
      HelloWorld,
    },
    mounted() {
      console.log(this)
    },
    methods: {
      handleClick() {
        this.useData.userName = 'Gavin'
      },
      handleClick3() {
        this.useDispatch('getUerInfo')
      },
    },
    destroyed() {
      console.log('destory')
    },
  },
  homeModel
)
</script>

```

## homeMdel.js

```js
/**
 * this的指向本身对象
 * 不能访问到组件本身，这也是model层的作用，专注数据处理和数据共享
 */

export default {
  state: {
    userName: ''
  },
  async getUerInfo() {
    this.state.userName = 'Gavin 2'
  }
}
```
