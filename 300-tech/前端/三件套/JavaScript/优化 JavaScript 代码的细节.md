---
title: 优化 JavaScript 代码的细节
date: 2024-08-22T01:48:46+08:00
updated: 2024-08-22T01:49:27+08:00
dg-publish: false
source: https://github.com/peng-yin/note/issues/5
---

## 一些原则

1. 多使用组合而不是继承
2. 命名：

- 名副其实，
- 常量 const 记录
- 类名应该是名词，方法名应该是动词

1. 函数

- 函数只做一件事
- 形参不超过三个，多了就使用对象参数。
- 不使用布尔值来作为参数，遇到这种情况时，一定可以拆分函数。

```
 // bad
 function createFile(name, temp) {
   if (temp) {
     fs.create(`./temp/${name}`);
   } else {
     fs.create(name);
   }
 }

 // good
 function createFile(name) {
   fs.create(name);
 }

 function createTempFile(name) {
   createFile(`./temp/${name}`);
 }

```

1. 避免副作用: 集中副作用：遇到不可避免的副作用时候，比如读写文件、上报日志，那就在一个地方集中处理副作用，不要在多个函数和类处理副作用。
2. 其它注意的地方：

- 常见就是陷阱就是对象之间共享了状态，使用了可变的数据类型，比如对象和数组。对于可变的数据类型，使用 immutable 等库来高效克隆。
- 避免用可变的全局变量。

```
// bad：注意到cart是引用类型！
const addItemToCart = (cart, item) =&gt; {
  cart.push({ item, date: Date.now() });
};

// good
const addItemToCart = (cart, item) =&gt; {
  return [...cart, { item, date: Date.now() }];
};
```

1. 封装复杂的判断条件，提高可读性。
2. 在方法中有多条件判断时候，为了提高函数的可扩展性，考虑下是不是可以使用能否使用多态性来解决。
3. 错误处理

- 不要忽略捕获的错误。而要充分对错误做出反应，比如 console.error() 到控制台，提交错误日志，提醒用户等操作

1. 入乡随俗: 形成统一的代码风格也是一种代码整洁。

- 不要漏了 catch promise 中的 reject。

## 1.处理 if 语句中对单个变量值的多次逻辑运算

> 没有优化过的代码如下：

```
function test(param) {
    if (param === 1 || param === 2 || param === 3) {
        // ...
    }
}
```

> 使用 includes 、Array.some 实现

```
function test(param) {
    let arr = [1, 2, 3];
    if (arr.includes(param)) {
        // ...
    }
}
```

```
function test(param) {
    let arr = [1, 2, 3];
    arr.some(item =&gt; {
        if (item === param) {
            // ...
        }
    });
}
```

## 2\. 用 return 提前返回，减少 if 嵌套

> 在开发中，我们可能会遇到这个代码场景，需要在第一层 if 中对某个值进行判断其是否符合条件，然后在该 if 判断语句中再进行其他的逻辑判断，这样就增加了 if 的嵌套层数：

```
function test(arr, param) {
    if (arr.length &gt; 0) {
        if (param === 1) {
            // ...
        }
    }
}
```

> 其实我们完全将第一层 if 判断抽离到顶部，在判断不符合条件就马上 return

```
function test(arr, param) {
    if (arr.length === 0) {
        return;
    }
    if (param === 1) {
        // ...
    }
}
```

## 3\. 使用 HOC, 为一个函数增加额外的功能 curry

```js
const func = (param) => {
  console.log("func", param);
};

const wrap = (param, func) => {
  console.log("wrap");
  return func(param);
};

// wrap 逻辑已被复用
wrap("1111", func);
```

## 4.使用 every/some 处理全部/部分满足条件

> 在处理全部/部分满足的条件时，我们通常可以使用 every/some 来进行处理：

```
const fruits = [
    { name: 'apple', color: 'red' },
    { name: 'banana', color: 'yellow' },
    { name: 'grape', color: 'purple' }
];

function test() {
  // 简短方式，所有的水果都必须是红色
  const isAllRed = fruits.every(f =&gt; f.color == 'red');
  console.log(isAllRed); // false
}
```

## 5.删除冗余的 if...else if...if 语句

```js
switch(a) {
   case 1:
    break;
   case 2:
    break;
   case 3:
    break;
  default:
    break;
}

let handler = {
  1: () => {},
  2: () => {},
  3: () => {},
  default: () => {}
}

handler[a]() || handler['default']()

// better👍

const functionA = ()=>{/*do Something*/}       // 单独业务逻辑
const functionB = ()=>{/*do Something*/}       // 单独业务逻辑
const functionC = ()=>{/*do default  */}       // 公共业务逻辑

const actions = new Map([
    ['guest_1', () => { functionA }],
    ['guest_2', () => {  functionB }],
    ['default', () => { functionC  }],
    //...
])

/**
  * @param {string} identity 身份标识：guest客态 master主态
  * @param {number} status 活动状态：1开票中 2开票失败 3 开票成功 4 商品售罄 5 有库存未开团
 */

const onButtonClick = (identity, status) => {
  let action = actions.get(`${identity}_${status}`) || actions.get('default')
  action.call(this)
}
```

