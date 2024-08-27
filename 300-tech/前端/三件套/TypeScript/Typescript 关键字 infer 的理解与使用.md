---
title: Typescript 关键字 infer 的理解与使用
date: 2024-08-22T02:34:21+08:00
updated: 2024-08-22T02:35:29+08:00
dg-publish: false
---

官方对 `infer` 的解释是这样的：

> Within the extends clause of a conditional type, it is now possible to have infer declarations that introduce a type variable to be inferred. Such inferred type variables may > be referenced in the true branch of the conditional type. It is possible to have multiple infer locations for the same type variable.

翻译后大概意思：

> `infer` 关键词常在条件类型中和 extends 关键词一同出现，表示将要推断的类型，作为类型变量可以在三元表达式的 True 部分引用。而 ReturnType 正是使用这种方式提取到了函数的返回类型

**`infer` 作用是在 extends 的条件语句中推断待推断的类型**

## 解析举例

```
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any;

type func = () => number;
type variable = string;
type funcReturnType = ReturnType<func>; // funcReturnType 类型为 number
type varReturnType = ReturnType<variable>; // varReturnType 类型为 string
```

在这个例子中，infer R 代表待推断的返回值类型，如果 T 是一个函数，则返回函数的返回值，否则返回 any

## infer 的常见使用方式

### 1. 简单的类型提取

```
type Unpacked<T> =  T extends (infer U)[] ? U : T;

type T0 = Unpacked<string[]>; // string
type T1 = Unpacked<string>; // string
```

### 2. 嵌套的按顺序类型提取

```
type Unpacked<T> =
    T extends (infer U)[] ? U :
    T extends (...args: any[]) => infer U ? U :
    T extends Promise<infer U> ? U :
    T;

type T0 = Unpacked<string>;  // string
type T1 = Unpacked<string[]>;  // string
type T2 = Unpacked<() => string>;  // string
type T3 = Unpacked<Promise<string>>;  // string
type T4 = Unpacked<Promise<string>[]>;  // Promise<string>
type T5 = Unpacked<Unpacked<Promise<string>[]>>;  // string
```

### 3. 使用多个 infer 变量进行类型提取

```
type Foo<T> = T extends { a: infer U, b: infer U } ? U : never;
type T10 = Foo<{ a: string, b: string }>;  // string
type T11 = Foo<{ a: string, b: number }>;  // string | number
```
