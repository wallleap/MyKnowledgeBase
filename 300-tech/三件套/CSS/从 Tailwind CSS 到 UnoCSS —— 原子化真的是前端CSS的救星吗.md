---
title: 从 Tailwind CSS 到 UnoCSS —— 原子化真的是前端CSS的救星吗
date: 2023-08-04 11:08
updated: 2023-08-04 11:12
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - 从 Tailwind CSS 到 UnoCSS —— 原子化真的是前端CSS的救星吗
rating: 1
tags:
  - CSS
  - web
category: web
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: //juejin.cn/post/7263049551136636983
url: //myblog.wallleap.cn/post/1
---

## 小编前言

追忆往昔，穿越前朝，CSS 也是当年前端三剑客之一，风光的很，随着前端跳跃式的变革，CSS 在现代前端开发中似乎有点默默无闻起来。

不得不说当看到 `UnoCSS` 之前，我甚至都还没听过原子化 CSS 这个概念（不够卷，惭愧，惭愧），很久没关注过 CSS 相关的内容了。

原子化 CSS 本身的概念和 Tailwind CSS、UnoCSS 设计都比较简单，这里主要想聊一下在现代前端中，原子化 CSS 到底是不是完美的解决方案，是不是解决 CSS 问题的正确方向。

## 概念 - 原子化 CSS

> 原子化 CSS 是一种 CSS 的架构方式，它倾向于小巧且用途单一的 class，并且会以视觉效果进行命名。

听起来厉害，但实现的最终方式超级简单，核心就是预置一大堆 class 样式，尽量将这些 class 样式简单化、单一化，在开发过程中，可以直接在 DOM 中写预置好的 class 名快速实现样式，而不需要每次写简单枯燥大量的 css 样式，如下代码所示：

先预置一组 class 列表

```css
.m-10 { margin: 10px; }
.p-5 { padding: 5px; }
.text-red { color: red; }
 // 无数个....
```

编码时在 dom 中直接写 class 名，快速实现样式

```html
<div class="m-10 p-5 text-red">
 测试dom
</div>
```

而预置的 class 列表中的样式，有着一定的规律，开发者可以通过学习快速掌握，利用多个 class 在 dom 中的**组合**快速实现效果

😮😮，是不是看起来有点熟悉？（死去的 **Bootstrap** 攻击我），所以你觉得这是冷饭新炒，还是实用主义？

## Tailwind CSS 广受欢迎

