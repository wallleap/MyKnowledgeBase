---
title: 近一万字的 ES 6 语法知识点补充
date: 2023-08-07T09:13:00+08:00
updated: 2024-08-21T10:33:30+08:00
dg-publish: false
aliases:
  - 近一万字的 ES 6 语法知识点补充
author: Luwang
category: web
comments: true
cover: //cdn.wallleap.cn/img/post/1.jpg
description: 文章描述
image-auto-upload: false
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
rating: 1
source: #
tags:
  - JS
  - web
url: //myblog.wallleap.cn/post/1
---

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9eaae700fc2~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

## 前言

ECMAScript 6.0（简称 ES6），作为下一代 JavaScript 的语言标准正式发布于 2015 年 6 月，至今已经发布 3 年多了，但是因为蕴含的语法之广，完全消化需要一定的时间，这里我总结了部分 ES6，以及 ES6 以后新语法的知识点，使用场景，希望对各位有所帮助

**本文讲着重是对 ES6 语法特性的补充，不会讲解一些 API 层面的语法，更多的是发掘背后的原理，以及 ES6 到底解决了什么问题**

**如有错误,欢迎指出,将在第一时间修改,欢迎提出修改意见和建议**

话不多说开始 ES6 之旅吧~~~

## let/const（常用）

let,const 用于声明变量，用来替代老语法的 var 关键字，与 var 不同的是，let/const 会创建一个块级作用域（通俗讲就是一个花括号内是一个新的作用域）

这里外部的 console.log(x) 拿不到前面 2 个块级作用域声明的 let:

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9eaaef8cc17~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

在日常开发中多存在于使用**if/for**关键字结合 let/const 创建的块级作用域，**值得注意的是使用 let/const 关键字声明变量的 for 循环和 var 声明的有些不同**

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9eaaed7a489~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

for 循环分为 3 部分，第一部分包含一个变量声明，第二部分包含一个循环的退出条件，第三部分包含每次循环最后要执行的表达式，也就是说第一部分在这个 for 循环中只会执行一次 var i = 0，而后面的两个部分在每次循环的时候都会执行一遍

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9eab004895e~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

而使用使用 let/const 关键字声明变量的 for 循环，除了会创建块级作用域，let/const 还会将它绑定到每个循环中，确保对上个循环结束时候的值进行重新赋值

什么意思呢？简而言之就是每次循环都会声明一次（对比 var 声明的 for 循环只会声明一次），可以这么理解 let/const 中的 for 循环

给每次循环创建一个块级作用域:

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9eab036229e~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

### 暂时性死区

使用 let/const 声明的变量，从一开始就形成了封闭作用域，在声明变量之前是无法使用这个变量的，这个特点也是为了弥补 var 的缺陷（var 声明的变量有变量提升）

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/19/169065634290c00f~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

剖析暂时性死区的原理，**其实 let/const 同样也有提升的作用**，但是和 var 的区别在于

- var 在创建时就被初始化，并且赋值为 undefined
- let/const 在进入块级作用域后，会因为提升的原因先创建，但不会被初始化，直到声明语句执行的时候才被初始化，初始化的时候如果使用 let 声明的变量没有赋值，则会默认赋值为 undefined，而 const 必须在初始化的时候赋值。而创建到初始化之间的代码片段就形成了暂时性死区

