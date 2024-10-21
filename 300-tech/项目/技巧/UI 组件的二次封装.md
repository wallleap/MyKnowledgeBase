---
title: UI 组件的二次封装
date: 2024-09-30T10:30:27+08:00
updated: 2024-10-09T09:43:33+08:00
dg-publish: false
---

1. **属性**的传递：会默认透传到组件的根元素上，在 3.3 开始可以直接禁用自动继承，然后再手动绑定（通过 $attrs）
2. **插槽**的保留：通过 [$slots](https://cn.vuejs.org/api/component-instance.html#slots) 获取传递的插槽，或者直接使用 h 函数（`h(虚拟节点, useAttrs(), useSlots())`，在模板中可以直接用动态组件）
3. ref 引用，让它在 el-input 上（React 很容易实现），或把所有**方法**提到组件最外层

可以使用动态组件直接传（h 函数返回的是 VNode），写成函数就是一个函数组件 `FunctionalComponent`

```vue
<componet :is="h(Comp, $attrs, $slots)"></component>
<componet :is="() => h(Comp, $attrs, $slots)"></component>


const Component: FunctionalComponent<{ count: number }> = (props, { slots, context }) => {
  const a = ref('aaa')
  return h('div', { style: { color: 'red' }, onClick() {console.log('click')} }, [slots?.header?.(a), 'container', slots?.default?.()])
}
<Component :count="1">
  <div>1111</div>
  <template #header={a}>
    <div>header {{ a }}</div>
  </template>
</Component>
```

`MyInput.vue`

```vue
<script setup>
import { ref, onMounted } from 'vue'
/* import { useAttrs } from 'vue'
const attrs = useAttrs() */
defineOptions({
  inheritAttrs: false
})

/*
const props = defineProps<{ type: string }>()
声明的属性就不会在 attrs 里了
*/

const inputRef = ref(null)

defineExpose(new Proxy({
  value: inputRef
}, {
  get(target, key) {
    return inputRef.value?.[key]
  },
  has(target, key) {
    return key in inputRef.value
  }
}))
</script>

<template>
  <div class="my-input">
    <el-input ref="inputRef" v-bind="$attrs">
      <template v-for="(value, name) in $slots" :key="name" #[name]="slotProps">
        <slot :name="name" v-bind="slotProps || {}" />
      </template>
    </el-input>
  </div>
</template>

<style scoped>
.my-input {
  width: 100%;
  transition: .3s;
}

.my-input:hover, .my-input:focus-within {
  filter: drop-shadow(0 0 10px rgba(0, 0, 0, 0.1));
}
</style>
```

`App.vue`

```vue
<script setup>
import MyInput from './components/MyInput.vue'
import { ref, onMounted } from 'vue'

const value = ref('')
const myInputRef = ref()

onMounted(() => {
  console.log(myInputRef.value)
})
</script>

<template>
  <el-button @click="myInputRef.focus()">Focus {{value}}</el-button>
  <MyInput ref="myInputRef" v-model="value" class="my-input1111" placeholder="请输入内容">
    <template #prepend>
      <el-button>前置</el-button>
    </template>
    <template #suffix>
      <el-icon>
        <Search />
      </el-icon>
    </template>
  </MyInput>
</template>
```

## 类型

`MyButton.vue`

```vue
<script setup lang="ts">
import { useAttrs, ref } from 'vue'
import type { ButtonProps } from 'element-plus'

defineOptions({
  inheritAttrs: false,
})

const loading = ref(false)
const attrs = useAttrs()
// TS 类型工具 Partial 表示所有属性都是可选的
// TS 类型工具 Omit 表示排除掉 loading 和 onClick 属性
const props = defineProps<Partial<Omit<ButtonProps, 'loading' | 'onClick'>>>()

async function handleClick() {
  loading.value = true
  try {
    // @ts-ignore
    await attrs?.onClick?.()
  } catch (error) {
    console.error(error)
  } finally {
    loading.value = false
  }
}
</script>

<template>
  <div>
    <!-- 这里使用 v-bind="$attrs" 将所有属性传递给 el-button -->
    <el-button
      :loading="loading"
      v-bind="props"
      @click="handleClick"
    >
      <slot></slot>
    </el-button>
  </div>
</template>

<style scoped>
</style>
```

`App.vue`

```vue
<script setup>
import MyButton from './components/MyButton.vue'

async function onClick() {
  await new Promise(resolve => setTimeout(resolve, 2000))
  console.log('click')
}
</script>

<template>
  <MyButton type="primary" @click="onClick">按钮</MyButton>
</template>
```
