---
title: Vue 中如何监听子组件的声明周期
date: 2024-10-09T10:50:15+08:00
updated: 2024-10-09T10:56:21+08:00
dg-publish: false
---

## 子组件中 emit 事件

```vue
<!-- 子组件 -->
<script setup>
import { onMounted } from 'vue'

const emits = defineEmits(['childMounted'])
onMounted(() => {
  emits('childMounted')
})
</script>

<template>
  <div>Hello</div>
</template>

<!-- 父组件 -->
<script setup>
import HelloWorld from '../components/HelloWorld.vue'

function childMounted() {
  console.log('HelloWorld Mounted')
}
</script>

<template>
  <HelloWorld @childMounted="childMounted"></HelloWorld>
</template>
```

子组件如果是第三方的，上面这种方式就不可以了

## Vue2 自带的

父组件中

```vue
<template>
  <HelloWorld @hook:mounted="childMounted"></HelloWorld>
</template>
```

## Vue3 迁移指南

```vue
<template>
  <HelloWorld @vue:mounted="childMounted"></HelloWorld>
</template>
```
