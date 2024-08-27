---
title: 总结React Hooks 使用中遇到的问题
date: 2024-08-22T02:19:37+08:00
updated: 2024-08-22T02:19:43+08:00
dg-publish: false
---

> ## Q: 使用单个 state 变量还是多个 state 变量？

```js
const [width, setWidth] = useState(100);
const [height, setHeight] = useState(100);
const [left, setLeft] = useState(0);
const [top, setTop] = useState(0);
```

或将所有的 state 放到一个 object 中，这样只需一个 state 变量即可：

```js
const [state, setState] = useState({
  width: 100,
  height: 100,
  left: 0,
  top: 0
});
```

如果使用单个 state 变量，**每次更新 state 时需要合并之前的 state**。因为 useState 返回的 setState 会替换原来的值。这一点和 Class 组件的 this.setState 不同。this.setState 会把更新的字段自动合并到 this.state 对象中。

```js
const handleMouseMove = (e) => {
  setState((prevState) => ({
    ...prevState,
    left: e.pageX,
    top: e.pageY,
  }))
};
```

在使用 state 之前，需要考虑状态拆分的「粒度」问题。如果粒度过细，代码就会变得比较冗余。如果粒度过粗，代码的可复用性就会降低。那么，到底哪些 state 应该合并，哪些 state 应该拆分呢？总结了下面两点：

- 将完全不相关的 state 拆分为多组 state。比如 size 和 position。
- 如果某些 state 是相互关联的，或者需要一起发生改变，就可以把它们合并为一组 state。比如 left 和 top。

```js
function Box() {
  const [position, setPosition] = usePosition();
  const [size, setSize] = useState({width: 100, height: 100});
  // ...
}

function usePosition() {
  const [position, setPosition] = useState({left: 0, top: 0});

  useEffect(() => {
    // ...
  }, []);

  return [position, setPosition];
}
```

> ## Q: deps 依赖过多，导致 Hooks 难以维护？

如果 useEffect、useMemo、useCallback、useImperativeHandle 的依赖数组依赖了过多东西，可能导致代码难以维护

```js
const refresh = useCallback(() => {
  // ...
}, [name, searchState, address, status, personA, personB, progress, page, size]);
```

遇到这些情况， 思考一下这些 deps 是否真的都需要？

```js
function Example({id}) {
  const requestParams = useRef({});
  requestParams.current = {page: 1, size: 20, id};

  const refresh = useCallback(() => {
    doRefresh(requestParams.current);
  }, []);


  useEffect(() => {
    id && refresh();
  }, [id, refresh]); // 思考这里的 deps list 是否合理？
}
```

虽然 useEffect 的回调函数依赖了 id 和 refresh 方法，但是观察 refresh 方法可以发现，它在首次 render 被创建之后，永远不会发生改变了。因此，把它作为 useEffect 的 deps 是多余的。

其次，如果这些依赖真的都是需要的，那么这些逻辑是否应该放到同一个 hook 中？

```js
function Example({id, name, address, status, personA, personB, progress}) {
  const [page, setPage] = useState();
  const [size, setSize] = useState();

  const doSearch = useCallback(() => {
    // ...
  }, []);

  const doRefresh = useCallback(() => {
    // ...
  }, []);


  useEffect(() => {
    id && doSearch({name, address, status, personA, personB, progress});
    page && doRefresh({name, page, size});
  }, [id, name, address, status, personA, personB, progress, page, size]);
}
```

可以看出，在 useEffect 中有两段逻辑，这两段逻辑是相互独立的，因此可以将这两段逻辑放到不同 useEffect 中：

```js
useEffect(() => {
  id && doSearch({name, address, status, personA, personB, progress});
}, [id, name, address, status, personA, personB, progress]);

useEffect(() => {
  page && doRefresh({name, page, size});
}, [name,  page, size]);
```

如果还是多，还可以把依赖的值合并成一个整体

```js
const [filters, setFilters] = useState({
  name: "",
  address: "",
  status: "",
  personA: "",
  personB: "",
  progress: ""
});

useEffect(() => {
  id && doSearch(filters);
}, [id, filters]);
```

如果 state 不能合并，在 callback 内部又使用了 setState 方法，那么可以考虑使用 setState callback 来减少一些依赖。比如：

```js
const useValues = () => {
  const [values, setValues] = useState({
    data: {},
    count: 0
  });

  const [updateData] = useCallback(
      (nextData) => {
        setValues({
          data: nextData,
          count: values.count + 1 // 因为 callback 内部依赖了外部的 values 变量，所以必须在依赖数组中指定它
        });
      },
      [values],
  );

  return [values, updateData];
};
```

