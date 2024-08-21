---
title: vite3+vue3+ts+pinia + Naive UI 项目实战 —— 国际化配置
date: 2023-08-04T06:03:00+08:00
updated: 2024-08-21T10:32:44+08:00
dg-publish: false
aliases:
  - vite3+vue3+ts+pinia + Naive UI 项目实战 —— 国际化配置
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

***本文正在参加 [「金石计划 . 瓜分6万现金大奖」](https://juejin.cn/post/7162096952883019783 "https://juejin.cn/post/7162096952883019783")***

> 接上篇 [《项目创建与初始配置》](https://juejin.cn/post/7171841238948118536 "https://juejin.cn/post/7171841238948118536") 继续搭建我们的后台项目。

因为项目是给设在新加坡、马来西亚的海外自提站使用的后台，所以国际化的配置必不可少。需要切换语言的内容主要分为两部分：

1. 我们自己添加的文案内容，需要用到 vue-i18n-next；
2. naive ui 的组件，可以使用 `n-config-provider` 配置。

## vue-i18n-next

## 安装

vue3 中使用 i18n 需要安装的是 [vue-i18n v9](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fintlify%2Fvue-i18n-next "https://github.com/intlify/vue-i18n-next") 的版本：

```
npm install vue-i18n@9
```

## 配置 i18n

创建 src\\lang\\index.ts，使用 `createI18n` 创建 i18n 实例：

```
// src\lang\index.ts
import { createI18n } from 'vue-i18n'
import { LANG_VALUE } from '@/common/enum'
import zhHans from './zh-Hans'
import en from './en'

const i18n = createI18n({
  legacy: false,
  locale: getLanguage(),
  messages: {
    [LANG_VALUE.Zh]: zhHans,
    [LANG_VALUE.En]: en
  }
})
export default i18n
```

下面对传入 `createI18n()` 的配置对象做些说明：

- **legacy**：默认为 `true`，但如果使用的是 Composition API 则需要定义为 `false`，否则会报类似下面的错误：

![](https://cdn.wallleap.cn/img/pic/illustration/202308041803073.png)

- **locale**：当前要展示的语言。值为之前用户的语言选择，从浏览器缓存中读取。如果缓存中没有数据，则通过 `navigator.language` 获取浏览器使用的语言：

```
// src\lang\index.ts
import { localCache } from '@/utils'
export function getLanguage() {
  const chooseLanguage = localCache.getItem(LANGUAGE)
  if (chooseLanguage) return chooseLanguage

  // 如果没有选择语言
  const language = navigator.language.toLowerCase()
  const locales = [LANG_VALUE.En, LANG_VALUE.Zh]
  for (const locale of locales) {
    if (language.indexOf(locale) > -1) {
      return locale
    }
  }
  return LANG_VALUE.Zh
}
```

因为 `locale` 值在多个文件中有使用到，所以我使用了 ts 新增类型 enum（枚举类型）来保存：

```
// src\common\enum.ts
export enum LANG_VALUE {
  En = 'en',
  Zh = 'zh-Hans'
}
```

- **messages**：不同语言对应的翻译文件：

中文翻译包：

```
// src\lang\zh-Hans.ts
export default {
  baoguochuku: '包裹出库',
  sousuo: '搜索'
}
```

英文翻译包：

```
// src\lang\en.ts
export default {
  baoguochuku: 'Outbound',
  sousuo: 'Search'
}
```

## 在 main.ts 引入

```
// src\main.ts，省略其它代码
import i18n from './lang/index'
app.use(i18n)
```

## 在文件中使用

### vue 文件

- **在 `<script lang="ts" setup>` 中使用**

下面以侧边导航菜单中，对每条导航的名称的配置为例。从 vue-i18n 中导入 `useI18n`，然后进行调用生成 `i18n` 实例，再从里面解构得到 `t` 方法，在第 6 行进行使用 —— `t(route.meta.title)`，`route.meta.title` 是我在路由里配置的内容，其值就是在语言翻译文件中定义的那些 key，比如 `'baoguochuku'` 等：

```
<!-- src\components\Navigation\Navigation.vue，省略其它代码 -->
<script lang="ts" setup>
import { useI18n } from 'vue-i18n'
const { t } = useI18n()
const menuOptions: MenuOption[] = modulesRoutes.map(route => ({
  label: () => t(route.meta.title),
  // ...
}))
</script>
```

这里注意下 label 的值得写成 getter 的形式，如果是写成 `label: t(route.meta.title)` 这样直接赋值，则切换语言时是没有效果的。

- **在 `<template>` 中使用**

使用方法与之前在 vue2 中一样：

```
<template>
  <n-button>
    <slot name="submitBtnText">{{ $t('sousuo') }}</slot>
  </n-button>
</template>
```

这里之所以可以直接使用 `$t` 是因为有个叫做 `globalInjection` 的配置项默认为 `true`，帮我们全局注入了 `$t` 方法，如果设置为 `false`：

```
// src\lang\index.ts
const i18n = createI18n({
  // ...
  globalInjection: false
})
```

则会报错：![](https://cdn.wallleap.cn/img/pic/illustration/202308041803075.png)

### ts 文件

在 ts 文件中，则需要引入我们之前在 src\\lang\\index.ts 配置生成的 `i18n` 实例，然后通过 `i18n.global.t()` 来定义文案：

```
// src\service\request\index.ts
import i18n from '@/lang'
if (!axios.isCancel(error)) {
  dialog.error({
    title: i18n.global.t('tishi'),
    // ...
  })
}
```

至此，项目里我们自己添加的内容的国际化就配置完成了，接下来是对 naive ui 组件的国际化配置。

## naive ui 组件

对于 naive ui 的组件，默认情况下均为英语，如果想改为中文，需要在根组件 src\\App.vue 中使用 n-config-provider 对语言进行配置，将用于设置全局语言的 `locale` 设为从 naive ui 导入的 `zhCN` ，设置全局日期语言的 `date-locale` 为从 naive ui 导入的 `dateZhCN`：

```
<!-- src\App.vue -->
<script setup lang="ts">
import { storeToRefs } from 'pinia'
import useAppStore from '@/stores/app'
  
const appStore = useAppStore()
const { locale, dateLocale } = storeToRefs(appStore)
</script>

<template>
  <n-config-provider
    :locale="locale"
    :date-locale="dateLocale"
  >
    <RouterView />
  </n-config-provider>
</template>
```

`locale` 和 `dateLocale` 作为由 pinia 管理的 getter 定义在 src\\stores\\app.ts：

```
// src\stores\app.ts
import { defineStore } from 'pinia'
import { zhCN, dateZhCN } from 'naive-ui'
import { LANG_VALUE } from '@/common/enum'
import { getLanguage } from '@/lang'

interface IAppState {
  language: string
  // ...
}

const useAppStore = defineStore('app', {
  state: (): IAppState => ({
    language: getLanguage(),
    // ...
  }),
  getters: {
    locale(state) {
      switch (state.language) {
        case LANG_VALUE.En:
          return null
        case LANG_VALUE.Zh:
          return zhCN
        default:
          break
      }
    },
    dateLocale(state) {
      switch (state.language) {
        case LANG_VALUE.En:
          return null
        case LANG_VALUE.Zh:
          return dateZhCN
        default:
          break
      }
    }
  },
  // ...
})

export default useAppStore
```

当 `language` 为 `'en'` 时，`locale` 和 `dateLocale` 就为 `null`；当 `language` 为 `'zh-Hans'` 时，`locale` 和 `dateLocale` 就为 `zhCN` 和 `dateZhCN`。而 `language` 的值由前面已经介绍过了的 `getLanguage()` 获取，首先是从浏览器缓存读取用户之前的选择，而用户是通过下面封装的组件进行语言选择的。

## 切换语言控件

我选择结合使用 naive ui 的 Popselect（弹出选择）和 Icon（图标）组件来实现切换语言的控件：

```
<!-- src\components\LangSelect\LangSelect.vue -->
<template>
  <div class="lang-select">
    <n-popselect
      v-model:value="value"
      :options="options"
      trigger="click"
      @update:value="handleUpdateValue"
      >
      <n-icon>
        <Language />
      </n-icon>
    </n-popselect>
  </div>
</template>
```

`value` 和 `options` 定义如下：

```
<!-- src\components\LangSelect\LangSelect.vue -->
<script lang="ts" setup>
import { ref } from 'vue'
import { storeToRefs } from 'pinia'
import useAppStore from '@/stores/app'
import { LANG_VALUE } from '@/common/enum'
import type { SelectOption, SelectGroupOption } from 'naive-ui'

const ENGLISH = 'English'
const ZHONG_WEN = '中文'
// 获取当前语言
const appStore = useAppStore()
const { language } = storeToRefs(appStore)
// 弹出选择 Popselect
type valueType = string | null
const value = ref<valueType>(getValue())
function getValue() {
  switch (language.value) {
    case LANG_VALUE.En:
      return ENGLISH
    case LANG_VALUE.Zh:
      return ZHONG_WEN
    default:
      return ZHONG_WEN
  }
}
const options: Array<SelectOption | SelectGroupOption> = [
  {
    label: ENGLISH,
    value: ENGLISH
  },
  {
    label: ZHONG_WEN,
    value: ZHONG_WEN
  }
]
</script>
```

选项 `options` 就两个：`'English'` 或 '`中文'`。`value` 的默认值根据存储在 pinia 的 `appStore` 的 state `language` 决定。这里第 13 行使用 pinia 提供的 `storeToRefs` 处理是为了在解构获取 `language` 时保留其响应式，其实用 vue3 的 `toRefs` 也可以。

切换时触发的 `handleUpdateValue` 定义如下：

```
<!-- src\components\LangSelect\LangSelect.vue -->
<script lang="ts" setup>
import { setLocale } from '@/lang'
  
let lang: langType
function handleUpdateValue(value: valueType) {
  switch (value) {
    case ENGLISH:
      lang = LANG_VALUE.En
      break
    case ZHONG_WEN:
      lang = LANG_VALUE.Zh
      break
    default:
      break
  }
  setLocale(lang)
}
</script>
```

将切换后的 value 传给定义于 src\\lang\\index.ts（也就是前面配置 i18n 的文件） 的 `setLocale()` 处理：

```
// src\lang\index.ts
export function setLocale(lang: langType) {
  // 获取当前语言
  const appStore = useAppStore()
  const { language } = storeToRefs(appStore)

  i18n.global.locale.value = lang // 修改 i18n
  language.value = lang // 修改 pinia
  localCache.setItem(LANGUAGE, lang) // 修改缓存
}
```

## 图标的使用

顺便说明下 naive ui 中图标的使用，图标选自 naive ui 文档推荐的 xicons 图标库，使用前需要根据所选用的图标做对应的安装：

```
npm i -D @vicons/ionicons5
```

然后进行引入，并作为 `<n-icon>` 的默认插槽内容使用：

```
<!-- src\components\LangSelect\LangSelect.vue -->
<script lang="ts" setup>
import { Language } from '@vicons/ionicons5'
</script>                                               
<template>
  <n-icon>
    <Language />
  </n-icon>
</template>
```

## 解决控制台警告

做到这一步，如果打开浏览器控制台，可能会看到如下警告：

![](https://cdn.wallleap.cn/img/pic/illustration/202308041803076.png)

解决办法是在 vite.config.ts 添加如下配置：

```
export default defineConfig({
  // ...
  define: {
    __VUE_I18N_FULL_INSTALL__: true,
    __VUE_I18N_LEGACY_API__: false,
    __INTLIFY_PROD_DEVTOOLS__: false
  }
})
```

因为项目里都是使用 Composition API，无需启用对选项式 api 的支持，所以 `__VUE_I18N_LEGACY_API__` 的值由默认的 `true` 改为了 `false`，其余两项则维持默认值。

## 效果

至此，本项目的国际化配置就完成了。最后来看一下效果（目前只做了部分文案的初步翻译）
