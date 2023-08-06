---
title: Vue 3 源码导读
date: 2023-08-04 17:47
updated: 2023-08-04 17:48
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - Vue 3 源码导读
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

5 号凌晨，尤雨溪公布了 [Vue 3 源代码](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue-next "https://github.com/vuejs/vue-next")。

![](https://cdn.wallleap.cn/img/pic/illustration/202308041747620.jpeg)

话不多说，我们趁热对 Vue 3 源码进行一些简要的分析。

如果你还没有阅读过 [Composition API RFC](https://link.juejin.cn/?target=https%3A%2F%2Fvue-composition-api-rfc.netlify.com%2F%23basic-example "https://vue-composition-api-rfc.netlify.com/#basic-example")，可能无法完全看懂下面的内容。

## 兼容性

目前打包后的代码是 ES2015+，不支持 IE 11。

## 对 TypeScript 的使用

目前的代码 98% 以上使用 TypeScript 编写。

如果你还没有学习 TypeScript，请尽快学习，否则可能看不懂源码。

另外有件事情说出来可能会让你非常惊讶，Vue 3 的源代码**完全没有**使用 class 关键字！（只在测试代码和示例代码里用到了 class 关键字）

## 什么时候发正式版

目前 Vue 3 处于 Pre-Alpha 版本。后面应该还会有 Alpha、Beta 等版本。

根据 [Vue 官方时间表](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue%2Fprojects%2F6 "https://github.com/vuejs/vue/projects/6")，至少要等到 2020 年第一季度才有可能发布 3.0 正式版。

## 阅读建议

强烈推荐大家用国庆假期这段时间把 Vue 3 的源码通看一遍，因为目前的代码结构清晰，而且代码量相对较少。

下载代码后，使用 yarn dev 命令就可以对其进行调试。（有人给出了详细的 [调试技巧](https://juejin.cn/post/6844903957836333069 "https://juejin.cn/post/6844903957836333069"))

关于阅读顺序，我的建议是

1. 先读 reactivity，能最快了解 Vue 3 的新特性；
2. 再读 rumtime，理解组件和生命周期的实现；
3. 如果还有时间再读 compiler，理解编译优化过程。

另外如果你想省时间，可以直接看所有 `__tests__` 目录里的测试用例来了解 Vue 3 的所有功能。目前有不到 700 个测试用例。

## 代码结构

代码仓库中有个 packages 目录，里面是 Vue 3 的主要功能的实现，包括

- reactivity 目录：数据响应式系统，这是一个单独的系统，可以与任何框架配合使用。
- runtime-core 目录：与平台无关的运行时。其实现的功能有虚拟 DOM 渲染器、Vue 组件和 Vue 的各种 API，我们可以利用这个 runtime 实现针对某个具体平台的高阶 runtime，比如自定义渲染器。
- runtime-dom 目录: 针对浏览器的 runtime。其功能包括处理原生 DOM API、DOM 事件和 DOM 属性等。
- runtime-test 目录: 一个专门为了测试而写的轻量级 runtime。由于这个 rumtime 「渲染」出的 DOM 树其实是一个 JS 对象，所以这个 runtime 可以用在所有 JS 环境里。你可以用它来测试渲染是否正确。它还可以用于序列化 DOM、触发 DOM 事件，以及记录某次更新中的 DOM 操作。
- server-renderer 目录: 用于 SSR。尚未实现。
- compiler-core 目录: 平台无关的编译器. 它既包含可扩展的基础功能，也包含所有平台无关的插件。
- compiler-dom 目录: 针对浏览器而写的编译器。
- shared 目录: 没有暴露任何 API，主要包含了一些平台无关的内部帮助方法。
- vue 目录: 用于构建「完整构建」版本，引用了上面提到的 runtime 和 compiler。

可以看出，新的 Vue 代码仓库是模块化的。接下来我们逐一来看看每个模块暴露的 API。

### @vue/runtime-core 模块

大部分 Vue 开发者应该不会用到这个模块，因为它是专门用于自定义 renderer 的。

使用方法示例：

```js
import { createRenderer, createAppAPI } from '@vue/runtime-core'

const { render, createApp } = createRenderer({
  pathcProp,
  insert,
  remove,
  createElement,
  // ...
})

// `render` 是底层 API
// `createApp` 会产生一个 app 实例，该实例拥有全局的可配置上下文
export { render, createApp }

export * from '@vue/runtime-core'
```

### @vue/runtime-dom 模块

这个模块是基于上面模块而写的浏览器上的 runtime，主要功能是适配了浏览器环境下节点和节点属性的增删改查。它暴露了两个重要 API：render 和 createApp，并声明了一个 ComponentPublicInstance 接口。

```js
export { render, createApp }

// re-export everything from core
// h, Component, reactivity API, nextTick, flags & types
export * from '@vue/runtime-core'

export interface ComponentPublicInstance {
  $el: Element
}
```

### @vue/runtime-test 模块

这个模块的主要功能是用对象来表示 DOM 树，方便测试。并且提供了很多有用的 API 方便测试：

```js
export { render, createApp }

// convenience for one-off render validations
export function renderToString(vnode: VNode) {
  const root = nodeOps.createElement('div')
  render(vnode, root)
  return serializeInner(root)
}

export * from './triggerEvent'
export * from './serialize'
export * from './nodeOps'
export * from './jestUtils'
export * from '@vue/runtime-core'
```

### @vue/reactivity 模块

这是一个极其重要的模块，它是一个数据响应式系统。其暴露的主要 API 有 ref（数据容器）、reactive（基于 Proxy 实现的响应式数据）、computed（计算数据）、effect（副作用） 等几部分：

```js
export { ref, isRef, toRefs, Ref, UnwrapRef } from './ref'
export {
  reactive,
  isReactive,
  readonly,
  isReadonly,
  toRaw,
  markReadonly,
  markNonReactive
} from './reactive'
export {
  computed,
  ComputedRef,
  WritableComputedRef,
  WritableComputedOptions
} from './computed'
export {
  effect,
  stop,
  pauseTracking,
  resumeTracking,
  ITERATE_KEY,
  ReactiveEffect,
  ReactiveEffectOptions,
  DebuggerEvent
} from './effect'
export { lock, unlock } from './lock'
export { OperationTypes } from './operations'
```

很明显，这个模块就是 Composition API 的核心了，其中的 ref 和 reactive 应该重点掌握。

### @vue/compiler-core 模块

这个编译器的暴露了 AST 和 baseCompile 相关的 API，它能把一个字符串变成一棵 AST。

```js
export function baseCompile(
  template: string | RootNode,
  options: CompilerOptions = {}
): CodegenResult {
  // 详情略 ...
  return generate(ast, options)
}

export { parse, ParserOptions, TextModes } from './parse'
export { transform /* ... */ } from './transform'
export { generate, CodegenOptions, CodegenContext, CodegenResult} from './codegen'
export { ErrorCodes, CompilerError, createCompilerError } from './errors'
export * from './ast'
```

### @vue/compiler-dom 模块

这个模块则基于上个模块，针对浏览器做了适配，如对 textarea 和 style 标签做了特殊处理。

### @vue/server-renderer 模块

目前这个模块没有实现任何功能。

### vue 模块

这个模块就是简单的引入了 runtime 和 compiler：

```js
import { compile, CompilerOptions } from '@vue/compiler-dom'
import { registerRuntimeCompiler, RenderFunction } from '@vue/runtime-dom'

function compileToFunction(
  template: string,
  options?: CompilerOptions
): RenderFunction {
  const { code } = compile(template, {
    hoistStatic: true,
    ...options
  })
  return new Function(code)() as RenderFunction
}

registerRuntimeCompiler(compileToFunction)

export { compileToFunction as compile }
export * from '@vue/runtime-dom'
```

## 关注我

关注我，如果 Vue 3 有任何新消息，我会第一时间给出分析。

完。
