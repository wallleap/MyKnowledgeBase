---
title: 规范 JavaScript 命名
date: 2023-02-27 13:49
updated: 2023-02-27 13:52
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - 规范 JavaScript 命名
rating: 1
tags:
  - JavaScript
category: web
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: null
url: //myblog.wallleap.cn/post/1
---

## 驼峰命名法

驼峰命名法按照第一个字母是否大写分为：

1. **Pascal Case 大驼峰式命名法**：首字母大写

```js
`StudentInfo`、`UserInfo`、`ProductInfo`
```

2. **Camel Case 小驼峰式命名法**：首字母小写

```js
 `studentInfo`、`userInfo`、`productInfo`
```

## 变量

- 命名方法：小驼峰式命名法

- 命名规范：前缀应当是名词。(函数的名字前缀为动词，以此区分变量和函数)

- 命名建议：尽量在变量名字中体现所属类型，如:length、count 等表示数字类型；而包含 name、title 表示为字符串类型。

- 示例

```js
// 好的命名方式
var maxCount = 10
var tableTitle = 'LoginTable'

// 不好的命名方式
var setCount = 10
var getTitle = 'LoginTable'
```

## 函数

- 命名方法：小驼峰式命名法

- 命名规范：前缀应当为动词。

- 命名建议：可使用常见动词约定

| 动词   | 含义                          | 返回值                                                |
| ------ | ----------------------------- | ----------------------------------------------------- |
| `can`  | 判断是否可执行某个动作 (权限) | 函数返回一个布尔值。true：可执行；false：不可执行     |
| `has`  | 判断是否含有某个值            | 函数返回一个布尔值。true：含有此值；false：不含有此值 |
| `is`   | 判断是否为某个值              | 函数返回一个布尔值。true：为某个值；false：不为某个值 |
| `get`  | 获取某个值                    | 函数返回一个非布尔值                                  |
| `set`  | 设置某个值                    | 无返回值、返回是否设置成功或者返回链式对象            |
| `load` | 加载某些数据                  | 无返回值或者返回是否加载完成的结果                    |

- 示例

```js
// 是否可阅读
function canRead() {
  return true
}

// 获取名称
function getName() {
  return this.name
}
```

## 常量

- 命名方法：名称全部大写。

- 命名规范：使用大写字母和下划线来组合命名，下划线用以分割单词。

- 命名建议：无

- 示例

```js
var MAX_COUNT = 10
var URL = 'http://www.baidu.com'
```

## 构造函数

- 命名方法：大驼峰式命名法，首字母大写。

- 命名规范：前缀为名称。

- 命名建议：无。

- 示例

```javascript
function Student(name) {
	this.name = name
}

var st = new Student('tom')
```

## 类的成员

① 公共属性和方法：跟变量和函数的命名一样。

② 私有属性和方法：前缀为_(下划线)，后面跟公共属性和方法一样的命名方式。

```javascript
function Student(name) {
  var _name = name // 私有成员

  // 公共方法
  this.getName = function () {
    return _name
  }

  // 公共方式
  this.setName = function (value) {
    _name = value
  }
}
var st = new Student('tom')
st.setName('jerry')
console.log(st.getName()) // => jerry：输出_name私有变量的值
```

## 参考文档

1. [JavaScript 开发规范](https://www.cnblogs.com/polk6/p/4660195.html)
