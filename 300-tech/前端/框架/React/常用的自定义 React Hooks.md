---
title: 常用的自定义 React Hooks
date: 2024-08-22T01:58:17+08:00
updated: 2024-08-22T01:58:38+08:00
dg-publish: false
source: https://github.com/peng-yin/note/issues/45
---

## react hooks 核心 API 使用注意事项

> 常用的 hooks 主要有:

- useState
- useEffect
- useCallback
- useMemo
- useRef

> 构建小型 redux

- useReducer
- useContext
- createContext

**需要考虑渲染性能，我们知道如果在不做任何处理时，我们在函数组件中使用 setState 都会导致组件内部重新渲染**

### 场景一

当我们在容器组件手动更新了任何 state 时，容器内部的各个子组件都会重新渲染，为了避免这种情况出现，我们一般都会使用 memo 将函数组件包裹，来达到 class 组件的 pureComponent 的效果：

```js
import React, { memo, useState, useEffect } from 'react';
const A = (props) => {
  console.log('A1');
  useEffect(() => {
    console.log('A2');
  });
  return <div>A</div>;
};

const B = memo((props) => {
  console.log('B1');
  useEffect(() => {
    console.log('B2');
  });
  return <div>B</div>;
});

const Home = (props) => {
  const [a, setA] = useState(0);
  useEffect(() => {
    console.log('start');
    setA(1);
  }, []);
  return (
    <div>
      <A n={a} />
      <B />
    </div>
  );
};
```

当我们将 B 用 memo 包裹后，状态 a 的更新将不会导致 B 组件重新渲染。其实仅仅优化这一点还远远不够的，比如说我们**子组件用到了容器组件的某个变量或者函数，那么当容器内部的 state 更新之后，这些变量和函数都会重新赋值，这样就会导致即使子组件使用了 memo 包裹也还是会重新渲染**，那么这个时候我们就需要使用**useMemo**和**useCallback**了。

**useMemo**可以帮我们将变量缓存起来，**useCallback**可以缓存回调函数，它们的第二个参数和 useEffect 一样，是一个依赖项数组，通过配置依赖项数组来决定是否更新。

```js
import React, { memo, useState, useEffect, useMemo } from 'react';
const Home = (props) => {
  const [a, setA] = useState(0);
  const [b, setB] = useState(0);
  useEffect(() => {
    setA(1);
  }, []);

  const add = useCallback(() => {
    console.log('b', b);
  }, [b]);

  const name = useMemo(() => {
    return b + 'xuxi';
  }, [b]);
  return (
    <div>
      <A n={a} />
      <B add={add} name={name} />
    </div>
  );
};
```

此时 a 更新后 B 组件不会再重新渲染。以上几个优化步骤主要是用来优化组件的渲染性能，我们平时还会涉及到获取组件 dom 和使用内部闭包变量的情景，这个时候我们就可以使用**useRef**。

**useRef**返回一个可变的 ref 对象，其 .current 属性被初始化为传入的参数（initialValue）。返回的 ref 对象在组件的整个生命周期内保持不变。

```js
function AutoFocusIpt() {
  const inputEl = useRef(null);
  const useEffect(() => {
    // `current` 指向已挂载到 DOM 上的文本输入元素
    inputEl.current.focus();
  }, []);
  return (
    <>
      <input ref={inputEl} type="text" />
    </>
  );
}
```

## 实现 redux

```js
// actionType.js
const actionType = {
  INSREMENT: 'INSREMENT',
  DECREMENT: 'DECREMENT',
  RESET: 'RESET',
};
export default actionType;

// actions.js
import actionType from './actionType';
const add = (num) => ({
  type: actionType.INSREMENT,
  payload: num,
});

const dec = (num) => ({
  type: actionType.DECREMENT,
  payload: num,
});

const getList = (data) => ({
  type: actionType.GETLIST,
  payload: data,
});
export { add, dec, getList };

// reducer.js
function init(initialCount) {
  return {
    count: initialCount,
    total: 10,
    user: {},
    article: [],
  };
}

function reducer(state, action) {
  switch (action.type) {
    case actionType.INSREMENT:
      return { count: state.count + action.payload };
    case actionType.DECREMENT:
      return { count: state.count - action.payload };
    case actionType.RESET:
      return init(action.payload);
    default:
      throw new Error();
  }
}

export { init, reducer };

// redux.js
import React, { useReducer, useContext, createContext } from 'react';
import { init, reducer } from './reducer';

const Context = createContext();
const Provider = (props) => {
  const [state, dispatch] = useReducer(reducer, props.initialState || 0, init);
  return <Context.Provider value={{ state, dispatch }}>{props.children}</Context.Provider>;
};

export { Context, Provider };
```