## 6\. 替换数组中的特定值

可以使用.splice(start、value to remove、valueToAdd)，这些参数指定希望从哪里开始修改、修改多少个值和替换新值。

```js
var userNames = ['tom1', 'tom2', 'tom3', 'tom4', 'tom5']

userNames.splice(0, 2, 'jack1', 'jack2')

console.log(userNames)
```

## 7\. Array.from 达到 .map 的效果

```
var userList = [
  {name: 'tom1', age: 1},
  {name: 'tom2', age: 1},
  {name: 'tom3', age: 1},
  {name: 'tom4', age: 1},
]

var userName = Array.from(userList, ({ name }) =&gt; name)
console.log(userName)
```

## 7\. 将数组转换为对象

```
var arr = ['tom1', 'tom2', 'tom3', 'tom4', 'tom5']

var obj = {...arr}

console.log(obj)

```

## 8\. 分组 groupBy

```js
var words = ['one', 'two', 'three'];

// 返回 {'3': ['one', 'two'], '5': ['three']}
groupBy(words, 'length');

var groupBy = function (arr, criteria) {
  return arr.reduce(function (obj, item) {

    // 判断criteria是函数还是属性名
    var key = typeof criteria === 'function' ? criteria(item) : item[criteria];

    // 如果属性不存在，则创建一个
    if (!obj.hasOwnProperty(key)) {
      obj[key] = [];
    }

    // 将元素加入数组
    obj[key].push(item);

    // 返回这个对象作为下一次循环的accumulator
    return obj;

  }, {});
};
```

## 9\. 策略模式

维基百科上说：策略模式作为一种软件设计模式，指对象有某个行为，但是在不同的场景中，该行为有不同的实现算法

```js
const ROLES = {
  GUEST: 0,
  USER: 1,
  ADMIN: 2
};
const userRole = await getUserRole();
if (userRole === ROLES.GUEST) {
} else if (userRole === ROLES.USER) {
} else if (userRole === ROLES.ADMIN) {};
```

👍better

```js
const ROLES = {
  GUEST: 0,
  USER: 1,
  ADMIN: 2
};
const ROLE_METHODS = {
  [ROLES.GUEST]() {},
  [ROLES.USER]() {},
  [ROLES.ADMIN]() {},
};
const userRole = await getUserRole();
ROLE_METHODS[userRole]();
```

当需要增加角色，或者修改角色数字的时候，只需要修改 ROLES 里对应的字段，以及 ROLE\_METHODS 里的方法即可，这样就可以将可能很冗长的 if...else 代码给抽离出来，颗粒度更细，更好维护。

## 10\. 函数参数解构

> 函数传参越少越好，多了改为对象传入

```js
// bad
doSomething({ foo: 'Hello', bar: 'Hey!', baz: 42 });
function doSomething(config) {
  const foo = config.foo !== undefined ? config.foo : 'Hi';
  const bar = config.bar !== undefined ? config.bar : 'Yo!';
  const baz = config.baz !== undefined ? config.baz : 13;
  // ...
}
// good
// 对象解构， 配置默认参数
function doSomething({ foo = 'Hi', bar = 'Yo!', baz = 13 }) {
// ...
}

// 你需要使配置对象也可选
function doSomething({ foo = 'Hi', bar = 'Yo!', baz = 13 } = {}) {
// ...
}
```

## 11\. 短路求值

> 与（&&）运算符将会返回第一个 false/‘falsy’的值。当所有的操作数都是 true 时，将返回最后一个表达式的结果。

```js
let one = 1, two = 2, three = 3;
console.log(one && two && three); // Result: 3
console.log(0 && null); // Result: 0
```

> 或（||）运算符将返回第一个 true/‘truthy’的值。当所有的操作数都是 false 时，将返回最后一个表达式的结果。

```js
let one = 1, two = 2, three = 3;
console.log(one || two || three); // Result: 1
console.log(0 || null); // Result: null
```

## 12\. 解构组合新对象

> 拆封对象

```js
const rawUser = {
   name: 'John',
   surname: 'Doe',
   email: 'john@doe.com',
   displayName: 'SuperCoolJohn',
   joined: '2016-05-05',
   image: 'path-to-the-image',
   followers: 45
}

let user = {}, userDetails = {};
({ name: user.name, surname: user.surname, ...userDetails } = rawUser);
```

> 使用解构赋值将对象的某个 key 值改变，其余不变，重新组合新对象

```js
const obj = {
  name: 'lisa',
  age: 25,
  phone: 17610638800,
  company: 'alibaba'
}

const { name: nickname, ...props } = obj

const newObj = {
  nickname,
  ...props
}
console.log(newObj)
```

> 从含有众多字段的一个对象中取出部分字段，来组成新的对象

