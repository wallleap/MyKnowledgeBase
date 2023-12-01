---
title: 被誉为 JavaScript 中最难理解的概念之一的闭包是什么鬼？
date: 2023-08-06 22:58
updated: 2023-08-06 23:00
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - 被誉为 JavaScript 中最难理解的概念之一的闭包是什么鬼？
rating: 1
tags:
  - JS
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

## 一、闭包的概念

> **当通过调用外部函数返回的内部函数后，即使外部函数已经执行结束了，但是被内部函数引用的外部函数的变量依然会保存在内存中，我们把引用了其他函数作用域变量的函数和这些被引用变量的集合，称为闭包（Closure），闭包是这些东西共同的组合**
> 在了解闭包的概念和用途之前，理解作用域和变量的生命周期等基础预备知识，对于理解闭包非常有帮助。

## 二、怎么实现闭包

闭包是指一个函数可以访问它定义时所在的词法作用域以及全局作用域中的变量。在 JavaScript 中，闭包可以通过函数嵌套和变量引用实现。

```js
function outerFunction() {
  let outerVariable = '我在outer函数里!';

  function innerFunction() {
    console.log(outerVariable);
  }

  return innerFunction;
}

const innerFunc = outerFunction();
innerFunc(); // 输出: 我在outer函数里!
```

在上面的代码示例中，`innerFunction` 引用了 `outerVariable`，因此 JavaScript 引擎会保留 `outerFunction` 的作用域链，以便 `innerFunction` 可以访问 `outerVariable`。

```js
function a(){
    function b(){
        var bb = 888
        console.log(aa);  //输出：666
    }
    var aa = 666
    return b
}
var demo = a()
demo()
```

在上面的代码示例中，`a` 函数定义了一个名为 `aa` 的变量和一个名为 `b` 的函数，`b` 函数引用了 `aa` 变量，因此 JavaScript 引擎会保留 `a` 函数的作用域链，`b` 函数可以访问 `a` 函数的执行上下文，`b` 函数内用到了外部函数 `a` 的变量 `aa`，在 `a` 函数调用结束后该函数执行上下文会销毁，但会保留一部分留在内存中供 `b` 函数使用，这就形成了闭包。

具体来说，当内部函数引用外部函数的变量时，外部函数的作用域链将被保留在内存中，以便内部函数可以访问这些变量。 这种函数嵌套和变量共享的方式就是闭包的核心概念。当一个函数返回另一个函数时，它实际上返回了一个闭包，其中包含了原函数定义时的词法作用域和相关变量。

## 三、闭包的用途

### 1.封装私有变量

闭包可以用于封装私有变量，以防止其被外部访问和修改。封装私有变量可以一定程度上防止全局变量污染，使用闭包封装私有变量可以将这些变量限制在函数内部或模块内部，从而减少了全局变量的数量，降低了全局变量被误用或意外修改的风险。

在下面这个例子中，调用函数，输出的结果都是 1，但是显然我们的代码效果是想让 `count` 每次加一的。

```js
function add() {
    let count = 0;
    count++;
    console.log(count);
}
add()   //输出1
add()   //输出1
add()   //输出1
```

一种显而易见的方法是将 `count` 提到函数体外，作为全局变量。这么做当然是可以解决问题，但是在实际开发中，一个项目由多人共同开发，你不清楚别人定义的变量名称是什么，这么做有点冒险，有什么其他的办法可以解决这个问题呢？

```js
function add(){
    let count = 0
    function a(){
        count++
        console.log(count);
    }
    return a
}
var res = add() 
res() //1 
res() //2
res() //3
```

答案是用闭包。在上面的代码示例中，`add` 函数返回了一个闭包 a，其中包含了 `count` 变量。由于 `count` 只在 `add` 函数内部定义，因此外部无法直接访问它。但是，由于 `a` 函数引用了 `count` 变量，因此 `count` 变量的值可以在闭包内部被修改和访问。这种方式可以用于封装一些私有的数据和逻辑。

### 2\. 做缓存

函数一旦被执行完毕，其内存就会被销毁，而闭包的存在，就可以保有内部环境的作用域。

```js
function foo(){
    var myName ='张三'
    let test1 = 1
    const test2 = 2 
    var innerBar={
        getName: function(){
            console.log(test1);
            return myName
        },
        setName:function(newName){
            myName = newName
        }
    }
    return innerBar
}
var bar = foo()   
console.log(bar.getName()); //输出：1 张三
bar.setName('李四')
console.log(bar.getName()); //输出：1 李四
```

这里 var bar = foo() 执行完后本来应该被销毁，但是因为形成了闭包，所以导致 foo 执行上下文没有被销毁干净，被引用了的变量 myName、test1 没被销毁，闭包里存放的就是变量 myName、test1，这个闭包就像是 setName、getName 的专属背包，setName、getName 依然可以使用 foo 执行上下文中的 test1 和 myName。

### 3\. 模块化编程（实现共有变量）

闭包还可以用于实现模块化编程。模块化编程是一种将程序拆分成小的、独立的、可重用的模块的编程风格。闭包可以用于封装模块的私有变量和方法，以便防止其被外部访问和修改。例如：

```js
const myModule = (function() {
  let privateVariable = '我是私有的!';

  function privateMethod() {
    console.log(privateVariable);
  }

  return {
    publicMethod: function() {
      privateMethod();
    }
  };
})();

myModule.publicMethod(); // 输出: 我是私有的!
```

在上面的代码示例中，myModule 实际上是一个立即执行的匿名函数，它返回了一个包含 publicMethod 的对象。在函数内部，定义了一个私有变量 privateVariable 和一个私有方法 privateMethod。publicMethod 是一个公共方法，它可以访问 privateMethod，但是无法访问 privateVariable。这种方式可以用于实现简单的模块化编程。

## 四、闭包的缺点

闭包也存在着一个潜在的问题，由于闭包会引用外部函数的变量，但是这些变量在外部函数执行完毕后没有被释放，那么这些变量会一直存在于内存中，总的内存大小不变，但是可用内存空间变小了。 一旦形成闭包，只有在页面关闭后，闭包占用的内存才会被回收，这就造成了所谓的**内存泄漏**。

因此我们在使用闭包时需要特别注意内存泄漏的问题，可以用以下两种方法解决内存泄露问题：

> 1.**及时释放闭包**：手动调用闭包函数，并将其返回值赋值为 null，这样可以让闭包中的变量及时被垃圾回收器回收。
> 2.**使用立即执行函数**：在创建闭包时，将需要保留的变量传递给一个立即执行函数，并将这些变量作为参数传递给闭包函数，这样可以保留所需的变量，而不会导致其他变量的内存泄漏。

## 五、最后的话

总之，闭包是一种非常重要的编程技术，可以让程序员更加灵活地处理数据和逻辑。在实际的开发过程中，合理地使用闭包可以帮助我们更加高效地编写代码，提高程序的性能和可维护性。感谢你阅读这篇文章，如果你觉得写得还行的话，不要忘记点赞、评论、收藏哦！
