---
title: JS 中字符串 28 种常用 API 总结
date: 2023-08-06T11:06:00+08:00
updated: 2024-08-21T10:33:28+08:00
dg-publish: false
aliases:
  - JS 中字符串 28 种常用 API 总结
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

> **在前端开发中，处理字符串是一项常见的任务。JavaScript 提供了一系列的字符串 API，用于操作和处理字符串数据。字符串常用的 API 方法有很多，包括查找字符串、截取字符串、替换字符串、分割字符串、大小写转换、字符串拼接和字符串比较等等。本文将介绍一些常用的字符串 API，并提供相应的代码示例。**

## 二、substring() 方法和 slice() 方法

### 1.substring() 方法

字符串的 `截取` 可以使用 substring() 方法和 slice() 方法。其中 substring() 方法接受两个参数，第一个参数是起始位置，第二个参数是结束位置，截取的字符串不包括结束位置的字符。

```js
let str = "Hello, World!";
let str2 = "卡布奇诺,拿铁,维也纳,摩卡,冰美式,库比卡,浓缩"
console.log(str.substring(1, 8));    //ello, W
console.log(str2.substring(5, 8));  //拿铁,
console.log(str2);  //卡布奇诺,拿铁,维也纳,摩卡,冰美式,库比卡,浓缩
//substring(1, 8) 返回从下标1开始8结束(不包括8)的长度为7的子字符串。
```

### 2.slice() 方法

slice() 方法也接受两个参数，第一个参数是起始位置，第二个参数是结束位置，但是可以使用 `负数` 表示从后往前数的位置。返回一个新字符串，不会影响原始字符串。如果省略第二个参数，该方法会提取字符串的剩余部分。提取的字符串为 `[start, end)` 长度的子字符串，其中起始位置包含第一个字符。

```js
let str = "123456789";
console.log(str.slice(0, 5));   //12345
console.log(str.slice(2));      //3456789
console.log(str.slice(3, -2));  //4567
console.log(str);     //123456789
let str2 = "给我来一杯卡布奇诺!";
console.log(str2.slice(0, 5));  //给我来一杯
console.log(str2.slice(2));     //来一杯卡布奇诺!
console.log(str2.slice(3, -2)); //一杯卡布奇
console.log(str2.slice(3, -4)); //一杯卡
console.log(str2);    //给我来一杯卡布奇诺!
```

## 三、replace() 方法和 replaceAll() 方法

### 1.replace() 方法

replace() 方法用于将指定的字符串或正则表达式匹配的部分 `替换` 为新的字符串，替换字符串中的 `第一个` 匹配项。

```js
const str = "Hello, World! World!";
const newStr = str.replace("World", "JavaScript");
console.log(newStr);  //Hello, JavaScript! World!

//将一个或多个空格替换成空字符
const str2 = "你    好 呀!"
const newStr2 = str2.replace(/\s+/g, "");
console.log(newStr2);  //你好呀!
```

replace("World", "JavaScript") 将字符串中的第一个 "World" 替换为 "JavaScript"。

正则表达式 `/\s+/g` 可以匹配所有连续的空格字符。其中，\\s 表示空白字符，包括空格、制表符和换行符，+ 表示匹配一个或多个。

### 2.replaceAll() 方法

replaceAll() 方法是在 ES2021（ES12）中引入的，用于 `替换` 字符串中 `所有` 匹配的部分。

```js
const str3 = "哎哟! 你干嘛啊啊啊!";
const newStr3 = str3.replaceAll("啊", "呀");
console.log(newStr3);  //哎哟! 你干嘛呀呀呀!
const str4 = "你    好 呀!"
const newStr4 = str4.replaceAll(" ", "");
console.log(newStr4);  //你好呀!
```

可以直接将字符串空格作为第一个参数，将空字符串作为第二个参数，这样可以把所有的空格字符替换为空字符。

## 四、trim() 方法

trim() 方法 `去除` 字符串两端的 `空格`，返回新的字符串。

```js
let str = "    卡布奇诺 拿铁 维也纳    摩卡 冰美式 浓缩  "
console.log(str.trim());        //卡布奇诺 拿铁 维也纳    摩卡 冰美式 浓缩
console.log(str.trimStart()); //卡布奇诺 拿铁 维也纳    摩卡 冰美式 浓缩    
console.log(str.trimEnd());   //    卡布奇诺 拿铁 维也纳    摩卡 冰美式 浓缩
```