上面的代码中，必须在 useCallback 的依赖数组中指定 values，否则无法在 callback 中获取到最新的 values 状态。但是，通过 setState 回调函数，不用再依赖外部的 values 变量，因此也无需在依赖数组中指定它。就像下面这样：

```js
const useValues = () => {
  const [values, setValues] = useState({});

  const [updateData] = useCallback((nextData) => {
    setValues((prevValues) => ({
      data: nextData,
      count: prevValues.count + 1, // 通过 setState 回调函数获取最新的 values 状态，这时 callback 不再依赖于外部的 values 变量了，因此依赖数组中不需要指定任何值
    }));
  }, []); // 这个 callback 永远不会重新创建

  return [values, updateData];
};
```

最后，还可以通过 ref 来保存可变变量。以前只把 ref 用作保持 DOM 节点引用的工具，可 useRef Hook 能做的事情远不止如此。可以用它来保存一些值的引用，并对它进行读写。举个例子：

```js
const useValues = () => {
  const [values, setValues] = useState({});
  const latestValues = useRef(values);
  latestValues.current = values;

  const [updateData] = useCallback((nextData) => {
    setValues({
      data: nextData,
      count: latestValues.current.count + 1,
    });
  }, []);

  return [values, updateData];
};
```

在使用 ref 时要特别小心，因为它可以随意赋值，所以一定要控制好修改它的方法。特别是一些底层模块，在封装的时候千万不要直接暴露 ref，而是提供一些修改它的方法。

**总结**

依赖数组依赖的值最好不要超过 3 个，否则会导致代码会难以维护。

如果发现依赖数组依赖的值过多，应该采取一些方法来减少它。

- 去掉不必要的依赖。
- 将 Hook 拆分为更小的单元，每个 Hook 依赖于各自的依赖数组。
- 通过合并相关的 state，将多个依赖值聚合为一个。
- 通过 setState 回调函数获取最新的 state，以减少外部依赖。
- 通过 ref 来读取可变变量的值，不过需要注意控制修改它的途径。

> ## Q: 该不该使用 useMemo？

需要知道 useMemo 本身也有开销。useMemo 会「记住」一些值，同时在后续 render 时，将依赖数组中的值取出来和上一次记录的值进行比较，如果不相等才会重新执行回调函数，否则直接返回「记住」的值。这个过程本身就会消耗一定的内存和计算资源。因此，过度使用 useMemo 可能会影响程序的性能。

**一、应该使用 useMemo 的场景**

1. 保持引用相等

- 对于组件内部用到的 object、array、函数等，如果用在了其他 Hook 的依赖数组中，或者作为 props 传递给了下游组件，应该使用 useMemo。
- 自定义 Hook 中暴露出来的 object、array、函数等，都应该使用 useMemo 。以确保当值相同时，引用不发生变化。
- 使用 Context 时，如果 Provider 的 value 中定义的值（第一层）发生了变化，即便用了 Pure Component 或者 React.memo，仍然会导致子组件 re-render。这种情况下，仍然建议使用 useMemo 保持引用的一致性。

1. 成本很高的计算

- 比如 cloneDeep 一个很大并且层级很深的数据

```ts
// 在编写自定义 Hook 时，返回值一定要保持引用的一致性。
// 因为无法确定外部要如何使用它的返回值。如果返回值被用做其他 Hook 的依赖，并且每次 re-render 时引用不一致（当值相等的情况），就可能会产生 bug

function Example() {
  const data = useData();
  const [dataChanged, setDataChanged] = useState(false);

  useEffect(() => {
    setDataChanged((prevDataChanged) => !prevDataChanged); // 当 data 发生变化时，调用 setState。如果 data 值相同而引用不同，就可能会产生非预期的结果。
  }, [data]);

  console.log(dataChanged);

  return <ExpensiveComponent data={data} />;
}

const useData = () => {
  // 获取异步数据
  const resp = getAsyncData([]);

  // 处理获取到的异步数据，这里使用了 Array.map。因此，即使 data 相同，每次调用得到的引用也是不同的。

  // so 这里需要useMemo

  const mapper = (data) => data.map((item) => ({...item, selected: false}));

  return resp ? mapper(resp) : resp;
};

// 另外，由于引用的不同，也会导致 ExpensiveComponent 组件 re-render，产生性能问题


```

**二、无需使用 useMemo 的场景**

- 如果返回的值是原始值： string, boolean, null, undefined, number, symbol（不包括动态声明的 Symbol），一般不需要使用 useMemo。
- 仅在组件内部用到的 object、array、函数等（没有作为 props 传递给子组件），且没有用到其他 Hook 的依赖数组中，一般不需要使用 useMemo。

