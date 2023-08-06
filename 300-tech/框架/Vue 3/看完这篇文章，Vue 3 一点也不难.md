---
title: 看完这篇文章，Vue 3 一点也不难
date: 2023-08-04 17:01
updated: 2023-08-04 17:05
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - 看完这篇文章，Vue 3 一点也不难
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

## 前言

感觉时间好快啊，也有一个月多没更文了。这图片还是 2020 年产的呢~

开启 2021 年之旅，这篇文章是 2021 年首发...希望读者掘友多多支持哈~

整理了一下这段日子里学习的 Vue3，我想之后 Vue3 会成为一种趋势。那就捉急学起来吧~

## setup

这个 api 的调用时机：创建组件实例，然后初始化 props ，紧接着就调用 `setup` 函数。从生命周期钩子的视角来看，它会在 `beforeCreate` 钩子之前被调用。

我们可以在这个函数写大部分的业务逻辑，在 Vue2 中我们是通过每个选项的形式将代码逻辑分离开，比如 `methods，data，watch` 选项。Vue3 现在 ~改变了~这样的模式（ps: 也不能说改变吧，因为它也是能兼容 Vue2 的写法）

这难道不是三国的理论吗？`分久必合，合久并分` 嘛

setup 是有 2 个可选的参数。

- props --- 属性 (响应式对象 且 可以监听 (watch))
- context 上下文对象 --- 使用这个对象再也不用担心 this 到底指向哪里啦

## 生成响应式对象

