---
title: 11-前端 Mock
date: 2024-05-30 11:27
updated: 2024-05-31 15:14
---

## Mock 出现原因

- 前端先做后端才出的接口和文档
- 作假

## 成熟的 Mock 产品

- 平台类 APIfox、PostMan 等
- 代码类

## 自制 Mock

### Mock 方法

安装 faker

```sh
pnpm add @faker-js/faker --save-dev
```

新建 `src/_mock/mock.ts`

```ts
import {faker} from "@faker-js/faker"
import {AxiosRequestConfig} from 'axios'

type Mock = (config: AxiosRequestConfig) => [number, any]

export const mockSession: Mock = () => {
  return [200, {
    jwt: window.atob(faker.lorem.words(21))
  }]
}

export const mockTagIndex: Mock = (config) => {
  let id = 0
  const createId = () => {
    id += 1
    return id
  }
  const createTag = (n = 1, attrs?: any) =>
    Array.from({length: n}).map(() => ({
      id: createId(),
      name: faker.lorem.word(),
      sign: faker.internet.emoji(),
      kind: config.params.kind,
      ...attrs
    }))
  if (config.params.kind === 'expenses') {
    return [200, {resources: createTag(26)}]
  } else {
    return [200, {resources: createTag(12)}]
  }
}
```

### 拦截

`src/shared/http.ts`

```ts
import axios, {AxiosError, AxiosInstance, AxiosRequestConfig, AxiosResponse} from "axios"
import {mockSession, mockTagIndex} from "../_mock/mock"

type JSONValue = string | number | null | boolean | JSONValue[] | { [key: string]: JSONValue }

export class Http {
  instance: AxiosInstance

  constructor(baseURL: string, options = {}) {
    this.instance = axios.create({
      baseURL,
      timeout: 5000,
      ...options,
    })
  }

  // read
  get<R = unknown>(url: string, query?: Record<string, string>, config?: Omit<AxiosRequestConfig, 'params' | 'url' | 'method'>) {
    return this.instance.request<R>({
      ...config,
      url,
      params: query,
      method: 'GET',
    })
  }

  // create
  post<R = unknown>(url: string, data?: Record<string, JSONValue>, config?: Omit<AxiosRequestConfig, 'data' | 'url' | 'method'>) {
    return this.instance.request<R>({
      ...config,
      url,
      data,
      method: 'POST',
    })
  }

  // update
  patch<R = unknown>(url: string, data?: Record<string, JSONValue>, config?: Omit<AxiosRequestConfig, 'data' | 'url' | 'method'>) {
    return this.instance.request<R>({
      ...config,
      url,
      data,
      method: 'PATCH',
    })
  }

  // destroy
  delete<R = unknown>(url: string, query?: Record<string, string>, config?: Omit<AxiosRequestConfig, 'params' | 'url' | 'method'>) {
    return this.instance.request<R>({
      ...config,
      url,
      params: query,
      method: 'DELETE',
    })
  }
}

export const http = new Http('/api/v1')

const isDev = (ip?: string) => {
  const ipArrange = /^(127\.0\.0\.1)|(localhost)|(10\.\d{1,3}\.\d{1,3}\.\d{1,3})|(172\.((1[6-9])|(2\d)|(3[01]))\.\d{1,3}\.\d{1,3})|(192\.168\.\d{1,3}\.\d{1,3})$/
  return ipArrange.test(ip || window.location.hostname)
}

const mock = (response: AxiosResponse) => {
  if (!isDev()) {
    return false
  }
  switch (response.config?.params?._mock) {
    case 'tagIndex':
      [response.status, response.data] = mockTagIndex(response.config)
      return true
    case 'session':
      [response.status, response.data] = mockSession(response.config)
      return true
  }
  return false
}

// 请求拦截器
http.instance.interceptors.request.use(config => {
  const token = localStorage.getItem('jwt')
  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }
  return config
})
// 响应拦截器
http.instance.interceptors.response.use(response => {
  mock(response)
  return response
}, error => {
  if (mock(error.response)) {
    return error.response
  } else {
    throw error
  }
})
http.instance.interceptors.response.use((response) => {
  return response
}, (error) => {
  if (error.response) {
    const axiosError = error.response as AxiosError
    switch (axiosError.status) {
      case 401:
        break
      case 403:
        break
      case 404:
        break
      case 429:
        alert('请求过于频繁，请稍后再试')
        break
      case 500:
        break
    }
  }
  throw error
})
```

## ItemCreate 页面请求 Mock 的 tags

### 初始代码

`src/views/items/ItemCreate.tsx`