```ts
interface IExampleProps {
  page: number;
  type: string;
}

const Example = ({page, type}: IExampleProps) => {
  const resolvedValue = useMemo(() => {
    return getResolvedValue(page, type);
  }, [page, type]);

  return <ExpensiveComponent resolvedValue={resolvedValue}/>;
};

// 如果 getResolvedValue 的开销不大，并且 resolvedValue 返回一个字符串之类的原始值，那完全可以去掉 useMemo

interface IExampleProps {
  page: number;
  type: string;
}

const Example = ({page, type}: IExampleProps) => {
  const resolvedValue = getResolvedValue(page, type);
  return <ExpensiveComponent resolvedValue={resolvedValue}/>;
};
```

> ## Q: Hooks 能替代高阶组件和 Render Props 吗？

高阶组件采用了装饰器模式，让可以增强原有组件的功能，并且不破坏它原有的特性。例如：

```jsx
const RedButton = withStyles({
  root: {
    background: "red",
  },
})(Button);
```

在上面的代码中，希望保留 Button 组件的逻辑，但同时又想使用它原有的样式。因此，通过 withStyles 这个高阶组件注入了自定义的样式，并且生成了一个新的组件 RedButton。

**Render Props**

Render Props 通过父组件将可复用逻辑封装起来，并把数据提供给子组件。至于子组件拿到数据之后要怎么渲染，完全由子组件自己决定，灵活性非常高。而高阶组件中，渲染结果是由父组件决定的。Render Props 不会产生新的组件，而且更加直观的体现了「父子关系」

```jsx
<Parent>
  {(data) => {
    // 怎么渲染，完全由子组件自己决定
    return <Child data={data} />;
  }}
</Parent>
```

Render Props 作为 JSX 的一部分，可以很方便地利用 React 生命周期和 Props、State 来进行渲染，在渲染上有着非常高的自由度。同时，它不像 Hooks 需要遵守一些规则，可以放心大胆的在它里面使用 if / else、map 等各类操作。 **Antd design Form.List 组件 采用的就是这种方式**

**HOC 和 Render Props，可以相互转换**

```jsx
<RedButton>
  {(styles)=>(
    <Button styles={styles}/>
  )}
</RedButton>
```

## 总结

- 没有 Hooks 之前，高阶组件和 Render Props 本质上都是将复用逻辑提升到父组件中
- Hooks 出现之后，将复用逻辑提取到组件顶层，而不是强行提升到父组件中。这样就能够避免 HOC 和 Render Props 带来的「嵌套地狱」
- 但是，像 Context 的 <Provider/> 和 <Consumer/> 这样有父子层级关系（树状结构关系）的，还是只能使用 Render Props 或者 HOC

**使用场景**

- Hooks：提取复用逻辑。除了有明确父子关系的，其他场景都可以使用 Hooks
- Render Props：在组件渲染上拥有更高的自由度，可以根据父组件提供的数据进行动态渲染。适合有明确父子关系的场景。
- 高阶组件 HOC：适合用来做注入，并且生成一个新的可复用组件。适合用来写插件。

**能使用 Hooks 的场景还是应该优先使用 Hooks，其次才是 Render Props 和 HOC， 这 3 种也可以互相配合使用**

> ## Q: 使用 Hooks 时还有哪些好的实践？

1、若 Hook 类型相同，且依赖数组一致时，应该合并成一个 Hook。否则会产生更多开销。

```jsx
const dataA = useMemo(() => {
  return getDataA();
}, [A, B]);

const dataB = useMemo(() => {
  return getDataB();
}, [A, B]);

// 应该合并为

const [dataA, dataB] = useMemo(() => {
  return [getDataA(), getDataB()]
}, [A, B]);
```

2、参考原生 Hooks 的设计，自定义 Hooks 的返回值可以使用 Tuple 类型，更易于在外部重命名。但如果返回值的数量超过三个，还是建议返回一个对象。

```ts
export const useToggle = (defaultVisible: boolean = false) => {
  const [visible, setVisible] = useState(defaultVisible);
  const show = () => setVisible(true);
  const hide = () => setVisible(false);

  return [visible, show, hide] as [typeof visible, typeof show, typeof hide];
};

const [isOpen, open, close] = useToggle(); // 在外部可以更方便地修改名字
const [visible, show, hide] = useToggle();
```

3、ref 不要直接暴露给外部使用，而是提供一个修改值的方法

4、在使用 useMemo 或者 useCallback 时，确保返回的函数只创建一次。也就是说，函数不会根据依赖数组的变化而二次创建。举个例子：

