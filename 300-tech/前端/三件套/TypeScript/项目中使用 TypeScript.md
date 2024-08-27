---
title: 项目中使用 TypeScript
date: 2024-08-22T01:56:25+08:00
updated: 2024-08-22T01:56:44+08:00
dg-publish: false
source: https://github.com/peng-yin/note/issues/25
---

## why use typescript ?

那么为什么应该使用 TypeScript 呢？（在代码质量这个层面）

- 代码中没有与参数或变量名的拼写错误相关的一些非常烦人的运行时错误
- 您可以建立清晰明了的对象之间的约定
- 不用 hack 的手段就能实现类似在 class 中使用 private 的事情
- 有来自编译器的即时反馈，很多错误都是在编译阶段捕获的，而不是在运行时
- 让非 JS 开发人员更容易阅读和理解代码
- 你可以使用 JavaScript 未来版本中的功能
- 为单元测试编写 mocks，stubs 和 fakes 要容易得多，因为你知道他们的确切接口。

## How to get started ?

```
1.  npm i -g typescript &amp;&amp; mkdir typescript &amp;&amp; cd typescript &amp;&amp; tsc -- init

2. touch index.ts &amp;&amp; tsc -w  // 创建一个 .ts 文件，并告诉 TypeScript 编译器监视文件变化
```

## tscongfig.json

```

{
    "compilerOptions": {
        // 不报告执行不到的代码错误。
        "allowUnreachableCode": true,
        // 必须标注为null类型,才可以赋值为null
        "strictNullChecks": true,
        // 严格模式, 强烈建议开启
        "strict": true,
        // 支持别名导入:
        // import * as React from "react"
        "esModuleInterop": true,
        // 目标js的版本
        "target": "es5",
        // 目标代码的模块结构版本
        "module": "es6",
        // 在表达式和声明上有隐含的 any类型时报错。
        "noImplicitAny": true,
        // 删除注释
        "removeComments": true,
        // 保留 const和 enum声明
        "preserveConstEnums": false,
        // 生成sourceMap    
        "sourceMap": true,
        // 目标文件所在路径
        "outDir": "./lib",
        // 编译过程中需要引入的库文件的列表
        "lib": [
            "dom",
            "es7"
        ],
        // 额外支持解构/forof等功能
        "downlevelIteration": true,
        // 是否生成声明文件
        "declaration": true,
        // 声明文件路径
        "declarationDir": "./lib",
        // 此处设置为node,才能解析import xx from 'xx'
        "moduleResolution": "node"
    },
    // 入口文件
    "include": [
        "src/main.ts"
    ]
}

```

## typescript 特性

### 基础类型

___

- ts 和 js 最大的区别就是 ts 多了类型注解功能， type 类型（基础类型和高级类型 => 通过基础类型组成的自定义类型）
- ts 中基础类型有如下几种:boolean / number / string / object / 数组 / 元组 / 枚举 / any / undefined / null / void / never
- 上面列出的类型, 是 ts 中表示类型的关键字, 其中 object 其实是包含数组/元祖/枚举, 在 ts 的概念中, 这个叫做类型兼容, 就是说数组类型数据, 也可以用 object 来标注

\**TS 最常用的功能是静态类型。没有使用严格类型校验也就没有使用 TypeScript 的意义*
\*

```
let array:object = [12, 311]
```

> 字面量

```
// 字面量 -&gt; 直接声明
const n:number = 123;
const s:string = '456';
const o:object = {a:1,b:'2'};

// 非字面量 -&gt; 非new关键词化出来的数据
const n:Number = new Number(123);
const s:String = new String('456');
const o:Object = new Object({a:1,b:'2'});
```

> boolean

```
const IS_MOBILE:boolean = true;
const IS_TABLE:boolean = false;
```

> number

```
let decLiteral: number = 6;
let hexLiteral: number = 0xf00d;
let binaryLiteral: number = 0b1010;
let octalLiteral: number = 0o744;
```

> string