```js
const pick = (...props) =>
  ob => props.reduce((other, e) => ob[e] !== undefined ? { ...other, [e]: ob[e] } : other, {});
```

这个 pick 方法是一个柯里化的函数，在向其中传入所需字段后，返回的函数可以将入参对象的对应字段提取到新的对象中，并过滤由于入参对象未定义 key 产生的 undefined 值。

调用方法如下：

```js
const source = { a: 1, b: 2, c: 3 };
const result = pick('a', 'c', 'd')(source);  // { a: 1, c: 3 }
```

## 13\. 判断一个空对象

```js
let obj1 = {
    name: 'oli',
    child: {
        name: 'oliver'
    }
}

let obj2 = {
    [Symbol('name')]: 'alice'
}

let obj3 = Object.defineProperty({}, 'name', {
    value: 'alice',
    enumerable: false
})

let obj4 = Object.create(null)

// 我们需要一个函数，判断是否不含自有属性

isEmpty(obj1) // false
isEmpty(obj2) // false
isEmpty(obj3) // false
isEmpty(obj4) // true
```

```js
方法1:

const isEmptyObj = object => {
    if (!!Object.getOwnPropertySymbols(object).length) {
        return false
    }
    for (const key in object) {
        if (object.hasOwnProperty(key)) {
            return false
        }
    }
    return true
}

方法二：keys 方法

const isEmptyObj = object => {
    if (!!Object.getOwnPropertySymbols(object).length) {
        return false
    }
    if (Object.keys(object).length) {
        return false
    }
    return true
}

方法三：JSON 方法

const isEmptyObj = object => {
    if (!!Object.getOwnPropertySymbols(object).length) {
        return false
    }
    return JSON.stringify(object) === '{}'
}

方法四：getOwnPropertyNames 方法

const isEmptyObj = object => {
    if (!!Object.getOwnPropertySymbols(object).length) {
        return false
    }
    if (!!Object.getOwnPropertyNames(object).length) {
        return false
    }
    return true
}
```

## 14\. js 快速生成数组序列

```js
const fn = length => Array.from({ length }).map((v, k) => k)

const fn = length => [...new Array(length).keys()]
```

## 15\. 通用验证函数

```js
const schema = {
  first: {
    required: true,
  },
  last : {
    required: true,
  }
}

const validate = (schema, values) => {
  for(fileld in schema) {
    if(schema[fileld].required) {
      if(!values[fileld]) {
        return false
      }
    }
  }
  return true
}

console.log(validate(schema, { first: 'Lisa' , last: 'Tom' }))
```

## 15\. 👍 map

```js
const commodity = new Map([
  ['phone', 9999],
  ['computer', 15999],
  ['television', 2999],
  ['gameBoy', 2599]
])

const price = (name) => {
  return commodity.get(name)
}
price('phone') // 9999

const commodity = new Map([
  [{name: 'phone', date: '11-11'}, () => {
    console.log(9999)
    // do Something
  }],
  [{name: 'phone', date: '12-12'}, () => {
    console.log(9888)
    // do Something
  }],
  [{name: 'phone', date: '06-18'}, () => {
    console.log(9799)
    // do Something
  }],
  [{name: 'phone', date: '09-09'}, () => {
    console.log(9699)
    // do Something
  }],
]);
const showPrice = (name, date) => {
  [...commodity].forEach(([key,value])=> {
    key.name === name && key.date === date ? value.call() : ''
  })
}
showPrice('phone', '12-12')

const CheckPrice = () => {
  const showMessage = () => {
    console.log('打开活动页面')
  }
  return new Map ([
    [{name: 'phone', date: '11-11'}, () => {
      console.log(9999)
      showMessage()
      // do Something
    }],
    [{name: 'phone', date: '12-12'}, showMessage],
    [{name: 'phone', date: '06-18'}, () => {
      console.log(9799)
      // do Something
    }],
    [{name: 'phone', date: '09-09'}, () => {
      console.log(9699)
      // do Something
    }],
  ])
}
const showPrice = (name, date) => {
  [...CheckPrice()].forEach(([key,value])=> {
    key.name === name && key.date === date ? value.call() : ''
  })
}
showPrice('phone', '11-11')

const CheckPrice = () => {
  const showMessage = () => {
    console.log('打开活动页面')
  }
  return new Map ([
   [/^phone_[0-4]$/, () => {
     console.log('活动时间价格是8888')
   }],
   [/^phone_[5-9]*$/, () => {
     console.log('其他时间10000')
   }],
   [/^phone_.*$/, () => {
    showMessage()
   }],
  ])
}
const showPrice = (name, date) => {
  [...CheckPrice()].forEach(([key,value])=> {
    key.test(`${name}_${date}`) ? value.call() : ''
  })
}
let date = [{date: '11-11', key: 1}, {date: '12-28', key: 9}]
let target = date.find((item, index) => item.date === '12-28')
showPrice('phone', target.key)
```

