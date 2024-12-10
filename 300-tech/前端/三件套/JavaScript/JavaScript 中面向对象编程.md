---
title: JavaScript 中面向对象编程
date: 2024-09-11T04:43:26+08:00
updated: 2024-11-25T15:52:14+08:00
dg-publish: false
---

面向对象编程

## 对象

对象：一个包含相关**数据和方法的集合**（通常由一些变量和函数组成，我们称之为对象里面的**属性和方法**）

```js
const person = {}
console.log(person)  // {}
console.log(person.toString())  // '[object Object]'
```

添加属性和方法（通过 对象字面量 方式创建一个对象）

```js
const person = {
  name: ["Bob", "Smith"],
  age: 32,
  bio: function () {  // 能简写成 bio() {}
    console.log(`${this.name[0]} ${this.name[1]} 现在 ${this.age} 岁了。`)
  },
  introduceSelf: function () {
    console.log(`你好！我是 ${this.name[0]}。`)
  },
}
```

访问对象的属性和调用它的方法

```js
console.log(person.name)  // ["Bob", "Smith"]
console.log(person.name[0])  // Bob
console.log(person.age)  // 32
person.bio()  // Bob Smith 现在 32 岁了。
person.introduceSelf()  // 你好！我是 Bob。
```

访问属性：

- `.属性` 访问
- `[属性]` 访问（这个可以传变量 `person.[age]`）

更新属性：

- `person.age = 18`
- `person['name'][1] = 'Tom'`

直接创建新的属性

- `person['eyes'] = 'hazel`
- `person.farewell = function() {}`

关键字 `this` 指向当前代码**运行时**的对象

## 函数工厂模式

批量创建对象

```js
function createPerson(name) {
  const obj = {}
  obj.name = name
  obj.introduceSelf = function () {
    console.log(`你好！我是 ${this.name}。`)
  }
  return obj
}

const salva = createPerson("Salva")
salva.name
salva.introduceSelf()
```

它需要创建空对象，初始化并返回，这些都没必要

## 构造函数

创建多个有相同属性的对象

构造函数约定以大写字母开头

```js
function Person(name) {
  this.name = name
  this.introduceSelf = function() {
    console.log(`Hello, I'm ${this.name}`)
  }
}
```

要将 `Person()` 作为构造函数调用，使用 `new`

```js
const salva = new Person('Salva')
console.log(salva.name)
salva.introduceSelf()
```

使用 `new` 关键字会

1. **创建**一个新对象
2. 把 `this` **绑定**到新对象
3. 运行构造函数中的代码
4. **返回**新对象

## 原型和原型链

JavaScript 中所有的对象都有一个内置属性，称为它的 **prototype**（原型）

原型本身也是对象，所有原型对象也会有自己的原型

构成原型链，最后终止于拥有 `null` 作为其原型的对象上

当你试图访问一个对象的属性时：如果在对象本身中找不到该属性，就会在原型中搜索该属性。如果仍然找不到该属性，那么就搜索原型的原型，以此类推，直到找到该属性，或者到达链的末端，在这种情况下，返回 `undefined`

可以通过 `Object.getPrototyeOf(obj)` 查看对象的原型、`obj.__proto__` 是它的隐式原型

例如 `function fun() {}` 的原型链 `Function.prototype` → `Object {}` → `null`

由于查找属性和方法会根据原型链查找，如果直接在对象上创建同名的属性和方法将会直接使用自己创建的

可以通过 `Object.create` 或 `x.prototype` 设置原型

```js
const personPrototype = {
  greet() {
    console.log("hello!")
  },
}
const carl = Object.create(personPrototype)
carl.greet() // hello!

const personPrototype = {
  greet() {
    console.log(`你好，我的名字是 ${this.name}！`)
  },
}
function Person(name) {
  this.name = name
}
Object.assign(Person.prototype, personPrototype)
// 或
// Person.prototype.greet = personPrototype.greet
const reuben = new Person("Reuben")
reuben.greet() // 你好，我的名字是 Reuben！
```

自有属性（不是在原型上定义的），可以使用 `Object.hasOwn` 或 `Object.hasOwnProperty()` 判断

```js
const irma = new Person("Irma");

console.log(Object.hasOwn(irma, "name")); // true
console.log(Object.hasOwn(irma, "greet")); // false
```

通过原型能够实现某种意义的继承

## 面向对象编程

面向对象编程（OOP）是如今多种编程语言所实现的一种编程范式

其中包括三个主要概念：**类与实例**、**继承**、**封装**

谈论面向对象编程时，通常来说是指**基于类的面向对象编程**（class-based OOP）

面向对象编程将一个系统**抽象**为许多**对象的集合**，每一个对象代表了这个系统的特定方面

通过类创建对象，对象中包括函数（方法）和数据（属性）

使用 `class` 关键字声明一个类

```js
class Person {
  // 属性
  name  // 可选，因为在 this.name = name 的时候会自动创建，也可以给一个初始值 name = ''

