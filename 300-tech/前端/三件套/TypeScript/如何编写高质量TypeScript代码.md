---
title: 如何编写高质量TypeScript代码
date: 2024-08-22T02:34:21+08:00
updated: 2024-08-22T02:34:28+08:00
dg-publish: false
---

### 一、减少重复代码

> 如何编写高质高效的 TypeScript 代码，根据以往开发经验，本文给出以下建议

对于 TypeScript 新手小伙伴来说，在定义接口时，可能会编写类似以下的代码：

```typescript
interface Person {
  firstName: string;
  lastName: string;
}

interface PersonWithBirthDate {
  firstName: string;
  lastName: string;
  birth: Date;
}
```

很明显，接口定义出现重复代码，相对于 `Person` 接口来说，`PersonWithBirthDate` 接口只是多了一个 `birth` 属性，其他的属性跟 `Person` 接口是一样的。那么如何避免出现例子中的重复代码呢？要解决这个问题，可以利用 extends 关键字：

```typescript
interface Person {
  firstName: string;
  lastName: string;
}

interface PersonWithBirthDate extends Person {
  birth: Date;
}
```

当然除了使用 extends 关键字之外，也可以使用交叉运算符（&）：

```typescript
type PersonWithBirthDate = Person & { birth: Date };
```

有时候你想要定义一个类型来匹配一个初始对象的属性，比如：

```typescript
// 初始化对象
const INIT_OPTIONS = {
  width: 640,
  height: 480,
  color: "#00FF00",
  label: "VGA",
};
// 你可能会这样定义初始化对象的类型：
interface Options {
  width: number;
  height: number;
  color: string;
  label: string;
}
```

其实，想要快速简洁无误的定义一个对象的属性类型时候，可以使用 typeof 操作符：

```typescript
type Options = typeof INIT_OPTIONS;
```

