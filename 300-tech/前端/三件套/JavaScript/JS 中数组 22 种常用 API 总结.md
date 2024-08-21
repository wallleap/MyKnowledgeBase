---
title: JS 中数组 22 种常用 API 总结
date: 2023-08-06T11:02:00+08:00
updated: 2024-08-21T10:33:28+08:00
dg-publish: false
aliases:
  - JS 中数组 22 种常用 API 总结
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
  - JS
  - web
url: //myblog.wallleap.cn/post/1
---

## 一、引言

> **在前端开发中，数组是一种常见且重要的数据结构。数组提供了许多便捷的方法来操作和处理其中的数据。本文将简单介绍前端中数组常用的 API，包括添加、删除、截取、合并、转换等操作。**

关于 `字符串` 的 API，大家可以去看这篇文章。[JS中字符串28种常用API总结](https://juejin.cn/post/7240697872160440380 "https://juejin.cn/post/7240697872160440380")。

## 二、push() 方法和 pop() 方法

push() 方法用于向数组末尾添加一个或多个元素，并返回修改后的数组的 `新长度`。

```js
const fruits = ['苹果', '香蕉'];
const res = fruits.push('橘子', '西瓜');
console.log(fruits);  //[ '苹果', '香蕉', '橘子', '西瓜' ]
console.log(res);     //4
```

pop() 方法用于删除并返回数组的最后一个元素。

```js
const fruits = ['苹果', '香蕉', '橘子'];
const lastFruit = fruits.pop();
console.log(fruits);     // ['苹果', '香蕉']
console.log(lastFruit); //橘子
```

## 三、shift() 方法和 unshift() 方法

shift() 方法用于删除并返回数组的 `第一个` 元素。

```js
const fruits = ['苹果', '香蕉', '橘子'];
const firstFruit = fruits.shift();
console.log(fruits);      //[ '香蕉', '橘子' ]
console.log(firstFruit);  //苹果
```

unshift() 方法用于向数组的 `开头添加` 一个或多个元素，并返回修改后的数组的新长度。

```js
const fruits = ['苹果', '香蕉'];
const newLength = fruits.unshift('橘子', '西瓜');
console.log(fruits);        //[ '橘子', '西瓜', '苹果', '香蕉' ]
console.log(newLength); //4
```

## 四、slice() 方法

slice() 方法用于从数组中截取指定位置的元素，返回一个新的数组。

语法是：`array.slice(start, end)`，其中，`start` 和 `end` 都是可选参数，表示选取的元素的起始位置和结束位置。如果不传入参数则默认选取整个数组。该方法返回的是一个新的数组，包含从 `start` 到 `end`（`不包括end`）的元素。

```js
const names = ['张三', '李四', '王五', '赵六'];
const slicedNames = names.slice(1, 3);
const slicedNames1 = names.slice();
const slicedNames2 = names.slice(0);
const slicedNames3 = names.slice(2);
const slicedNames4 = names.slice(3);
const slicedNames5 = names.slice(4);

//slice不改变原数组
console.log(names);          //[ '张三', '李四', '王五', '赵六' ] 
console.log(slicedNames);  //[ '李四', '王五' ]
console.log(slicedNames1); //[ '张三', '李四', '王五', '赵六' ] 
console.log(slicedNames2); //[ '张三', '李四', '王五', '赵六' ] 
console.log(slicedNames3); //[ '王五', '赵六' ]
console.log(slicedNames4); //[ '赵六' ]
console.log(slicedNames5); //[] 参数大于等于4时就输出空数组
```

## 五、splice() 方法

splice() 方法用于从数组中删除、替换或添加元素，并返回被删除的元素组成的数组，它会直接修改原数组。

语法：`array.splice(start, deleteCount, item1, item2, ...)`

其中，start 表示要修改的起始位置，deleteCount 表示要删除的元素个数，item1、item2 等表示要添加的元素。如果 `deleteCount为0`，则表示只添加元素，不删除元素。

```js
//实现删除
let arr = [1,2,3,4,5]
console.log(arr.splice(0,2));  //[ 1, 2 ] 返回被删除的元素
console.log(arr);  //[ 3, 4, 5 ]   该方法会改变原数组

//实现添加
let arr2 = [1,2,3,4,5]
console.log(arr2.splice(1,0,666,777)); //[] 返回空数组，没有删除元素
console.log(arr2);  //[ 1, 666, 777, 2, 3, 4, 5 ]  原数组改变了

// 实现替换
let arr3 = [1,2,3,4,5]
console.log(arr3.splice(2,1,"aaa","bbb")); //[ 3 ]  返回被删除的一个元素
console.log(arr3);  //[ 1, 2, 'aaa', 'bbb', 4, 5 ]  可以添加字符串

let arr4 = [1,2,3,4,5]
console.log(arr4.splice(1,4,666)); //[ 2, 3, 4, 5 ]  返回被删除的四个元素
console.log(arr4);  //[ 1, 666 ]
```

## 六、join() 方法

join() 方法用于将数组中的所有元素以指定的分隔符连接成一个 `字符串`。

```js
const fruits = ['苹果', '香蕉', '橘子'];
const joinedString = fruits.join(',');
console.log(fruits);  //[ '苹果', '香蕉', '橘子' ]
console.log(joinedString); //苹果,香蕉,橘子
```

## 七、concat() 方法

concat() 方法用于合并两个或多个数组，返回一个新的数组。

```js
const fruits1 = ['苹果', '橘子'];
const fruits2 = ['西瓜', '蓝莓'];
const combinedFruits = fruits1.concat(fruits2);
console.log(combinedFruits); //[ '苹果', '橘子', '西瓜', '蓝莓' ]
```

## 八、forEach() 方法

forEach() 方法用于对数组中的每个元素执行一个回调函数。

```js
const fruits = ['火龙果', '蓝莓', '西瓜', '葡萄'];
fruits.forEach(fruit => {
  console.log(fruit); //火龙果 蓝莓 西瓜 葡萄
});
```

## 九、map() 方法

map() 方法用于对数组中的每个元素执行一个回调函数，并返回一个新的数组，新数组中的元素为回调函数的返回值。

```js
const numbers = [1, 2, 3, 4, 5];
const squaredNumbers = numbers.map(num => num * num);
console.log(squaredNumbers); //[ 1, 4, 9, 16, 25 ]
```

## 十、filter() 方法

filter() 方法用于筛选、过滤数组中符合条件的元素，并返回一个新的数组。

```js
//筛选出偶数
const numbers = [1, 2, 3, 4, 5];
const evenNumbers = numbers.filter(num => num % 2 === 0);
console.log(evenNumbers); //[ 2, 4 ]
```

## 十一、reduce() 方法

reduce() 方法是数组对象的一个方法，用于将数组中的所有元素按照指定的规则进行归并计算，返回一个最终值。

语法：`array.reduce(callback, initialValue)`

该方法接收两个参数，第一个参数是一个回调函数，第二个参数是一个初始值。回调函数中可以接收四个参数，分别是：

1. accumulator：累加器，用于存储上一次回调函数的返回值或初始值。
2. currentValue：当前元素的值。
3. currentIndex：当前元素的索引。
4. array：数组对象本身。

initialValue 是初始值，可选参数。

```js
const arr = [1, 2, 3, 4, 5];
const sum = arr.reduce((acc, num) => {
  return acc + num
}, 10);
console.log(sum); //25
//如果初始值是设为0，则输出15
```

## 十二、fill() 方法

JS 中的 fill 方法可以填充一个数组中的所有元素，它会直接修改原数组。

语法：`array.fill(value, start, end)`

其中，value 表示要填充的值，start 和 end 表示要填充的起始位置和结束位置。如果不传入 start 和 end，则默认填充整个数组。该方法返回的是被修改后的原数组。

```js
let arr = [1, 2, 3, 4, 5];
arr.fill(0);
console.log(arr); // [0, 0, 0, 0, 0]

//将数组中从位置2到位置4(不包括位置4)的元素都填充为6
arr.fill(6, 2, 4);
console.log(arr);  //[ 0, 0, 6, 6, 0 ]
```

## 十三、数组查找 API

### 1.includes() 方法

includes 方法用于检查数组中是否包含某个元素，如果包含则返回 true，否则返回 false。

与 indexOf() 方法不同，includes() 方法不支持指定起始位置，它从数组的开头开始搜索。

```js
const fruits = ['苹果', '香蕉', '橘子', '西瓜', 1, 2, 3];
console.log(fruits.includes('橘子')); //true
console.log(fruits.includes('葡萄')); //false
console.log(fruits.includes(3));      //true
console.log(fruits.includes(4));      //false
```

### 2.indexOf() 方法

需要注意的是，indexOf 方法只会返回第一个匹配项的位置。如果数组中存在多个相同的元素，该方法只会返回第一个元素的位置。

此外，indexOf 方法还可以接受一个可选的第二个参数，用于指定从哪个位置开始查找。

```js
const fruits = ['苹果', '蓝莓', '橘子', '西瓜', '葡萄'];
console.log(fruits.indexOf('橘子', 1));  //2  返回元素下标
console.log(fruits.indexOf('橘子', 3));  //-1  没有该元素
const arr = [1,2,3,4,5,6,7,6,6,5];
// 从下标6开始查找元素5
console.log(arr.indexOf(5,6)); //9
```

### 3.lastIndexOf()() 方法

lastIndexOf() 方法用于查找数组中某个元素最后一次出现的索引（位置），如果找到则返回该索引值，否则返回 -1。

```js
const fruits = ['火龙果', '橘子', '蓝莓', '西瓜', '葡萄', '橘子'];
console.log(fruits.lastIndexOf('橘子')); //5
console.log(fruits.lastIndexOf('柚子')); //-1
```

### 4.findIndex() 方法

findIndex() 方法用于查找数组中满足条件的元素的索引，如果找到则返回该索引值，否则返回 -1。

```js
const arr = [1, 2, 3, 4, 5, 6];
const index = arr.findIndex(num => num > 3);
console.log(index);   //3  返回第一个符合条件的元素的下标
const index2 = arr.findIndex(num => num > 10);
console.log(index2); //-1  不存在符合条件的元素
```

## 十四、sort() 方法

sort() 方法用于对数组进行原地排序，会直接修改原始数组，而不会创建新的数组。默认情况下，它将数组元素视为字符串，并按照 Unicode 码点进行排序。但是，可以传入自定义的比较函数来实现基于其他规则的排序。

`比较函数`：比较函数接收两个参数，通常被称为 a 和 b，表示进行比较的两个元素。它应该返回一个负数、零或正数，表示 a 应该在 b 之前、与 b 相同位置还是在 b 之后。比较函数的返回值规则如下：

1. 如果返回值小于 0，则 a 排在 b 前面。
2. 如果返回值等于 0，则 a 和 b 的相对位置不变。
3. 如果返回值大于 0，则 a 排在 b 后面。

```js
const arr = [3, 1, 5, 2, 4];
arr.sort();
console.log(arr);  //[1, 2, 3, 4, 5]

//默认排序（按照 Unicode 码点排序）
const arr = ['f', 'k', 'c', 'a', 'b'];
arr.sort();
console.log(arr);  //[ 'a', 'b', 'c', 'f', 'k' ]

//使用比较函数进行自定义排序
const arr = [10, 2, 66, 26, 13, 1];
arr.sort((a, b) => a - b);
console.log(arr);  //[ 1, 2, 10, 13, 26, 66 ]

const arr = [10, 2, 66, 26, 13, 1];
arr.sort((a, b) => b - a);
console.log(arr);  //[ 66, 26, 13, 10, 2, 1 ]
```

## 十五、reverse() 方法

reverse() 方法用于反转数组中的元素顺序，即将数组元素进行逆序排列。

```js
const arr = [1, 2, 3, 4, 5];
arr.reverse();
console.log(arr); //[ 5, 4, 3, 2, 1 ]
```

## 十六、toString() 和 toLocaleString() 方法

toString 方法将数组转换为一个由数组元素组成的字符串，元素之间用逗号分隔。

```js
const arr = [1, 2, 3, 4, 5];
console.log(arr.toString());  //1,2,3,4,5
const arr2 = ['苹果', '蓝莓', '橘子', '西瓜', '葡萄'];
const arr3 = ['a', 'null', 'b', 'c', 'undefined', 'd', 'e']
console.log(arr2.toString());  //苹果,蓝莓,橘子,西瓜,葡萄
console.log(arr3.toString());  //a,null,b,c,undefined,d,e
```

toLocaleString 方法将数组转换为一个由数组元素组成的字符串，元素之间同样用逗号分隔，但是它会根据当前环境的语言和地区设置来决定元素的格式。

```js
//根据当前环境的语言和地区设置来决定元素的格式
const arr = [123456.789, new Date()];
//我补充写作的时间
console.log(arr.toLocaleString()); //123,456.789,2023/5/29 07:57:19

const arr2 = [1000, 2000, 3000];
const str = arr2.toLocaleString();
console.log(str); //1,000,2,000,3,000
```

## 十七、Array.from() 方法

Array.from() 是 JavaScript 中一个用于从 `类数组或可迭代对象` 创建新数组的静态方法。它接收一个可迭代对象或类数组的对象，并返回一个新的数组实例。

`Array.from(iterable, mapFn, thisArg)`

1. iterable: 必需，一个可迭代对象或类似数组的对象，用于创建新的数组。
2. mapFn (可选): 一个映射函数，用于对每个元素进行处理后返回新数组中的元素。
3. thisArg (可选): 可选参数，执行 mapFn 函数时的 this 值。

```js
//使用字符串创建数组
const str = "Hello";
const arr = Array.from(str);
console.log(arr); //[ 'H', 'e', 'l', 'l', 'o' ]
```

```js
//使用类似数组的对象创建数组
const obj = {
  0: "榴莲",
  1: "牛油果",
  2: "蓝莓",
  length: 3
};
const arr = Array.from(obj);
console.log(arr);  //[ '榴莲', '牛油果', '蓝莓' ]
```

```js
//使用映射函数处理数组元素
const numbers = [1, 2, 3, 4, 5];
const doubledNumbers = Array.from(numbers, num => num * 2);
console.log(doubledNumbers); //[ 2, 4, 6, 8, 10 ]
```

## 十八、最后的话

本文介绍了前端中数组常用的 API，涵盖了添加、删除、截取、合并、转换等常见操作。熟练掌握这些方法可以提高一定的开发效率。在实际开发中，请根据具体需求选择适合的数组方法。

能力一般，水平有限，本文可能存在纰漏或错误，如有问题欢迎指正，感谢你阅读这篇文章，如果你觉得写得还行的话，不要忘记点赞、评论、收藏哦！祝生活愉快！