## useDebounce（防抖）

```js
import { useEffect, useRef } from 'react';

const useDebounce = (fn, ms = 30, deps = []) => {
  let timeout = useRef();
  useEffect(() => {
    if (timeout.current) clearTimeout(timeout.current);
    timeout.current = setTimeout(() => {
      fn();
    }, ms);
  }, deps);

  const cancel = () => {
    clearTimeout(timeout.current);
    timeout = null;
  };

  return [cancel];
};

export default useDebounce;
```

**useDebounce**接受三个参数，分别为回调函数，时间间隔以及依赖项数组，它暴露了 cancel API，主要是用来控制何时停止防抖函数用的。具体使用如下：

```js
// ...
import { useDebounce } from 'hooks';
const Home = (props) => {
  const [a, setA] = useState(0);
  const [b, setB] = useState(0);
  const [cancel] = useDebounce(
    () => {
      setB(a);
    },
    2000,
    [a],
  );

  const changeIpt = (e) => {
    setA(e.target.value);
  };
  return (
    <div>
      <input type="text" onChange={changeIpt} />
      {b} {a}
    </div>
  );
};
```

## useThrottle(节流)

```js
import { useEffect, useRef, useState } from 'react';

const useThrottle = (fn, ms = 30, deps = []) => {
  let previous = useRef(0);
  let [time, setTime] = useState(ms);
  useEffect(() => {
    let now = Date.now();
    if (now - previous.current > time) {
      fn();
      previous.current = now;
    }
  }, deps);

  const cancel = () => {
    setTime(0);
  };

  return [cancel];
};

export default useThrottle;
```

## 实现自定义 useTitle

```js
import { useEffect } from 'react';

const useTitle = (title) => {
  useEffect(() => {
    document.title = title;
  }, []);

  return;
};

export default useTitle;

// 使用
const Home = () => {
  // ...
  useTitle('趣谈前端');

  return <div>home</div>;
};
```

## 实现自定义的 useScroll

```js
// 监听一个元素滚动位置的变化来决定展现那些内容

import { useState, useEffect } from 'react';

const useScroll = (scrollRef) => {
  const [pos, setPos] = useState([0, 0]);

  useEffect(() => {
    function handleScroll(e) {
      setPos([scrollRef.current.scrollLeft, scrollRef.current.scrollTop]);
    }
    scrollRef.current.addEventListener('scroll', handleScroll, false);
    return () => {
      scrollRef.current.removeEventListener('scroll', handleScroll, false);
    };
  }, []);

  return pos;
};

export default useScroll;
```

由以上代码可知，我们在钩子函数里需要传入一个元素的引用，这个我们可以在函数组件中采用 ref 和 useRef 来获取到，钩子返回了滚动的 x，y 值，即滚动的左位移和顶部位移，具体使用如下：

```js
import React, { useRef } from 'react';

import { useScroll } from 'hooks';
const Home = (props) => {
  const scrollRef = useRef(null);
  const [x, y] = useScroll(scrollRef);

  return (
    <div>
      <div ref={scrollRef}>
        <div className="innerBox"></div>
      </div>
      <div>
        {x}, {y}
      </div>
    </div>
  );
};
```

## useUpdate(实现组件的强制更新)

```js
import { useState } from 'react'

const useUpdate = () => {
    const [, setFlag] = useState()
    const update = () => {
        setFlag(Date.now())
    }
  
    return update
  }

export default useUpdate
```