```tsx
import {defineComponent, onMounted, ref} from "vue"
import {http} from "../../shared/http"
import Icon from "../../components/common/Icon"
import MainLayout from "../../components/layouts/MainLayout"
import {Tabs, Tab} from "../../components/Tabs"
import {useRouter} from "vue-router"
import InputPad from "../../components/InputPad"
import s from './ItemCreate.module.scss'

export default defineComponent({
  setup() {
    const router = useRouter()
    const kind = ref('支出')
    const expensesTags = ref<Tag[]>([])
    const incomeTags = ref<Tag[]>([])
    onMounted(async () => {
      const response = await http.get<{resources: Tag[]}>('/tags', {
        kind: 'expenses',
        _mock: 'tagIndex',
      })
      expensesTags.value = response.data.resources
    })
    onMounted(async () => {
      const response = await http.get<{resources: Tag[]}>('/tags', {
        kind: 'income',
        _mock: 'tagIndex',
      })
      incomeTags.value = response.data.resources
    })
    return () => (
      <MainLayout onNavClick={() => router.back()}>{
        {
          title: () => '记一笔',
          icon: () => <Icon name="left" width="1em" height="1em" />,
          other: () => <>
            <Tabs selected={kind.value} v-model:selected={kind.value}>
              <Tab name="支出">
                <div class={s.expenseWrapper}>
                  <div class={s.tag}><Icon class={s.tagSign} name='add' /></div>
                  {expensesTags.value.map(tag => (<div class={s.tag} key={tag.id}>
                    <span class={s.tagSign}>{tag.sign}</span>
                    <span class={s.tagName}>{tag.name}</span>
                  </div>))}
                </div>
              </Tab>
              <Tab name="收入">
                <div class={s.incomeWrapper}>
                  <div class={s.tag}><Icon class={s.tagSign} name='add'/></div>
                  {incomeTags.value.map(tag => (<div class={s.tag} key={tag.id}>
                    <span class={s.tagSign}>{tag.sign}</span>
                    <span class={s.tagName}>{tag.name}</span>
                  </div>))}
                </div>
              </Tab>
            </Tabs>
          </>,
          default: () => <>
            <div class={s.inputPad_wrapper}>
              <InputPad />
            </div>
          </>
        }
      }</MainLayout>
    )
  },
})
```

类型声明

`src/vite-env.d.ts`

```ts
/// <reference types="vite/client" />
type JSONValue = null | boolean | string | number | JSONValue[] | Record<string, JSONValue>

type Tag = {
  id: number
  user_id: number
  name: string
  sign: string
  kind: 'expenses' | 'income'
}
```

### 封装 useAsyncTags

### 封装 Tags 组件

也可以在 useAsyncTags 中提供一个 `view = computed` 的 DOM

```tsx
import {computed, onMounted, ref} from "vue"
import {AxiosResponse} from "axios"
import {http} from "../shared/http"
import Icon from "../components/common/Icon"
import Button from "../components/common/Button"
import s from "../views/items/ItemCreate.module.scss"

type Fetcher = (page: number, kind: 'expenses' | 'incomes') => Promise<AxiosResponse<Resources<Tag>>>
const fetcher: Fetcher = (page, kind) => http.get<Resources<Tag>>('/tags', {
  kind,
  _mock: 'tagIndex',
  page: (page + 1).toString()
})
export const useAsyncTags = (kind: 'expenses' | 'incomes') => {
  const page = ref(0)
  const hasMore = ref(false)
  const tags = ref<Tag[]>([])
  const selectedTagId = ref<number>()
  const fetchTags = async () => {
    const response = await fetcher(page.value, kind)
    const {resources, pager} = response.data
    tags.value.push(...resources)
    page.value += 1
    hasMore.value = (page.value - 1) * pager.per_page + resources.length < pager.count
  }
  const onSelect = (tag: Tag) => {
    selectedTagId.value = tag.id
  }
  const view = computed(() => {
    return <>
      <div class={s.incomeWrapper}>
        <div class={s.tag}><Icon class={s.tagSign} name='add'/></div>
        {tags.value.map(tag => (<div class={[s.tag, selectedTagId.value === tag.id ? s.selected : '']} key={tag.id} onClick={() => onSelect(tag)}>
          <span class={s.tagSign}>{tag.sign}</span>
          <span class={s.tagName}>{tag.name}</span>
        </div>))}
      </div>
      <div class={s.btnWrapper}>
        {
          hasMore.value && <Button class={s.btn} onClick={fetchTags}>加载更多</Button>
        }
      </div>
    </>
  })
  onMounted(fetchTags)
  return {
    page,
    hasMore,
    tags,
    fetchTags,
    view,
    selectedTagId,
  }
}
```

使用

```tsx
const {view: expensesTags, selectedTagId: expensesTagId} = useAsyncTags('expenses')  
const {view: incomesTags, selectedTagId: incomesTagId} = useAsyncTags('incomes')  
// TODO: 将选择的 tagId 传给后端  
watchEffect(() => {  
  console.log(expensesTagId.value, incomesTagId.value)  
})


<Tabs selected={kind.value} v-model:selected={kind.value}>  
  <Tab name="支出">  
    <div class={s.tabContent}>{expensesTags.value}</div>  
  </Tab>  <Tab name="收入">  
    <div class={s.tabContent}>{incomesTags.value}</div>  
  </Tab>
</Tabs>
```

> `ref` 和 `reactive`

一般使用 `ref`（简单变量），对象用两个都可以

- `ref` 需要 `.value`
- `reactive` 可以直接用

### 解决组件缓存

如果使用的是 `Tags` 组件的方式

点击切换 Tab

发现不是重新渲染而是更新了

**方案一：在 update 的时候重置和请求**

```ts
onUpdated(() => {
  console.log('我被更新了')
})
```

**方案二：使用 key**

两个组件的 key 不一样，这样区分组件就不会被缓存

```ts
<Tab>
  <Tags key="expenses" />
</Tab>
<Tab>
  <Tags key="incomes" />
</Tab>
```

也可以在封装 Tab 的时候就给它上层设置 key

```ts
<div key="props.name">  
  {tabs.find(node => node.props?.name === props.selected)}  
</div>
```

但是这样不太好，点击切换了就会重新渲染，这样需要重新进行请求

**方案三：Tabs 都渲染，但是通过样式判断显隐**

```ts
<div>
  {tabs.map(node => <div style={{
    display: node.props?.name === props.selected ? 'block' : 'none'
  }}>{ndoe}</div>)}
  // 或使用 v-show 指令
  {tabs.map(node => <div v-show={node.props?.name === props.selected}>{ndoe}</div>)} 
</div>
```

### 滚动位置

添加一个变量，设置滚动高度，切换回来的时候再设置这个滚动高度

`keep-alive` 做了这个