## 16\. 如何平滑滚动到页面顶部

```js
const scrollToTop = () => {
  const c = document.documentElement.scrollTop || document.body.scrollTop;
  if (c > 0) {
    window.requestAnimationFrame(scrollToTop);
    window.scrollTo(0, c - c / 8);
  }
}

// 事例
scrollToTop()
```

## 17\. 功能性集合

```js
const calc = {
    add : function(a,b) {
        return a + b;
    },
    sub : function(a,b) {
        return a - b;
    },
    mult : function(a,b) {
        return a * b;
    },
    div : function(a,b) {
        return a / b;
    },
    run: function(fn, a, b) {
        return fn && fn(a,b);
    }
}

calc.run(calc.mult, 7, 4); //28
```

## 几个优雅的运算符使用技巧

> 可选链接运算符【？.】

```js
// 操作符一旦为空值就会终止执行

object?.property // 对于静态属性用法

object?.[expression]  // 对于动态属性

object.fun?.() // 对于方法
```

> 空值合并运算符 「??」

??与||的功能是相似的，区别在于 ??在左侧表达式结果为 null 或者 undefined 时，才会返回右侧表达式

```js
// 与无效合并一起使用

let title = data?.children?.[0]?.title ?? 'jack';
console.log(title); //jack
```

> 逻辑空分配（?? =）

```js
// 逻辑空值运算符仅在空值（空值或未定义undefined）时才将值分配给x
x ??= y

x = x ?? y;
```

> 逻辑或分配（|| =）

```js
// 此逻辑赋值运算符仅在左侧表达式为 falsy值时才赋值

x ||= y

x || (x = y)
```

> 逻辑与分配（&& =）

```js
// 逻辑赋值运算符仅在左侧为真时才赋值
x &&= y

x && (x = y)
```

## 数据类型检测

```js
Object.prototype.toString.call('string');       //"[object String]"
Object.prototype.toString.call(1111);           //"[object Number]"
Object.prototype.toString.call(true);           //"[object Boolean]"
Object.prototype.toString.call(null);           //"[object Null]"
Object.prototype.toString.call(undefined);      //"[object Undefined]"
Object.prototype.toString.call(Symbol('111'));  //"[object Symbol]"
Object.prototype.toString.call({});             //"[object Object]"
```

> 1.2 比较对象的构造函数与类的构造函数是否相等

```js
var a = {}
a.constructor === Object     // true

var b = '111';
b.constructor === String    // true

var strN = new String('11111');
strN.constructor === String // true

var c = true;
c.constructor === Boolean   // true

var d = Symbol('symbol')
d.constructor === Symbol    // true
```

> 1.2 propotype
> 比较对象的原型与构造函数的原型是否相等

```js
var a = {}
a.__proto__ === Object.prototype     // true

var t = new Date();
t.__proto__ === Date.prototype      // true

var str = 'sting';
str.__proto__ === String.prototype  // true

var strN = new String('11111');
strN.__proto__ === String.prototype // true
```

## 数据特殊操作

> 交换两个值

1. 利用一个数异或本身等于 0 和异或运算符合交换率

```js
var a = 3;
var b = 4
a ^= b; // a = a ^ b
b ^= a;
a ^= b;

console.log(a, b);
```

1. 小数取整

```js
var num = 123.123

// 常用方法
console.log(parseInt(num)); // 123
// “双按位非”操作符
console.log(~~ num); // 123
// 按位或
console.log(num | 0); // 123
// 按位异或
console.log(num ^ 0); // 123
// 左移操作符
console.log(num << 0); // 123
```

## 数字金额千分位格式化

1. 使用 Number.prototype.toLocaleString()

```js
var num = 123455678;
var num1 = 123455678.12345;

var formatNum = num.toLocaleString('en-US');
var formatNum1 = num1.toLocaleString('en-US');

console.log(formatNum); // 123,455,678
console.log(formatNum1); // 123,455,678.123
```

1. 使用正则表达式

```js
var num = 123455678;
var num1 = 123455678.12345;

var formatNum = String(num).replace(/\B(?=(\d{3})+(?!\d))/g, ',');
var formatNum1 = String(num1).replace(/\B(?=(\d{3})+(?!\d))/g, ',');

console.log(formatNum); // 123,455,678
console.log(formatNum1); // 123,455,678.12,345
```

## 设计模式