```
export const useCount = () => {
  const [count, setCount] = useState(0);

  const [increase, decrease] = useMemo(() => {
    const increase = () => {
      setCount(count + 1);
    };

    const decrease = () => {
      setCount(count - 1);
    };
    return [increase, decrease];
  }, [count]);

  return [count, increase, decrease];
};
```

在 useCount Hook 中， count 状态的改变会让 useMemo 中的 increase 和 decrease 函数被重新创建。由于闭包特性，如果这两个函数被其他 Hook 用到了，应该将这两个函数也添加到相应 Hook 的依赖数组中，否则就会产生 bug。比如：

```jsx
function Counter() {
  const [count, increase] = useCount();

  useEffect(() => {
    const handleClick = () => {
      increase(); // 执行后 count 的值永远都是 1
    };

    document.body.addEventListener("click", handleClick);
    return () => {
      document.body.removeEventListener("click", handleClick);
    };
  }, []);

  return <h1>{count}</h1>;
}
```

在 useCount 中，increase 会随着 count 的变化而被重新创建。但是 increase 被重新创建之后， useEffect 并不会再次执行，所以 useEffect 中取到的 increase 永远都是首次创建时的 increase 。而首次创建时 count 的值为 0，因此无论点击多少次， count 的值永远都是 1。

那把 increase 函数放到 useEffect 的依赖数组中不就好了吗？事实上，这会带来更多问题：

- increase 的变化会导致频繁地绑定事件监听，以及解除事件监听。
- 需求是只在组件 mount 时执行一次 useEffect，但是 increase 的变化会导致 useEffect 多次执行，不能满足需求。

**如何解决这些问题呢？**

1. 通过 setState 回调，让函数不依赖外部变量。例如：

```jsx
export const useCount = () => {
  const [count, setCount] = useState(0);

  const [increase, decrease] = useMemo(() => {
    const increase = () => {
      setCount((latestCount) => latestCount + 1);
    };

    const decrease = () => {
      setCount((latestCount) => latestCount - 1);
    };
    return [increase, decrease];
  }, []); // 保持依赖数组为空，这样 increase 和 decrease 方法都只会被创建一次

  return [count, increase, decrease];
};
```

2. 通过 ref 来保存可变变量。例如：

```jsx
export const useCount = () => {
  const [count, setCount] = useState(0);
  const countRef = useRef(count);
  countRef.current = count;

  const [increase, decrease] = useMemo(() => {
    const increase = () => {
      setCount(countRef.current + 1);
    };

    const decrease = () => {
      setCount(countRef.current - 1);
    };
    return [increase, decrease];
  }, []); // 保持依赖数组为空，这样 increase 和 decrease 方法都只会被创建一次

  return [count, increase, decrease];
};
```

## 总结

1. 将完全不相关的 state 拆分为多组 state。
2. 如果某些 state 是相互关联的，或者需要一起发生改变，就可以把它们合并为一组 state。
3. 依赖数组依赖的值最好不要超过 3 个，否则会导致代码会难以维护。
4. 如果发现依赖数组依赖的值过多，应该采取一些方法来减少它。
	- 去掉不必要的依赖。
	- 将 Hook 拆分为更小的单元，每个 Hook 依赖于各自的依赖数组。
	- 通过合并相关的 state，将多个依赖值聚合为一个。
	- 通过 setState 回调函数获取最新的 state，以减少外部依赖。
	- 通过 ref 来读取可变变量的值，不过需要注意控制修改它的途径。

1. 应该使用 useMemo 的场景：

- 保持引用相等
- 成本很高的计算

1. 无需使用 useMemo 的场景：

- 如果返回的值是原始值： string, boolean, null, undefined, number, symbol（不包括动态声明的 Symbol），一般不需要使用 useMemo。
- 仅在组件内部用到的 object、array、函数等（没有作为 props 传递给子组件），且没有用到其他 Hook 的依赖数组中，一般不需要使用 useMemo。

1. Hooks、Render Props 和高阶组件都有各自的使用场景，具体使用哪一种要看实际情况。
2. 若 Hook 类型相同，且依赖数组一致时，应该合并成一个 Hook。
3. 自定义 Hooks 的返回值可以使用 Tuple 类型，更易于在外部重命名。如果返回的值过多，则不建议使用。
4. ref 不要直接暴露给外部使用，而是提供一个修改值的方法。
5. 在使用 useMemo 或者 useCallback 时，可以借助 ref 或者 setState callback，确保返回的函数只创建一次。也就是说，函数不会根据依赖数组的变化而二次创建。

参考文章：

[You’re overusing useMemo: Rethinking Hooks memoization](https://blog.logrocket.com/rethinking-hooks-memoization/?from=singlemessage&isappinstalled=0)
