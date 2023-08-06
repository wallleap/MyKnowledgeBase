---
title: JavaScript 骚操作之操作符
date: 2023-08-04 17:29
updated: 2023-08-04 17:29
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - JavaScript 骚操作之操作符
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

## 三目运算符 ?

每到周末我都会问自己，怎么安排？于是我写了个程序跑了一下

```
    if (hasMoney) {
        console.log('周末嗨翻天');
    } else {
        console.log('周末睡一天');
    }
```

看上去没有什么问题，但总感觉代码有点臃肿，能不能稍微简约一点？

换成三目运算符后

```
    hasMoney ? console.log('周末嗨翻天') : console.log('周末睡一天');
```

look，当 hasMoney 为 true 的时候，嗨翻天，为 false 的时候的睡一天。这种操作符在我们的根据简单的判断条件赋值的时候非常有用

```
    let weekendPlan = hasMoney ? '周末嗨翻天' : '周末睡一天';
```

如此，我们的代码就不会充斥着大量的 if 判断条件句，还能略显骚气。但是，别骚过度了

```
    val = simpleCondition
        ? 1
        : simpleConditionAgain
        ? 2
        : simpleConditionAgainAndAgain
        ? 3
        : 4;
```

上面的例子有点为了装逼而装逼的感觉，在代码可读性方面有点糟糕，即使换成了 if 条件句也不甚美观；像这种情况，可以使用 switch 语句或者策略模式的设计思想优化。

## 逻辑与操作符 &&

如果有钱的话，周末就去嗨翻天，但没钱的情况还没有想好要干嘛（虽然没钱的情况干不了什么），常规操作为

```
    if (hasMoney) {
        console.log('周末嗨翻天');
    }
```

这个时候使用三目运算符改写的话

```
    hasMoney ? console.log('周末嗨翻天') : undefined;
```

有点奇怪，我们不得不写个 undefined 去告诉程序判断条件为 false 的时候应该怎么做。使用逻辑与操作符就可以避免这个问题

```
    hasMoney && console.log('周末嗨翻天');
```

逻辑与操作符会使用 Boolean 函数判断每一个条件 (表达式) 是否为 true，当所有的判断条件都为 true 的情况下，就会返回最后的表达式的运算结果；但是如果有判断条件为 false 的话，就会返回第一个判断条件为 false 的运算结果

```
    true && console.log('It is true');                         // It is true
    true && false && console.log('It is true');                // 返回 false
    true && 0 && console.log('It is true');                    // 返回 0
    true && undefined == null && console.log('It is true');    // It is true, 表达式undefined == null的运算结果为true
    true && undefined === null && console.log('It is true');   // 返回 false, 表达式undefined === null的运算结果为false
```

所以在 react 中使用逻辑与操作符渲染元素的时候，一定要注意

```
    render() {
        return (
            <Fragment>{
                data.length && data.map(item => <span>{ item }</span>)
            }</Fragment>
        );
    }

    // 当没有数据的时候，页面会渲染出一个0
```

上面的例子的本意是当有数据的时候就将数据渲染出来，没有数据的时候，不做任何操作。但是逻辑与操作符会将第一个为 false 的表达式的运算结果返回，导致返回了个 0！

## 逻辑或操作符 ||

说到逻辑与操作符，就不得不提它的好基友：逻辑或操作符。考虑下面的场景

```
    // 当自身为undefined时，赋值为0，否则还是赋值为自身
    val = val !== undefined ? val : 0;
```

使用三目运算符去处理上述例子的逻辑时，我们需要显式的判断 val 是否为空的，然后再决定变量 val 是否应该等于本身。那么本着能省就省的原则，我们可以使用逻辑或操作符

```
    val = val || 0;
```

逻辑或操作符，其会将第一个表达式的运算结果传入 Boolean 函数，如果 Boolean 函数返回 true，就会返回这个运算结果；否则将会尝试下一个，直到结束；如果所有的表达式的运算结果对应的布尔值都为 false，则返回最后一个表达式的运算结果

故此操作符在向下兼容、设置函数参数的默认值时非常有用

```
    // ES5设置函数默认值
    function testFunc(arg1) {
        arg1 = arg1 || 1;

        // do something else
    }

    let a = 1,
        b = 2,
        c = null;
    
    console.log(a || b || c);         // 1
    console.log(0 || b || c);         // 2
    console.log(0 || false || c);     // null
```

> note: 故此猜想，if 条件句里面使用逻辑或、逻辑与操作符，实际上也只是返回表达式的运算结果，然后再隐式调用了 Boolean 函数得到一个最终的布尔值

## 逻辑取反

上文说到，有钱的周末可以为所欲为，没钱的周末，估计也就只能选择睡一天了。这种情况下，如果使用 hasMoney 作为判断标准，我们的代码是这样的

```
    hasMoney === false && console.log('周末睡一天');
```

当 hasMoney 不是一个布尔值的时候，hasMoney === false 语句就会一直返回 false，造成有钱的假象。所以我们必须的将 hasMoney 转换成一个布尔值才能判断，恰好取反操作符可以同时给我们做这两件事

```
    !hasMoney && console.log('周末睡一天');
```

这样就能在没钱的时候睡一天了。

取反操作符能够将一个非 Boolean 类型的值转化为 Boolean 类型的值且取反

```
    !true && console.log('666');             // 返回false
    !!true && console.log('666');            // 666
    {} && console.log('666');                // 报错
    !{} && console.log('666');               // 返回false
    !!{} && console.log('666');              // 666
```

## 按位取反操作符 ~

加入我们需要判断一个数组里面是否存在某个元素，在 ES6 里面可以使用 includes

```
    let arr = ['we', 'are', 'the', 'BlackGold', 'team'];

    arr.includes('the') && console.log('in');      // in
```

但是这个方法有比较大的限制：没办法传入一个筛选函数。

```
    let arr = ['we', 'are', 'the', 'BlackGold', 'team'];

    arr.includes(element => element === 'the') && console.log('in');     // 返回false
```

这种情况下，我们通常可以使用 findIndex 方法

```
    let arr = ['we', 'are', 'the', 'BlackGold', 'team'];

    arr.findIndex(element => element === 'the') !== -1 && console.log('in');      // in
```

由于 findIndex 方法返回的索引是从 0 开始的，所以我们必须得判断其返回的索引是否不等于 -1 或者是大于等于 0。如果使用按位取反操作符，将不需要显示去判断

```
    let arr = ['we', 'are', 'the', 'BlackGold', 'team'];

    ~arr.findIndex(element => element === 'we') && console.log('in');      // in
    ~arr.findIndex(element => element === 'the') && console.log('in');      // in
```

按位取反操作符，顾名思义，就是将变量的每一个比特位取反：0->1、1->0

```
    console.log(~-1);       // 0 转换为Boolean值即为false
    console.log(~0);        // -1 转换为Boolean值即为true
    console.log(~1);        // -2 转换为Boolean值即为true
```

## 结语

JavaScript 骚操作系列主要总结 JavaScript 一些好用的特性、方法，以供交流学习之用，不喜勿喷。如有错漏，欢迎指正。

相关文章：

[JavaScript骚操作之遍历、枚举与迭代（上篇）](https://juejin.cn/post/6844903724897271816 "https://juejin.cn/post/6844903724897271816")

[JavaScript骚操作之遍历、枚举与迭代（下篇）](https://juejin.cn/post/6844903731809501197 "https://juejin.cn/post/6844903731809501197")

> @Author: PaperCrane
