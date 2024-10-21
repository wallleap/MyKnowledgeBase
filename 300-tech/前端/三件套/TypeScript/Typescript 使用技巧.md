---
title: Typescript 使用技巧
date: 2024-08-22T02:34:21+08:00
updated: 2024-10-14T03:14:26+08:00
dg-publish: false
---

## 注释

```
/** I'm a nice boy */
interface Person {
  /** A cool name. */
  name: string,
}
```

通过 `/** */` 形式的注释可以给 TS 类型做标记，编辑器会有更好的提示

## 巧用 Record 类型

业务中，我们经常会写枚举和对应的映射:

```
type AnimalType = 'cat' | 'dog' | 'fish';
const AnimalMap = {
  cat: { name: '猫', age: 1},
  dog: { name: '狗', age: 2 },
  fish: { name: '鱼', age: 1 },
};
```

但是没有给每个 Animal 描述设置 ts 类型，怎么做呢？

使用 `Record`:

```
// 动物名
type AnimalType = 'cat' | 'dog' | 'fish';
// 动物属性描述
interface AnimalDescription {
  name: string;
  age: number;
}
// 使用Record给动物的描述定义类型
const AnimalMap: Record<AnimalType, AnimalDescription> = {
  cat: { name: '猫', age: 1},
  dog: { name: '狗', age: 2 },
  fish: { name: '蛙', age: 1 },
};
// 此时 cat、dog、fish的name类型是string，age是number类型
```

## 巧用 tsx+extends

在 React 中.tsx 文件里，泛型可能会被当做 jsx 标签

```
const toArray = <T>(element: T) => [element]; // Error in .tsx file.
```

TypeScript 泛型写法 `<T>` 在 React 中被误认为是 html 标签（类似 `<div>`）

加 extends 可破

```
const toArray = <T extends {}>(element: T) => [element]; // No errors.
```

## 解决类型错误问题

- **对象属性不存在错误**:

这种情况一般在于，该对象值 TS 知道其有明确类型（不是 any，如果是 any 就不会报错了），但是当前要访问的属性不存在与其已知类型结构。这种情况分两种办法解决：

> 如果能修改该值的类型声明，那么添加上缺损值的属性即可；
> 否则，使用 `// @ts-ignore` 注释，或者使用类型断言，强制为 any 类型

- **类型不明确的错误**:

即一个值的类型可能被注解为联合类型，那么在直接访问时，TS 无法确定当前值到底属于哪个精确的类型，所以会报告错误。这种情况有以下解决拌饭：

> 使用类型保护 (type guards)
> 使用类型断言：定义变量时不指定类型，但一定要赋值，TypeScript 会根据赋值自动推断出类型，此后再改变此值时也只能是首次赋值的类型
> 使用 // @ts-ignore 注释

应该优先考虑类型保护，因为类型保护本质上就是增加代码逻辑，帮助 TS 理解确定当前类型，所以代码也会更健壮。而后两种办法，除非明确知道此时该值就是确定的类型，否则即使通过了 TS 编译器，在代码执行阶段，依然有可能出错！

- **值可能不存在的或为 undefined 的错误**:

这种情况其实是上面提到的类型不明确错误的一种，一般发生在可选属性或者可选参数时。解决这些情况，最简单的就是使用非空类型断言（前提是确认该值确实是非空）：

非空类型断言的形式是在值后面添加半角感叹号：

```
someVar!.toString();
```

## typeof、keyof、InstanceType

组件实例

```ts
const Foo = defineComponent()

type FooInstance = InstanceType<typeof Foo>
```

例如

```ts
// useCompRef.ts
import { ref } from 'vue'

export function useCompRef<T extends abstract new (...args: any) => any>(
  _comp: T  // 自行推导 typeof ElForm
) {
  return ref<InstanceType<T>>()
}
```

使用

```ts
import { ElForm } from 'element-plus'
import { useCompRef } from 'useCompRef'
const formRef = useCompRef(ElForm)
formRef.value?.validate
```

## 可以是任何类型但不能是 Date

```ts
function log<T>(x: T extends Date ? never : T) {
  console.log(x)
}

type BanDate<T> = T extends Date ? never : T
function log<T>(x: BanDate<T>) {}

log(new Date())  // 不能传递日期

type BanType<T, E> = T extends E ? never : T
```
