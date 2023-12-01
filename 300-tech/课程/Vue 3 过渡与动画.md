---
title: Vue 3 过渡与动画
date: 2023-08-15 16:51
updated: 2023-08-15 17:37
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - Vue 3 过渡与动画
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

## 使用 class 切换实现过渡和动画

在 CSS 里写好样式，通过切换 class 实现效果切换

```html
<div id="app">
  <div :class="['box', {active: isActive}]">
    {{isActive?'大': '小'}}
    <button @click="isActive = !isActive">Toggle size</button>
  </div>
</div>
<script src="https://unpkg.com/vue@next"></script>
<script>
  Vue.createApp({
    data() { return { isActive: false } }
  }).mount('#app')
</script>
<style>
.box {
  width: 200px;
  height: 200px;
  margin: 30px;
  box-shadow: 0 0 4px 0px rgba(0, 0, 0, 0.2);
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all .3s;
}
.box.active { transform: scale(1.2); }
</style>
```

## 修改 style 实现过渡和动画

修改 style 和 data 中数据绑定

```html
<div id="app">
  <div class="color" :style="{backgroundColor: `rgb(${R},${G},${B})`}"></div>
  <div>
    <input type="range" v-model="R" min="0" max="255" /> R  {{R}} <br>
    <input type="range" v-model="G" min="0" max="255" /> G  {{G}} <br>
    <input type="range" v-model="B" min="0" max="255" /> B  {{B}} <br>
  </div>
</div>
<script src="https://unpkg.com/vue@next"></script>
<script>
  Vue.createApp({
    data() {
      return { R: 0, G: 0, B: 0 }
    }
  }).mount('#app')
</script>
<style>
.color {
  width: 40px;
  height: 40px;
}
</style>
```

## transition 组件

可包裹

- v-if、v-show
- 动态组件
- 组件根节点

创建的 class

- `v-enter-from` 在元素被插入之前生效，在元素被插入之后的下一帧移除
- `v-enter-active` 定义进入过渡生效时的状态
- `v-enter-to` 定义进入过渡的结束状态，在元素被插入之后下一帧生效，在过渡/动画完成之后移除
- `v-leave-from` 在离开过渡被触发时立刻生效，下一帧被移除
- `v-leave-active` 定义离开过渡生效时的状态
- `v-leave-to` 离开过渡的结束状态，在离开过渡被触发之后下一帧生效，在过渡/动画完成之后移除

```html
<div id="app">
  <button @click="isShow = !isShow">Toggle</button>
  <transition  name="slide">
    <p v-if="isShow">hello world</p>
  </transition>
  
</div>
<script src="https://unpkg.com/vue@next"></script>
<script>
  Vue.createApp({
    data() {
      return {
        isShow: true
      }
    }
  }).mount('#app')
</script>

<style>
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.fade-enter-active,
.fade-leave-active,
.slide-enter-active,
.slide-leave-active {
  transition: all .3s;
}

.slide-enter-from {
  opacity: 0;
  transform: translateX(-200px);
}
.slide-leave-to {
  opacity: 0;
  transform: translateX(200px);
}
</style>
```

![](https://cdn.wallleap.cn/img/pic/illustration/202308151709674.png)

注意 name

```html
<div id="app">
  <button @click="show = !show">Toggle</button>
  <transition  name="fade">
    <p v-if="show">hello</p>
  </transition>
</div>
<script src="https://unpkg.com/vue@next"></script>
<script>
  Vue.createApp({
    data() {
      return {
        show: true
      }
    }
  }).mount('#app')
</script>

<style>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
```

## transition 组件 + animate.css 库

官网：<https://animate.style>

自定义过渡 class 类名，结合 animate.css 写动画更简单

```html
<div id="app">
  <button @click="show = !show">Toggle</button>
  <transition  name="fade" 
    enter-active-class="animate__animated animate__fadeInDown"
    leave-active-class="animate__animated animate__fadeInUp"
  >
    <p v-if="show">hello</p>
  </transition>
</div>
<script src="https://unpkg.com/vue@next"></script>
<script>
  Vue.createApp({
    data() {
      return {  show: true }
    }
  }).mount('#app')
</script>

<link href="https://cdnjs.cloudflare.com/ajax/libs/animate.css/4.1.0/animate.min.css"
  rel="stylesheet"/>
```

## 多个元素轮流切换过渡

- 多元素过渡指的是多个元素进行切换，同一时间只展示一个（`v-if`、`v-else`）
- 使用不同的 key 提升性能
- mode 属性解决两个元素同时存在的现象
	- `out-in` 当前元素先出，下个元素再进
	- `in-out` 下个元素先进，当前元素再出

```html
<div id="app">
  <button @click="show = !show">Toggle</button>
  <transition 
    mode="out-in"
    enter-active-class="animate__animated animate__fadeInUp"
    leave-active-class="animate__animated animate__fadeOutDown"
  >
    <p v-if="show" key="hello">hello</p>
    <p v-else key="world">world</p>
  </transition>
</div>
<script src="https://unpkg.com/vue@next"></script>
<script>
  Vue.createApp({
    data() {
      return {  show: true }
    }
  }).mount('#app')
</script>

<link href="https://cdnjs.cloudflare.com/ajax/libs/animate.css/4.1.0/animate.min.css"
  rel="stylesheet"/>
```

## 多组件切换

- 使用动态组件实现 Tab 切换效果
- 如果动态组件使用了 keep-alive，需要放在 transition 内部

```html
<div id="app">
  <button v-for="tab in tabs" :key="tab" :class="{ active: currentTab === tab }" @click="currentTab = tab">
    {{ tab }}
  </button>
  <transition mode="out-in" name="fade">
    <keep-alive>
      <component :is="currentTab" class="tab"></component>
    </keep-alive>
  </transition>
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
  })
  app.component('Tab2', {
    template: `<div>
      <input v-model="value" /> {{value}}
    </div>`,
    data() { return { value: 'hello' } },
  })
  app.mount('#app')
</script>
<link href="https://cdnjs.cloudflare.com/ajax/libs/animate.css/4.1.0/animate.min.css" rel="stylesheet" />

<style>
  .active {
    background: #ccc;
  }

  .fade-enter-active,
  .fade-leave-active {
    transition: all .3s;
  }

  .fade-enter-from,
  .fade-leave-to {
    opacity: 0;
  }

</style>
```

## 列表的过渡效果

使用 transition-group 实现列表过渡

```html
<div id="app">
  <button @click="add">添加</button>
   <transition-group name="list" tag="div">
    <div v-for="(value, index) in news" :key="value">
      {{value}} 
      <button @click="news.splice(index, 1)">删除</button>
    </div>
  </transition-group>
</div>
<script src="https://unpkg.com/vue@next"></script>
<script>
  Vue.createApp({
    data() {
      return {
        news: []
      }
    },
    methods: {
      add() {
        let value = window.prompt('输入新闻')
        if(value) {
          this.news.push(value)
          this.news = this.news.sort()
        }
      }
    }
  }).mount('#app')
</script>
<style>
  .list-enter-active,
  .list-leave-active {
    transition: all .4s ease;
  }
  .list-enter-from {
    opacity: 0;
    transform: translateX(-30px);
  }
  .list-leave-to {
    opacity: 0;
    transform: translateX(30px);
  }
</style>
```