如何理解响应式对象？我在另外一篇文章中使用例子讲述过一个场景，可查看：[【Vue.js进阶】总结我从Vue源码学到了什么（上）](https://juejin.cn/post/6899620558087356423#heading-5 "https://juejin.cn/post/6899620558087356423#heading-5")

现在来看看 Vue3 是如何生成响应式数据

### reactive

该函数接收一个对象作为参数，并返回一个代理对象。`reactive` 函数生成的对象如果没有合理的使用会丢失响应式。

先来看看什么情况下会失去响应式：

```js
setup(){
  const obj=reactive({
    name:'金毛',
    age:10
  });
  return{
    ...obj, //失去响应式了 因为obj失去了引用了
  }
},
```

如何解决这个失去响应式的问题？

```js
return {
 obj, //这样写那么模板就要都基于obj来调取, 类型{{obj.age}}
 ...toRefs(obj)   //用toRefs包装，必须是reactive生成的对象, 普通对象不可以, 它把每一项都拿出来包了一下, 这样就可以在模板中使用 {{age}}。
}
```

至于 toRefs 是什么接下来会讲述，现在先知道它是可以解决失去响应式的问题。

### ref

因为 `reactive` 函数可以代理一个对象，但无法代理基本数据类型，所以需要使用 `ref` 函数来间接对基本数据类型进行处理。该函数对基本数据类型数据进行装箱操作使得成为一个响应式对象，可以跟踪数据变化。

```xml
<span @click="addN">快来点击我呗~~</span>
<span>{{n}}</span> 
```

```js
setup(){
  const n=ref(1); //生成的n是一个对象, 这样方便vue去监控它
  function addN(){
    console.log(n.value,n,'.....')
    n.value++;  //注意这里要使用.value的形式, 因为n是对象, value才是他的值
  }
  return {
    n,      //返回出去页面的template才可以使用它, {{n}} 不需要.value
    addN,
  }
}
```

#### 如何实现 ref？

在实现 ref 之前，我们先来了解认识 Vue3 源码中提供的 `track` 和 `trigger` 这个两个 api。

> track 和 trigger 是依赖收集的核心

`track` 是用来跟踪收集依赖 (收集 effect)：接收三个参数。那 `trigger` 用来触发响应 (执行 effect)。（ps: 文章的后续会讲述如何去实现这两个 api，现在先留个疑问...）

这里是利用 js 是单线程的，那就可以在获取值的时候进行拦截依赖收集，在设置更新值时触发依赖的更新。所以 ref 的实现大致实现思路就可以写成：

```js
function myRef(val: any) {
  let value = val
  const ref = {
    get value() {
      // 收集依赖
      track(r, TrackOpTypes.GET, 'value')
      return value
    },
    set value(newVal: any) {
      if (newVal !== value) {
        value = newVal
        // 触发响应
        trigger(r, TriggerOpTypes.SET, 'value')
      }
    }
  }
  return ref
}
```

### toRef 与 toRefs

如上在 `reactive` 模块的内容中通过使用 `toRefs` 来包装生成的对象，那么所生成的对象是指向了对象相应 property 的 `ref`。

利用 Vue3 官网的🌰来加深理解：

```js
const state = reactive({
  foo: 1,
  bar: 2
})

const stateAsRefs = toRefs(state)
// ref 和 原始property “链接”
state.foo++
console.log(stateAsRefs.foo.value) // 2

stateAsRefs.foo.value++
console.log(state.foo) // 3
```

toRef 和 toRefs 实现原理是一致的。toRef 用来把一个响应式对象的的某个 key 值转换成 ref。而 toRefs 函数是把一个响应式对象的所有的 key 都转成了 ref。

#### 如何实现 toRef

因为 target 目标对象本身就是一个响应式数据，已经经历过依赖收集，响应触发的这个拦截了，所以在实现 toRef 时就不需要了。

这里实现了 toRef，那么 toRefs 也自然而来就可以实现了...(ps: 通过遍历得到即可)

```js
function toRef(target, key) {
  return {
    get value() {
      return target[key]
    },
    set value(newVal){
      target[key] = newVal
    }
  }
}
```

### 思考

在使用关于 ref 相关的内容时，我们看到了在模板中没有.value 去获取数据，但是在 js 代码块中使用.value 去访问属性。

可以总结为 Vue3 的自动拆装箱：

JS ：需要通过.value 访问包装对象

模板: 自动拆箱，就是在模板中不需要使用.value 访问

## 副作用

在 Vue3 中有一个副作用的概念，那什么是副作用呢？

接下来讲述的 api 就是和这个副作用相关！

### effect & watchEffect

`effect` 该函数用于定义副作用，它的参数就是副作用函数，这个函数可能会产生副作用。默认情况下，这个副作用会先执行。

#### 如何理解这个副作用？

可以通过下面的代码来知晓：

```js
import { effect,reactive } from '@vue/reactivity';
// import {watchEffect} from '@vue/runtime-core';
// 使用 reactive() 函数定义响应式数据
const obj = reactive({ text: 'hello' })
// 使用 effect() 函数定义副作用函数
effect(() => {
   document.body.innerText = obj.text
})

// watchEffect(() => {
//      document.body.innerText = obj.text
// })

// 一秒后修改响应式数据，这会触发副作用函数重新执行
setTimeout(() => {
  obj.text += ' world'
}, 1000)
```

也就是说 effect 接收的回调函数 cb 就是一个副作用，当数据发生变化的时候，这个 cb 就会被触发...

#### 思考

```js
import {effect,reactive } from '@vue/reactivity';
const obj = reactive({ a: 1 })
effect(() => {
  console.log(obj.foo)
}
obj.a++
obj.a++
obj.a++
//结果：2，3，4
```

当 effect 函数和响应式数据建立了联系，那么只要响应式数据一发生改变，那么 effect 函数回调就会被执行。也就是说变几次执行几次。这样的性能是不是不好？？

effect 可以传递第二个参数 `{ scheduler: XXX }`， 指定调度器:XXX。

所谓调度器就是用来指定如何运行副作用函数的。

而 `watchEffect` 函数就是基于这个调度器的原理来优化实现副作用。

```js
import {reactive } from '@vue/reactivity';
 import {watchEffect} from '@vue/runtime-core';
const obj = reactive({ a: 1 })
watchEffect(() => {
  console.log(obj.foo)
}
obj.a++
obj.a++
obj.a++
//结果：4
```

那这个实现思路是什么呢？ 相当于用一个队列 queue 来收集这些 cb,在收集之前做一个判断看看队列是否已经存在这个 cb。然后通过 while 循环执行队列里的 cb 即可。 伪代码：

```js
const queue=  [];
let dirty = false;
function queueHandle(job) {
  if (!queue.includes(job)) queue.push(job)
  if (!dirty) {
    dirty = true
    Promise.resolve().then(() => {
      let fn;
      while(fn = queue.shift()) {
        fn();
      }
    })
  }
}
```

一般在开发环境下不使用 effect 而是使用 watchEffect。

#### 异步副作用

刚刚上面讲述的是在同步的情况下，那么异步的副作用比如数据发生变更时又会发生一次 ajax 请求。我们没办法判断哪一次的请求更快，这无形当中就给我们带来了不确定性...

那如何解决这种不确定呢？

想办法清理失效时的回调，我所想的是通过在执行这一次的副作用时，清理上一次的异步副作用，使得之前挂起的异步操作无效。

Vue3 通过在 watchEffect 传入的回调函数中可以接收一个 `onInvalidate` 函数作入参。

可以基于 effect 实现大致的原理：

```js
import { effect } from '@vue/reactivity'

function watchEffect(fn: (onInvalidate: (fn: () => void) => void) => void) {
  let cleanup: Function
  function onInvalidate(fn: Function) {
    cleanup = fn
  }
  // 封装一下 effect
  // 在执行副作用函数之前，先使上一次无作用无效
  effect(() => {
    cleanup && cleanup();
    fn(onInvalidate)
  })
}
```

### 如何停止副作用

Vue3 提供了一个 stop 函数，用来停止副作用。

在 effect 函数会返回一个值，这个值其实就是 effect 本身。将这个返回值传入到 stop 函数中，那么后续在更改数据，都无法实现 effect 的回调函数被调用。

### 区别

`watchEffect` 会维护与组件实例以及组件状态 (是否被卸载等) 的关系，如果一个组件被卸载，那么 `watchEffect` 也将被 `stop`，但 `effect` 不会。

`effect` 是需要我们收的去清楚副作用的，要不然它本身是不会主动被清除的。

## watch

再来谈谈 `watch`，它等同于组件的侦听器。`watch` 需要侦听特定的数据源，并在回调函数中执行副作用。默认情况下，它也是惰性的，即只有当被侦听的源发生变化时才执行回调。

这个 api 的实现和 Vue2 没什么很大的理解区别，关键 Vue3 在 Vue2 的基础上又扩展了一些功能，相对于 Vue2 更加完美了...

```js
// 特定响应式对象监听
// 也可开启immediate: true, 这个和2.0没什么区别
watch(
  text,
  () => {
    console.log("watch text:");
  }
);

// 特定响应式对象监听 可以获取新旧值
watch(
  text,
 (newVal, oldVal) => {
    console.log("watch text:", newVal, oldVal);
  },
);

// 多响应式对象监听
watch(
  [firstName,lastName],
 ([newFirst,newLast], [oldFirst,oldlast]) => {
   console.log(newFirst，'新的first值',newLast,'新的last值')
  },
  
);

```

与 watchEffect 比较，watch 允许我们：

- 懒执行副作用；
- 更具体地说明什么状态应该触发侦听器重新运行；
- 访问侦听状态变化前后的值。

## triggerRef

还记得 Vue2 的 `$forceUpdate` 这个强制性刷新吗？ 在 Vue3 中也有一个强制去触发副作用的 api。先来看看下面的代码：

```js
//shallowRef它只代理 ref 对象本身，也就是说只有 .value 是被代理的，而 .value 所引用的对象并没有被代理
const shallow = shallowRef({
  greet: 'Hello, world'
})

// 第一次运行时记录一次 "Hello, world"
watchEffect(() => {
  console.log(shallow.value.greet)
})

// 这不会触发作用，因为 ref 很浅层
shallow.value.greet = 'Hello, universe'

// 手动触发，记录 "Hello, universe"
triggerRef(shallow)

```

## 生命周期这件事

2.x 与 3.0 的对照

```
  beforeCreate -> 使用 setup()
  created -> 使用 setup()
  beforeMount -> onBeforeMount  ---只能在setup里面使用
  mounted -> onMounted---只能在setup里面使用
  beforeUpdate -> onBeforeUpdate---只能在setup里面使用
  updated -> onUpdated---只能在setup里面使用
  beforeDestroy -> onBeforeUnmount---只能在setup里面使用
  destroyed -> onUnmounted---只能在setup里面使用
  errorCaptured -> onErrorCaptured---只能在setup里面使用
```

## 获取真实的 Dom 元素

Vue2 的 ref 获取真实的 dom 元素 `this.$refs.XXX`,而 Vue3 也是通过 ref 获取真实的 dom 元素，但是写法上发生更改。

```js
 <div v-for="item in list" :ref="setItemRef"></div>
  <p ref="content"></p>
```

```js
import { ref, onBeforeUpdate, onUpdated } from 'vue'

export default {
setup() {
//定义一个变量接收dom
  let itemRefs = [];
  let content=ref(null);
  const setItemRef = el => {
    itemRefs.push(el)
  }
  onBeforeUpdate(() => {
    itemRefs = []
  })
  onUpdated(() => {
    console.log(itemRefs)
  })
  //返出去的名称要与dom的ref相同, 这样就可以接收到dom的回调
  return {
    itemRefs,
    setItemRef,
    content
  }
}
}
```

## 提供/注入 -- 开发插件

之前开发一个公共组件或者是要封装一个公共的插件时会将功能挂在原型上或者是使用 mixins。

其实这样子不好，挂在原型上使得 Vue 显得臃肿而且还会命名冲突的可能，mixins 混入会使得代码跳跃，让阅读者的逻辑跳跃。

现在有个新的方案**CompositionAPI**，来实现插件的开发

1.可以将插件的公共功能使用 provide 函数和 inject 函数包装。

```js
import {provide, inject} from 'vue';
// 这里使用symbol就不会造成变量名的冲突了, 这个命名权交给用户才是真正合理的架构设计
const StoreSymbol = Symbol()

export function provideString(store){
  provide(StoreSymbol, store)  //
}

//目标插件
export function useString() {
  const store = inject(StoreSymbol)  
  /*拿到具体的值了，可以做相关的功能了*/
  return store
}
```

2.在根组件进行数据的初始化，引入 provideString 函数，并进行数据传输

```js
export default {
  setup(){
    // 一些初始化'配置/操作'可以在这里进行
    // 需要放在对应的根节点, 因为依赖provide 和 inject
     provideString({
       a:'可能我是axios',
       b:'可能我是一个message弹框'
     })
  }
}
```

3.在想要的组件中引入插件，完成相关的功能的使用

```js
import { useString } from '../插件';

export default {
  setup(){
    const store = useString(); //可以使用这个插件了
  }
}
```

## Vue3 响应式原理

### vue2 的响应式缺点

- 默认就会递归
- 不支持数组改变长度来响应式
- 对象不存在的属性不会被拦截

### 如何实现

上述留下来的疑问，现在在这里实现。

我们先来分析一下 Vue3 响应式原理是如何实现的：

- Vue3 不在使用 `Object.defineProperty` 进行拦截了，相反替代的方案是 ES6 的 `Proxy`。
- 依赖收集不是通过 `Dep` 类和 `Watch` 类，而是通过 `track` 函数将目标对象和副作用进行相关联，通过 `trigger` 进行依赖的响应。
- 副作用的以栈形式的方式进行存储，先进后出的思想。 这只是大体的思路，实现过程中还有挺多细节的分析，接下来一步一步讲解...

#### 第一步

先来实现一个 reactivity，实现这个函数要思考的问题是：

1.{a:{b:2}}像这种多层嵌套的对象，如何进行响应更新？

2.一个对象多次调用 reactivity 函数，那么应该怎么处理？

3.一个对象的代理对象调用了 reactivity 函数，又应该怎么处理？

4.如何去判断对象是新增属性还是修改属性？

```js
//工具类函数
function isObject(obj){
  return typeof obj==='object' && obj!==null?true:false;
}
function isOwnKey(target,key){
  return Object.hasOwnProperty(target,key)?true:false;
}

```

```js
function reactivity(target){
 return  createReactivity(target);
}
let toProxy=new WeakMap();   //用来连接目标对象（key）与代理对象（value）的关系
let toRaw=new WeakMap();   //用来连接代理对象（key）与目标对象（value）的关系

function createReactivity(target){
  if(!isObject(target)){
    return ;
  }
  let mapProxy=toProxy.get(target); //处理目标对象被响应式处理多次的情况
  if(mapProxy){
    return mapProxy;
  }
  if(toRaw.has(target)){  //处理目标对象的代理对象被响应式处理  let proxy=reactivity（obj）;reactivity（proxy）;
    return target;
  }
  let proxy=new Proxy(target,{
    get(target,key,receiver){
      let res=Reflect.get(target,key);
      //在获取属性的时候，收集依赖
      track(target,key);  ++++
      return isObject(res)?reactivity(res):res;  //递归实现多层嵌套的对象
    },
    set(target,key,value,receiver){
      let res=Reflect.set(target, name, value,receiver);  
      let oldValue=target[key];  //获取老值，用于比较新值和老值的变化，改变了才修改
      
      /**通过判断key是否已经存在来判断是新增属性还是修改属性，并且在新增的时候可能会改变原有老的属性，这一点大多数人都不会被考虑到 */
      if(!isOwnKey(target,key)){
        console.log('新增属性');
        //在设置属性的时候发布
        trigger(target,'add',key); +++
      }else if(oldValue!==value){
        console.log('修改属性');
        trigger(target,'set',key); +++
      }
      return res;
    },
  })
  toProxy.set(target,proxy);
  toRaw.set(proxy,target);
  return proxy;
}
```

#### 第二步

现在来实现一个副作用 effect 函数，这个函数我们考虑最简单的方式，就是传入一个 fn 作为入参。

我们用一个全局的队列来存储 effect，这个队列的存储的方式正如我刚刚所说的是栈队列的方式。

那么在一上来 effect 就会默认执行一次，那么先收集 effect，然后利用 js 单线程的原理进行目标对象和 effect 进行相关联。

```js
let effectStack=[];  //栈队列，先进后出

function effect(fn){
  let effect=createEffect(fn);
  effect();  //默认先执行一遍
}
function createEffect(){
  let effect=function(){
    run(effect,fn);
  }
  return effect;
}
//run执行函数，功能1.收集effect,2.执行fn
function run(effect,fn){
  //利用try-finally来防止当发生错误的时候也会执行finally的代码
  //利用js是单线程执行的。先收集再关联
  try{
    effectStack.push(effect);
    fn();
  }finally{
    effectStack.pop();
  }
}
```

#### 第三步

这一步的关键是在收集依赖的时候如何让目标对象的 key 和 effect 产生联系。 在这里有个特殊的数据结构：

```js
 {
   target:{
     key1:[effect1,effect12],
     key2:[effect3,effect4],
     key3:[effect5],
   }
 }
```

每个目标对象（作为 key）都有对应的 value（是个对象），然后对象 (value) 又映射了 key 和 effect。

那么在响应依赖的时候，因为我们得到了 effect 和目标对象所对应的 key 的关系了，那么遍历触发即可。

```js
let targetMap=new WeakMap();
//收集依赖
function track(target,key){
  let effect=effectStack[effectStack.length-1]; //从栈获取effect看看是否有副作用
  if(effect){  //有关系才创建依赖
    let depMap=targetMap.get(target);
    if(!mapRarget){
      targetMap.set(target,(depMap=new Map()));
    }
    let deps=depMap.get(key);
    if(!deps){
      mapDep.set(key,(deps=new Set()));
    }
    if(!deps.has(effect)){
      deps.add(effect)
    }
  }
}
//响应依赖
function trigger(target,type,key){
  let depMap=targetMap.get(target);
  if(depMap){
   let deps= depMap.get(key);  //当前的key所对应的effect
   if(deps){
     deps.forEach(effect=>effect());
   }
  }
}

```

好了，到这里 Vue3 的响应式原理基本上是实现了，当然你可以看完本篇文章再去看看源码，我相信你会更容易理解源码了~

## 总结

在学习的过程中，翻过 Vue3 的源码看了 3 天，说实话还真挺头疼。后来我决定不先看源码了，先学会如何去使用 Vue3 开始,然后再想想为何会这么用，最后到如何实现。

求知的路上真的是 `路漫漫其修远兮，吾将上下而求索`

## 参考资料

[Vue3中文文档](https://link.juejin.cn/?target=https%3A%2F%2Fvue3js.cn%2Fdocs%2Fzh%2F "https://vue3js.cn/docs/zh/")

[【Vue3官方教程】🎄万字笔记 | 同步导学视频](https://juejin.cn/post/6909247394904702984 "https://juejin.cn/post/6909247394904702984")