```ts
/* 从多个函数取结果 */

function mount () {
  console.log('mount');
  return 'b'
}

function unmount () {
  console.log('unmount');
  return 'a'
}

const funs = [
  () => mount(),
  () => unmount()
]

console.log(funs.map(patch => patch()));

// ========================== 封装判断条件 =========================

function canActivateService(subscription, account) {
  return subscription.isTrial || account.balance > 0
}

if (canActivateService(subscription, account)) {

}

// ========================== 避免判断条件 =========================

class A {
  getA() {}
  getB() {}
}

class B extends A {
  getExcuteA() {
    return this.getA()
  }
}

class C extends A {
  getExcuteA() {
    return this.getB()
  }
}

/* 函数只做一件事 */
console.log(new B().getExcuteA());

// ========================== 对象和数据结构 =========================

/* 
  使用 getter 和 setter 从对象中访问数据比简单地在对象上查找属性要好。原因如下:

  当需要在获取对象属性之前做一些事情时，不必在代码中查找并修改每个访问器。
  执行set时添加验证更简单。
  封装内部表示。
  更容易添加日志和错误处理。
  可以延迟加载对象的属性，比如从服务器获取它。
*/

class BankAccount {
  accountBalance = 0

  get balance() {
    return this.accountBalance
  } 

  set balance(value) {
    if (value < 0) {
      throw new Error('Cannot set negative balance.');
    }
    this.accountBalance = value;
  }

  // ....
}

const account = new BankAccount();

account.balance = 100;

// ========================== 高内聚低耦合 =======================================================================

/* 
* 内聚：定义了类成员之间相互关联的程度。理想情况下，高内聚类的每个方法都应该使用类中的所有字段，实际上这不可能也不可取。但我们依然提倡高内聚。
* 耦合：指的是两个类之间的关联程度。如果其中一个类的更改不影响另一个类，则称为低耦合类。
*/

class UserService {
  constructor(private readonly db: Database) {

  }

  async getUser(id: number): Promise<User> {
    return await db.users.findOne({ id })
  }

  async getTransactions(userId: number): Promise<Transaction[]> {
    return await db.transactions.find({ userId })
  }
}

class UserNotifier {
  constructor(private readonly emailSender: EmailSender) {

  }

  async sendGreeting(): Promise<void> {
    await emailSender.send('Welcome!');
  }

  async sendNotification(text: string): Promise<void> {
    await emailSender.send(text);
  }

  async sendNewsletter(): Promise<void> {
    // ...
  }
}

// ========================== 组合大于继承 =======================================================================

/* 
  什么时候应该使用继承？这取决于你面临的问题。以下场景使用继承更好:
* 继承代表的是“is-a”关系，而不是“has-a”关系 (人 -> 动物 vs. 用户 -> 用户详情)。
* 可复用基类的代码 (人类可以像所有动物一样移动)。
* 希望通过更改基类对派生类进行全局更改(改变所有动物在运动时的热量消耗)。
*/

class Employee {
  private taxData: EmployeeTaxData;

  constructor(
    private readonly name: string, 
    private readonly email:string) {
  }

  setTaxData(ssn: string, salary: number): Employee {
    this.taxData = new EmployeeTaxData(ssn, salary);
    return this;
  }
  // ...
}

class EmployeeTaxData {
  constructor(
    public readonly ssn: string, 
    public readonly salary: number) {
  }
  // ...
}

// ========================== 使用方法链 =======================================================================
class QueryBuilder {
  private collection: string;
  private pageNumber: number = 1;
  private itemsPerPage: number = 100;
  private orderByFields: string[] = [];

  from(collection: string): this {
    this.collection = collection;
    return this;
  }

  page(number: number, itemsPerPage: number = 100): this {
    this.pageNumber = number;
    this.itemsPerPage = itemsPerPage;
    return this;
  }

  orderBy(...fields: string[]): this {
    this.orderByFields = fields;
    return this;
  }

  build(): Query {
    // ...
  }
}

// ...

const query = new QueryBuilder()
  .from('users')
  .page(1, 100)
  .orderBy('firstName', 'lastName')
  .build();

//  ========================== SOLID原则 =======================================================================

/* 
* 单一职责原则 (SRP)
* 减少修改类的次数
* 如果一个类功能太多，修改了其中一处很难确定对代码库中其他依赖模块的影响
*/

class UserAuth {
  constructor(private readonly user: User) {
    
  }
  verifyCredentials() {

  }
}

class UserSettings {
  private readonly auth: UserAuth
  constructor(private readonly user: User) {
    this.auth = new UserAuth(user)
  }

  changeSettings(settings: UserSettings) {
    if(this.auth.verifyCredentials())
  }
}

/* 
* 开闭原则 (OCP)
* “软件实体(类、模块、函数等)应该对扩展开放，对修改封闭。” 换句话说，
* 就是允许在不更改现有代码的情况下添加新功能。
*/
abstract class Adapter {
  abstract async request<T>(url: string): Promise<T>;
}

class AjaxAdapter extends Adapter{
  constructor() {
    super()
  }

  async request<T>(url: string): Promise<T> {
    // request and return promise
  }
}

class NodeAdapter extends Adapter{
  constructor() {
    super()
  }

  async request<T>(url: string): Promise<T> {
    // request and return promise
  }
}

class HttpRequester {
  constructor(private readonly adapter: Adapter) {
  }

  async fetch<T>(url: string): Promise<T> {
    const response = await this.adapter.request<T>(url);
    // transform response and return
  }
}

new HttpRequester()

/* 
* 里氏替换原则 (LSP)
*/

abstract class Shape {
  setColor(color: string) {
    // ...
  }

  render(area: number) {
    // ...
    console.log(area);
  }

  abstract getArea(): number;
}

class Rectangle extends Shape {
  constructor(
    private readonly width = 0, 
    private readonly height = 0) {
    super();
  }

  getArea(): number {
    return this.width * this.height;
  }
}

class Square extends Shape {
  constructor(private readonly length: number) {
    super();
  }

  getArea(): number {
    return this.length * this.length;
  }
}

function renderLargeShapes(shapes: Shape[]) {
  shapes.forEach((shape) => {
    const area = shape.getArea();
    shape.render(area);
  });
}

const shapes = [new Rectangle(4, 5), new Rectangle(4, 5), new Square(5)];

renderLargeShapes(shapes);

/* 
* 接口隔离原则 (ISP)
* “客户不应该被迫依赖于他们不使用的接口。” 这一原则与单一责任原则密切相关。
* 不应该设计一个大而全的抽象，否则会增加客户的负担，因为他们需要实现一些不需要的方法
*/

interface IPrinter {
  print();
}

interface IFax {
  fax();
}

interface IScanner {
  scan();
}

class AllInOnePrinter implements IPrinter, IFax, IScanner {
  print() {
    // ...
  }  

  fax() {
    // ...
  }

  scan() {
    // ...
  }
}

class EconomicPrinter implements IPrinter {
  print() {
    // ...
  }
}

/* 
* 依赖反转原则(Dependency Inversion Principle)
* 这个原则有两个要点:
* 高层模块不应该依赖于低层模块，两者都应该依赖于抽象。
* 抽象不依赖实现，实现应依赖抽象。
* 这样做的一个巨大好处是减少了模块之间的耦合。耦合非常糟糕，它让代码难以重构。
*/

import { readFile as readFileCb } from 'fs';
import { promisify } from 'util';

const readFile = promisify(readFileCb);

type ReportData = {
  // ..
}

interface Formatter {
  parse<T>(content: string): T;
}

class XmlFormatter implements Formatter {
  parse<T>(content: string): T {
    // Converts an XML string to an object T
  }
}

class JsonFormatter implements Formatter {
  parse<T>(content: string): T {
    // Converts a JSON string to an object T
  }
}

class ReportReader {
  constructor(private readonly formatter: Formatter){

  }

  async read(path: string): Promise<ReportData> {
    const text = await readFile(path, 'UTF8');
    return this.formatter.parse<ReportData>(text);
  }
}

// ...

const reader = new ReportReader(new XmlFormatter());

await report = await reader.read('report.xml');

// or if we had to read a json report:

const reader = new ReportReader(new JsonFormatter());

await report = await reader.read('report.json');

/* 
* 错误处理
* 它表示着运行时已经成功识别出程序中的错误，通过停止当前堆栈上的函数执行，终止进程(在Node.js)，以及在控制台中打印堆栈信息来让你知晓。
* 建议使用 Error 类型的 throw 语法。因为你的错误可能在写有 catch语法的高级代码中被捕获。
* 使用 Error 类型的好处是 try/catch/finally 语法支持它，并且隐式地所有错误都具有 stack 属性，该属性对于调试非常有用。
*/
function calculateTotal(items: Item[]): number {
  throw new Error('Not implemented.');
}

function get(): Promise<Item[]> {
  return Promise.reject(new Error('Not implemented.'));
}

// or equivalent to:

async function get(): Promise<Item[]> {
  throw new Error('Not implemented.');
}

import { logger } from './logging'

try {
  get();
} catch (error) {
  logger.log(error);
}

/* 
* 调用函数的函数和被调函数应靠近放置
*/
class PerformanceReview {
  constructor(private readonly employee: Employee) {

  }

  review() {
    this.getPeerReviews();
    this.getManagerReview();
    this.getSelfReview();

    // ...
  }

  private getPeerReviews() {
    const peers = this.lookupPeers();
    // ...
  }

  private lookupPeers() {
    return db.lookup(this.employee.id, 'peers');
  }

  private getManagerReview() {
    const manager = this.lookupManager();
  }

  private lookupManager() {
    return db.lookup(this.employee, 'manager');
  } 

  private getSelfReview() {
    // ...
  }
}

const review = new PerformanceReview(employee);

review.review();

// ========================== simple enhancer =========================

const sayName = (name) => {
  console.log(name);
}

const sayAge = (age) => {
  console.log(age)
}

// 增强一个函数
const manage = (name, fn = sayName) => {
  fn(name)
}

manage('Lisa')
manage(1, sayAge)

// ========================== nice enhancer =========================

// 针对某个函数或者组件, 在符合条件的情况下, 增加对应的功能

const withSay = ({uuid, name}) => {
  const text = `${uuid}-${name}`
  return function(fn) {
    fn(text)
  }
}

const withWork = ({uuid, name}) => {
  const text = `${uuid}-${name}-weber`
  return function(fn) {
    fn(text)
  }
}

const compose = (...funcs) => funcs.reduce((a, b) => (...args) => a(b(...args)),(arg) => arg)

const middleware = [
  {
    hoc: withSay,
    filter(uuid) {
      return uuid === 1
    }
  },
  {
    hoc: withWork,
    filter(uuid) {
      return uuid === 2
    }
  }
]

const person = [
  {
    uuid: 1,
    name: 'Lisa',
    action: (text) => {
      console.log(text, '1');
    }
  },
  {
    uuid: 2,
    name: 'jack',
    action: (text) => {
      console.log(text, '2');
    }
  }
]

for (const p of person) {
  const { uuid, name, action } = p
  const map = middleware.reduce((prev, item) => {
    const { hoc, filter } = item
    if(filter(uuid)) {
      prev.push(
        hoc({
          uuid,
          name
        })
      )
    }
    return prev
  }, [])
  compose(...map)(action)
}

// 分支优化
const describeForNameMap = [
  [
      (name) => name.length > 3, // 判断条件
      (params) => `${params}-output1` // 执行函数
  ],
  [
      (name) => name.length < 2, 
      (params) => `${params}-output2`
  ]
];
  
function getUserDescribe(name) {
  const getDescribe = describeForNameMap.find((item) => item[0](name));

  return getDescribe ? getDescribe[1] : console.log("此人比较神秘！");
}

console.log(getUserDescribe('aaaaaa')(1111));

const DISPOSAL_METHOD_MAP = {
  standard_algorithm_evangelist: () => {},
  standard_UX_evangelist: () => {},
  standard_algorithm_follower: () => {},
  standard_UX_follower: () => {},
  proEdition_algorithm_evangelist: () => {},
  proEdition_UX_evangelist: () => {},
  proEdition_algorithm_follower: () => {},
  proEdition_UX_follower: () => {},
  normal_algorithm_evangelist: () => {},
  normal_UX_evangelist: () => {},
  normal_algorithm_follower: () => {},
  normal_UX_follower: () => {}
}
const disposal_method =
  DISPOSAL_METHOD_MAP[[user.vipType, user.groupType, user.role].join('_')]
  
try {
  disposal_method()
} catch (error) {}
```