trim() 去掉了字符串开头和结尾的空格。trimStart()(trimLeft) 只删除字符串开头的空格，trimEnd()(trimRight) 只删除字符串结尾的空格。trimLeft() 和 trimRight() 方法现已舍弃。

## 五、match() 方法

match 方法用于在一个字符串中查找匹配的子串，并返回一个数组，包含所有匹配的子串及其位置。

语法: `string.match(regexp)`，其中，string 是要匹配的字符串，regexp 是一个正则表达式。

如果 regexp`没有全局标志g`，则 match() 方法返回一个数组，该数组的第一个元素是匹配到的子串，后面的元素是正则表达式中的捕获组（如果有的话）。

如果 regexp`有全局标志g`，则 match() 方法返回所有匹配到的子串组成的数组。如果没有匹配到任何子串，则返回 null。

```js
const str = "Hello, World!";
const matches = str.match(/l/g);
console.log(matches);  //[ 'l', 'l', 'l' ]

//使用"/(l)(o)/g"正则表达式来查找字符串中所有的 "lo" 组合。
const str2 = "Hello, World!";
const matches2 = str2.match(/(l)(o)/g);
console.log(matches2);  //[ 'lo' ]

//使用"/o/g"正则表达式来查找字符串中所有的小写字母 "o"。
const str3 = 'Hello, World!';
console.log(str3.match('o'));//['o', index: 4, input: 'Hello, World!', groups: undefined]
console.log(str3.match('l'));//['l', index: 2, input: 'Hello, World!', groups: undefined]
console.log(str3.match(/o/g)); //['o', 'o']
```

## 六、split() 方法

split() 方法将字符串按指定的分隔符 `拆分` 成数组。

```js
//s.split("-")使用-作为分隔符将字符串拆分成数组。
const s = "卡布奇诺-拿铁-维也纳-摩卡-冰美式-浓缩";
const arr = s.split("-");
console.log(arr); //[ '卡布奇诺', '拿铁', '维也纳', '摩卡', '冰美式', '浓缩' ]

const str = "Hello     World!";
console.log(str.split(/\s+/));   //['Hello', 'World!']
const str2 = "Hello,World!";
console.log(str2.split("-"));     //['Hello,World!']
```

`/\s+/` 正则表达式匹配一个或多个空白字符。如果分隔符没有在字符串中出现，split 函数会返回一个包含 `整个字符串` 作为唯一元素的数组。

## 七、search() 方法

search 方法用于在一个字符串中 `查找匹配的子串`，并返回 `第一个` 匹配的子串的位置。它可以接受一个参数，用于指定要查找的子串。

```js
const str = "Hello, World!";
console.log(str.search(/World/i));  //7
console.log(str.search(/world/i));  //7
console.log(str.search("World"));  //7
console.log(str.search("l"));  //2
const str2 = "维也纳,拿铁,摩卡,冰美式,浓缩";
console.log(str2.search("拿铁"));      //4
console.log(str2.search("卡布奇诺")); //-1
```

`/World/i` 是一个正则表达式，用于匹配字符串中的 "World"，其中 `i` 是忽略大小写的标志。 如果找不到匹配的子串则返回 -1。

## 八、valueOf() 方法

valueOf() 方法返回字符串对象的 `原始值`。

```js
const num = 123;
const str = "卡布奇诺 拿铁 维也纳 摩卡 冰美式";
const bool = true;
const obj = {name:"咖啡", type:"卡布奇诺"}
const arr = ["拿铁", "摩卡", "美式", "卡布奇诺"]
const date = new Date();

console.log(num.valueOf());   //123
console.log(str.valueOf());     //"卡布奇诺 拿铁 维也纳 摩卡 冰美式"
console.log(bool.valueOf());  //true
console.log(obj.valueOf());   //{name: '咖啡', type: '卡布奇诺'}
console.log(arr.valueOf());    //['拿铁', '摩卡', '美式', '卡布奇诺']
console.log(date.valueOf());  //1685850114551  (获取时间戳)
```