![](https://cdn.wallleap.cn/img/pic/illustration/202308041109569.jpeg)

在 [tailwindcss 的 github](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Ftailwindlabs%2Ftailwindcss "https://github.com/tailwindlabs/tailwindcss") 上目前居然已经有了 `70.8K` 个 star ！！ ，最开始我是难以置信它竟会如此受欢迎的，npm 周下载量也超六百多万 。

Tailwind CSS 主张让你无需离开 HTML 即可快速构建网站，并有许多非常非常实用的**优点**：

- 对 class 极致的封装，最大程度提高复用性
- 易扩展的的各种自定义配置
- 构建时清除未引用的 css 样式，解决生产环境 CSS 冗余的问题
- 舒适的响应式开发体验
- 函数、指令、布局、动画，......等等

几乎囊括了开发者日常原生 CSS 开发中遇到的所有头痛，不舒服的开发体验，所以说他确实是很实用的工具。Tailwind CSS 真的是为前端开发者造福了。

在前端构建工具链中 Tailwind CSS 作为 PostCSS 插件可与其他预处理结合使用，并优化 CSS 生产产物。

## UnoCSS

Tailwind CSS 在过去几年广受欢迎，近两年 `UnoCSS` 又脱颖而出，这里需要先夸一下 UnoCSS 官网的图标&文字&背景色的色彩渐变联动主题，很酷！！

![](https://cdn.wallleap.cn/img/pic/illustration/202308041109571.jpeg)

按 UnoCSS 作者的说法，**UnoCSS 并非要替代 Tailwind CSS 而是从另一个角度使原子化 CSS 在业务中融合的更完美**。

UnoCSS 作者是 Vite 团队成员，我觉得正是因为他作为 Vite 的开发者，对于工程化构建具有很高的敏感度，所以才能创造出 UnoCSS 将原子化的 CSS 与前端工程融合到极致。

那么 UnoCSS 做了哪些事呢：

**按需生成**

- 生成业务真正使用到的 class ，同时在开发和生产环境使用
- 对比 Tailwind CSS 只在生产环节清除无引用代码，UnoCSS 在开发环节也通过文件监听按需传输，获得更快地性能（虽然已经很快了，但再快一点总归是个提升嘛）

![](https://cdn.wallleap.cn/img/pic/illustration/202308041109572.jpeg)

![](https://cdn.wallleap.cn/img/pic/illustration/202308041109573.jpeg)

**极具灵活性**

UnoCSS 对自己定位是一个 CSS 引擎而非一个框架，所以它与 Tailwind CSS 应该是包含关系，UnoCSS 作为规则的制定者，而 Tailwind CSS 可以作为其中的一组 preset

```js
import UnocssPlugin from '@unocss/vite'

import PresetTachyons from '@unocss/preset-tachyons'
import PresetBootstrap from '@unocss/preset-bootstrap'
import PresetTailwind from '@unocss/preset-tailwind'
import PresetWindi from '@unocss/preset-windi'
import PresetAntfu from '@antfu/oh-my-cool-unocss-preset'

export default {
  plugins: [
    UnocssPlugin({
      presets: [
        // PresetTachyons,
        PresetBootstrap,
        // PresetTailwind,
        // PresetWindi,
        // PresetAntfu
        // 选择其中一个...或多个！
      ]
    })
  ]
}
```

属性化书写 class 名

```vue
// 将冗长的 calss 按类型区分，更方便阅读理解
<button class="bg-blue-400 hover:bg-blue-500 text-sm text-white font-mono font-light py-2 px-4 rounded border-2 border-blue-200 dark:bg-blue-500 dark:hover:bg-blue-600">
  Button
</button>

// 改变为
<button 
  bg="blue-400 hover:blue-500 dark:blue-500 dark:hover:blue-600"
  text="sm white"
  font="mono light"
  p="y-2 x-4"
  border="2 rounded blue-200"
>
  Button
</button>
```

在自定义规则上，UnoCSS 提供更加灵活的静态&动态匹配规则

编译进一步优化（比如不再解析 AST），生产构建速度再度提升

UnoCSS 等于是做了个更上层的引擎，以后再有新的原子化 CSS 框架也可以兼容进来了，省得你有选择困难症。

## 原子化 CSS 并非现代前端 CSS 的救星

Tailwind CSS 与 UnoCSS 的特性和使用方法并非本文主要想讲的，具体细节大家可移步官网查看，这里主要想讨论下

> **Tailwind CSS 或 UnoCSS 的原子化 CSS 会是现代前端解决 CSS 问题的最佳实践吗？**
> 
> 我觉得答案是否定的

从我的直接观感，**原子化 CSS 更像是一个辅助型的实用工具，而非 CSS 问题的解决方案，作为辅助工具，它确实是能在某类业务场景或者业务发展的某个阶段提供非常大的帮助，但从整个复杂多变的前端业务场景上看，它的能力是有限的**。

- 最直观的的结果就是当业务复杂度提升到某个阶段后，原子化 CSS 的性价比将急剧下降，带来的 HTML 代码冗余，可读性差，难以维护的问题将直接影响到业务开发。
- 尤其是现在无论是 Vue 的单文件组件还是 React 的函数式组件，都会将部分 JS 逻辑注入 HTML 中，如果 CSS 逻辑也要通过 class 名组合的方式注入其中，那就太混乱了。
- **但 Vue 或 React 各自的组件化设计思想，都可以通过各自的组件化拆分来解决代码冗余的问题，使其可适用于各种简单&复杂的业务场景，是一套完善的最佳实践，而原子化 CSS 没有办法做组件化拆分的，所以随着业务复杂度的上升，代码冗余迟早会发生，这直接限制了此类框架的普及，所以它无法作为前端 CSS 问题的根本解决方案。**

从设计思想上，原子化 CSS 看起来也与目前主流的组件化，函数式编程显得格格不入。

### 适用场景

在一些简单的业务场景上，原子化 CSS 确实有非常大的优势，比如快速开发响应式 H5，业务复杂度低的中后台系统，简单的官网页面。

而在一些复杂的业务场景，比如复杂的 C 端业务，大型的系统就不在那么适用了。

## 参考文献

[重新构想原子化CSS](https://link.juejin.cn/?target=https%3A%2F%2Fantfu.me%2Fposts%2Freimagine-atomic-css-zh "https://antfu.me/posts/reimagine-atomic-css-zh")