## 随机生成字母和数组的组合

```
Math.random().toString(36).substr(2)
```

## 在方法中有多条件判断时候，为了提高函数的可扩展性，考虑下是不是可以使用能否使用多态性来解决

```
 // 地图接口可能来自百度，也可能来自谷歌
 const googleMap = {
  show: function (size) {
      console.log('开始渲染谷歌地图', size);
  }
};
const baiduMap = {
  render: function (size) {
      console.log('开始渲染百度地图', size);
  }
};

 // bad: 出现多个条件分支。如果要加一个腾讯地图，就又要改动renderMap函数。
 function renderMap(type) {
     const size = getSize();
     if (type === 'google') {
         googleMap.show(size);
     } else if (type === 'baidu') {
         baiduMap.render(size);
     }
 };
 renderMap('google')

 // good：实现多态处理。如果要加一个腾讯地图，不需要改动renderMap函数。
 // 细节：函数作为一等对象的语言中，作为参数传递也会返回不同的执行结果，也是“多态性”的体现。
function renderMap (renderMapFromApi) {
  const size = 'aaaa';
  renderMapFromApi(size);
}
renderMap((size) =&gt; googleMap.show(size));
```

## 对象

多使用 getter 和 setter（getXXX 和 setXXX）。好处：

- 在 set 时方便验证。
- 可以添加埋点，和错误处理。
- 可以延时加载对象的属性。