```
let s1:string = 'hello world!';
let s2:string = 'hello ${name}`;
```

> 数组

第 1 种, 通过在指定类型后面增加\[\], 表示该数组内的元素都是该指定类型:

```
let numbers:number[] = [1,2,3,4,5];
// number|string代表联合类型, 下面的高级类型中会讲
let numbers:(number|string)[] = [1,2,3,4,'5'];
```

第 2 种, 通过泛型表示, Array<元素类型>, 泛型会在后面课讲解, 先做了解即可:

```
let numbers:Array&lt;number&gt; = [1,2,3,4,5];
```

> 元组 (Tuple)

元组类型表示一个已知元素数量和类型的数组, 各元素的类型不必相同:

```
let list1:[number, string] = [1, '2', 3]; // 错误, 数量不对, 元组中只声明有2个元素
let list2:[number, string] = [1, 2]; // 错误, 第二个元素类型不对, 应该是字符串'2'
let list3:[number, string] = ['1', 2]; // 错误, 2个元素的类型颠倒了
let list4:[number, string] = [1, '2']; // 正确
```

> 枚举 (enum)

**枚举是 ts 中有而 js 中没有的类型, 编译后会被转化成对象**, 默认元素的值从 0 开始, 如下面的 Color.Red 的值为 0, 以此类推 Color.Green 为 1, Color.Blue 为 2:

**使用枚举来存储应用程序状态**

```
enum Color {Red, Green, Blue}
// 等价
enum Color {Red=0, Green=1, Blue=2}
```

当然也可以自己手动赋值:

```
enum Color {Red=1, Green=2, Blue=4}
```

并且我们可以反向通过值得到他的键值:

```
enum Color {Red=1, Green=2, Blue=4}
Color[2] === 'Green' // true
```

看下编译成 js 后的枚举代码, 你就明白为什么可以反向得到键值:

```
var Color;
(function (Color) {
    Color[Color["Red"] = 0] = "Red";
    Color[Color["Green"] = 1] = "Green";
    Color[Color["Blue"] = 2] = "Blue";
})(Color || (Color = {}));
// Color的值为: {0: "Red", 1: "Green", 2: "Blue", Red: 0, Green: 1, Blue: 2}
```

> any(任意类型)

any 代表任意类型, 也就是说, 如果你不清楚变量是什么类型, 就可以用 any 进行标记, 比如引入一些比较老的 js 库, 没有声明类型, 使用的时候就可以标记为 any 类型, 这样 ts 就不会提示错误了. 当然不能所有的地方都用 any, 那样 ts 就没有使用的意义了

```
let obj:any = {};
// ts自己推导不出forEach中给obj增加了'a'和'b'字段.
['a', 'b'].forEach(letter=&gt;{
    obj[letter] = letter;
});

// 但是因为标记了any, 所以ts认为a可能存在
obj.a = 123
```

> void

void 的意义和 any 相反, 表示不是任何类型, 一般出现在函数中, 用来标记函数没有返回值:

```
function abc(n:number):void{
    console.log(n);
}
```

void 类型对应 2 个值, 一个是 undefined,一个 null:

```
const n1:void = undefined;
const n2:void = null;
```

> null 和 undefined
> 默认情况下 null 和 undefined 是所有类型的子类型, 比如:

```
const n1:null = 123;
const n2:undefined = '123';
```

> 如果一个变量的值确实需要是 null 或者 undefined, 可以像下面这么用, ts 会自动根据 if/else 推导出正确类型:

```
// 这是"联合类型", 在"高级类型"中会有详细介绍, 表示num可能是undefined也可能是number
let num: undefined|number;

if(Math.random()&gt;0.5) num = 1;

if(undefined !== num) {
    num++;
}
```

> never

never 表示不可达, 用文字还真不好描述, 主要使用在 throw 的情况下:

```
function error():never{
    throw '错了!';
}
```

> object

object 表示非原始类型, 也就是除 number/ string/ boolean/ symbol/ null/ undefined 之外的类型:

```
let o1:object = [];
let o2:object = {a:1,b:2};
```

**实际上基本不用 object 类型的, 因为他标注的非常不具体, 一般都用接口来标注更具体的对象类型**

### 高级类型

___

通过基础类型组合而来的, 我们可以叫他高级类型. 包含: 交叉类型 / 联合类型 / 接口 ......

#### 接口 (interface)

1. 可选类型 -- 用 '?' 号修饰

- 现在我们发现不是接口中提供的每一个参数我们都需要，为此 'TS' 提供了
		一种 '?' 来修饰，被问号修饰的参数就是可以选传，不是必须参数

```
interface Baseinfo {
    name:string,
    sex?:string
}
// 人 
function printPesonInfo(parmasinfo: Baseinfo) {
    console.log(`姓名：${parmasinfo.name }`)
}

let paramsinfo = {name:'wang'} 
printPesonInfo(paramsinfo) // 姓名：wang
```

1. 索引签名 -- 可以添加不确定参数

- 1.刚才的问号让我们可以选填一些参数，但是如果需要增加一些不确定
		参数就需要使用 ' 索引签名 '
- 2.利用这个可以规定你的索引签名的类型 (即使你的索引规定是 string 类型
		但也可以使用数字，案例三有说明)
- 3.主要注意这里有个问题：需要注意的是，一旦定义了任意属性，
		那么确定属性和可选属性的类型都必须是它的类型的子集

```
interface Baseinfo {
    name:string,
    sex?:string,
    [other:string]:any
}
// 人 
function printPesonInfo(parmasinfo: Baseinfo) {
    console.log(`姓名：${parmasinfo.name }`)
}

