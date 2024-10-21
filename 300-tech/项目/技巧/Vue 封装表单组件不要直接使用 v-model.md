---
title: Vue 封装表单组件不要直接使用 v-model
date: 2024-10-09T04:32:45+08:00
updated: 2024-10-09T05:33:20+08:00
dg-publish: false
---

## 直接使用 v-model

子组件

```vue
<!-- SearchBar 搜索栏 -->
<template>
  <div class="search-bar">
    <el-input v-model="modelValue.keyword" :placeholder="modelValue.placeholder">
      <template #prepend>
        <el-select v-model="modelValue.selectValue" style="width: 100px;">
          <el-option
            v-for="item in modelValue.selectOptions"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          ></el-option>
        </el-select>
      </template>
      <template #append>
        <el-button :icon="Search" />
      </template>
    </el-input>
  </div>
</template>

<script setup>
import { Search } from '@element-plus/icons-vue'

const props = defineProps({
  modelValue: {
    type: Object,
    required: true,
  },
})
</script>
```

父组件

```vue
<script setup>
import { ref } from 'vue'
import SearchBar from './components/SearchBar.vue'

const searchBar = ref({
  keyword: '',
  placeholder: '请输入搜索内容',
  selectValue: 'user',
  selectOptions: [
    { label: '用户', value: 'user' },
    { label: '文章', value: 'article' },
  ],
})
</script>

<template>
  <SearchBar v-model="searchBar" />
</template>
```

数据传输

**父组件** 数据
↓                                          ↑
modelValue v-model      emit: update:modelValue
↓                                          ↑
**子组件**  事件

现在父组件把 modelValue 数据传到子组件，子组件直接使用 v-model 绑定到文本框，将来文本框改动直接就**修改了父组件的数据**，打破了单向数据流

## 子组件不使用 v-model

v-model 是语法糖，可以拆成两个

```vue
<!-- SearchBar 搜索栏 -->
<template>
  <div class="search-bar">
    <el-input :modelValue="modelValue.keyword" @update:modelValue="handleKeywordChange" :placeholder="modelValue.placeholder">
      <template #prepend>
        <el-select v-model="modelValue.selectValue" style="width: 100px;">
          <el-option
            v-for="item in modelValue.selectOptions"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          ></el-option>
        </el-select>
      </template>
      <template #append>
        <el-button :icon="Search" />
      </template>
    </el-input>
  </div>
</template>

<script setup>
import { Search } from '@element-plus/icons-vue'

const props = defineProps({
  modelValue: {
    type: Object,
    required: true,
  },
})

const emit = defineEmits(['update:modelValue'])

const handleKeywordChange = (value) => {
  emit('update:modelValue', { ...props.modelValue, keyword: value })
}
</script>
```

## 使用计算属性包一层

**父组件** 数据
↓                                          ↑
modelValue v-model      emit:update:modelvalue
↓                                          ↑
get                                       set
↓                                          ↑
计算属性                         计算属性
↓ v-model            v-model ↑
文本框

```vue
<!-- SearchBar 搜索栏 -->
<template>
  <div class="search-bar">
    <el-input v-model="keyword" :placeholder="modelValue.placeholder">
      <template #prepend>
        <el-select v-model="modelValue.selectValue" style="width: 100px;">
          <el-option
            v-for="item in modelValue.selectOptions"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          ></el-option>
        </el-select>
      </template>
      <template #append>
        <el-button :icon="Search" />
      </template>
    </el-input>
  </div>
</template>

<script setup>
import { Search } from '@element-plus/icons-vue'
import { computed } from 'vue'
const props = defineProps({
  modelValue: {
    type: Object,
    required: true,
  },
})

const emit = defineEmits(['update:modelValue'])

const keyword = computed({
  get() {
    return props.modelValue.keyword
  },
  set(value) {
    emit('update:modelValue', { ...props.modelValue, keyword: value })
  },
})
</script>
```

但是这样的话每个都需要转计算属性

## 计算属性 + Proxy

```vue
<!-- SearchBar 搜索栏 -->
<template>
  <div class="search-bar">
    <el-input v-model="model.keyword" :placeholder="model.placeholder">
      <template #prepend>
        <el-select v-model="model.selectValue" style="width: 100px;">
          <el-option
            v-for="item in model.selectOptions"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          ></el-option>
        </el-select>
      </template>
      <template #append>
        <el-button :icon="Search" />
      </template>
    </el-input>
  </div>
</template>

<script setup>
import { Search } from '@element-plus/icons-vue'
import { computed } from 'vue'
const props = defineProps({
  modelValue: {
    type: Object,
    required: true,
  },
})

const emit = defineEmits(['update:modelValue'])

// 并不能监听到变化
/* const model = computed({
  get() {
    return props.modelValue
  },
  set(value) {
    emit('update:modelValue', value)
  },
}) */

const model = computed({
  get() {
    return new Proxy(props.modelValue, {
      set(target, key, value) {
        emit('update:modelValue', {
          ...target,
          [key]: value,
        })
        return true  // 返回 true 表示设置成功
      },
    })
  },
  set(value) {
    emit('update:modelValue', value)
  },
})
</script>
```

## 抽离成辅助函数

```js
import { computed } from 'vue'

export default function useVModel(props, propName, emit) {
  return computed({
    get() {
      return new Proxy(props[propName], {
        set(target, key, value) {
          emit(`update:${propName}`, {
            ...target,
            [key]: value,
          })
          return true
        },
      })
    },
    set(value) {
      emit(`update:${propName}`, value)
    },
  })
}
```

直接使用

```vue
<!-- SearchBar 搜索栏 -->
<template>
  <div class="search-bar">
    <el-input v-model="model.keyword" :placeholder="model.placeholder">
      <template #prepend>
        <el-select v-model="model.selectValue" style="width: 100px;">
          <el-option
            v-for="item in model.selectOptions"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          ></el-option>
        </el-select>
      </template>
      <template #append>
        <el-button :icon="Search" />
      </template>
    </el-input>
  </div>
</template>

<script setup>
import { Search } from '@element-plus/icons-vue'
import useVModel from './useVModel'

const props = defineProps({
  modelValue: {
    type: Object,
    required: true,
  },
})

const emit = defineEmits(['update:modelValue'])
const model = useVModel(props, 'modelValue', emit)
</script>
```