```
// good
function makeBankAccount() {
  let balance = 0;

  function getBalance() {
    return balance;
  }

  function setBalance(amount) {
    balance = amount;
  }

  return {
    getBalance,
    setBalance
  };
}

const account = makeBankAccount();
account.setBalance(100);
```

## 类

### solid

1. 单一职责原则 (SRP) - 保证“每次改动只有一个修改理由”。因为如果一个类中有太多功能并且您修改了其中的一部分，则很难预期改动对其他功能的影响。

```
// bad：设置操作和验证权限放在一起了
class UserSettings {
  constructor(user) {
    this.user = user;
  }

  changeSettings(settings) {
    if (this.verifyCredentials()) {
      // ...
    }
  }

  verifyCredentials() {
    // ...
  }
}
// good: 拆出验证权限的类
class UserAuth {
  constructor(user) {
    this.user = user;
  }

  verifyCredentials() {
    // ...
  }
}

class UserSettings {
  constructor(user) {
    this.user = user;
    this.auth = new UserAuth(user);
  }

  changeSettings(settings) {
    if (this.auth.verifyCredentials()) {
      // ...
    }
  }
}
```

1. 开闭原则 (OCP) - 对扩展放开，但是对修改关闭。在不更改现有代码的情况下添加新功能。比如一个方法因为有 switch 的语句，每次出现新增条件时就要修改原来的方法。这时候不如换成多态的特性。

