---
title: 总结 React 运用 TypeScript 的问题
date: 2024-08-22T02:19:08+08:00
updated: 2024-08-22T02:19:16+08:00
dg-publish: false
---

> 个人觉得 React 的 PropType 和 PropDefault 几乎能做 ts 的静态类型检查能做到的事情，甚至做的还能比 ts 做的多。比如说对于组件间设置默认值，ts 对于支持的就是不太好

> 虽然 TS 在第一次开发的时候, 需要花一些时间去编写各种类型，但是对于后续维护、重构非常有帮助，还是推荐长期维护的项目中使用它

---

## 组件定义

- 函数组件

```ts

import React from 'react';

// 函数声明式写法, 注明了这个函数的返回值是 React.ReactNode 类型
function func({}:IProps): React.ReactNode {
  return <h1>My Website Heading</h1>;
}

// 函数表达式写法, 返回一个函数而不是值或者表达式，所以注明这个函数的返回值是 React.FC 类型
const func:React.FC<IProps> = () => <h1>My Website Heading</h1>;

```

- 类组件

```ts
interface StateProps { }

interface IProps { }

class Page extends React.Component<IProps, StateProps> {
    constructor(pops) {
        
    }
    
    componentDidMount(){
        
    }

    render() {
        return (
            <div>
                
            </div>
        );
    }
}
```

## Hooks

**useState**

如果默认值已经可以说明类型，那么不用手动声明类型，交给 TS 自动推断即可：

```ts
// val: boolean
const [val, toggle] = React.useState(false);

toggle(false)
toggle(true)
```

如果初始值是 null 或 undefined，那就要通过泛型手动传入期望的类型。

```ts
const [user, setUser] = React.useState<IUser | null>(null);

// later...
setUser(newUser);
```

直接访问 user 上的属性时，提示它有可能是 null。

通过 optional-chaining 语法（TS 3.7 以上支持），可以避免这个错误。

```ts
// ✅ ok
const name = user?.name
```

## useReducer