valueOf() 方法返回一个对象的原始值。如果对象本身就是一个原始值（如数字、字符串、布尔值），则直接返回该值；否则，返回对象本身。

## 九、concat() 方法

concat 用于将多个字符串 `连接` 成一个新的字符串。该方法不会改变原有的字符串，而是返回一个新的字符串。

语法：`string.concat(str1, str2, ..., strN)`

其中，string 是要连接的字符串，str1, str2, ..., strN 是要连接的其他字符串，可以有多个参数。

```js
const str1 = "冰美式";
const str2 = "和摩卡";
const str3 = "!";
const result = str1.concat(str2, str3);
console.log(result);  //冰美式和摩卡!
console.log(str1);    //冰美式
console.log(str2);    //和摩卡
console.log(str3);    //!
```

concat() 方法返回的是一个新的字符串，原有的字符串不会改变。如果要改变原有的字符串，可以使用赋值运算符（+=）或字符串模板 (用 \`\` 和 ${} ) 来实现。

```js
let str = "冰美式";
str += "和摩卡!";
console.log(str);   //冰美式和摩卡!

let str2 = "冰美式";
str2 = `${str2}和摩卡!`;
console.log(str2);  //冰美式和摩卡!
```

## 十、JSON.stringify() 方法

JSON.stringify() 是 JavaScript 中的一个方法，用于将 JavaScript 对象 `转换为JSON字符串`。该方法接受一个 JavaScript 对象作为参数，并返回一个 JSON 字符串。

语法：`JSON.stringify(value, replace, space)`

1. value 是要转换为 JSON 字符串的 JavaScript 对象；
2. replace 是一个函数，用于控制转换过程中哪些属性应该被包含在 JSON 字符串中；
3. space 是一个用于缩进输出的空格数或缩进字符串。

```js
const person = {
  name: "张三",
  age: 20,
  city: "南昌",
  fn: function(){console.log(666);},
  test: undefined,
  hobbies: ["读书", "旅游", "羽毛球"]
};
console.log(JSON.stringify(person));
//{"name":"张三","age":20,"city":"南昌","hobbies":["读书","旅游","羽毛球"]}
console.log(JSON.stringify(person, ["name", "hobbies"]));
//{"name":"张三","hobbies":["读书","旅游","羽毛球"]}
console.log(JSON.stringify(person, null, 2));
// {
//   "name": "张三",
//   "age": 20,
//   "city": "南昌",
//   "hobbies": [
//     "读书",
//     "旅游",
//     "羽毛球"
//   ]
// }
```

1. 如果对象中包含函数、undefined、symbol 或循环引用等特殊类型的属性，则在转换为 JSON 字符串时会被忽略或转换为 null。
2. 如果需要控制转换过程中哪些属性应该被包含在 JSON 字符串中，可以使用 replace 参数。这里只输出 `name` 和 `hobbies`。
3. 第三个参数表示缩进的空格数，这里设置为 2。由于缩进输出的 JSON 字符串更易于阅读和调试，因此在开发过程中常常会使用这种方式。
4. 可以用 `JSON.parse()` 方法将这些 JSON 字符串转换回 JS 对象。

## 十一、includes() 方法、indexOf() 方法和 lastIndexOf() 方法

### 1.includes() 方法

includes 方法用于检查数组中是否 `包含` 某个元素，如果包含则返回 true，否则返回 false。includes() 方法 `区分大小写`。

```js
const str = "Hello world";
console.log(str.includes("world")); //true
console.log(str.includes("Hello")); //true
console.log(str.includes("o"));      //true
console.log(str.includes("h"));      //false
console.log(str.includes("H"));      //true
console.log(str.includes("JS"));     //false
console.log(str.includes("WORLD"));  //false
```

### 2.indexOf() 方法

需要注意的是，indexOf 方法只会返回 `第一个` 匹配项的位置。如果数组中存在多个相同的元素，该方法只会返回第一个元素的位置，indexOf() 方法是 `区分大小写` 的。

```js
const str = "Hello world";
console.log(str.indexOf("o"));      //4
console.log(str.indexOf("e"));      //1
console.log(str.indexOf("E"));      //-1
console.log(str.indexOf("llo"));    //2
console.log(str.indexOf("world"));  //6
console.log(str.indexOf("Hello"));  //0
console.log(str.indexOf("hello"));  //-1
```

### 3.lastIndexOf() 方法

lastIndexOf() 方法与 indexOf() 方法类似，不同之处在于它返回指定字符串或字符在原字符串中 `最后一次` 出现的位置，而不是第一次出现的位置。

```js
const str = "Hello world, world";
console.log(str.lastIndexOf("world"));//13
console.log(str.lastIndexOf("l", 8));   //3
console.log(str.lastIndexOf("l"));      //16
console.log(str.lastIndexOf("L"));      //-1
console.log(str.lastIndexOf("k"));      //-1
console.log(str.lastIndexOf("hello")); //-1
```

我们使用 `str.lastIndexOf("l", 8)` 查找字符串 "Hello world, world" 中字符 "l" 最后一次出现的位置，但是限定了查找范围只在前 8 个字符内。也就相当于在字符串 `"Hello wo"` 中使用 `lastIndexOf("l")` 方法，`l` 最后出现在下标 3 的位置，所以返回 3。

如果 lastIndexOf() 方法没有找到指定的字符串或字符，它将返回 -1。

## 十二、endsWith() 方法和 startsWith() 方法

### 1.endsWith() 方法

endsWith() 方法用于判断一个字符串是否以指定的子字符串结尾。

```js
const str = "Hello, World!";
console.log(str.endsWith("!"));     //true
console.log(str.endsWith("ld"));   //false
console.log(str.endsWith("l"));     //false
console.log(str.endsWith("W", 7)); //false  "Hello, " 不以W结尾
console.log(str.endsWith("W", 8)); //true   "Hello, W" 以W结尾
```

endsWith() 方法接受两个参数，第一个参数是要查找的子字符串，第二个参数是查找的终止位置。当省略第二个参数时，默认从结尾开始查找。如果找到指定的子字符串，则返回 true，否则返回 false。

### 2.startsWith() 方法

startsWith() 方法用于判断一个字符串是否以指定的子字符串开头。

```js
const str = "Hello, World!";
console.log(str.startsWith("He"));   //true
console.log(str.startsWith("l"));     //false
console.log(str.startsWith("l", 3));  //true  "lo, World!"以l开头
console.log(str.startsWith("l", 4));  //false  "o, World!"不以l开头
console.log(str.startsWith("e", 1));  //true  "ello, World!"以e开头
console.log(str.startsWith("e", 4));  //false "o, World!"不以e开头
```

startsWith() 方法接受两个参数，第一个参数是要查找的子字符串，第二个参数是查找的 `起始位置`。当省略第二个参数时，默认从头开始查找。如果找到指定的子字符串，则返回 true，否则返回 false。

## 十三、normalize() 方法

normalize() 用于将字符串 `标准化` 为一致的 Unicode 形式。因为在 Unicode 中，同样的字符可以有多种不同的表示方式，这可能会导致某些比较或排序操作的混淆。normalize() 方法可以将这些不同表示方式的字符转换为一致的形式，以便在比较和排序等操作中得到正确的结果。

normalize() 方法接受一个参数，表示需要使用哪种标准化形式，共有四种：

1. NFC：使用组合字符标准化形式。
2. NFD：使用分解字符标准化形式。
3. NFKC：使用组合字符同时兼容性标准化形式。
4. NFKD：使用分解字符同时兼容性标准化形式。

```js
const str1 = "café";
const str2 = "cafe\u0301";
console.log(str1.normalize());  //café
console.log(str2.normalize());  //café
console.log(str1 === str2);     //false
console.log(str1.normalize() === str2.normalize()); // true
console.log(str1.normalize('NFD'));   //café
console.log(str1.normalize('NFC'));   //café
console.log(str1.normalize('NFKC'));  //café
console.log(str1.normalize('NFKD'));  //café
console.log(str2.normalize('NFD'));   //café
console.log(str2.normalize('NFC'));   //café
console.log(str2.normalize('NFKC'));  //café
console.log(str2.normalize('NFKD'));  //café
```

## 十四、repeat() 方法

repeat() 方法用于将字符串 `重复` 指定的次数，并返回重复后的字符串。

```js
const str = "冰美式";
console.log(str.repeat(-1)); //报错
console.log(str.repeat(0));  //空字符
console.log(str.repeat(3));  //冰美式冰美式冰美式
console.log(str);  //冰美式
```

repeat() 方法只接受一个参数，表示需要重复的次数。少于 0 的次数会报错，等于 0 的次数会返回空字符串，大于等于 1 的次数会返回重复后的字符串。

## 十五、localeCompare() 方法

字符串的 `比较` 可以使用比较运算符和 localeCompare() 方法。其中比较运算符可以直接比较两个字符串的大小，localeCompare() 方法可以比较两个字符串在字典序中的大小。

返回值是一个数字，目前的主流浏览器都返回的是 1、0、-1 三个值。

1. 返回值大于 0：说明当前字符串 string 大于对比字符串 targetString
2. 返回值小于 0：说明当前字符串 string 小于对比字符串 targetString
3. 返回值等于 0：说明当前字符串 string 等于对比字符串 targetString

```js
var str1 = "hello";
var str2 = "world";
var str3 = "hello";
console.log(str1 > str2); //false
console.log("a" > "b");   //false
console.log("f" > "c");    //true
console.log(str1.localeCompare(str2)); // -1
console.log(str1.localeCompare(str3)); // 0
console.log(str2.localeCompare(str3)); // 1
```

## 十六、toUpperCase() 方法和 toLowerCase() 方法

### 1.toUpperCase() 方法

toUpperCase：将字符串中的所有字母变为 `大写字母`，非字母不受影响，不改变原字符串。

```js
const str = "hello world 冰美式";
const upperStr = str.toUpperCase();
console.log(upperStr);  //HELLO WORLD 冰美式
console.log(str);  //hello world 冰美式
```

### 2.toLowerCase() 方法

toLowerCase：将字符串中的所有字母变为 `小写字母`，非字母不受影响，不改变原字符串。

```js
const str = "HELLO WORLD 冰美式";
const lowerStr = str.toLowerCase();
console.log(lowerStr);  //hello world 冰美式
console.log(str);  //HELLO WORLD 冰美式
```

## 十七、toLocaleUpperCase() 方法和 toLocaleLowerCase() 方法

`toLocaleUpperCase()` 和 `toLocaleLowerCase()` 是 JavaScript 中的字符串方法，用于将字符串转换为大写字母和小写字母，与 `toUpperCase()` 和 `toLowerCase()` 方法的区别在于它们使用特定的语言环境设置来进行转换，可以根据用户所处的地区，将不同的字符转换为大写或小写。

### 1.toLocaleUpperCase() 方法

toLocaleUpperCase：将字符串中的所有字母转换为大写字母，同时根据本地化规则将字符转换为特定于语言和区域设置的大小写形式。

```js
// 将字符转换为特定于土耳其语本土化规则的大写形式
const str = "hello world";
const upperStr = str.toLocaleUpperCase('tr-TR');
console.log(upperStr);  //HELLO WORLD

// 希腊语字符示例：
const str = 'Γειά σου';
console.log(str.toLocaleUpperCase()); //ΓΕΙΆ ΣΟΥ

// 阿拉伯语字符示例：
const str2 = 'مرحبا بالعالم';
console.log(str2.toLocaleUpperCase()); //مرحبا بالعالم
```

### 2.toLocaleLowerCase() 方法

toLocaleLowerCase：将字符串中的所有字母转换为小写字母，同时根据本地化规则将字符转换为特定于语言和区域设置的大小写形式。

```js
//将字符转换为特定于土耳其语本土化规则的小写形式。
const str = "HELLO WORLD";
const lowerStr = str.toLocaleLowerCase('tr-TR');
console.log(lowerStr);  //hello world
//土耳其语中的I转换为ı
const str2 = 'Türkİye Cumhurİyetİ';
console.log(str2.toLocaleLowerCase()); //türkiye cumhuriyeti
```

## 十八、toString() 方法

toString() 方法返回一个对象的 `字符串形式`。对于自定义对象，如果没有为其定义 toString() 方法，会使用默认的 toString() 实现，即返回 “\[Object object\]” 字符串。

```js
const num = 666
console.log(num.toString());  //666

const arr = [1, 2, 3];
console.log(arr.toString());  //1,2,3

const b = true;
console.log(b.toString());    //true

const obj = {name: "张三", age: 20};
console.log(obj.toString());  //[object Object]
```

我们可以通过为自定义对象定义 toString() 方法来自定义它们的字符串表示形式。

```js
function Person(name, age) {
  this.name = name;
  this.age = age;
}
Person.prototype.toString = function() {
  return `Person(name='${this.name}', age=${this.age})`;
};
const str = new Person("张三", 20);
console.log(str.toString());  //Person(name='张三', age=20)
```

## 十九、at() 方法和 charAt() 方法

### 1.charAt() 方法

charAt() 方法用于 `返回指定位置的字符`，如果索引超出字符串范围，则返回空字符串。

```js
//返回指定位置的字符
const str = "Hello, World!";
console.log(str.charAt(1));   //e
console.log(str.charAt(8));   //o
console.log(str.charAt(20));  //输出空字符串
```

### 2.at() 方法

at() 方法用于 `返回指定位置的Unicode字符`，如果索引超出字符串范围，则返回 undefined。

```js
const str = '😀hello';
console.log(str.at(0)); //�
console.log(str.at(4)); //l
console.log(str.at(6)); //o
const str1 = "Oh, what are you doing?";
const str2 = "卡布奇诺 拿铁 维也纳 摩卡 冰美式";
const str3 = "ΠΑΡΆΔΕΙΓΜΑ ΕΝΌΤΗΤΑΣ";
const str4 = "The 🐕 jumps over the 🦊.";
console.log(str1.at(4)); //w
console.log(str2.at(2)); //奇
console.log(str3.at(4)); //Δ
console.log(str4.at(4)); //�
```

## 二十、codePointAt() 方法和 charCodeAt() 方法

### 1.codePointAt() 方法

codePointAt() 方法返回指定索引位置处的字符的 Unicode 编码。

```js
const str = "Hello world";
const str2 = "卡布奇诺";
//字符"e"的Unicode编码为101
console.log(str.codePointAt(1));   //101
console.log(str.codePointAt(2));   //108
console.log(str2.codePointAt(0));  //21345
console.log(str2.codePointAt(1));  //24067

const str = "彩虹 🌈";
const str2 = "独角兽 🦄";
console.log(str.codePointAt(0)); //24425
console.log(str.codePointAt(3)); //127752
console.log(str.codePointAt(5)); //undefined
console.log(str2.codePointAt(1)); //35282
console.log(str2.codePointAt(4)); //129412
console.log(str2.codePointAt(6)); //undefined
```

如果指定的索引位置不存在字符，则 codePointAt() 方法返回 undefined。

### 2.charCodeAt() 方法

charCodeAt() 方法可返回指定位置的字符的 Unicode 编码，返回值是 0 - 65535 之间的整数。

```js
const str = "Hello world";
const str2 = "卡布奇诺";
//字符"e"的Unicode编码为101
console.log(str.charCodeAt(1));   //101
console.log(str.charCodeAt(2));   //108
console.log(str2.charCodeAt(0));  //21345
console.log(str2.charCodeAt(1));  //24067

const str = "彩虹 🌈";
const str2 = "独角兽 🦄";
console.log(str.charCodeAt(0)); //24425
console.log(str.charCodeAt(3)); //55356
console.log(str.charCodeAt(5)); //NaN
console.log(str2.charCodeAt(1)); //35282
console.log(str2.charCodeAt(4)); //55358
console.log(str2.charCodeAt(6)); //NaN
```

如果指定的索引位置不存在字符，则 codePointAt() 方法返回 NaN。

## 二十一、最后的话

本文介绍了前端中字符串常用的 API，涵盖了添加、删除、截取、合并、转换等常见操作。这些方法可以帮助我们更方便地处理字符串，提高一定的开发效率。在实际开发中，请根据具体需求选择适合的字符串处理方法。

能力一般，水平有限，本文可能存在纰漏或错误，如有问题欢迎各位大佬指正，感谢你阅读这篇文章，如果你觉得写得还行的话，不要忘记点赞、评论、收藏哦！祝生活愉快！
