---
title: 用 Echarts 写一个低仿甘特图
date: 2023-08-04 11:42
updated: 2023-08-04 11:48
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - 用 Echarts 写一个低仿甘特图
rating: 1
tags:
  - Echarts
  - web
category: web
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: https://juejin.cn/post/7020213150473535524
url: //myblog.wallleap.cn/post/1
---

小知识，大挑战！本文正在参与“[程序员必备小知识](https://juejin.cn/post/7008476801634680869 "https://juejin.cn/post/7008476801634680869")”创作活动。

本文已参与 [「掘力星计划」](https://juejin.cn/post/7012210233804079141 "https://juejin.cn/post/7012210233804079141") ，赢取创作大礼包，挑战创作激励金。

## 开始之前

前两天用 Html 原生标签配合 Vue 完成了一个低仿版的甘特图 [相关文章](https://juejin.cn/post/7017643739849965575 "https://juejin.cn/post/7017643739849965575")，那么同样的功能用 Echarts 写起来有何不同？欲知详情，请按滚轮下滑😉。

老规矩，先来看看长啥样 (大致功能点)

![](https://cdn.wallleap.cn/img/pic/illustration/202308041143485.gif)

## 做些准备

开发这样的一个小型甘特图需要准备些什么？

1.本文案例使用 Vue 和 Echarts，交互上的组件使用了 Element-ui。

2.Echarts 的相关知识点 (会看**文档**就行)。

## 梳理思路

简而言之就是魔改 Echarts 中的**横向柱状图**。跟上个 Demo 一样，在横向时间轴的渲染及给定区域上色是共同的核心点，不同的是这里面的点击类似事件没 div 中好处理 (比较而言)，先提一嘴，我们后面会分析。

## 初始化

> 用 vue 脚手架脚手架创建一个项目 (Vue2.x)，安装好 Echarts 和 Element,**项目源码**在文末查看。 创建两个组件 Home.vue 和 Gantt.vue，后者作为前者的子组件。在 Home 组件中使用

```vue
 <Gantt :baseDate="baseDate" ref="gantt" :ganttData="ganttData" @getInfoCallback="getGanttInfo"  :roomData="roomData"></Gantt>
```

Gantt 组件中接收了三个 props 值

> baseDate：时间选择器,(yyyy-mm-dd 格式)
> roomData：左侧会议室数据，对应 Y 轴。
> ganttData：甘特图内容数据。

## 要点分析

在前面的效果图中可以看到展示的是 8:00 到 18:00，对应的是 Echarts X 轴的配置。

### 时间轴渲染

```js
xAxis: {
  type: 'time',
  position: 'top',
  interval: 3600 * 1000,         // 以一个小时递增
  // max:`${this.baseDate} 24:00`,
  max: `${this.baseDate} 19:00`, // 设置最大时间为19点,不包括19点
  min: `${this.baseDate} 08:00`, //最小时间8点
  axisLabel: {
    formatter: function (value, index) {
      var data = new Date(value)
      var hours = data.getHours()
      return hours + ':00'
    },
    textStyle: {
      color: 'rgba(0,0,0,0.65)', // 更改坐标轴文字颜色
      fontSize: 14 // 更改坐标轴文字大小
    }
  },
  axisLine: {
    lineStyle: {
      color: '#e5e5e5'
    },
    onZero: false
  },
  splitLine: {
    show: true,
    lineStyle: {
      type: 'dashed'
    }
  }
},
```

就是修改 Echarts 配置项中的 xAxis 对象，type 设置为 'time'，因为我们这里要用时间; 顶部显示 position 为 top；interval 是以毫秒为单位的，所以这里我们换算成小时，再设置 min 和 max 区间就行了，这里用到了传入的时间**baseDate**。由于显示上的优化，横坐标的显示有做格式化，只显示**小时:00**。

![](https://cdn.wallleap.cn/img/pic/illustration/202308041143486.png)

### 左侧类别渲染

home.vue

```js
created(){
 this.roomData = [
    '会议室一',
    '会议室二',
    '会议室三',
    '会议室四'
  ]
}
```

gantt.vue

```js
yAxis: {
  inverse: true, // 是否反转
  type: 'category',
  axisLine: {
    lineStyle: {
      color: '#e5e5e5'
    }
  },
  data: this.roomData,   //会议室数据
  axisLabel: {
    textStyle: {
      color: 'rgba(0, 0, 0, 0.65)', // 刻度颜色
      fontSize: 14 // 刻度大小
    }
  }
},
```

使用传入的 roomData 数据给到 Echarts 配置项中的 yAxis，这里的 inverse 属性顾名思义，翻转 Y 轴，Echarts 默认的 Y 轴数据是从下到上渲染的，假如你想翻过来，inverse 设置为 true 即可。

### 主体内容渲染 (核心)

前面提到的三个传入的 props 我们已经用了两个，ganttData 该派上用场了，之所以是这样的结构，是因为 Echarts 数据格式是这么要求的。

Home.vue

```js
created () {
  this.ganttData = [
    {
      value: [
        {
          index: 0,
          roomName: '会议室一',
          RoomId: '2234',
          id: '444',
          startTime: `${this.baseDate} 10:28`,
          endTime: `${this.baseDate} 12:28`,
          status: '0',
          content: '吃饭'
        }
      ]
    }
  ]
},
```

Gantt.vue

```js
series: [
  {
    type: 'custom',
    clickable: false,
    renderItem: function (params, api) {
      var categoryIndex = api.value(0).index // 使用 api.value(0) 取出当前 dataItem 中第一个维度的数值。
      var start = api.coord([api.value(0).startTime, categoryIndex]) //使用 api.coord(...) 将数值在当前坐标系中的值转换成为屏幕上的点的像素值。
      var end = api.coord([api.value(0).endTime, categoryIndex])
      var height = 40
      return {
        type: 'rect',  //矩形
        shape: echarts.graphic.clipRectByRect({
         // 矩形的位置和大小。
          x: start[0],
          y: start[1] - height / 2,
          width: end[0] - start[0],
          height: height
        }, {
          // 当前坐标系的包围盒。
          x: params.coordSys.x,
          y: params.coordSys.y,
          width: params.coordSys.width,
          height: params.coordSys.height
        }),
        style: api.style()
      }
    },
    label: {
      normal: {
        show: true,
        position: 'insideBottom',
        formatter: function (params) {
          return params.value[0].content
        },
        textStyle: {
          align: 'center',
          fontSize: 14,
          fontWeight: '400',
          lineHeight: '30' 
        }
      }
    },
    encode: {
      x: [0], 
      y: 0      
    },

    itemStyle: {
      normal: {
        color: function (params) {
          if (params.value[0].status === '1') return '#4dc394'
          else return '#e5835b'
        }
      }
    },

    data: this.ganttData
  }
]
```

由于我们这个例子的特殊性，设置 type 属性值为 'custom'，开启 Ecahrts 的自定义渲染函数 renderItem，api.value(0) 即为需要渲染的数据项。

![](https://cdn.wallleap.cn/img/pic/illustration/202308041143487.png)

关于这里用到的详细 api，可以查看 Echarts 官网关于这部分的详细说明，下面我们简要说明一下。 [echarts.apache.org/zh/option.h…](https://link.juejin.cn/?target=https%3A%2F%2Fecharts.apache.org%2Fzh%2Foption.html%23series-custom "https://echarts.apache.org/zh/option.html#series-custom")

#### [renderItem(params,api)](https://link.juejin.cn/?target=https%3A%2F%2Fecharts.apache.org%2Fzh%2Foption.html%23series-custom.renderItem "https://echarts.apache.org/zh/option.html#series-custom.renderItem")

```js
var categoryIndex = api.value(0).index 
var start = api.coord([api.value(0).startTime, categoryIndex]) 
var end = api.coord([api.value(0).endTime, categoryIndex])
var height = 40
```

> categoryIndex 就是拿到这一项数据在 ganttData 中的 index 值 (已经给定的，非排序的 index）。通过 api.coord 计算出这条数据在坐标系中的位置信息,即 start 和 end 数据。
> height 给定了 40，没有用 api 去计算这一行的高度，原因是渲染占满一行感觉挺丑，读者可自行修改 (参考上文链接)

```js
x: start[0],
y: start[1] - height / 2,
width: end[0] - start[0],
height: height
```

> 根据计算出 X,Y 便得出了矩形在坐标系中的位置，并设置宽高

### [encode](https://link.juejin.cn/?target=https%3A%2F%2Fecharts.apache.org%2Fzh%2Foption.html%23series-custom.encode "https://echarts.apache.org/zh/option.html#series-custom.encode")

```js
encode: {
  x: 1, // 这个例子中的可选参数,因为这个例子中的维度根据一个对象，可以写不为0参数或者不写这个参数。
  y: 0 // 把"维度0"映射到 Y 轴。
},
```

如你所见，ganttData 数据中的 value 数组里是一个对象。

```js
this.ganttData = [
  {
    value: [
      {
        index: 1,
        roomName: '会议室二',
        RoomId: '123',
        id: '333',
        startTime: `${this.baseDate} 8:28`,
        endTime: `${this.baseDate} 9:28`,
        status: '1',
        content: '睡觉'
      }
    ]
  }
]
```

### itemStyle

很简单，根据不同的状态返回不同的**颜色**

```js
itemStyle: {
  normal: {
    color: function (params) {
      if (params.value[0].status) return '#4dc394'
      else return '#e5835b'
    }
  }
},
```

渲染的部分就告一段落，看看交互。

## 事件交互

鼠标移入图例显示缩略信息，点击新增或者编辑维护数据。

> Gantt.vue

### 鼠标移入

> 首先增加一个移入显示 tooltip 的功能，这部分其实也是配置，echarts 中 option 的 tooltip 属性。

```js
tooltip: {
  trigger: 'item',
  show: true,
  hideDelay: 100,
  backgroundColor: 'rgba(255,255,255,1)',
  borderRadius: 5,
  textStyle: {
    color: '#000'
  },
  formatter: function (params) {
    const item = params.data.value[0]
    return item.content + '<br/>' +
    (item.status==='1' ? '<span style="color:#4dc394;">已完成</span>'
      : '<span style="color:#e5835b;">进行中</span>') + '<br/>' +
      item.startTime + ' - ' + item.endTime
  }
},
```

### 新增&编辑

这部分我开始做的时候其实有点小头疼，原因在于要区分点击区域，新增的内容应该隶属哪个会议室？修改的话修改的又是哪个?我是这么做的:

```js
// 任意位置点击事件----注册双击
myChart.getZr().on('click', params => {
  if (!params.target) {
  // 点击在了空白处，做些什么。
    const point = [params.offsetX, params.offsetY]
    if (myChart.containPixel('grid', point)) {
    // 获取被点击的点在y轴上的索引
      const idxArr = myChart.convertFromPixel({ seriesIndex: 0 }, point)
      const xValue = new Date(+idxArr[0]).getHours()
      const yValue = idxArr[1]
      const sendData = [xValue, yValue]
      this.$emit('getInfoCallback', sendData)
    }
  }
})
// 图例点击事件-返回数据给父组件---单击事件
myChart.on('click', params => {
  this.$emit('getInfoCallback', params.data.value)
})
}
```

使用 Echarts 提供的**任意区域点击事件**api 根据 params 有没有 targat 属性来判断是**新增**还是**编辑**。

#### 新增

进入 if 逻辑，yValue 得到的是点击区域 Y 轴的索引值，也就是 roomData 对应会议室的索引值。 emit 给父组件 (Home）返回 sendData:\[横轴时刻，y 轴的索引\]，父组件中取第二项就可以拿到点击的对应会议室索引。

#### 编辑

编辑则直接注册 click 事件，通过参数拿到图例的信息给父组件，你可能会问，为什么不在上面的 if(!params.target）的 else 逻辑拿？因为没有，任意区域点击事件只返回它的**位置信息**。

Gantt.vue 的逻辑就是这么简单。

#### 小坑

有一个小坑，就是 Echarts 图例销毁，因为时间选择器选择之后要重新初始化 Echarts，那么就需要先销毁再创建

```js
myEcharts () {
  const container = document.getElementById('main')
  this.$echarts.init(container).dispose()
  var myChart = this.$echarts.init(container)
}
```

> Home.vue

父组件中接受到子组件的数据就可以新增或者编辑，这里的逻辑看文末代码就可以直接明白。 值得一提的是一个数据的**注意点**

ganttData 中的 index 要和 roomData 中会议室名字索引值一一对应，在工作中的话，这些数据全部由后端返回，读者可自行组织 (该数据结构只适用于本案例)，大功告成😁。如果你看到了这里，点个赞再走吧，谢谢。

![](https://cdn.wallleap.cn/img/pic/illustration/202308041143488.png)

## 项目源码

> [沙盒地址](https://link.juejin.cn/?target=https%3A%2F%2Fcodesandbox.io%2Fs%2Fvue-echarts-gantt-forked-z5kfk "https://codesandbox.io/s/vue-echarts-gantt-forked-z5kfk")

## 往期文章

[手写一个精确到秒的低仿甘特图](https://juejin.cn/post/7017643739849965575 "https://juejin.cn/post/7017643739849965575")

[Teleport，Vue3.0新特性里的"任意门](https://juejin.cn/post/7016866669968490532 "https://juejin.cn/post/7016866669968490532")

[移动端Picker组件滚动穿透问题踩坑](https://juejin.cn/post/7013600126715461640 "https://juejin.cn/post/7013600126715461640")

[Ant Design of Vue没有表格合计行api，换种思路](https://juejin.cn/post/7012784170128637966 "https://juejin.cn/post/7012784170128637966")
