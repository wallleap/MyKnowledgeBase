---
title: 谁是 lottie?
date: 2023-08-09T10:17:00+08:00
updated: 2024-08-21T10:32:45+08:00
dg-publish: false
aliases:
  - 谁是 lottie?
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
  - web
url: //myblog.wallleap.cn/post/1
---

## 背景

在项目中，loading 常用的动画方案是 Gif 动画。

Gif 动画存在一些问题，例如：文件较大、无法缩放匹配不同屏幕大小和密度、易出现锯齿、无法控制动画等。

其他常用的动画方案有：

- Png 序列帧：文件大，可能会在不同屏幕分辨率下失真
- SVG 动画：实现成本高，易出现动画还原度低的情况

目前，项目需要经过调研，Lottie 动画是一种具有高可行性的方案。

![](https://cdn.wallleap.cn/img/pic/illustration/202308091018583.png)

## Lottie 简介

[Lottie](https://link.juejin.cn/?target=http%3A%2F%2Fairbnb.io%2Flottie%2F%23%2F "http://airbnb.io/lottie/#/") 是 airbnb 开源的动画库，支持多个平台如 Android、iOS、Web、React Native 和 Windows。其提供从 AE 到各终端的完整工具流程。设计师可以通过 AE 的 Bodymovin 插件将动画导出为 `json 文件`，然后通过 `Lottie 实现动画效果`，`确保动画的还原度`。

`lottie实现的动画效果：`

## lottie-web

[lottie-web](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fairbnb%2Flottie-web "https://github.com/airbnb/lottie-web") 支持多种特性，提供复杂的动画控制和监听功能。

使用示例如下：

```js
lottie.loadAnimation({
  container: animationWindow,
  renderer: 'svg',
  loop: true,
  autoplay: true,
  path: 'https://linwu-hi.github.io/lottie/animation_ll2ddfyg.json'
});
```

控制动画播放的方法：

| 名称 | 描述 |
| --- | --- |
| animation.play | 播放该动画，从目前停止的帧开始播放 |
| stop | 停止播放该动画，回到第 0 帧 |
| pause | 暂停该动画，在当前帧停止并保持 |
| goToAndStop | animation.goToAndStop(value, isFrame); 跳到某个时刻/帧并停止。isFrame(默认 false) 指示 value 表示帧还是时间 (毫秒) |
| goToAndPlay | animation.goToAndPlay(value, isFrame); 跳到某个时刻/帧并进行播放 |
| goToAndStop | animation.goToAndStop(30, true); 跳转到第 30 帧并停止 |
| playSegments | animation.playSegments(arr, forceFlag);arr 可以包含两个数字或者两个数字组成的数组，forceFlag 表示是否立即强制播放该片段 animation.playSegments(\[10,20\], false); 播放完之前的片段，播放 10-20 帧 animation.playSegments(\[\[0,5\],\[10,18\]\], true); 直接播放 0-5 帧和 10-18 帧 |
| setSpeed | animation.setSpeed(speed); 设置播放速度，speed 为 1 表示正常速度 |
| setDirection | animation.setDirection(direction); 设置播放方向，1 表示正向播放，-1 表示反向播放 |
| destroy | animation.destroy(); 删除该动画，移除相应的元素标签等。在 unmount 的时候，需要调用该方法 |

监听事件：

| 名称 | 描述 |
| --- | --- |
| data\_ready | 加载完 json 动画 |
| complete | 播放完成（循环播放下不会触发） |
| loopComplete | 当前循环下播放（循环播放/非循环播放）结束时触发 |
| enterFrame | 每进入一帧就会触发，播放时每一帧都会触发一次，stop 方法也会触发 |
| segmentStart | 每进入一帧就会触发，播放时每一帧都会触发一次，stop 方法也会触发 |
| DOMLoaded | 动画相关的 dom 已经被添加到 html 后触发 |
| destroy | 将在动画删除时触发 |

## Lottie 动画性能

![](https://cdn.wallleap.cn/img/pic/illustration/202308091018584.png)

> 对比 Lottie 和 Gif 动画，数据显示 Lottie 动画文件更小，帧率更高，而且其性能表现更好。

- GIF 动图

![](https://cdn.wallleap.cn/img/pic/illustration/202308091018585.gif)

- Lottie 动画

## react-lottie

> [react-lottie](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fchenqingspring%2Freact-lottie "https://github.com/chenqingspring/react-lottie") 将 lottie-web 封装成 React 组件，使其更加易于在 React 中使用。

```jsx
import React from 'react'
import Lottie from 'react-lottie';
import * as animationData from './pinjump.json'
 
export default class LottieControl extends React.Component {
 
  constructor(props) {
    super(props);
    this.state = {isStopped: false, isPaused: false};
  }
 
  render() {
    const buttonStyle = {
      display: 'block',
      margin: '10px auto'
    };
 
    const defaultOptions = {
      loop: true,
      autoplay: true, 
      animationData: animationData,
      rendererSettings: {
        preserveAspectRatio: 'xMidYMid slice'
      }
    };
 
    return <div>
      <Lottie options={defaultOptions}
              height={400}
              width={400}
              isStopped={this.state.isStopped}
              isPaused={this.state.isPaused}/>
      <button style={buttonStyle} onClick={() => this.setState({isStopped: true})}>stop</button>
      <button style={buttonStyle} onClick={() => this.setState({isStopped: false})}>play</button>
      <button style={buttonStyle} onClick={() => this.setState({isPaused: !this.state.isPaused})}>pause</button>
    </div>
  }
}
```

## vue-lottie

> [vue-lottie](https://link.juejin.cn/?target=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2Fvue-lottie "https://www.npmjs.com/package/vue-lottie") 将 lottie-web 封装成 vue 组件，使其更加易于在 vue 中使用。

也有 `vue3-lottie`

```vue
<template>
    <div id="app">
        <lottie :options="defaultOptions" :height="400" :width="400" v-on:animCreated="handleAnimation"/>
        <div>
            <p>Speed: x{{animationSpeed}}</p>
            <input type="range" value="1" min="0" max="3" step="0.5"
                   v-on:change="onSpeedChange" v-model="animationSpeed">
        </div>
        <button v-on:click="stop">stop</button>
        <button v-on:click="pause">pause</button>
        <button v-on:click="play">play</button>
    </div>
</template>
 
<script>
  import Lottie from './lottie.vue';
  import * as animationData from './assets/pinjump.json';
 
  export default {
    name: 'app',
    components: {
      'lottie': Lottie
    },
    data() {
      return {
        defaultOptions: {animationData: animationData},
        animationSpeed: 1
      }
    },
    methods: {
      handleAnimation: function (anim) {
        this.anim = anim;
      },
 
      stop: function () {
        this.anim.stop();
      },
 
      play: function () {
        this.anim.play();
      },
 
      pause: function () {
        this.anim.pause();
      },
 
      onSpeedChange: function () {
        this.anim.setSpeed(this.animationSpeed);
      }
    }
  }
</script>
```