引用 [一篇博客](https://link.juejin.cn/?target=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000008213835 "https://segmentfault.com/a/1190000008213835") 对于 ES6 标准翻译出来的一段话

> 由 let/const 声明的变量，当它们包含的词法环境 (Lexical Environment) 被实例化时会被创建，但只有在变量的词法绑定 (LexicalBinding) 已经被求值运算后，才能够被访问

回到例子，这里因为使用了 let 声明了变量 name,在代码执行到 if 语句的时候会先进入预编译阶段，依次创建块级作用域,词法环境,name 变量（没有初始化）,随后进入代码执行阶段，只有在运行到 let name 语句的时候变量才被初始化并且默认赋值为 undefined，但是因为暂时性死区导致在运行到声明语句**之前**使用到了 name 变量，所以报错了

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168e0aa07109eb76~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

上面这个例子,因为使用 var 声明变量,会有变量提升,同样也是发生在预编译阶段,var 会提升到当前函数作用域的顶部并且默认赋值为 undefined,如果这几行代码是在全局作用域下,则 name 变量会直接提升到全局作用域,随后进入执行阶段执行代码,name 被赋值为 "abc",并且可以成功打印出字符串 abc

相当于这样

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168e0abf693ab68b~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

暂时性死区其实是为了防止 ES5 以前在变量声明前就使用这个变量,这是因为 var 的变量提升的特性导致一些不熟悉 var 原理的开发者习以为常的以为变量可以先使用在声明,从而埋下一些隐患

关于 JS 预编译和 JS 的 3 种作用域 (全局,函数,块级) 这里也不赘述了,否则又能写出几千字的博客,有兴趣的朋友自行了解一下,同样也有助于了解 JavaScript 这门语言

### const

使用 const 关键字声明一个常量，常量的意思是不会改变的变量，const 和 let 的一些区别是

1. const 声明变量的时候必须赋值，否则会报错，同样使用 const 声明的变量被修改了也会报错

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9ead3323a16~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

1. const 声明变量不能改变，如果声明的是一个引用类型，则不能改变它的内存地址（这里牵扯到 JS 引用类型的特点，有兴趣可以看我另一篇博客 [对象深拷贝和浅拷贝](https://juejin.cn/post/6844903749270372365 "https://juejin.cn/post/6844903749270372365")）

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9eaff4bf8e6~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

有些人会有疑问，为什么日常开发中没有显式的声明块级作用域，let/const 声明的变量却没有变为全局变量

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9eb0879648b~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

这个其实也是 let/const 的特点，ES6 规定它们不属于顶层全局变量的属性，这里用 chrome 调试一下

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9eb0a403854~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

可以看到使用 let 声明的变量 x 是在一个叫 script 作用域下的，而 var 声明的变量因为变量提升所以提升到了全局变量 window 对象中，这使我们能放心的使用新语法，不用担心污染全局的 window 对象

### 建议

在日常开发中，我的建议是全面拥抱 let/const，一般的变量声明使用 let 关键字，而当声明一些配置项（类似接口地址，npm 依赖包，分页器默认页数等一些一旦声明后就不会改变的变量）的时候可以使用 const，来显式的告诉项目其他开发者，这个变量是不能改变的 (const 声明的常量建议使用全大写字母标识,单词间用下划线)，同时也建议了解 var 关键字的缺陷（变量提升，污染全局变量等），这样才能更好的使用新语法

## 箭头函数（常用）

ES6 允许使用**箭头**（=>）定义函数

箭头函数对于使用 function 关键字创建的函数有以下区别

1. 箭头函数没有 arguments（建议使用更好的语法，剩余运算符替代）
2. 箭头函数没有 prototype 属性，不能用作构造函数（不能用 new 关键字调用）
3. 箭头函数没有自己 this，它的 this 是词法的，引用的是上下文的 this，即在你写这行代码的时候就箭头函数的 this 就已经和外层执行上下文的 this 绑定了 (这里个人认为并不代表完全是静态的,因为外层的上下文仍是动态的可以使用 call,apply,bind 修改,这里只是说明了箭头函数的 this 始终等于它上层上下文中的 this)

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9eb0c098401~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

因为 setTimeout 会将一个匿名的回调函数推入异步队列，而回调函数是具有全局性的，即在非严格模式下 this 会指向 window，就会存在丢失变量 a 的问题，而如果使用箭头函数，在书写的时候就已经确定它的 this 等于它的上下文（这里是 makeRequest 的函数执行上下文，相当于将箭头函数中的 this 绑定了 makeRequest 函数执行上下文中的 this）因为是 controller 对象调用的 makeRequest 函数，所以 this 就指向了 controller 对象中的 a 变量

箭头函数的 this 指向即使使用 call,apply,bind 也无法改变（这里也验证了为什么 ECMAScript 规定不能使用箭头函数作为构造函数，因为它的 this 已经确定好了无法改变）

### 建议

箭头函数替代了以前需要显式的声明一个变量保存 this 的操作，使得代码更加的简洁

ES5 写法:

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9eb128894f7~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

ES6 箭头函数:

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9eb2071ae0b~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

再来看一个例子

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168e0cd852903baa~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

值得注意的是 makeRequest 后面的 function 不能使用箭头函数，因为这样它就会再使用上层的 this，而再上层是全局的执行上下文，它的 this 的值会指向 window,所以找不到变量 a 返回 undefined

在数组的迭代中使用箭头函数更加简洁，并且省略了 return 关键字

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9eb3a73b69d~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

不要在可能改变 this 指向的函数中使用箭头函数，类似 Vue 中的 methods,computed 中的方法,生命周期函数，Vue 将这些函数的 this 绑定了当前组件的 vm 实例，如果使用箭头函数会强行改变 this，因为箭头函数优先级最高（无法再使用 call,apply,bind 改变指向）

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168e0d5ae99d7e7c~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

在把箭头函数作为日常开发的语法之前,个人建议是去了解一下箭头函数的是如何绑定 this 的,而不只是当做省略 function 这几个单词拼写,毕竟那才是 ECMAScript 真正希望解决的问题

## iterator 迭代器

iterator 迭代器是 ES6 非常重要的概念，但是很多人对它了解的不多，但是它却是另外 4 个 ES6 常用特性的实现基础（解构赋值，剩余/扩展运算符，生成器，for of 循环），了解迭代器的概念有助于了解另外 4 个核心语法的原理，另外 ES6 新增的 Map,Set 数据结构也有使用到它，所以我放到前面来讲

对于可迭代的数据解构，ES6 在内部部署了一个\[Symbol.iterator\] 属性，它是一个函数，执行后会返回 iterator 对象（也叫迭代器对象），而生成 iterator 对象\[Symbol.iterator\] 属性叫 iterator 接口,有这个接口的数据结构即被视为可迭代的

数组中的 Symbol.iterator 方法 (iterator 接口) 默认部署在数组原型上:

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9eb2a2e09da~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

默认部署 iterator 接口的数据结构有以下几个，注意普通对象默认是没有 iterator 接口的（可以自己创建 iterator 接口让普通对象也可以迭代）

- Array
- Map
- Set
- String
- TypedArray（类数组）
- 函数的 arguments 对象
- NodeList 对象

iterator 迭代器是一个对象，它具有一个 next 方法所以可以这么调用

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9eb360753c9~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

next 方法返回又会返回一个对象，有 value 和 done 两个属性，value 即每次迭代之后返回的值，而 done 表示是否还需要再次循环，可以看到当 value 为 undefined 时，done 为 true 表示循环终止

梳理一下

- 可迭代的数据结构会有一个\[Symbol.iterator\] 方法
- \[Symbol.iterator\] 执行后返回一个 iterator 对象
- iterator 对象有一个 next 方法
- 执行一次 next 方法 (消耗一次迭代器) 会返回一个有 value,done 属性的对象

借用 [冴羽博客](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fmqyqingfeng%2FBlog%2Fissues%2F90 "https://github.com/mqyqingfeng/Blog/issues/90") 中 ES5 实现的迭代器可以更加深刻的理解迭代器是如何生成和消耗的

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/3/19/16994a020924cb2e~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

## 解构赋值（常用）

解构赋值可以直接使用对象的某个属性，而不需要通过属性访问的形式使用，对象解构原理个人认为是通过寻找相同的属性名，然后原对象的这个属性名的值赋值给新对象对应的属性

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9eb466d9679~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

这里左边真正声明的其实是 titleOne,titleTwo 这两个变量，然后会根据左边这 2 个变量的位置寻找右边对象中 title 和 test\[0\] 中的 title 对应的值，找到字符串 abc 和 test 赋值给 titleOne,titleTwo（如果没有找到会返回 undefined）

**数组解构的原理其实是消耗数组的迭代器，把生成对象的 value 属性的值赋值给对应的变量**

数组解构的一个用途是交换变量，避免以前要声明一个临时变量值存储值

ES6 交换变量:

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9eb57d68be3~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

### 建议

同样建议使用，因为解构赋值语意化更强，对于作为对象的函数参数来说，可以减少形参的声明，直接使用对象的属性（如果嵌套层数过多我个人认为不适合用对象解构，不太优雅）

一个常用的例子是 Vuex 中 actions 中的方法会传入 2 个参数，第一个参数是个对象，你可以随意命名，然后使用<名字>.commit 的方法调用 commit 函数，或者使用对象解构直接使用 commit

不使用对象解构:

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9eb4dca1cbf~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

使用对象解构:

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9eb58a3cd0d~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

另外可以给使用 axios 的响应结果进行解构 (axios 默认会把真正的响应结果放在 data 属性中)

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9eb5c2686d1~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

## 剩余/扩展运算符（常用）

剩余/扩展运算符同样也是 ES6 一个非常重要的语法，使用 3 个点（...），后面跟着一个含有 iterator 接口的数据结构

### 扩展运算符

以数组为例,使用扩展运算符使得可以 " 展开 " 这个数组，可以这么理解，数组是存放元素集合的一个容器，而使用扩展运算符可以将这个容器拆开，这样就只剩下元素集合，你可以把这些元素集合放到另外一个数组里面

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9eb69467378~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

扩展运算符可以代替 ES3 中数组原型的 concat 方法

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/14/168eb00e090d1386~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

这里将 arr1,arr2 通过扩展运算符展开,随后将这些元素放到一个新的数组中,相对于 concat 方法语义化更强

### 剩余运算符

剩余运算符最重要的一个特点就是替代了以前的 arguments

访问函数的 arguments 对象是一个很昂贵的操作，以前的 arguments.callee,arguments.caller 都被废止了，建议在支持 ES6 语法的环境下不要在使用 arguments 对象，使用剩余运算符替代（箭头函数没有 arguments，必须使用剩余运算符才能访问参数集合）

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9eb71537568~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

剩余运算符可以和数组的解构赋值一起使用，但是必须放在**最后一个**，因为剩余运算符的原理其实是利用了数组的迭代器，它会消耗 3 个点后面的数组的所有迭代器，读取所有迭代器生成对象的 value 属性，剩运算符后不能在有解构赋值，因为剩余运算符已经消耗了所有迭代器，而数组的解构赋值也是消耗迭代器，但是这个时候已经没有迭代器了，所以会报错

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9eb6122c311~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

这里 first 会消耗右边数组的一个迭代器，...arr 会消耗剩余所有的迭代器，而第二个例子...arr 直接消耗了所有迭代器，导致 last 没有迭代器可供消耗了，所以会报错，因为这是毫无意义的操作

**剩余运算符和扩展运算符的区别就是，剩余运算符会收集这些集合，放到右边的数组中，扩展运算符是将右边的数组拆分成元素的集合，它们是相反的**

### 在对象中使用扩展运算符

这个是 ES9 的语法，ES9 中支持在对象中使用扩展运算符，之前说过数组的扩展运算符原理是消耗所有迭代器，但对象中并没有迭代器，我个人认为可能是实现原理不同，但是仍可以理解为将键值对从对象中拆开，它可以放到另外一个普通对象中

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9eb76380489~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

其实它和另外一个 ES6 新增的 API 相似，即 Object.assign，它们都可以合并对象，但是还是有一些不同 Object.assign 会触发目标对象的 setter 函数，而对象扩展运算符不会，这个我们放到后面讨论

### 建议

使用扩展运算符可以快速的将类数组转为一个真正的数组

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9eb82d7b593~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

合并多个数组

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9eb854df5fb~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

函数柯里化

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9eb8982ea6e~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

## 对象属性/方法简写 (常用)

### 对象属性简写

es6 允许当对象的属性和值相同时，省略属性名

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9eb8cbe0f2d~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

需要注意的是

- **省略的是属性名而不是值**
- 值必须是一个变量

对象属性简写经常与解构赋值一起使用

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9eb9359bc8b~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

结合上文的解构赋值，这里的代码会其实是声明了 x,y,z 变量，因为 bar 函数会返回一个对象，这个对象有 x,y,z 这 3 个属性，解构赋值会寻找等号右边表达式的 x,y,z 属性，找到后赋值给声明的 x,y,z 变量

### 方法简写

es6 允许当一个对象的属性的值是一个函数（即是一个方法），可以使用简写的形式

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9eba5a9ac98~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

在 Vue 中因为都是在 vm 对象中书写方法，完全可以使用方法简写的方式书写函数

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9eba26780ea~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

## for ... of 循环

for ... of 是作为 ES6 新增的遍历方式,允许遍历一个含有 iterator 接口的数据结构并且返回各项的值,和 ES3 中的 for ... in 的区别如下

1. for ... of 只能用在可迭代对象上,获取的是迭代器返回的 value 值,for ... in 可以获取所有对象的键名
2. for ... in 会遍历对象的整个原型链,性能非常差不推荐使用,而 for ... of 只遍历当前对象不会遍历它的原型链
3. 对于数组的遍历,for ... in 会返回数组中所有可枚举的属性 (包括原型链上可枚举的属性),for ... of 只返回数组的下标对应的属性值

for ... of 循环的原理其实也是利用了可迭代对象内部部署的 iterator 接口,如果将 for ... of 循环分解成最原始的 for 循环,内部实现的机制可以这么理解

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9eba84cafbe~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

可以看到只要满足第二个条件 (iterator.next() 存在且 res.done 为 true) 就可以一直循环下去,并且每次把迭代器的 next 方法生成的对象赋值给 res,然后将 res 的 value 属性赋值给 for ... of 第一个条件中声明的变量即可,res 的 done 属性控制是否继续遍历下去

for... of 循环同时支持 break,continue,return(在函数中调用的话) 并且可以和对象解构赋值一起使用

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9ebb0fe5712~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

arr 数组每次使用 for ... of 循环都返回一对象 ({a:1},{a:2},{a:3}),然后会经过对象解构,寻找属性为 a 的值,赋值给 obj.a,所以在每轮循环的时候 obj.a 会分别赋值为 1,2,3

## Promise（常用）

Promise 作为 ES6 中推出的新的概念，改变了 JS 的异步编程，现代前端大部分的异步请求都是使用 Promise 实现，fetch 这个 web api 也是基于 Promise 的，这里不得简述一下之前统治 JS 异步编程的回调函数，回调函数有什么缺点，Promise 又是怎么改善这些缺点

### 回调函数

众所周知，JS 是单线程的，因为多个线程改变 DOM 的话会导致页面紊乱，所以设计为一个单线程的语言，但是浏览器是多线程的，这使得 JS 同时具有异步的操作，即定时器，请求，事件监听等，而这个时候就需要一套事件的处理机制去决定这些事件的顺序，即 Event Loop（事件循环），这里不会详细讲解事件循环，只需要知道，前端发出的请求，一般都是会进入浏览器的 http 请求线程，等到收到响应的时候会通过回调函数推入异步队列，等处理完主线程的任务会读取异步队列中任务，执行回调

在《你不知道的 JavaScript》下卷中，这么介绍

使用回调函数处理异步请求相当于把你的回调函数置于了一个黑盒，虽然你声明了等到收到响应后执行你提供的回调函数,可是你并不知道这个第三方库会在什么具体会怎么执行回调函数

使用第三方的请求库你可能会这么写:

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168e0109224ac6ef~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

收到响应后，执行后面的回调打印字符串，但是如果这个第三方库有类似超时重试的功能，可能会执行多次你的回调函数，如果是一个支付功能，你就会发现你扣的钱可能就不止 1000 元了 -.-

第二个众所周知的问题就是，在回调函数中再嵌套回调函数会导致代码非常难以维护，这是人们常说的“回调地狱”

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9ebc8e9311d~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

另外你使用的第三方 ajax 库还有可能并没有提供一些错误的回调，请求失败的一些错误信息可能会被吞掉，而你确完全不知情 (nodejs 提供了 err-first 风格的回调,即异步操作的第一个回调永远是错误的回调处理,但是你还是不能保证所有的库都提供了发送错误时的执行的回调函数)

总结一下回调函数的一些缺点

1. 多重嵌套，导致回调地狱
2. 代码跳跃，并非人类习惯的思维模式
3. 信任问题，你不能把你的回调完全寄托与第三方库，因为你不知道第三方库到底会怎么执行回调（多次执行）
4. 第三方库可能没有提供错误处理
5. 不清楚回调是否都是异步调用的（可以同步调用 ajax，在收到响应前会阻塞整个线程，会陷入假死状态，非常不推荐）

```
xhr.open("GET","/try/ajax/ajax_info.txt",false); //通过设置第三个async为false可以同步调用ajax
```

### Promise

针对回调函数这么多缺点，ES6 中引入了一个新的概念 Promise，Promise 是一个构造函数，通过 new 关键字创建一个 Promise 的实例，来看看 Promise 是怎么解决回调函数的这些问题

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9ebca9f5e7b~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

**Promise 并不是回调函数的衍生版本，而是 2 个概念，所以需要将之前的回调函数改为支持 Promise 的版本，这个过程成为 " 提升 "，或者 "promisory"，现代 MVVM 框架常用的第三方请求库 axios 就是一个典型的例子，另外 nodejs 中也有 bluebird，Q 等**

1. 多重嵌套，导致回调地狱

Promise 在设计的时候引入了链式调用的概念，每个 then 方法**同样也是一个 Promise**，因此可以无限链式调用下去

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9ebced32817~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

配合箭头函数，明显的比之前回调函数的多层嵌套优雅很多

1. 代码跳跃，并非人类习惯的思维模式

Promise 使得能够同步思维书写代码，上述的代码就是先请求 3000 端口，得到响应后再请求 3001，再请求 3002，再请求 3003，而书写的格式也是符合人类的思维，从先到后

1. 信任问题，你不能把你的回调完全寄托与第三方库，因为你不知道第三方库到底会怎么执行回调（多次执行）

Promise 本身是一个状态机，具有以下 3 个状态

- pending（等待）
- fulfilled（成功）
- rejected（拒绝）

当请求发送没有得到响应的时候为 pending 状态，得到响应后会 resolve(决议) 当前这个 Promise 实例,将它变为 fulfilled/rejected(大部分情况会变为 fulfilled)

当请求发生错误后会执行 reject(拒绝) 将这个 Promise 实例变为 rejected 状态

一个 Promise 实例的状态只能从 pending => fulfilled 或者从 pending => rejected，即当一个 Promise 实例从 pending 状态改变后，就不会再改变了（不存在 fulfilled => rejected 或 rejected => fulfilled）

而 Promise 实例必须主动调用 then 方法，才能将值从 Promise 实例中取出来（前提是 Promise 不是 pending 状态），这一个“主动”的操作就是解决这个问题的关键，即第三方库做的只是改变 Promise 的状态，而响应的值怎么处理，这是开发者主动控制的，这里就实现了控制反转，将原来第三方库的控制权转移到了开发者上

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9ebcee4bf6f~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

1. 第三方库可能没有提供错误处理

Promise 的 then 方法会接受 2 个函数，第一个函数是这个 Promise 实例被 resolve 时执行的回调，第二个函数是这个 Promise 实例被 reject 时执行的回调，而这个也是开发者主动调用的

使用 Promise 在异步请求发送错误的时候，即使没有捕获错误，也不会阻塞主线程的代码（准确的来说，异步的错误都不会阻塞主线程的代码）

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9ebd0318392~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

1. 不清楚回调是否都是异步调用的

Promise 在设计的时候保证所有响应的处理回调都是异步调用的，不会阻塞代码的执行，Promise 将 then 方法的回调放入一个叫微任务的队列中（MicroTask），确保这些回调任务在同步任务执行完以后再执行，这部分同样也是事件循环的知识点，有兴趣的朋友可以深入研究一下

对于第三个问题中,为什么说执行了 resolve 函数后 " 大部分情况 " 会进入 fulfilled 状态呢?考虑以下情况

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/14/168eb59272b3154f~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/14/168eb5ae056a4568~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

(这里用一个定时器在下轮事件循环中打印这个 Promise 实例的状态,否则会是 pending 状态)

很多人认为 promise 中调用了 resolve 函数则这个 promise 一定会进入 fulfilled 状态,但是这里可以看到,即使调用了 resolve 函数,仍返回了一个拒绝状态的 Promise,原因是因为如果在一个 promise 的 resolve 函数中又传入了一个 Promise,会展开传入的这个 promise

这里因为传入了一个拒绝状态的 promise,resolve 函数展开这个 promise 后,就会变成一个拒绝状态的 promise,所以把 resolve 理解为决议比较好一点

等同于这样

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/14/168eb60451743e8a~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

### 建议

在日常开发中，建议全面拥抱新的 Promise 语法，其实现在的异步编程基本也都使用的是 Promise

建议使用 ES7 的 async/await 进一步的优化 Promise 的写法，async 函数始终返回一个 Promise，await 可以实现一个 " 等待 " 的功能，async/await 被成为异步编程的终极解决方案，即用同步的形式书写异步代码，并且能够更优雅的实现异步代码顺序执行以及在发生异步的错误时提供更精准的错误信息,详细用法可以看阮老师的 ES6 标准入门

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/21/1690e7a5ff97364e~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

关于 Promise 还有很多很多需要讲的，包括它的静态方法 all，race，resolve，reject，Promise 的执行顺序，Promise 嵌套 Promise，thenable 对象的处理等，碍于篇幅这里只介绍了一下为什么需要使用 Promise。但很多开发者在日常使用中只是了解这些 API，却不知道 Promise 内部具体是怎么实现的，遇到复杂的异步代码就无从下手，非常建议去了解一下 Promise A+ 的规范，自己实现一个 Promise

## ES6 Module(常用)

在 ES6 Module 出现之前，模块化一直是前端开发者讨论的重点，面对日益增长的需求和代码，需要一种方案来将臃肿的代码拆分成一个个小模块，从而推出了 AMD,CMD 和 CommonJs 这 3 种模块化方案，前者用在浏览器端，后面 2 种用在服务端，直到 ES6 Module 出现

**ES6 Module 默认目前还没有被浏览器支持，需要使用 babel，在日常写 demo 的时候经常会显示这个错误**

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9ebeb392397~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

可以在 script 标签中使用 tpye="module" 在**同域**的情况下可以解决（非同域情况会被同源策略拦截，webstorm 会开启一个同域的服务器没有这个问题，vscode 貌似不行）

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9ebedf62b58~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

ES6 Module 使用 import 关键字导入模块，export 关键字导出模块，它还有以下特点

1. ES6 Module 是静态的，也就是说它是在编译阶段运行，和 var 以及 function 一样具有提升效果（这个特点使得它支持 tree shaking）
2. 自动采用严格模式（顶层的 this 返回 undefined）
3. ES6 Module 支持使用 export {<变量>}导出具名的接口，或者 export default 导出匿名的接口

module.js 导出:

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9ebef0bed93~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

a.js 导入:

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9ebf3ade585~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

这两者的区别是，export {<变量>}导出的是一个变量的引用，export default 导出的是一个值

什么意思呢，就是说在 a.js 中使用 import 导入这 2 个变量的后，在 module.js 中因为某些原因 x 变量被改变了，那么会立刻反映到 a.js，而 module.js 中的 y 变量改变后，a.js 中的 y 还是原来的值

module.js:

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9ec0ee34ff0~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

a.js:

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9ec1621b560~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

可以看到给 module.js 设置了一个一秒后改变 x,y 变量的定时器,在一秒后同时观察导入时候变量的值,可以发现 x 被改变了,但 y 的值仍是 20,因为 y 是通过 export default 导出的,在导入的时候的值相当于只是导入数字 20,而 x 是通过 export {<变量>}导出的,它导出的是一个变量的引用,即 a.js 导入的是当前 x 的值,只关心**当前**x 变量的值是什么,可以理解为一个 " 活链接 "

export default 这种导出的语法其实只是指定了一个命名导出,而它的名字叫 default,换句话说,将模块的导出的名字重命名为 default,也可以使用 import <变量> from <路径> 这种语法导入

module.js 导出:

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9ec0954d5de~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

a.js 导入:

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9ebf3ade585~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

**但是由于是使用 export {<变量>}这种形式导出的模块,即使被重命名为 default,仍然导出的是一个变量的引用**

这里再来说一下目前为止主流的模块化方案 ES6 Module 和 CommonJs 的一些区别

1. CommonJs 输出的是一个值的拷贝,ES6 Module 通过 export {<变量>}输出的是一个变量的引用,export default 输出的是一个值
2. CommonJs 运行在服务器上,被设计为运行时加载,即代码执行到那一行才回去加载模块,而 ES6 Module 是静态的输出一个接口,发生在编译的阶段
3. CommonJs 在第一次加载的时候运行一次并且会生成一个缓存,之后加载返回的都是缓存中的内容

### import()

关于 ES6 Module 静态编译的特点,导致了无法动态加载,但是总是会有一些需要动态加载模块的需求,所以现在有一个提案,使用把 import 作为一个函数可以实现动态加载模块,它返回一个 Promise,Promise 被 resolve 时的值为输出的模块

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9ec16da2511~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9ec186c4715~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

使用 import 方法改写上面的 a.js 使得它可以动态加载 (使用静态编译的 ES6 Module 放在条件语句会报错,因为会有提升的效果,并且也是不允许的),可以看到输出了 module.js 的一个变量 x 和一个默认输出

Vue 中路由的懒加载的 ES6 写法就是使用了这个技术,使得在路由切换的时候能够动态的加载组件渲染视图

## 函数默认值

ES6 允许在函数的参数中设置默认值

ES5 写法:

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9ec37511eed~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

ES6 写法:

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9ec2bcb6af0~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

相比 ES5,ES6 函数默认值直接写在参数上,更加的直观

如果使用了函数默认参数,在函数的参数的区域 (括号里面),它会作为一个单独的**块级作用域**,并且拥有 let/const 方法的一些特性,比如暂时性死区

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9ec1f367047~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

这里当运行 func 的时候,因为没有传参数,使用函数默认参数,y 就会去寻找 x 的值,在沿着词法作用域在外层找到了值为 1 的变量 x

再来看一个例子

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9ec35ac50b5~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

这里同样没有传参数,使用函数的默认赋值,x 通过词法作用域找到了变量 w,所以 x 默认值为 2,y 同样通过词法作用域找到了刚刚定义的 x 变量,y 的默认值为 3,但是在解析到 z = z + 1 这一行的时候,JS 解释器先会去解析 z+1 找到相应的值后再赋给变量 z,但是因为暂时性死区的原因 (let/const" 劫持 " 了这个块级作用域,无法在声明之前使用这个变量,上文有解释),导致在 let 声明之前就使用了变量 z,所以会报错

这样理解函数的默认值会相对容易一些

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/13/168e47dff16b7e77~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

当传入的参数为 undefined 时才使用函数的默认值 (显式传入 undefined 也会触发使用函数默认值,传入 null 则不会触发)

在举个例子:

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9ec395bebe1~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

这里借用阮一峰老师书中的一个例子,func 的默认值为一个函数,执行后返回 foo 变量,而在函数内部执行的时候,相当于对 foo 变量的一次变量查询 (LHS 查询),而查询的起点是这个单独的块级作用域,即 JS 解释器不会去查询去函数内部查询变量 foo,而是沿着词法作用域先查看同一作用域 (前面的函数参数) 中有没有 foo 变量,再往函数的外部寻找 foo 变量,最终找不到所以报错了,这个也是函数默认值的一个特点

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/3/18/1698f6f2b3f8adb6~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

通过 debugger 可以更加直观的发现在这个函数内部可以通过词法作用域访问 func 函数,foo 变量,还有 this,但是当查看 func 函数的词法作用域时,发现它只能访问到 Global,即全局作用域,foo 变量并不存在于它的词法作用域中

### 函数默认值配合解构赋值

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9ec50606fd1~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

第一行给 func 函数传入了 2 个空对象,所以函数的第一第二个参数都不会使用函数默认值,然后函数的第一个参数会尝试解构对象,提取变量 x,因为第一个参数传入了一个空对象,所以解构不出变量 x,但是这里又在内层设置了一个默认值,所以 x 的值为 10,而第二个参数同样传了一个空对象,不会使用函数默认值,然后会尝试解构出变量 y,发现空对象中也没有变量 y,但是 y 没有设置默认值所以解构后 y 的值为 undefined

第二行第一个参数显式的传入了一个 undefined,所以会使用函数默认值为一个空对象,随后和第一行一样尝试解构 x 发现 x 为 undefined,但是设置了默认值所以 x 的值为 10,而 y 和上文一样为 undefined

第三行 2 个参数都会 undefined,第一个参数和上文一样,第二个参数会调用函数默认值,赋值为{y:10},然后尝试解构出变量 y,即 y 为 10

第四行和第三行相同,一个是显式传入 undefined,一个是隐式不传参数

第五行直接使用传入的参数,不会使用函数默认值,并且能够顺利的解构出变量 x,y

## Proxy

Proxy 作为一个 " 拦截器 ",可以在目标对象前架设一个拦截器,他人访问对象,必须先经过这层拦截器,Proxy 同样是一个构造函数,使用 new 关键字生成一个拦截对象的实例,ES6 提供了非常多对象拦截的操作,几乎覆盖了所有可能修改目标对象的情况 (Proxy 一般和 Reflect 配套使用,前者拦截对象,后者返回拦截的结果,Proxy 上有的的拦截方法 Reflect 都有)

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9ec5addffad~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

### Object.definePropery

提到 Proxy 就不得不提一下 ES5 中的 Object.defineProperty,这个 api 可以给一个对象添加属性以及这个属性的属性描述符/访问器 (这 2 个不能共存,同一属性只能有其中一个),属性描述符有 configurable,writable,enumerable,value 这 4 个属性,分别代表是否可配置,是否只读,是否可枚举和属性的值,访问器有 configurable,enumerable,get,set,前 2 个和属性描述符功能相同,后 2 个都是函数,定义了 get,set 后对元素的读写操作都会执行后面的 getter/setter 函数,并且覆盖默认的读写行为

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9ec56c9a8f0~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

定义了 obj 中 a 属性的表示为只读,且不可枚举,obj2 定义了 get,但没有定义 set 表示只读,并且读取 obj2 的 b 属性返回的值是 getter 函数的返回值

ES5 中的 Object.defineProperty 这和 Proxy 有什么关系呢?个人理解 Proxy 是 Object.defineProperty 的增强版,ES5 只规定能够定义属性的属性描述符或访问器.而 Proxy 增强到了 13 种,具体太多了我就不一一放出来了,这里我举几个比较有意思的例子

### handler.apply

apply 可以让我们拦截一个函数 (JS 中函数也是对象,Proxy 也可以拦截函数) 的执行,我们可以把它用在函数节流中

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9ec6239f7d9~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

调用拦截后的函数:

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/13/168e52a570e4d7d1~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

### handler.contruct

contruct 可以拦截通过 new 关键字调用这个函数的操作,我们可以把它用在单例模式中

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9ec6bf3b894~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

这里通过一个闭包保存了 instance 变量,每次使用 new 关键字调用被拦截的函数后都会查看这个 instance 变量,如果存在就返回闭包中保存的 instance 变量,否则就新建一个实例,这样可以实现全局只有一个实例

### handler.defineProperty

defineProperty 可以拦截对这个对象的 Object.defineProerty 操作

**注意对象内部的默认的\[\[SET\]\] 操作 (即对这个对象的属性赋值) 会间接触发 defineProperty 和 getOwnPropertyDescriptor 这 2 个拦截方法**

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9ec717f7a0c~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

这里有几个知识点

1. 这里使用了递归的操作,当需要访问对象的属性时候,会判断代理的对象属性的值仍是一个可以代理的对象就递归的进行代理,否则通过错误捕获执行默认的 get 操作
2. 定义了 defineProperty 的拦截方法,当对这个代理对象的某个属性进行赋值的时候会执行对象内部默认的\[\[SET\]\] 操作进行赋值,这个操作会间接触发 defineProperty 这个方法,随后会执行定义的 callback 函数

这样就实现了无论对象嵌套多少层,只要有属性进行赋值就会触发 get 方法,对这层对象进行代理,随后触发 defineProperty 执行 callback 回调函数

### 其他的使用场景

Proxy 另外还有很多功能,比如在实现验证器的时候,可以将业务逻辑和验证器分离达到解耦,通过 get 拦截对私有变量的访问实现私有变量,拦截对象做日志记录，实现微信 api 的 promise 化等

### Vue

尤大预计 2019 年下半年发布 Vue3.0,其中一个核心的功能就是使用 Proxy 替代 Object.defineProperty

我相信了解过一点 Vue 响应式原理的人都知道 Vue 框架在对象拦截上的一些不足

```
<template>
   <div>
       <div>{{arr}}</div>
       <div>{{obj}}</div>
       <button @click="handleClick">修改arr下标</button>
       <button @click="handleClick2">创建obj的属性</button>
   </div>
</template>

<script>

    export default {
        name: "index",
        data() {
            return {
                arr:[1,2,3],
                obj:{
                    a:1,
                    b:2
                }
            }
        },
        methods: {
            handleClick() {
                this.arr[0] = 10
                console.log(this.arr)
            },
            handleClick2() {
                this.obj.c = 3
                console.log(this.obj)
            }
        },
   }
</script>
```

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9ec77c2b72f~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

可以看到这里数据改变了,控制台打印出了新的值,但是视图没有更新,这是因为 Vue 内部使用 Object.defineProperty 进行的数据劫持,而这个 API 无法探测到**对象根属性的添加和删除,以及直接给数组下标进行赋值**,所以不会通知渲染 watcher 进行视图更新,而理论上这个 API 也无法探测到数组的一系列方法 (push,splice,pop),但是 Vue 框架修改了数组的原型,使得在调用这些方法修改数据后会执行视图更新的操作

```
//源码位置:src/core/observer/array.js
methodsToPatch.forEach(function (method) {
  // cache original method
  var original = arrayProto[method];
  def(arrayMethods, method, function mutator () {
    var args = [], len = arguments.length;
    while ( len-- ) args[ len ] = arguments[ len ];

    var result = original.apply(this, args);
    var ob = this.__ob__;
    var inserted;
    switch (method) {
      case 'push':
      case 'unshift':
        inserted = args;
        break
      case 'splice':
        inserted = args.slice(2);
        break
    }
    if (inserted) { ob.observeArray(inserted); }
    // notify change
    ob.dep.notify(); //这一行就会主动调用notify方法,会通知到渲染watcher进行视图更新
    return result
  });
});
```

在 [掘金翻译的尤大Vue3.0计划](https://juejin.cn/post/6844903687207272462 "https://juejin.cn/post/6844903687207272462") 中写到

> 3.0 将带来一个基于 Proxy 的 observer 实现，它可以提供覆盖语言 (JavaScript——译注) 全范围的响应式能力，消除了当前 Vue 2 系列中基于 Object.defineProperty 所存在的一些局限，如： 对属性的添加、删除动作的监测 对数组基于下标的修改、对于 .length 修改的监测 对 Map、Set、WeakMap 和 WeakSet 的支持

Proxy 就没有这个问题,并且还提供了更多的拦截方法,完全可以替代 Object.defineProperty,唯一不足的也就是浏览器的支持程度了 (IE: 谁在说我?)

所以要想深入了解 Vue3.0 实现机制,学会 Proxy 是必不可少的

## Object.assign

这个 ES6 新增的 Object 静态方法允许我们进行多个对象的合并

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9ec7eb6298e~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

可以这么理解,Object.assign 遍历需要合并给 target 的对象 (即 sourece 对象的集合) 的属性,用**等号**进行赋值,这里遍历{a:1}将属性 a 和值数字 1 赋值给 target 对象,然后再遍历{b:2}将属性 b 和值数字 2 赋值给 target 对象

这里罗列了一些这个 API 的需要注意的知识点

1. Object.assign 是浅拷贝,对于值是引用类型的属性,拷贝仍旧的是它的引用
2. 可以拷贝 Symbol 属性
3. 不能拷贝不可枚举的属性
4. Object.assign 保证 target 始终是一个对象,如果传入一个基本类型,会转为基本包装类型,null/undefined 没有基本包装类型,所以传入会报错
5. source 参数如果是不可枚举的数据类型会忽略合并 (字符串类型被认为是可枚举的,因为内部有 iterator 接口)
6. 因为是用**等号**进行赋值,如果被赋值的对象的属性有 setter 函数会触发 setter 函数,同理如果有 getter 函数,也会调用赋值对象的属性的 getter 函数 (这就是为什么 Object.assign 无法合并对象属性的访问器,因为它会直接执行对应的 getter/setter 函数而不是合并它们,如果需要合并对象属性的 getter/setter 函数,可以使用 ES7 提供的 Object.getOwnPropertyDescriptors 和 Object.defineProperties 这 2 个 API 实现)

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/13/168e64e6f6872a09~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/13/168e64bd2eb1e640~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

可以看到这里成功的复制了 obj 对象中 a 属性的 getter/setter

为了加深了解我自己模拟了 Object.assign 的实现,可供参考

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9ec8acf44b3~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

这里有一个坑不得不提,对于 target 参数传入一个字符串,内部会转换为基本包装类型,而字符串基本包装类型的属性是只读的 (属性描述符的 writable 属性为 false),这里感谢 [木易杨的专栏](https://juejin.cn/post/6844903753490006029 "https://juejin.cn/post/6844903753490006029")

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/13/168e66d1185ec325~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/13/168e66541e1da350~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

打印对象属性的属性描述符可以看到下标属性的值都是只读的,即不能再次赋值,所以尝试以下操作会报错

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/13/168e65efa1ab106f~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

字符串 abc 会转为基本包装类型,然后将字符串 def 合并给这个基本包装类型的时候会将字符串 def 展开,分别将字符串 def 赋值给基本包装类型 abc 的 0,1,2 属性,随后就会在赋值的时候报错 (非严格模式下会只会静默处理,ES6 的 Object.assign 默认开启了严格模式)

### 和 ES9 的对象扩展运算符对比

ES9 支持在对象上使用扩展运算符,实现的功能和 Object.assign 相似,唯一的区别就是在含有 getter/setter 函数的对象的属性上有所区别

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/14/168e9b0427f2d2ec~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/14/168e9b09b5b84f7d~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

(最后一个字符串 get 可以忽略,这是控制台为了显示 a 变量触发的 getter 函数)

分析一下这个例子

ES9:

- 会合并 2 个对象,并且只触发 2 个对象对应属性的 getter 函数
- 相同属性的后者覆盖了前者,所以 a 属性的值是第二个 getter 函数 return 的值

ES6:

- 同样会合并这 2 个对象,并且只触发了 obj 上 a 属性的 setter 函数而不会触发它的 getter 函数 (结合上述 Object.assgin 的内部实现理解会容易一些)
- obj 上 a 属性的 setter 函数替代默认的赋值行为,导致 obj2 的 a 属性不会被复制过来

除去对象属性有 getter/setter 的情况,Object.assgin 和对象扩展运算符功能是相同的,两者都可以使用,两者都是浅拷贝,使用 ES9 的方法相对简洁一点

### 建议

1. Vue 中重置 data 中的数据

这个是我最常用的小技巧,使用 Object.assign 可以将你目前组件中的 data 对象和组件默认初始化状态的 data 对象中的数据合并,这样可以达到初始化 data 对象的效果

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9ec96c22302~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

在当前组件的实例中 $data属性保存了当前组件的data对象,而$options 是当前组件实例初始化时的一些属性,其中有个 data 方法,即在在组件中写的 data 函数,执行后会返回一个初始化的 data 对象,然后将这个初始化的 data 对象合并到当前的 data 来初始化所有数据

1. 给对象合并需要的默认属性

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9eca34511aa~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

可以封装一个函数,外层声明一个 DEFAULTS 常量,options 为每次传入的动态配置,这样每次执行后会合并一些默认的配置项

1. 在传参的时候可以多个数据合并成一个对象传给后端

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/12/168df9ec9ea1c032~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.jpg)

## 参考资料

1. [阮一峰：ES6标准入门](https://link.juejin.cn/?target=http%3A%2F%2Fes6.ruanyifeng.com "http://es6.ruanyifeng.com")
2. [慕课网：ES6零基础教学](https://link.juejin.cn/?target=https%3A%2F%2Fcoding.imooc.com%2Fclass%2F98.html "https://coding.imooc.com/class/98.html")
3. 你不知道的 JavaScript 下卷
