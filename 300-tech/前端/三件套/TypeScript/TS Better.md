---
title: TS Better
date: 2024-08-22T02:21:07+08:00
updated: 2024-08-22T02:21:21+08:00
dg-publish: false
---

```ts
// 1. 通过 /** */ 形式的注释可以给 TS 类型做标记提示，编辑器会有更好的提示：

/** This is a cool guy. */
interface P {
  /** This is name. */
  name: string,
}

const p: P = {
  name: 'cool'
}

/* 2. 优先使用 接口继承而不是交叉类型 ===================> */

interface Names {
  name: string;
}

interface Ages {
  age: number;
}

interface Person extends Names, Ages {
  sex: number;
}

let Lisa = <Person>{};

Lisa.age = 11
Lisa.name = 'Lisa'
Lisa.sex = 1


/* 3. interface & type (类型别名) =======================================> */

interface Point {
  x: number,
  y: number
}

interface SetPoint {
  (x: number, y: number): void
}

type Point1 = {
  x: number,
  y: number
}

type SetPoint1 = (x: number, y: number) => void


/* <====== 能用 interface 实现，就用 interface , 如果不能就用 type=======> */

// Interface extends interface
interface P1 { x: number }
interface Y1 extends P1 { y: number }


// Type alias extends type alias
type P2 = { x: number; };
type Y2 = P2 & { y: number; };


// Interface extends type alias
type P3 = { x: number }
interface Y3 extends P3 { y: number }

// Type alias extends interface
interface P4 { x: number }
type Y4 = P4 & { y: number }



/* typeof ==============================================>  */
interface Option {
  timeout: number
}
const defaultOpt: Option = {
  timeout: 500
}

// 反过来
const defaultOption = {
  timeout: 500,
  name: 'Lisa'
}
type Opt = typeof defaultOption

/* keyof 提取其属性的名称, 与 Object.keys 略有相似，只不过 keyof 取 interface 的键 ======> */

interface K {
  name: string,
  age: number
}

type keys = keyof K


// 写一个方法获取对象里面的属性值
function get<T extends object, K extends keyof T>(o: T, name: K): T[K] {
  return o[name]
}

const article = {
  time: 2021,
  text: 'hello world'
}

const time = get(article, 'time')
const text = get(article, 'text')


/* 查找类型 ====================================================================> */
interface Person {
  addr: {
    city: string,
    street: string,
  }
}

// 当需要使用 addr 的类型时，除了把类型提出来
interface Address {
  city: string,
  street: string,
}

interface Person {
  addr: Address,
}

// 还可以
const addr: Person["addr"] = {
  city: 'string',
  street: 'string',
}

/* 查找类型 + 泛型 + keyof ======================================================================>*/

// 泛型（Generics）是指在定义函数、接口或类的时候，不预先指定具体的类型，而在使用的时候再指定类型的一种特性。

interface API {
  '/user': { name: string },
  '/menu': { foods: string[] }
}

const getAPI = <URL extends keyof API>(url: URL): Promise<API[URL]> => {
  return fetch(url).then(res => res.json());
}

getAPI('/user')
getAPI('/menu').then(user => user.foods);


/* 类型断言 ===================================================================================> */
const a = { name: 'Lisa' } as any

console.log(a.name);


/* 显式泛型 ===================================================================================> */

// $('button') 是个 DOM 元素选择器，可是返回值的类型是运行时才能确定的，除了返回 any ，还可以
function $<T extends HTMLElement>(id: string): T {
  return (document.getElementById(id)) as T;
}

// 不确定 input 的类型
// const input = $('input');

// Tell me what element it is.
const input = $<HTMLInputElement>('input');
console.log('input.value: ', input.value);


/* (深度声明 readonly) DeepReadonly ===================================================================================> */

// readonly 只能用在类（TS 里也可以是接口）中的属性上，相当于一个只有 getter 没有 setter 的属性的语法糖

type DeepReadonly<T> = {
  readonly [P in keyof T]: DeepReadonly<T[P]>;
}

const deep = { foo: { bar: 22 } }
const res = deep as DeepReadonly<typeof deep>
// res.foo.bar = 33 // Cannot assign to 'bar' because it is a read-only property.ts(2540)


/* 内置的类型 Required（必填） & Partial（可选） & Pick（挑选部分）  ===================================================================================> */

// type Partial<T> = {
//   [P in keyof T]?: T[P];
// };

// type Required<T> = {
//   [P in keyof T]-?: T[P];
// };

// type Pick<T, K extends keyof T> = {
//   [P in K]: T[P];
// };

interface User {
  id: number;
  age: number;
  name: string;
};

// 相当于: type PartialUser = { id?: number; age?: number; name?: string; }

type PartialUser = Partial<User>

// 相当于: type PickUser = { id: number; age: number; }
type PickUser = Pick<User, "id" | "age">


/* Condition Type  ===================================================================================> */

// T extends U ? X : Y

type isTrue<T> = T extends true ? true : false
// 相当于 type t = false
type t = isTrue<number>

// 相当于 type t = false
type t1 = isTrue<false>

/* never & Exclude & Omit ===================================================================================> */

// type Exclude<T, U> = T extends U ? never : T;

// 相当于: type A = 'a'
type A = Exclude<'x' | 'a', 'x' | 'y' | 'z'>


// type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
interface User {
  id: number;
  age: number;
  name: string;
};

// 相当于: type PickUser = { age: number; name: string; }
type OmitUser = Omit<User, "id">

// 函数类型接口   ===================================================================================> */

interface keyMap {
  (key: string, value: string): string;
}
let logKeyMap: keyMap = function (key1: string, value: string): string {
  return key1 + value;
};
console.log(logKeyMap("key1", "value"));


// 泛型就是解决类、接口、方法的复用性，以及对不特定数据类型的支持 ===================================================================================> */
function getDate<T>(value: T): T {
  return value;
}
console.log(getDate<number>(123));

// 类的泛型 =====================>
class List<T> {
  public list: T[] = [];
  add(value: T): void {
    this.list.push(value);
  }
}
let list = new List<number>();

// 接口的泛型 =========================>
interface ConfigFn {
  <T>(value: T): T;
}

let getData: ConfigFn = function <T>(value: T): T {
  return value;
};

console.log(getData<string>("z11"));

// interface ConfigFn<T> {
//   (value: T): T;
// }

// function getData<T>(value: T): T {
//   return value;
// }

// let myGetDate: ConfigFn<string> = getData;
// console.log(myGetDate("3"));
```

## 更多参考

1. [typescript高级用法](https://juejin.cn/post/6926794697553739784)