而在使用 [可辨识联合（Discriminated Unions）](https://kothing.github.io/typescript-discriminated-unions) 的过程中，也可能出现重复代码。比如：

```typescript
interface SaveAction {
  type: 'save';
  // ...
}

interface LoadAction {
  type: 'load';
  // ...
}

type Action = SaveAction | LoadAction;
type ActionType = 'save' | 'load'; // Repeated types!
```

为了避免重复定义 `'save'` 和 `'load'`，我们可以使用成员访问语法，来提取对象中属性的类型：

```typescript
type ActionType = Action['type']; // 类型是 "save" | "load"
```

然而在实际的开发过程中，重复的类型并不总是那么容易被发现。有时它们会被语法所掩盖。比如有多个函数拥有相同的类型签名：

```typescript
function get(url: string, opts: Options): Promise<Response> { /* ... */ }
function post(url: string, opts: Options): Promise<Response> { /* ... */ }
```

对于上面的 get 和 post 方法，为了避免重复的代码，你可以提取统一的类型签名：

```typescript
type HTTPFunction = (url: string, opts: Options) => Promise<Response>;

const get: HTTPFunction = (url, opts) => { /* ... */ };
const post: HTTPFunction = (url, opts) => { /* ... */ };
```

<br/>

## 二、使用更精确的类型替代字符串类型

假设你正在构建一个音乐集，并希望为专辑定义一个类型。这时你可以使用 `interface` 关键字来定义一个 `Album` 类型：

```typescript
interface Album {
  artist: string; // 艺术家
  title: string; // 专辑标题
  releaseDate: string; // 发行日期：YYYY-MM-DD
  recordingType: string; // 录制类型："live" 或 "studio"
}
```

对于 `Album` 类型，你希望 `releaseDate` 属性值的格式为 `YYYY-MM-DD`，而 `recordingType` 属性值的范围为 `live` 或 `studio`。但因为接口中 `releaseDate` 和 `recordingType` 属性的类型都是字符串，所以在使用 `Album` 接口时，可能会出现以下问题：

```typescript
const dangerous: Album = {
  artist: "Michael Jackson",
  title: "Dangerous",
  releaseDate: "November 31, 1991", // 与预期格式不匹配
  recordingType: "Studio", // 与预期格式不匹配
};
```

虽然 `releaseDate` 和 `recordingType` 的值与预期的格式不匹配，但此时 TypeScript 编译器并不能发现该问题。为了解决这个问题，你应该为 `releaseDate` 和 `recordingType` 属性定义更精确的类型，比如这样：

```typescript
interface Album {
  artist: string; // 艺术家
  title: string; // 专辑标题
  releaseDate: Date; // 发行日期：YYYY-MM-DD
  recordingType: "studio" | "live"; // 录制类型："live" 或 "studio"
}
```

重新定义 Album 接口之后，对于前面的赋值语句，TypeScript 编译器就会提示以下异常信息：

```typescript
const dangerous: Album = {
  artist: "Michael Jackson",
  title: "Dangerous",
  releaseDate: "November 31, 1991", // Error: 不能将类型“string”分配给类型“Date”
  recordingType: "Studio", // Error: 不能将类型“"Studio"”分配给类型“"studio" | "live"”
};
```

为了解决上面的问题，你需要为 releaseDate 和 recordingType 属性设置正确的类型，比如这样：

```typescript
const dangerous: Album = {
  artist: "Michael Jackson",
  title: "Dangerous",
  releaseDate: new Date("1991-11-31"),
  recordingType: "studio",
};
```

另一个错误使用字符串类型的场景是设置函数的参数类型。

假设你需要写一个函数，用于从一个对象数组（如下源数组）中抽取某个属性的值并保存到数组中，在 Underscore 库中，这个操作被称为 “pluck”。

```javascript
// 源数组
const albums = [
  {
    artist: "Michael Jackson",
    title: "Dangerous",
    releaseDate: new Date("1991-11-31"),
    recordingType: "studio",
  },
  {
    artist: "Eminem",
    title: "Beautiful",
    releaseDate: new Date("2018-10-06"),
    recordingType: "live",
  },
  {
    artist: "Adele",
    title: "Hello",
    releaseDate: new Date("2019-06-10"),
    recordingType: "studio",
  },
];

// 目标数组
const songs = ['Dangerous', 'Beautiful', 'Hello'];
```

要实现该功能，你可能最先想到以下代码：

```typescript
function pluck(record: any[], key: string): any[] {
  return record.map((r) => r[key]);
}
```

对于以上的 pluck 函数并不是很好，因为它使用了 `any` 类型，特别是作为返回值的类型。那么如何优化 pluck 函数呢？首先，可以通过引入一个泛型参数来改善类型签名：

```typescript
function pluck<T>(record: T[], key: string): any[] {
  // Element implicitly has an 'any' type because expression of type 'string' can't be used to index type 'unknown'.
  // No index signature with a parameter of type 'string' was found on type 'unknown'.
  return record.map((r) => r[key]); // Error
}
```

通过以上的异常信息，可知字符串类型的 `key` 不能被作为 `unknown` 类型的索引类型。要从对象上获取某个属性的值，你需要保证参数 `key` 是对象中的属性。

说到这里相信有一些小伙伴已经想到了 keyof 操作符，它是 TypeScript 2.1 版本引入的，可用于获取某种类型的所有键，其返回类型是联合类型。接着使用 keyof 操作符来完善下 pluck 函数：

```typescript
function pluck<T>(record: T[], key: keyof T) {
  return record.map((r) => r[key]);
}
```

对于第一次完善后的 pluck 函数，你的 IDE 将会为你自动推断出该函数的返回类型：

```typescript
function pluck<T>(record: T[], key: keyof T): T[keyof T][]
```

对于完善后的 pluck 函数，你可以使用前面定义的 Album 类型来测试一下：

```typescript
const albums: Album[] = [
  {
    artist: "Michael Jackson",
    title: "Dangerous",
    releaseDate: new Date("1991-11-31"),
    recordingType: "studio",
  },
  {
    artist: "Eminem",
    title: "Beautiful",
    releaseDate: new Date("2018-10-06"),
    recordingType: "live",
  },
  {
    artist: "Adele",
    title: "Hello",
    releaseDate: new Date("2019-06-10"),
    recordingType: "studio",
  }
];

// let releaseDateArr: (string | Date)[]
let releaseDateArr = pluck(albums, 'releaseDate');
```

示例中的 releaseDateArr 变量，它的类型被推断为 (string | Date)[]，很明显这并不是你所期望的，它的正确类型应该是 Date[]。那么应该如何解决该问题呢？这时你需要引入第二个泛型参数 K，然后使用 extends 来进行约束：

```typescript
function pluck<T, K extends keyof T>(record: T[], key: K): T[K][] {
  return record.map((r) => r[key]);
}

// let releaseDateArr: Date[]
let releaseDateArr = pluck(albums, 'releaseDate');
```

<br/>

## 三、定义的类型总是表示有效的状态

加入编写一个分页的组件，允许用户指定页码，然后加载并显示该页面对应内容的 `Web` 内容。

你可能会先定义 State 对象：

```typescript
interface State {
  pageContent: string;
  isLoading: boolean;
  errorMsg?: string;
}
```

接着你会编写一个渲染页面函数 `renderPage` 函数，用来渲染页面：

```typescript
function renderPage(state: State) {
  if (state.errorMsg) {
    return `呜呜呜，加载页面出现异常了...${state.errorMsg}`;
  } else if (state.isLoading) {
    return `页面加载中~~~`;
  }
  return `<div>${state.pageContent}</div>`;
}

// 输出结果：页面加载中~~~
console.log(renderPage({isLoading: true, pageContent: ""}));

// 输出结果：<div>Hello TypeScript !</div>
console.log(renderPage({isLoading: false, pageContent: "Hello TypeScript !"}));
```

创建好 `renderPage` 函数，你可以继续定义一个分页函数 `changePage` 函数，用于根据页码获取对应的页面数据：

```typescript
async function changePage(state: State, newPage: string) {
  state.isLoading = true;
  try {
    const response = await fetch(getUrlForPage(newPage));
    if (!response.ok) {
      throw new Error(`Unable to load ${newPage}: ${response.statusText}`);
    }
    const text = await response.text();
    state.isLoading = false;
    state.pageContent = text;
  } catch (e) {
    state.errorMsg = "" + e;
  }
}
```

对于以上的 `changePage` 函数，它存在以下问题：

- 在 `catch` 语句中，未把 `state.isLoading` 的状态设置为 `false`；
- 未及时清理 `state.errorMsg` 的值，因此如果之前的请求失败，那么你将继续看到错误消息，而不是加载消息。

出现上述问题的原因是，前面定义的 `State` 类型允许同时设置 `isLoading` 和 `errorMsg` 的值，尽管这是一种无效的状态。针对这个问题，你可以考虑引入可辨识联合类型来定义不同的页面请求状态：

```typescript
interface RequestPending {
  state: "pending";
}

interface RequestError {
  state: "error";
  errorMsg: string;
}

interface RequestSuccess {
  state: "ok";
  pageContent: string;
}

type RequestState = RequestPending | RequestError | RequestSuccess;

interface State {
  currentPage: string;
  requests: { [page: string]: RequestState };
}
```

在以上代码中，通过使用可辨识联合类型分别定义了 3 种不同的请求状态，这样就可以很容易的区分出不同的请求状态，从而让业务逻辑处理更加清晰。接下来，需要基于更新后的 `State` 类型，来分别更新一下前面创建的 `renderPage` 和 `changePage` 函数：

```typescript
// 完善后的 renderPage 函数
function renderPage(state: State) {
  const { currentPage } = state;
  const requestState = state.requests[currentPage];
  switch (requestState.state) {
    case "pending":
      return `页面加载中~~~`;
    case "error":
      return `呜呜呜，加载第${currentPage}页出现异常了...${requestState.errorMsg}`;
    case "ok":
      `<div>第${currentPage}页的内容：${requestState.pageContent}</div>`;
  }
}
```

```typescript
// 完善后的 changePage 函数
async function changePage(state: State, newPage: string) {
  state.requests[newPage] = { state: "pending" };
  state.currentPage = newPage;
  try {
    const response = await fetch(getUrlForPage(newPage));
    if (!response.ok) {
      throw new Error(`无法正常加载页面 ${newPage}: ${response.statusText}`);
    }
    const pageContent = await response.text();
    state.requests[newPage] = { state: "ok", pageContent };
  } catch (e) {
    state.requests[newPage] = { state: "error", errorMsg: "" + e };
  }
}
```

在 `changePage` 函数中，会根据不同的情形设置不同的请求状态，而不同的请求状态会包含不同的信息。这样 `renderPage` 函数就可以根据统一的 `state` 属性值来进行相应的处理。因此，通过使用可辨识联合类型，让请求的每种状态都是有效的状态，不会出现无效状态的问题。

<br/>

## 四、选择条件类型而不是重载声明

假设你要使用 TS 实现一个 `double` 函数，该函数支持 `string` 或 `number` 类型。这时，你可能已经想到了使用联合类型和函数重载：

```typescript
function double(x: number | string): number | string;
function double(x: any) {
  return x + x;
}
```

虽然这个 double 函数的声明是正确的，但它有一点不精确：

```typescript
const num = double(10);
// const num: string | number

const str = double('ts');
// const str: string | number
```

对于 double 函数，你期望传入的参数类型是 `number` 类型，其返回值的类型也是 `number` 类型。当你传入的参数类型是 `string` 类型，其返回的类型也是 `string` 类型。而上面的 `double` 函数却是返回了 `string | number` 类型。为了实现上述的要求，你可能想到了引入泛型变量和泛型约束：

```typescript
function double<T extends number | string>(x: T): T;
function double(x: any) {
  return x + x;
}
```

在上面的 `double` 函数中，引入了泛型变量 `T`，同时使用 `extends` 约束了其类型范围。

```typescript
const num = double(10); // const num: 10
const str = double('ts');// const str: "ts"
console.log(str);
```

不幸的是，我们对精确度的追求超过了预期。现在的类型有点太精确了。当传递一个字符串类型时，`double` 声明将返回一个字符串类型，这是正确的。但是当传递一个字符串字面量类型时，返回的类型是相同的字符串字面量类型，这是错误的，因为 ts 经过 `double` 函数处理后，返回的是 tsts，而不是 ts。

另一种方案是提供多种类型声明。虽然 TypeScript 只允许你编写一个具体的实现，但它允许你编写任意数量的类型声明。你可以使用函数重载来改善 `double` 的类型：

```typescript
function double(x: number): number;
function double(x: string): string;
function double(x: any) {
  return x + x;
}

const num = double(10); // const num: number
const str = double("ts"); // const str: string
```

很明显此时 `num` 和 `str` 变量的类型都是正确的，但不幸的是，double 函数还有一个小问题。因为 `double` 函数的声明只支持 `string` 或 `number` 类型的值，而不支持 `string | number` 联合类型，比如：

```typescript
function doubleFn(x: number | string) {
  // Argument of type 'string | number' is not assignable to
  // parameter of type 'number'.
  // Argument of type 'string | number' is not assignable to
  // parameter of type 'string'.
  return double(x); // Error
}
```

为什么会提示以上的错误呢？因为当 TypeScript 编译器处理函数重载时，它会查找重载列表，直到找一个匹配的签名。对于 `number | string` 联合类型，很明显是匹配失败的。

然而对于上述的问题，虽然可以通过新增 `string | number` 的重载签名来解决，但最好的方案是使用条件类型。在类型空间中，条件类型就像 `if` 语句一样：

```typescript
function double<T extends number | string>(
  x: T
): T extends string ? string : number;

function double(x: any) {
  return x + x;
}
```

这与前面引入泛型版本的 `double` 函数声明类似，只是它引入更复杂的返回类型。条件类型使用起来很简单，与 JavaScript 中的三目运算符（?:）一样的规则。`T extends string ? string : number` 的意思是，如果 `T` 类型是 `string` 类型的子集，则 `double` 函数的返回值类型为 `string` 类型，否则为 `number` 类型。

在引入条件类型之后，前面的所有例子就可以正常工作了：

```typescript
const num = double(10);  // const num: number
const str = double("ts");  // const str: string

// function f(x: string | number): string | number
function f(x: number | string) {
  return double(x);
}
```

<br/>

## 五、一次性创建对象

在 `JavaScript` 中可以很容易地创建一个表示二维坐标点的对象：

```typescript
const pt = {};
pt.x = 3;
pt.y = 4;
```

然而对于同样的代码，在 `TypeScript` 中会提示以下错误信息：

```typescript
const pt = {};
// Property 'x' does not exist on type '{}'
pt.x = 3; // Error
// Property 'y' does not exist on type '{}'
pt.y = 4; // Error
```

这是因为第一行中 pt 变量的类型是根据它的值 `{}` 推断出来的，你只能对已知的属性赋值。针对这个问题，你可能会想到一种解决方案，即新声明一个 `Point` 类型，然后把它作为 `pt` 变量的类型：

```typescript
interface Point {
  x: number;
  y: number;
}

// Type '{}' is missing the following properties from type 'Point': x, y
const pt: Point = {}; // Error
pt.x = 3;
pt.y = 4;
```

那么如何解决上述问题呢？其中一种最简单的解决方案是一次性创建对象：

```typescript
const pt = {
  x: 3,
  y: 4,
}; // OK
```

如果你想一步一步地创建对象，你可以使用类型断言（`as`）来消除类型检查：

```typescript
const pt = {} as Point;
pt.x = 3;
pt.y = 4; // OK
```

但是更好的方法是一次性创建对象并显式声明变量的类型：

```typescript
const pt: Point = {
  x: 3,
  y: 4,
};
```

而当你需要扩展对象 (给对象添加属性) 时，你可能会这样处理，比如：

```typescript
const pt = { x: 3, y: 4 };
const id = { name: "Pythagoras" };
const namedPoint = {};
Object.assign(namedPoint, pt, id);

// Property 'id' does not exist on type '{}'
namedPoint.name; // Error
```

为了解决上述问题，你可以使用对象展开运算符 ... 来一次性构建大的对象：

```typescript
const namedPoint = {...pt, ...id};
namedPoint.name; // OK, type is string
```

此外，你还可以使用对象展开运算符，以一种类型安全的方式逐个字段地构建对象。关键是在每次更新时使用一个新变量，这样每个变量都会得到一个新类型：

```typescript
const pt0 = {};
const pt1 = {...pt0, x: 3};
const pt: Point = {...pt1, y: 4}; // OK
```

虽然这是构建这样一个简单对象的一种迂回方式，但对于向对象添加属性并允许 `TypeScript` 推断新类型来说，这可能是一种有用的技术。要以类型安全的方式有条件地添加属性，可以使用带 `null` 或 `{}` 的对象展开运算符，它不会添加任何属性：

```typescript
declare var hasMiddle: boolean;
const firstLast = {first: 'Harry', last: 'Truman'};
const president = {...firstLast, ...(hasMiddle ? {middle: 'S'} : {})};
```

如果在编辑器中鼠标移到 `president`，你将看到 `TypeScript` 推断出的类型：

```typescript
const president: {
  middle?: string;
  first: string;
  last: string;
}
```

最终通过设置 hasMiddle 变量的值，你就可以控制 president 对象中 middle 属性的值：

```typescript
declare var hasMiddle: boolean;
var hasMiddle = true;
const firstLast = {first: 'Harry', last: 'Truman'};
const president = {...firstLast, ...(hasMiddle ? {middle: 'S'} : {})};

let mid = president.middle
console.log(mid); // S
```

<br/>

## 六、参考资源

[Effective TypeScript]<https://github.com/danvk/effective-typescript>)
