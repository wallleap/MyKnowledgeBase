---
title: Vue 3 优雅地监听 localStorage 变化
date: 2023-08-03 15:37
updated: 2023-08-03 15:40
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - Vue 3 优雅地监听 localStorage 变化
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

> 最近在研究框架，也仔细用了 `Vue3` 一些功能，今天分享一次我的实践：

**「`Vue3` 如何监听 `localStorage` 的变化。」**

## 💡 为什么要这样做？

原生的 `localStorage` 只能监听同源地址下不同页面的 `localStorage` 变化，作为单页面应用，显然不实用。所以我打算自定义一个 `hook` 监听 `localStorage` 的变化。

## 💎 思路

首先我们需要重写 `localStorage` 下的所有方法，这样在每个方法被使用的时候就可以被监听到了。

此时就需要一个事件机制，用于传递消息。

在 `Vue3` 移除了 `$on`、`$emit` 事件接口后，我们可以借助第三方库实现：`mitt`、`tiny-emitter`.

不过我打算使用自己实现的 `中介者模式` 作为通信方法。

## 💎 实现

### 🚗 实现 `中介者模式`

```ts
// mediator.ts
export interface MediatorProps {
  uuid?: number;
  publish?: (topic: string, ...args: unknown[]) => void;
  subscribe?: (topic: string, callback: (...args: unknown[]) => void) => void;
}

const mediator = (function () {
  let topics = [],
    uuid = 0;

  function subscribe(topic: string, callback: (...args: unknown[]) => void) {
    uuid++;
    topics[topic] = topics[topic]
      ? [...topics[topic], { callback, uuid }]
      : [{ callback, uuid }];
  }

  function publish(topic: string, ...args: unknown[]) {
    if (topics[topic]) {
      topics[topic].map((item) => item.callback(...args));
    }
  }
  return {
    install: function (obj: MediatorProps) {
      obj.uuid = uuid;
      obj.publish = publish;
      obj.subscribe = subscribe;
      return obj;
    },
  };
})();

export default mediator;
```

### 🚗 重写 `localStorage`

```ts
// localStorage.ts
import mediator from "./mediator";

const keys: string[] = [];
const createMediator = () => mediator.install({});
const sub = createMediator();

export const $localStorage = {
  getItem: (key: string) => {
    return window.localStorage.getItem(key);
  },

  setItem: (key: string, value: any) => {
    // 防止重复发布
    if (!keys.includes(key)) keys.push(key);

    // 被修改就发布事件
    sub.publish(key, value);    

    window.localStorage.setItem(key, value);
  },
  clear: () => {
    // 被删除就每个key发布事件
    keys.map((key) => sub.publish(key, undefined));
    // 发布后清空记录key的数组
    keys.length = 0;

    window.localStorage.clear();
  },
  removeItem: (key: string) => {
    keys.splice(keys.indexOf(key), 1);

    // 被移除就发布 undefined
    sub.publish(key, undefined);

    window.localStorage.removeItem(key);
  },
  key: window.localStorage.key,
  length: window.localStorage.length,
};
```

### 🚗 实现 `useStorage hook`

```ts
// useStorage.ts
import { ref } from "vue";
import mediator from "./mediator";
const createMediator = () => mediator.install({});

 export const useStorage = (key: string) => {
  
  const string = ref(null);

  const sub = createMediator();

  sub.subscribe(key, (value) => string.value = value);

  return string;
};
```

## 💎 测试

### 🚗 使用 `localStorage`

```vue
// One.vue
// 使用localStorage
import { watch } from "vue";
import { useStorage } from "./hook";

const key = useStorage("yourKey");

watch([key], (a) => console.log(a));
```

### 🚗 监听 `localStorage` 变化

```vue
// Two.vue
// 监听localStorage
<script setup lang="ts">
import { ref } from "vue";
import { localStorage } from "./hook";

const count = ref(0);
</script>

<template>
  <div>
      <button
        type="button"
        @click="$localStorage.setItem('a', count++);"
      >
        count is {{ count }}
      </button>
    </div>
</template>
```

### 🚗 结果

![](https://cdn.wallleap.cn/img/pic/illustration/202308031540483.gif)

打印结果

好了今天的分享的就到了，希望今天的分享可以帮助到你！

源码点这里：<https://codesandbox.io/p/sandbox/hardcore-hodgkin-qp5lwu>