```
// bad: 注意到fetch用条件语句了，不利于扩展
class AjaxAdapter extends Adapter {
  constructor() {
    super();
    this.name = "ajaxAdapter";
  }
}

class NodeAdapter extends Adapter {
  constructor() {
    super();
    this.name = "nodeAdapter";
  }
}

class HttpRequester {
  constructor(adapter) {
    this.adapter = adapter;
  }

  fetch(url) {
    if (this.adapter.name === "ajaxAdapter") {
      return makeAjaxCall(url).then(response =&gt; {
        // transform response and return
      });
    } else if (this.adapter.name === "nodeAdapter") {
      return makeHttpCall(url).then(response =&gt; {
        // transform response and return
      });
    }
  }
}

function makeAjaxCall(url) {
  // request and return promise
}

function makeHttpCall(url) {
  // request and return promise
}

// good
class AjaxAdapter extends Adapter {
  constructor() {
    super();
    this.name = "ajaxAdapter";
  }

  request(url) {
    // request and return promise
  }
}

class NodeAdapter extends Adapter {
  constructor() {
    super();
    this.name = "nodeAdapter";
  }

  request(url) {
    // request and return promise
  }
}

class HttpRequester {
  constructor(adapter) {
    this.adapter = adapter;
  }

  fetch(url) {
    return this.adapter.request(url).then(response =&gt; {
      // transform response and return
    });
  }
}
```

1. 里氏替换原则 (LSP)

- 如果 S 是 T 的子类，则 T 的对象可以替换为 S 的对象，而不会破坏程序。
- 所有引用其父类对象方法的地方，都可以透明的替换为其子类对象。

```
// bad: 用正方形继承了长方形
class Rectangle {
  constructor() {
    this.width = 0;
    this.height = 0;
  }

  setColor(color) {
    // ...
  }

  render(area) {
    // ...
  }

  setWidth(width) {
    this.width = width;
  }

  setHeight(height) {
    this.height = height;
  }

  getArea() {
    return this.width * this.height;
  }
}

class Square extends Rectangle {
  setWidth(width) {
    this.width = width;
    this.height = width;
  }

  setHeight(height) {
    this.width = height;
    this.height = height;
  }
}

function renderLargeRectangles(rectangles) {
  rectangles.forEach(rectangle =&gt; {
    rectangle.setWidth(4);
    rectangle.setHeight(5);
    const area = rectangle.getArea(); // BAD: 返回了25，其实应该是20
    rectangle.render(area);
  });
}

const rectangles = [new Rectangle(), new Rectangle(), new Square()];// 这里替换了
renderLargeRectangles(rectangles);

// good: 取消正方形和长方形继承关系，都继承Shape
class Shape {
  setColor(color) {
    // ...
  }

  render(area) {
    // ...
  }
}

class Rectangle extends Shape {
  constructor(width, height) {
    super();
    this.width = width;
    this.height = height;
  }

  getArea() {
    return this.width * this.height;
  }
}

class Square extends Shape {
  constructor(length) {
    super();
    this.length = length;
  }

  getArea() {
    return this.length * this.length;
  }
}

function renderLargeShapes(shapes) {
  shapes.forEach(shape =&gt; {
    const area = shape.getArea();
    shape.render(area);
  });
}

const shapes = [new Rectangle(4, 5), new Rectangle(4, 5), new Square(5)];
renderLargeShapes(shapes);
```

1. 接口隔离原则 (ISP) - 定义是 " 客户不应被迫使用对其而言无用的方法或功能 "。常见的就是让一些参数变成可选的。

```
 // bad
 class Dog {
     constructor(options) {
         this.options = options;
     }

     run() {
         this.options.run(); // 必须传入 run 方法，不然报错
     }
 }

 const dog = new Dog({}); // Uncaught TypeError: this.options.run is not a function
  
 dog.run()

 // good
 class Dog {
     constructor(options) {
         this.options = options;
     }

     run() {
         if (this.options.run) {
             this.options.run();
             return;
         }
         console.log('跑步');
     }
 }

```

1. 依赖倒置原则（DIP） - 程序要依赖于抽象接口 (可以理解为入参)，不要依赖于具体实现。这样可以减少耦合度。

```
 // bad
 class OldReporter {
   report(info) {
     // ...
   }
 }

 class Message {
   constructor(options) {
     // ...
     // BAD: 这里依赖了一个实例，那你以后要换一个，就麻烦了
     this.reporter = new OldReporter();
   }

   share() {
     this.reporter.report('start share');
     // ...
   }
 }

 // good
 class Message {
   constructor(options) {
     // reporter 作为选项，可以随意换了
     this.reporter = this.options.reporter;
   }

   share() {
     this.reporter.report('start share');
     // ...
   }
 }
 class NewReporter {
   report(info) {
     // ...
   }
 }
 new Message({ reporter: new NewReporter });

```

**持续更新中......**