  // 构造函数
  constructor(name) {  // 可以在 new 的时候传参数
    this.name = name  // 用于初始化新的对象的 name 属性
  }

  // 方法
  introduceSelf() {
    console.log(`Hi, I'm ${this.name}`)
  }
}
```

构造函数使用 `constructor` 关键字来声明，它会

- 创建一个新的对象
- 将 `this` 绑定到这个新的对象，你可以在构造函数代码中使用 `this` 来引用它
- 执行构造函数中的代码
- 返回这个新的对象

里面什么关键字都没加的成员和方法叫做实例成员/属性和实例方法，附着在**实例**上

如果不需要初始化内容，可以省略构造函数 `class A { method() {} }`

创建实例对象

```js
const giles = new Person('Giles')
giles.introduceSelf()
```

可以根据基类声明它的子类，子类会继承 `extends` 基类的属性和方法

```js
class Professor extends Person {
  teaches  // 自己的属性

  constructor(name, teaches) {
    super(name)  // 使用 super 调用父类的构造函数
    this.teaches = teaches
  }

  introduceSelf() {  // 覆盖父类的方法
    console.log(`My name is ${this.name}, and I will be your ${this.teaches} professor.`)
  }

  grade(paper) {  // 自己的方法
    const grade = Math.floor(Math.random() * (5 - 1) + 1)
    console.log(paper, grade)
  }
}
```

创建并使用实例对象

```js
const tom = new Professor('Tom', 'Math')
tom.introduceSelf()
tom.grade('my paper')
```

私有属性、私有方法 `#`

```js
class Student extends Person {
  #year  // 私有属性

  constructor(name, year) {
    super(name);
    this.#year = year
  }

  introduceSelf() {
    console.log(`Hi! I'm ${this.name}, and I'm in year ${this.#year}.`)
  }

  #privateMethod() {  // 私有方法
    return this.#year > 1
  }

  canStudyArchery() {
    return this.#privateMethod()
  }
}
```

之前在类声明中的方法和属性是可以直接在外部使用的，但是使用私有属性或方法的时候它会抛出错误

```js
const summers = new Student("Summers", 2)

summers.introduceSelf() // Hi! I'm Summers, and I'm in year 2.
summers.canStudyArchery() // true

summers.#year // SyntaxError
```

`static` 关键字声明成员或方法，只能在类上使用，不能在实例上使用（静态属性或方法）

```js
class Rabbit {
  furColor  // 实例成员
  age
  nickname
  static total = 0  // 静态成员，不是实例的
  
  constructor(furColor, age, nickname) {
    // this 是实例
    this.furColor = furColor
    this.age = age
    this.nickname = nickname
    
    Rabbit.total++
  }

  jump() {
    console.log(`${this.nickname} jumped`)
  }
}

const r1 = new Rabbit('white', 2, '小白兔')
const r2 = new Rabbit('gray', 3, '小灰兔')
r1.jump()
console.log(r2.nickname)
console.log(Rabbit.total)
```

- 实例成员或方法：例如 `const n = new Number(1); n.toString();` 这样的跟实例有关
- 静态成员或方法：例如 `Number.MAX_VALUE;Number.isNaN(NaN)` 和实例没有关系

`abstract` 抽象类，必须由继承类实现，不能 new（抽象类不能创建实例）

```ts
abstract class Person {
  public name: string
  constructor(name: string) {
    this.name = name
  }

  abstract sayHi(n: number): void  // 在基类中不实现，到子类中实现这个方法
  goHome() {
    this.sayHi(1)  // 在基类中方便调用，在
    console.log('回家')
  }
}

class Student extends Person {
  constructor(name: string) {
    super(name)
  }
  sayHi() {  // 子类必须实现父类中的抽象成员，可以选择不使用参数
    console.log(`我是一名学生，我的名字是${this.name}`)
  }
}

class Student extends Person {
  constructor(name: string) {
    super(name)
  }
  sayHi(n: number) {  // 使用了参数就必须一致
    console.log(`我是一名老师，我的名字是${this.name}，${n}岁`)
  }
}

const s = new Student('晓明')
const t = new Teacher('老王')
s.sayHi()
t.sayHi()
```

## getter

需要一个属性，它依赖于对象/类中已经存在的属性（计算得出），就可以使用 `getter` 而不是重新声明新的属性或方法

Getters 给你一种方法来定义一个对象的属性，但是在访问它们之前不会计算属性的值

**对象** 使用 `defineProperty`

```js
const good = { price: 14, count: 4 }
Object.defineProperty(good, 'total', {
  get() {
    return this.price * this.count
  }
})
console.log(good.total)
```

定义的时候，甚至可以使用变量作为属性名

```js
const expr = 'foo'
const obj = {
  get [expr]() {
    return 'bar'
  }
}
console.log(obj.foo)
```

使用 `delete` 删除 getter `delete obj.foo`

**class** 中

```js
class Good {
  constructor(price, count) {
    this.price = price
    this.count = count
  }

  get total() {
    return price * count
  }
}

const good = new Good(3, 12)
console.log(obj.total)
```