需要用 [Discriminated Unions](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes-func.html#discriminated-unions) 来标注 Action 的类型

```ts
const initialState = { count: 0 };

type ACTIONTYPE =
  | { type: "increment"; payload: number }
  | { type: "decrement"; payload: string };

function reducer(state: typeof initialState, action: ACTIONTYPE) {
  switch (action.type) {
    case "increment":
      return { count: state.count + action.payload };
    case "decrement":
      return { count: state.count - Number(action.payload) };
    default:
      throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = React.useReducer(reducer, initialState);
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({ type: "decrement", payload: "5" })}>
        -
      </button>
      <button onClick={() => dispatch({ type: "increment", payload: 5 })}>
        +
      </button>
    </>
  );
}
```

「Discriminated Unions」一般是一个联合类型，其中每一个类型都需要通过类似 type 这种特定的字段来区分，当传入特定的 type 时，剩下的类型 payload 就会自动匹配推断。

- 当写入的 type 匹配到 decrement 的时候，TS 会自动推断出相应的 payload 应该是 string 类型。
- 当写入的 type 匹配到 increment 的时候，则 payload 应该是 number 类型。

当在 dispatch 的时候，输入对应的 type，就自动提示剩余的参数类型

## useEffect

useEffect 执行一个 async 函数

```ts
useEffect(() => {
  const getUser = async () => {
    const user = await getUser()
    setUser(user)
  }
  getUser()
}, [])
```

## useRef

这个 Hook 在很多时候是没有初始值的，这样可以声明返回对象中 current 属性的类型：

**以一个按钮场景为例：**

```ts
function TextInputWithFocusButton() {
  const inputEl = React.useRef<HTMLInputElement>(null);
  const onButtonClick = () => {
    if (inputEl && inputEl.current) {
      inputEl.current.focus();
    }
  };
  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}
```

当 onButtonClick 事件触发时，可以肯定 inputEl 也是有值的，因为组件是同级别渲染的，但是还是依然要做冗余的非空判断。

```ts
// ✅ ok
inputEl.current?.focus()

// 👎
const ref1 = useRef<HTMLElement>(null!);
```

null! 这种语法是非空断言，跟在一个值后面表示断定它是有值的，所以在使用 inputEl.current.focus() 的时候，TS 不会给出报错。但是这种语法比较危险，需要尽量减少使用。

## useImperativeHandle

推荐使用一个自定义的 innerRef 来代替原生的 ref，否则要用到 forwardRef 会搞的类型很复杂

```ts
type ListProps = {
  innerRef?: React.Ref<{ scrollToTop(): void }>
}

function List(props: ListProps) {
  useImperativeHandle(props.innerRef, () => ({
    scrollToTop() { }
  }))
  return null
}
```

结合刚刚 useRef 的知识，使用是这样的：

```ts

function Use() {
  const listRef = useRef<{ scrollToTop(): void }>(null!)

  useEffect(() => {
    listRef.current.scrollToTop()
  }, [])

  return (
    <List innerRef={listRef} />
  )
}
```

## 自定义 Hook

如果想仿照 useState 的形式，返回一个数组给用户使用，一定要记得在适当的时候使用 as const，标记这个返回值是个常量，告诉 TS 数组里的值不会删除，改变顺序等等

否则，每一项都会被推断成是「所有类型可能性的联合类型」

```ts
export function useLoading() {
  const [isLoading, setState] = React.useState(false);
  const load = (aPromise: Promise<any>) => {
    setState(true);
    return aPromise.finally(() => setState(false));
  };
  // ✅ 加了 as const 会推断出 [boolean, typeof load]
  // ❌ 否则会是 (boolean | typeof load)[]
  return [isLoading, load] as const;[]
}
```

如果在用 React Hook 写一个库，别忘了把类型也导出给用户使用。

## 组件 Props

基础类型

```ts
type BasicProps = {
  message: string;
  count: number;
  disabled: boolean;
  /** 数组类型 */
  names: string[];
  /** 用「联合类型」限制为下面两种「字符串字面量」类型 */
  status: "waiting" | "success";
};

```

对象类型

```ts
type ObjectOrArrayProps = {
  /** 如果不需要用到具体的属性 可以这样模糊规定是个对象 ❌ 不推荐 */
  obj: object;
  obj2: {}; // 同上
  /** 拥有具体属性的对象类型 ✅ 推荐 */
  obj3: {
    id: string;
    title: string;
  };
  /** 对象数组 😁 常用 */
  objArr: {
    id: string;
    title: string;
  }[];
  /** key 可以为任意 string，值限制为 MyTypeHere 类型 */
  dict1: {
    [key: string]: MyTypeHere;
  };
  dict2: Record<string, MyTypeHere>; // 基本上和 dict1 相同，用了 TS 内置的 Record 类型。
}
```

函数类型

```ts
type FunctionProps = {
  /** 任意的函数类型 ❌ 不推荐 不能规定参数以及返回值类型 */
  onSomething: Function;
  /** 没有参数的函数 不需要返回值 😁 常用 */
  onClick: () => void;
  /** 带函数的参数 😁 非常常用 */
  onChange: (id: number) => void;
  /** 另一种函数语法 参数是 React 的按钮事件 😁 非常常用 */
  onClick(event: React.MouseEvent<HTMLButtonElement>): void;
  /** 可选参数类型 😁 非常常用 */
  optional?: OptionalType;
}
```

React 相关类型

```ts
export declare interface AppProps {
  children1: JSX.Element; // ❌ 不推荐 没有考虑数组
  children2: JSX.Element | JSX.Element[]; // ❌ 不推荐 没有考虑字符串 children
  children4: React.ReactChild[]; // 稍微好点 但是没考虑 null
  children: React.ReactNode; // ✅ 包含所有 children 情况
  functionChildren: (name: string) => React.ReactNode; // ✅ 返回 React 节点的函数
  style?: React.CSSProperties; // ✅ 推荐 在内联 style 时使用
  // ✅ 推荐原生 button 标签自带的所有 props 类型
  // 也可以在泛型的位置传入组件 提取组件的 Props 类型
  props: React.ComponentProps<"button">;
  // ✅ 推荐 利用上一步的做法 再进一步的提取出原生的 onClick 函数类型
  // 此时函数的第一个参数会自动推断为 React 的点击事件类型
  onClickButton：React.ComponentProps<"button">["onClick"]
}
```

### 函数式组件

React.FC 内置类型，会自动加上一个 children 类型

```ts
interface AppProps = { message: string };

const App: React.FC<AppProps> = ({ message, children }) => {
  return (
    <>
     {children}
     <div>{message}</div>
    </>
  )
}

```

包含 children 的：

#### 参考

[react-typescript-cheatsheets](https://github.com/typescript-cheatsheets/react)
