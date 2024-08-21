---
title: Vue 超长列表渲染性能优化实战
date: 2023-08-04T06:01:00+08:00
updated: 2024-08-21T10:32:43+08:00
dg-publish: false
aliases:
  - Vue 超长列表渲染性能优化实战
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

本文已参与好文召集令活动，点击查看：[后端、大前端双赛道投稿，2万元奖池等你挑战！](https://juejin.cn/post/6978685539985653767/ "https://juejin.cn/post/6978685539985653767/")

> 本文是关于超长列表渲染性能优化的学习笔记，这里做个总结与分享，有不足之处还望斧正~

长列表的优化策略主要有两种：

- 分片渲染（通过浏览器事件环机制，也就是 EventLoop，分割渲染时间）
- 虚拟列表（只渲染可视区域）
		在正式优化前，先介绍一些用得到的基本概念：

## 进程与线程

进程是系统进行资源分配和调度的一个独立单位，一个进程内包含多个线程。人们常说的 JS 是单线程的，是指 JS 的主进程是单线程的。

## 渲染进程

渲染进程包含以下线程：

- GUI 渲染线程（页面渲染）
- JS 引擎线程（执行 js 脚本）
- 事件触发线程（EventLoop 轮询处理线程）
- 事件（onclick）、定时器和 ajax 也会开启独立线程
		注意，GUI 渲染线程和 JS 引擎线程是互斥的，所谓的 JS 是单线程的，就是指这条共享的主线程。

### 浏览器中的 EventLoop

![](https://cdn.wallleap.cn/img/pic/illustration/202308041802462.jpeg)

## 分片渲染

## 简单的长列表观察渲染时间

我们先写一个有 10000 条简单数据的列表，并打印时间，注意，这个时间是 js 语句执行的时间，而不是浏览器渲染的时间。

```
<ul id="list"></ul>
<script type="text/javascript">
  const time = Date.now()
  for (let i = 0; i < 100000; i++) {
    const li = document.createElement('li')
    li.innerText = i
    list.appendChild(li)
  }
  console.log(Date.now() - time) // 160
</script>
```

为了获得浏览器的渲染时间，根据上面的 EventLoop 可知，每次 GUI 渲染完都会执行一个宏任务，所以我们可以在后面添加一个定时器（宏任务），渲染完成后执行得到渲染时间。

```
setTimeout(() => {
  console.log(Date.now() - time)
}, 0) // 2800
```

效果如下图：

![](https://cdn.wallleap.cn/img/pic/illustration/202308041802463.gif)

可以看到，js 执行时间为 160 ms，加上渲染总共用了 2801 ms。在渲染完成前页面一直是空白的加载状态，这是因为新版本的浏览器做了优化，会等待 for 循环执行完毕后再将 dom 节点插入到页面中，避免了频繁的重排和重绘。

## 进行分片优化

```
const time = Date.now()
/**
 * index: 记录循环到哪了
 * id: 往 li 里添加的内容
 */
let index = 0, id = 0
function load() {
  index += 50
  if (index < 10000) {
    requestAnimationFrame(() => { // 用 requestAnimationFrame（也是宏任务）代替了 setTimeout，性能更好点
      const fragment = document.createDocumentFragment() // IE 浏览器需要使用文档碎片，一般可不用
      for (let i = 0; i < 50; i++) {
        const li = document.createElement('li')
        li.innerText = id++
        fragment.appendChild(li)
      }
      list.appendChild(fragment)
    })
    load()
  }
}
load()
console.log(Date.now() - time)
setTimeout(() => {
  console.log(Date.now() - time)
})
```

因为 `requestAnimationFrame` 或定时器是一个宏任务，所以每执行一次 GUI 渲染后就执行一次相关的回调，也就实现了每次添加 50 个 li 节点，从而达到了分片加载的目的。现在加载时间则如下：

![](https://cdn.wallleap.cn/img/pic/illustration/202308041802464.gif)

与分片加载之前对比，确实快了许多。但这种方案有个问题：会导致页面的 dom 元素过多，依旧容易造成卡顿。解决这个问题，就要用下面介绍的第 2 种优化策略。

## 虚拟列表（只渲染当前可视区域）

虚拟列表优化依据列表每一项（item）的高度是否为固定值分为两种情况：

## item 高度固定

为了代码的可复用性，我们将封装一个 VirtualList 组件，先放代码（基于 vue 2.6）

### 用到长列表组件的页面 App.vue（父组件）

```
// App.vue
<template>
  <div id="app">
    <virtual-list :size="40" :keeps="8" :arrayData="list">
      <template #default="{ item }">
        <div style="height: 40px; border: 1px solid cadetblue;">
          {{ item.value }}
        </div>
      </template>
    </virtual-list>
  </div>
</template>

<script>
import VirtualList from './components/VirtualList.vue'
const list = []
for (let i = 0; i < 1000; i++) {
  list.push({
    id: i,
    value: i
  })
}
export default {
  name: 'App',
  components: {
    VirtualList
  },
  data() {
    return { list }
  }
}
</script>

<style lang="less">
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
</style>
```

### 长列表组件 VirtualList.vue

```
// VirtualList.vue
<template>
  <!-- 展示区域 -->
  <div class="wrap" ref="wrap" @scroll="handleScroll">
    <!-- 为了显示滚动条 -->
    <div ref="scrollHeight"></div>
    <!-- 展示的内容 -->
    <div class="visible-wrap" :style="{transform: `translateY(${offset}px)`}">
      <div v-for="item in visibleData" :key="item.id" :id="item.id">
        <slot :item="item"></slot>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'VirtualList',
  props: {
    size: Number,
    keeps: Number,
    arrayData: Array
  },
  data() {
    return {
      start: 0,
      end: this.keeps,
      offset: 0 // 列表内容的偏移量
    }
  },
  computed: {
    visibleData() {
      return this.arrayData.slice(this.start, this.end)
    }
  },
  mounted() {
    this.$refs.scrollHeight.style.height = this.arrayData.length * this.size + 'px'
    this.$refs.wrap.style.height = this.keeps * this.size + 'px'
  },
  methods: {
    handleScroll() {
      const scrollTop = this.$refs.wrap.scrollTop
      // 计算从下标为几的一项开始渲染，减 1 是因为渲染的数据是从第 0 项开始的
      this.start = Math.ceil(scrollTop / this.size) - 1 >= 0 ? Math.ceil(scrollTop / this.size) - 1 : 0
      this.end = this.start + this.keeps
      // 当列表向上（下）滚动时，为了让渲染的列表一直处于可视范围内，就要把列表向下（上）挪
      this.offset = this.start * this.size
    }
  }
}
</script>


<style scoped lang="less">
.wrap {
  position: relative;
  overflow-y: scroll;
}

.visible-wrap {
  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
}
</style>
```

### 解释说明

- **列表的数据来源是一个长度为 1000 的简单的 list 数组**
- **我们的目标是只需要传递 3 个值给 VirtualList 组件，就可以正常使用：**
		- `size`：每一项的高度
		- `keeps`：希望展示几条数据
		- `arrayData`：列表数据
- **VirtualList 组件里得有 3 个部分：**
		- 最外层容器区域。高度固定，超出区域出现滚动条，高度为传入的 `size` 乘上 `keeps`；
		- 列表本应该有的高度区域，也就是列表如果全部渲染的总高度。因为只渲染 `keeps` 指定的条数的数据，就会导致没有滚动条或滚动条无法起到预告总的列表长度的功能，所以要用一个高度为列表总长度的 div 让滚动条正确显示；
		- 要展示的内容。展示的数据应该是总数据 `arrayData` 的某一部分。展示的数据 `item` 还得传给父组件，在父组件进行使用，这里就用到了插槽。
- **当滚动列表时（`handleScroll` 触发），我们要及时的根据滚动的距离更新应该显示的数据：**
		- `onscroll` 处理的是对象内部内容区的滚动事件，所以是对最外部固定高度的 wrap 容器进行监听。
		- 如下图所示：蓝色矩形为可视区域，假设传入的 `keeps` 为 3 ，当滚动列表（红色矩形）时，渲染的列表区域，也就是 3 个 `item`（深蓝绿色矩形） 占据的区域也会跟着滚动，如果仅仅改变渲染的内容，也就是根据滚动距离从 item1 开始渲染，那么此时这个 item1 就会替换下图的 item0 ，位于可视区域之外，无法被看见。
				![](https://cdn.wallleap.cn/img/pic/illustration/202308041802465.png)
				所以需要根据已经滚动出可视区域的 `item` 的个数和每一项 `item` 的高度的乘积（`offset`）进行反向的移动，移动的距离为 `this.start * this.size`。注意： `offset` 的值在多数情况下不会等于 `scrollTop` 的值。

![](https://cdn.wallleap.cn/img/pic/illustration/202308041802466.png)

> 注：一个元素的 scrollTop 值是这个元素的内容顶部到它的视口可见内容（的顶部）的距离的度量。当一个元素的内容没有产生垂直方向的滚动条，那么它的 scrollTop 值为 0。

解决滚动时，如果刚好渲染的第一项只显示了部分，那么可视区域的最底下就会出现相应高度的空白的问题，就如下图中，可以看到右侧滚动条的高度超过 23 那一项的区域为空白：

![](https://cdn.wallleap.cn/img/pic/illustration/202308041802467.png)

解决方案为在原先的渲染项数基础上，再多向前和后渲染若干项，那么决定渲染哪些数据的 `visibleData` 的计算就会发生变化，原先是定义了 `start` 和 `end` 用于标记到底切割哪一部分，代码如下：

```
visibleData() {
  return this.arrayData.slice(this.start, this.end)
}
```

现在则是新定义了 `prevCount`、`renderStart`、`nextCount` 和 `renderEnd` 4 个参数，另外 `offset` 也需要改变为 `(this.start - this.prevCount) * this.size`，因为假设原本渲染 8 项，往上滚动了 1 项的距离，那么渲染的 8 项由 0 ~ 7 变为 1 ~ 8，0 项被删除，这时需要把 `class="visible-wrap"` 的这个 div 往下移动 1 项，才能刚好在可视区域的顶部看到第 1 项；现在则是往上移动 1 项时，`class="visible-wrap"` 这个 div 里渲染的项数就会变为 1+8+8=17 项，0 项不会被删除，不需要再往下移动 1 项，所以会有 `this.start - this.prevCount`

## item 高度不固定

接下来就是实际项目中更为常见的每一项高度不确定的情况了，还是先放代码：

### App.vue

跟之前 `item` 高度固定的情况相比变化不大，只需用 **mockJS** 随机生成段落文本，模拟每项 `item` 高度不固定的情况：

```
import Mock from 'mockjs'

const list = []
for (let i = 0; i < 1000; i++) {
  list.push({
    id: i,
    value: Mock.Random.cparagraph( 5, 10 ) // 随机生成有 5 ~ 10 个句子的中文段落
  })
}
```

多传给组件一个参数 `variable`，每一项的 style 也做了相应调整：

```
<virtual-list :size="30" :keeps="8" :arrayData="list" :variable="true">
  <template #default="{ item }">
    <div style="border: 1px solid cadetblue; padding: 20px;">
      {{ item.id}}. {{ item.value }}
    </div>
  </template>
</virtual-list>
```

### VirtualList.vue

先放完整的代码如下（省略了样式部分）：

```
// VirtualList.vue
<template>
  <!-- 展示区域 -->
  <div class="wrap" ref="wrap" @scroll="scrollFn">
    <!-- 为了显示滚动条 -->
    <div ref="scrollHeight"></div>
    <!-- 展示的内容 -->
    <div class="visible-wrap" :style="{transform: `translateY(${offset}px)`}">
      <div v-for="item in visibleData" :key="item.id" :id="item.id" ref="items">
        <slot :item="item"></slot>
      </div>
    </div>
  </div>
</template>

<script>
import throttle from 'lodash/throttle'
export default {
  name: 'VirtualList',
  props: {
    size: Number, // 每一项的高度或预估高度
    keeps: Number, // 渲染的项目数
    arrayData: Array, // 总列表的数据
    variable: Boolean // 每一项是否高度固定
  },
  data() {
    return {
      start: 0, // 展示的开始项
      end: this.keeps, // 展示的结尾项（不包括）
      offset: 0 // 列表内容的偏移量
    }
  },
  computed: {
    visibleData() {
      // 下面有用到 Math.min 是防止前面或后面没有 this.keeps 项，也就是刚开始或快到底的情况
      const prevCount = Math.min(this.start, this.keeps) // 展示的项数之前多渲染几项
      this.prevCount = prevCount // handleScroll 方法里要用
      const renderStart = this.start -  prevCount // 真正的渲染开始项
      const nextCount = Math.min(this.arrayData.length - this.end, this.keeps) // 展示的项数之后多渲染几项
      const renderEnd = this.end +  nextCount // 真正的渲染结束项的后一项，因为 slice 是不包括 end 参数的
      return this.arrayData.slice(renderStart, renderEnd) 
    }
  },
  created() {
    this.scrollFn = throttle(this.handleScroll, 200, { 'leading': false })
  },
  mounted() {
    // 计算列表如果全部渲染应该有的高度
    this.$refs.scrollHeight.style.height = this.arrayData.length * this.size + 'px'
    // 计算可视区域的高度
    this.$refs.wrap.style.height = this.keeps * this.size + 'px'
    // 缓存每一项的高度
    this.cacheListPosition()
  },
  updated() {
    this.$nextTick(() => {
      // 确保 dom 已经更新
      const domArr = this.$refs.items
      if (!(domArr && domArr.length > 0)) return // 如果没有 dom 就返回
      domArr.forEach(item => {
        const { height } = item.getBoundingClientRect() // 获取每个节点的高度
        // 更新缓存在 positionListArr 里的数据，先拿到这个节点的 id，id 跟 index 是一一对应的
        const id = item.getAttribute('id') // 先拿到这个节点的 id
        const oldHeight = this.positionListArr[id].height // 获取缓存数组里对应的这一项的原来的高度
        const difference = oldHeight - height
        if (difference) { // 如果高度变了，更新这一项的高度和 bottom（height 的变化不影响 top）
          this.positionListArr[id].height = height
          this.positionListArr[id].bottom = this.positionListArr[id].bottom - difference
          // 后面的每一项相应的也要调整
          for (let i = id + 1; i < this.positionListArr.length; i++) {
            if (this.positionListArr[i]) {
              this.positionListArr[i].top = this.positionListArr[i - 1].bottom // 后一项的 top 等于前一项的 bottom
              this.positionListArr[i].bottom = this.positionListArr[i].bottom - difference
            }
          }
        }
      })
      // 重新计算列表如果全部渲染应该有的高度
      this.$refs.scrollHeight.style.height = this.positionListArr[this.positionListArr.length - 1].bottom + 'px'
    })
  },
  methods: {
    handleScroll() {
      const scrollTop = this.$refs.wrap.scrollTop // 滚动条滚动距离
      if (this.variable) { // 每一项高度不固定
        this.start = this.getStartIndex(scrollTop) // 获取开始展示的那一项的下标
        this.end = this.start + this.keeps
        // 下面这句用三元运算符是因为 this.positionListArr[this.start - this.prevCount] 的值可能为 undefined，
        // 比如当从下往回滚动的时候 this.prevCount 的值有可能会有等于前一次滚动时的 this.start 的值的时候，大于现在 this.start 的值
        this.offset = this.positionListArr[this.start - this.prevCount] ? this.positionListArr[this.start - this.prevCount].top : 0 
      } else { // 每一项高度固定
        // 计获取开始展示的那一项的下标，减 1 是因为展示的数据是从第 0 项开始的
        this.start = Math.ceil(scrollTop / this.size) - 1 >= 0 ? Math.ceil(scrollTop / this.size) - 1 : 0
        this.end = this.start + this.keeps
        // 当列表向上（下）滚动时，为了让展示的列表一直处于可视范围内，就要把列表向下（上）挪
        this.offset = (this.start - this.prevCount) * this.size
      }
    },
  
    getStartIndex(scrollTop) {
      // 用二分法查找滚动距离是 positionListArr 哪一项的 bottom 的值
      let start = 0, // positionListArr 的第一项的下标
      end = this.positionListArr.length - 1, // positionListArr 的最后一项的下标
      // 用 temp 存储得不到精确值时最终返回的值，因为可能滚动的距离的值不会等于 positionListArr 数组里
      // 任何一项的 bottom 的值，所以只好存储一个最接近的值 
      temp = null
      while (start <= end) {
        let midIndex = parseInt(start + (end - start) / 2), // 中间那一项的 index，也可以用 Math.floor 代替 parseInt
        midVal = this.positionListArr[midIndex].bottom // 中间那一项的 bottom 值
        if (scrollTop === midVal) {
          // 滚动的值刚好等于中间那一项的 bottom 值，则此时的展示起点 start 应该为中间那一项的下一项 
          return midIndex + 1
        } else if (scrollTop < midVal) {
          // 滚动的值小于中间那一项的 bottom 值:
          end = midIndex - 1 // 让 end 移动到中间的那一项的前面一项
          if (temp === null || temp > midIndex ) // temp > midIndex 保证了目标值所在范围越变越小
            temp = midIndex
        } else if (scrollTop > midVal) {
          // 滚动的值大于中间那一项的 bottom 值:
          start = midIndex + 1 // 让 start 移动到中间的那一项的后面一项
          if (temp === null || temp < midIndex )
            temp = midIndex
        }
      }
      return temp
    },
    
    cacheListPosition() {
      // 缓存数据数组每一项的高度，top 值和 bottom 值
      // 注意: () => ({}) 为返回对象的写法
      this.positionListArr = this.arrayData.map((item, index) => ({
        height: this.size,
        top: index * this.size,
        bottom: (index + 1) * this.size
      }))
    }
  }
}
</script>
```

### 解释说明

原来对于列表如果全部渲染应该有的高度的计算 `this.arrayData.length * this.size + 'px'` 显然不合适了，因为每项 `item` 的高度 `size` 不确定了。在开始重新计算之前，先介绍一下二分法：

#### 二分法

- **使用前提**：数组已经按升序排列
- **基本原理**：首先将要查找的值（value）同数组中间那一项的值（midValue）进行比较，当 `start <= end`
	1. 如果 `value < midValue`，则 `end = mid - 1`，只需要在数组的前一半元素中继续查找
	2. 如果 `value = midValue`，匹配成功，查找结束
	3. 如果 `value > midValue`，则 `start = mid + 1`，只需要在数组的后一半元素中继续查找
	4. 如果 `while` 循环结束后都没有找到 `value`，返回 `-1`

```
const arr = [-1, 5, 6, 12, ...]
const start = 0,
end = arr.length -1,
mid = start + (end - start) / 2
```

**注意：** 在二分法中，计算中间项的索引时用的是 mid = start + (end - start) / 2 而不是直接使用更简单的公式 mid = (start + end) / 2，是为了防止值溢出的情况，因为 start + end 的值可能会大于 js 最大的能表示的数。（如果 start < 0 或 end < 0 时，end - start 也可能会溢出）

#### 利用二分法重新计算 start

1. 在页面加载完毕后，对数据数组里每一项的 `height`, `top` 和 `bottom` 的值做个缓存（此时的 `size` 为我们预估的，滚动条的高度并不准确），存放在数组 `positionListArr` 里；
2. 用二分法开始查找，我们页面滚动的距离 `scrollTop` 对应于 `positionListArr` 里的哪一项的 `bottom` 的值。之所以用二分法是因为后面会根据真实 dom 重行计算每一项的 `height`, `top` 和 `bottom`，到时候每一项的 `size` 就可能不一样了；
3. 之后对于 `end` 和 `offset` 计算原理就跟 `item` 高度固定的情况一样了。

#### 页面更新后

页面渲染完成后，获取到真实的 dom，更正缓存在 `positionListArr` 里的数据，实现更新滚动条的高度。这部分代码用到了 `ref` 和 `getBoundingClientRect` 的相关知识：

- 如果 `ref` 是写在 `v-for` 的元素或组件的时候，引用信息将是包含 DOM 节点或组件实例的数组。
- `Element.getBoundingClientRect()` 方法返回元素的大小及其相对于视口的位置，除了 `width` 和 `height` 以外的属性是相对于视图窗口的左上角来计算的，如下图：
		![](https://cdn.wallleap.cn/img/pic/illustration/202308041802468.png)

## 添加节流及效果展示

## 滚动节流

最后，给 `handleScroll` 添加个节流效果，提升性能：

1. 安装 lodash：`npm i --save lodash`
2. 在 VirtualList.vue 引入：`import throttle from 'lodash/throttle'`
3. 使用：新定义 `scrollFn` 方法，替换 `<div class="wrap" ref="wrap" @scroll="handleScroll">` 里的 `handleScroll`

## 效果图

![](https://cdn.wallleap.cn/img/pic/illustration/202308041802469.gif)

## One More Thing

## 仅用 css 就搞定长列表渲染的优化

最近看到一个新的 css 属性 `content-visibility`，利用这个属性（大概率还得配合 `contain-intrinsic-size`）就可以实现只渲染当前可视窗口区域内的内容，跳过不在屏幕上的内容渲染。但目前兼容性极差，只能说未来可期~