// 接口中的索引签名other 就会收到age
printPesonInfo( {name:'wang',age:13}) // wang
```

1. 限制接口中的参数是只读 -- readonly

- 1.readonly 和 const 都是只读，但两者如何用区别在哪里，首先 'readonly '
		是 'TS' 在接口提出，只针对接口中的参数赋值后就禁止更改，而 const 是
		es6 提出的针对变量，不让变量进行更改
- 2.用 'ts' 文档的总结来说: 最简单判断该用 readonly 还是 const 的方法是看要
		把它做为变量使用还是做为一个属性。 做为变量使用的话用 const，若做
		为属性则使用 readonly。

1. 接口 数组 和只读
2. 接口定义函数
3. 接口做闭包 (混合类型)
4. 绕过接口的多余参数检查 -- 三种方式

```
interface Article {
    title: string;
    count: number;
    content:string;
    fromSite?: string; // 非必填
}

// 不会报错
const article: Article = {
    title: '为vue3学点typescript(2), 类型',
    count: 9999,
    content: 'xxx...',
}
```

> 用接口定义函数

接口不仅可以定义对象, 还可以定义函数:

```
// 声明接口
interface Core {
    (n:number, s:string):[number,string]
}

// 声明函数遵循接口定义
const core:Core = (a,b)=&gt;{
    return [a,b];
}
```

> 用接口定义类

```
// 定义
interface Animal {
    head:number;
    body:number;
    foot:number;
    eat(food:string):void;
    say(word:string):string;
}

// implements
class Dog implements Animal{
    head=1;
    body=1;
    foot=1;
    eat(food:string){
        console.log(food);
    }
    say(word:string){
        return word;
    }
}
```

#### 交叉类型 (&)

交叉类型是将多个类型合并为一个类型, 表示 " 并且 " 的关系,用&连接多个类型, 常用于对象合并:

```
interface A {a:number};
interface B {b:string};

const a:A = {a:1};
const b:B = {b:'1'};
const ab:A&amp;B = {...a,...b};
```

#### 联合类型 (|)

联合类型也是将多个类型合并为一个类型, 表示 " 或 " 的关系,用|连接多个类型:

```
function setWidth(el: HTMLElement, width: string | number) {
    el.style.width = 'number' === typeof width ? `${width}px` : width;
}
```

注意: 我这里标记了 el 为 HTMLElement, 可以在 typescript 的仓库看到 ts 还定义了很多元素, 更多 [请查看](https://github.com/microsoft/TypeScript/blob/master/lib/lib.dom.d.ts)

#### " 类型变量 " 和 " 泛型 "

变量的概念我们都知道, 可以表示任意数据, 类型变量也一样, 可以表示任意类型:

**泛型方法**

```
// 在函数名后面用"&lt;&gt;"声明一个类型变量
// 输入T[], 返回T
function convert&lt;T&gt;(input:T):T{
    return input;
}
```

convert 中参数我们标记为类型 T, 返回值也标记为**T**, 从而表示了: 函数的输入和输出的类型一致. 这样使用了 " 类型变量 " 的函数叫做泛型函数, 那有 " 泛型类 " 吗?
**注意:** T 是我随便定义的, 就和变量一样, 名字你可以随便起, 只是建议都是大写字母,比如 U / RESULT.

**泛型类**

```
// 类名后面通过"&lt;&gt;"声明一个类型变量U, 类的方法和属性都可以用这个U
class Person&lt;U&gt; {
    who: U;
    
    constructor(who: U) {
        this.who = who;
    }

    say(code:U): string {
        return this.who + ' :i am ' + code;
    }
}

let a =  new Person&lt;string&gt;('詹姆斯邦德');
a.say(007) // 错误, 会提示参数应该是个string
a.say('007') // 正确

// 自动推断类型变量的类型（不用你标类型了,ts自己猜）

let a =  new Person('詹姆斯邦德');
// 等价 let a =  new Person&lt;string&gt;('詹姆斯邦德');
a.say(007) // 错误, 会提示参数应该是个string
a.say('007') // 正确

ts可以根据实例化时传入的参数的类型推断出U为string类型

// 浏览器自带api
document.ontouchstart = ev=&gt;{ // 能自动推断出ev为TouchEvent
    console.log(ev.touches);  // 不报错, TouchEvent上有touches属性
}

```

### 泛型是什么?

___

- 用动态的类型 (类型变量) 描述函数和类的方式

### 泛型类型

___

- 用类型变量去描述一个类型 (类型范围), ts 的数组类型 Array 本身就是一个泛型类型, 他需要传递具体的类型才能变的精准:

```
let arr : Array&lt;number&gt;;
arr = ['123']; // 错误, 提示数组中只可以有number类型
arr = [123];
```

- 定义一个泛型类型

```
function convert&lt;T&gt;(input:T):T{
    return input;
}

// 定义泛型类型
interface Convert {
    &lt;T&gt;(input:T):T
}

// 验证下
let convert2:Convert = convert // 正确不报错
```

### 泛型接口

___

- 泛型接口

```
interface Goods&lt;T&gt;{
    id:number;
    title: string;
    size: T;
}

let apple:Goods&lt;string&gt; = {id:1,title: '苹果', size: 'large'};
let shoes:Goods&lt;number&gt; = {id:1,title: '苹果', size: 43};
```

### 扩展类型变量 (泛型约束)

___

```
function echo&lt;T&gt;(input: T): T {
    console.log(input.name); // 报错, T上不确定是否有name属性
    return input;
}

```

- 前面说过 T 可以代表任意类型, 但对应的都是基础类型, 所以当我们操作 input.name 的时候就需要标记 input 上有 name 属性, 这样就相当于我们缩小了类型变量的范围, 对泛型进行了约束:

```
// 现在T是个有name属性的类型
function echo&lt;T extends {name:string}&gt;(input: T): T {
    console.log(input.name); // 正确
    return input;
}
```

### 一个泛型的应用, 工厂函数

___

```
function create&lt;T,U&gt;(O: {new(): T|U; }): T|U {
    return new O();
}
```

- 可以定义多个类型变量.
- 类型变量和普通类型用法一直, 也支持联合类型/交叉类型等类型.
- 如果一个数据是可以实例化的, 我们可以用{new(): any}表示.

### 不要乱用泛型

> 泛型主要是为了约束, 或者说缩小类型范围, 如果不能约束功能, 就代表不需要用泛型:

```
function convert&lt;T&gt;(input:T[]):number{
    return input.length;
}
```

**这样用泛型就没有什么意义了, 和 any 类型没有什么区别.**

### 类型断言 (你告诉 ts 是什么类型, 他都信)

___

> 断言有 2 种语法, 一种是通过 "<>", 一种通过 "as", 举例说明:

```
let obj = 0.5 &lt; Math.random() ? 1 : [1]; // number|number[]

// 断言, 告诉ts, obj为数组
(&lt;number[]&gt;obj).push(1);

//等价
(obj as number[]).push(1);

```

### 类型别名 (type)

> 类型别名可以表示很多接口表示不了的类型, 比如字面量类型 (常用来校验取值范围):

```
type A = 'top'|'right'|'bottom'|'left'; // 表示值可能是其中的任意一个
type B = 1|2|3;
type C = '红'|'绿'|'黄';
type D = 150;

let a:A = 'none'; // 错误, A类型中没有'none'
```

**更多组合:**

```
interface A1{
    a:number;
}
type B = A1 | {b:string};
type C = A1 &amp; {b:string};

// 与泛型组合
type D&lt;T&gt; = A1 | T[];
```

### 索引类型 (keyof)

> 和 js 中的 Object.keys 类似， 用来获取对象类型的键值:

```
type A = keyof {a:1,b:'123'} // 'a'|'b'
type B = keyof [1,2] // '0'|'1'|'push'... , 获取到内容的同时, 还得到了Array原型上的方法和属性(实战中暂时没遇到这种需求, 了解即可)
```

可以获得键值, 也可以获取对象类型的值的类型:

```
type A = {a:1,b:'123'};
type C = A['a'] // 等于type C = 1;
let c:C = 2 // 错误, 值只能是1
```

### 映射类型 (Readonly, Pick, Record 等...)

> 映射类型比较像修改类型的工具函数, 比如 Readonly 可以把每个属性都变成只读:

```
type A  = {a:number, b:string}
type A1 = Readonly&lt;A&gt; // {readonly a: number;readonly b: string;}
```

### extends(条件类型)

___

用来表示类型是不确定的, 如果 U 的类型可以表示 T, 那么返回 X, 否则 Y. 举几个例子:

```
type A =  string extends '123' ? string :'123' // '123'
type B =  '123' extends string ? string :123 // string
```

### infer(类型推断)

___

单词本身的意思是 " 推断 ", 实际表示在 extends 条件语句中声明待推断的类型变量. 我们上面介绍的映射类型中就有很多都是 ts 在 lib.d.ts 中实现的, 比如 Parameters:

```
type Parameters&lt;T extends (...args: any) =&gt; any&gt; = T extends (...args: infer P) =&gt; any ? P : never;

```

**应用 infer**

> 利用 infer 来实现 " 删除元祖类型中第一个元素 ", 这常用于简化函数参数

```
export type Tail&lt;Tuple extends any[]&gt; = ((...args: Tuple) =&gt; void) extends ((a: any, ...args: infer T) =&gt; void) ? T : never;
```
