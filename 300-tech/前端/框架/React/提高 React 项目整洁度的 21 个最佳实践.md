---
title: 提高 React 项目整洁度的 21 个最佳实践
date: 2023-08-04T04:21:00+08:00
updated: 2024-08-21T10:33:30+08:00
dg-publish: false
aliases:
  - 提高 React 项目整洁度的 21 个最佳实践
author: Luwang
category: web
comments: true
cover: //cdn.wallleap.cn/img/post/1.jpg
description: 文章描述
image-auto-upload: true
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
rating: 1
source: #
tags:
  - React
  - web
url: //myblog.wallleap.cn/post/1
---

React 在如何组织结构方面非常开放。这正是为什么我们有责任保持项目的整洁和可维护性。

今天，我们将讨论一些改善 React 应用程序健康状况的最佳实践。这些规则被广泛接受。因此，掌握这些知识至关重要。

所有内容都将以代码展示，所以做好准备！

## 1\. 使用 JSX 简写

尝试使用 JSX 简写来传递 Boolean 布尔值变量。假设您想要控制导航栏组件的标题可见性。

### Bad ❌

```
return <Navbar showTitle={true} />;
```

### Good ✔

```
return <Navbar showTitle />;
```

## 2\. 使用三元运算符

假设你想根据角色显示用户的详细信息。

### Bad ❌

```
const { role } = user;

if (role === ADMIN) {
  return <AdminUser />;
} else {
  return <NormalUser />;
}
```

### Good ✔

```
const { role } = user;

return role === ADMIN ? <AdminUser /> : <NormalUser />;
```

## 3\. 利用对象字面量

对象字面量可以帮助我们的代码更具可读性。假设你想根据角色显示三种类型的用户。不能使用三元，因为选项数量超过两个。

### Bad ❌

```
const { role } = user;

switch (role) {
  case ADMIN:
    return <AdminUser />;
  case EMPLOYEE:
    return <EmployeeUser />;
  case USER:
    return <NormalUser />;
}
```

### Good ✔

```
const { role } = user;

const components = {
  ADMIN: AdminUser,
  EMPLOYEE: EmployeeUser,
  USER: NormalUser,
};

const Component = components[role];

return <Component />;
```

现在看起来好多了。

## 4\. 使用 Fragments 语法

始终使用 Fragment 而不是 Div。它可以保持代码整洁，并且也有利于性能，因为在虚拟 DOM 中创建的节点少了一个。

### Bad ❌

```
return (
  <div>
    <Component1 />
    <Component2 />
    <Component3 />
  </div>
);
```

### Good ✔

```
return (
  <>
    <Component1 />
    <Component2 />
    <Component3 />
  </>
);
```

## 5\. 不要在渲染中定义函数

不要在渲染中定义函数。尝试将渲染内部的逻辑保持在绝对最低限度。

### Bad ❌

```
return (
  <button onClick={() => dispatch(ACTION_TO_SEND_DATA)}>
    {" "}
    // NOTICE HERE This is a bad example
  </button>
);
```

### Good ✔

```
const submitData = () => dispatch(ACTION_TO_SEND_DATA);

return <button onClick={submitData}>This is a good example</button>;
```

## 使用 Memo

React.PureComponent 和 Memo 可以显着提高应用程序的性能。它们帮助我们避免不必要的渲染。

### Bad ❌

```
import React, { useState } from "react";

export const TestMemo = () => {
  const [userName, setUserName] = useState("faisal");
  const [count, setCount] = useState(0);

  const increment = () => setCount((count) => count + 1);

  return (
    <>
      <ChildrenComponent userName={userName} />
      <button onClick={increment}> Increment </button>
    </>
  );
};

const ChildrenComponent = ({ userName }) => {
  console.log("rendered", userName);
  return <div> {userName} </div>;
};
```

尽管子组件应该只渲染一次，因为 count 的值与 ChildComponent 无关。但是，每次单击按钮时它都会呈现。 ![](https://cdn.wallleap.cn/img/pic/illustration/202308041622589.jpeg)

### Good ✔

让我们将 ChildrenComponent 编辑为：

```
import React, { useState } from "react";

const ChildrenComponent = React.memo(({ userName }) => {
  console.log("rendered");
  return <div> {userName}</div>;
});
```

现在，无论您单击该按钮多少次，它只会在必要时呈现。

## 7\. 将 CSS 放入 JavaScript 中

编写 React 应用程序时避免使用原始 JavaScript，因为组织 CSS 比组织 JS 困难得多

### Bad ❌

```
// CSS FILE

.body {
  height: 10px;
}

//JSX

return <div className='body'>

</div>
```

### Good ✔

```
const bodyStyle = {
  height: "10px",
};

return <div style={bodyStyle}></div>;
```

## 8\. 使用对象解构

使用对象解构对你有利。假设你需要显示用户的详细信息。

### Bad ❌

```
return (
  <>
    <div> {user.name} </div>
    <div> {user.age} </div>
    <div> {user.profession} </div>
  </>
);
```

### Good ✔

```
const { name, age, profession } = user;

return (
  <>
    <div> {name} </div>
    <div> {age} </div>
    <div> {profession} </div>
  </>  
)
```

## 9\. 字符串参数不需要大括号

将字符串作为参数传递给子组件时。

### Bad ❌

```
return <Navbar title={"My Special App"} />;
```

### Good ✔

```
return <Navbar title="My Special App" />;
```

## 10\. 从 JSX 中删除 JS 代码

如果任何 JS 代码不能用于渲染或 UI 功能，请将其移出 JSX。

### Bad ❌

```
return (
  <ul>
    {posts.map((post) => (
      <li
        onClick={(event) => {
          console.log(event.target, "clicked!"); // <- THIS IS BAD
        }}
        key={post.id}
      >
        {post.title}
      </li>
    ))}
  </ul>
);

```

### Good ✔

```
const onClickHandler = (event) => {
  console.log(event.target, "clicked!");
};

return (
  <ul>
    {posts.map((post) => (
      <li onClick={onClickHandler} key={post.id}>
        {post.title}
      </li>
    ))}
  </ul>
);
```

## 11\. 使用模板字符串

使用模板文字构建大字符串。避免使用字符串连接。它又漂亮又干净。

### Bad ❌

```
const userDetails = user.name + "'s profession is" + user.proffession;

return <div> {userDetails} </div>;
```

### Good ✔

```
const userDetails = `${user.name}'s profession is ${user.proffession}`;

return <div> {userDetails} </div>;
```

## 12\. 按顺序引入

始终尝试按特定顺序导入内容。它提高了代码的可读性。

### Bad ❌

```
import React from "react";
import ErrorImg from "../../assets/images/error.png";
import styled from "styled-components/native";
import colors from "../../styles/colors";
import { PropTypes } from "prop-types";
```

### Good ✔

经验法则是保持导入顺序如下：

- 内置;
- 外部的;
- 自己编写的;

所以上面的例子就变成了:

```
import React from "react";

import { PropTypes } from "prop-types";
import styled from "styled-components/native";

import ErrorImg from "../../assets/images/error.png";
import colors from "../../styles/colors";
```

## 13\. 使用隐式返回

使用 JavaScript 的隐式返回功能来编写漂亮的代码。假设您的函数执行简单的计算并返回结果。

### Bad ❌

```
const add = (a, b) => {
  return a + b;
};
```

### Good ✔

```
const add = (a, b) => a + b;
```

## 14\. 组件命名

始终对组件使用 PascalCase，对实例使用 CamelCase。

### Bad ❌

```
import reservationCard from "./ReservationCard";

const ReservationItem = <ReservationCard />;
```

### Good ✔

```
import ReservationCard from "./ReservationCard";

const reservationItem = <ReservationCard />;
```

## 15\. 保留属性命名

不要使用 DOM 组件的属性名称用于在组件之间传递 props，因为其他人可能不会预期这些名称。

### Bad ❌

```
<MyComponent style="dark" />

<MyComponent className="dark" />
```

### Good ✔

```
<MyComponent variant="fancy" />
```

## 16\. 引号

对 JSX 属性使用双引号，对所有其他 JS 使用单引号。

### Bad ❌

```
<Foo bar='bar' />

<Foo style={{ left: "20px" }} />
```

### Good ✔

```
<Foo bar="bar" />

<Foo style={{ left: '20px' }} />
```

## 17\. Prop 参数命名

如果 prop 值是 React 组件，则始终使用驼峰命名法作为 prop 名称或 PascalCase。

### Bad ❌

```
<Component UserName="hello" phone_number={12345678} />
```

### Good ✔

```
<MyComponent
  userName="hello"
  phoneNumber={12345678}
  Component={SomeComponent}
/>
```

## 18\. 括号中的 JSX

如果您的组件跨越一行以上，请始终将其括在括号中。

### Bad ❌

```
return (
  <MyComponent variant="long">
    <MyChild />
  </MyComponent>
);
```

![](https://cdn.wallleap.cn/img/pic/illustration/202308041622591.jpeg)

> 我这破垃圾编辑器自动格式化了,根部不存在这个问题哈哈哈哈哈

### Good ✔

```
return (
  <MyComponent variant="long">
    <MyChild />
  </MyComponent>
);
```

## 19\. 自动闭合标签

如果您的组件没有任何子组件，请使用自闭合标签。它提高了可读性。

### Bad ❌

```
<SomeComponent variant="stuff"></SomeComponent>
```

### Good ✔

```
<SomeComponent variant="stuff" />
```

## 20\. 方法名称中的下划线

不要在任何内部 React 方法中使用下划线。

### Bad ❌

```
const _onClickHandler = () => {
  // do stuff
};
```

### Good ✔

```
const onClickHandler = () => {
  // do stuff
};
```

## 21\. 替代文本

在你的 `<img>` 标签中始终要包括 alt 属性。不要在 alt 属性中使用 "picture" 或 "image"，因为屏幕阅读器已经默认将 `<img>` 元素识别为图像，无需重复说明。

### Bad ❌

```
<img src="hello.jpg" />

<img src="hello.jpg" alt="Picture of me rowing a boat" />
```

### Good ✔

```
<img src="hello.jpg" alt="Me waving hello" />
```

## 参考资源

- [21 Best Practices for a Clean React Project](https://link.juejin.cn/?target=https%3A%2F%2Fdev.to%2Fmohammadfaisal%2F21-best-practices-for-a-clean-react-project-jdf "https://dev.to/mohammadfaisal/21-best-practices-for-a-clean-react-project-jdf")

## 总结

就这样吧。如果你已经做到了这一步，那么恭喜您！我希望你能从本文中学到一两件事。

最后分享两个我的两个开源项目,它们分别是:

- [前端脚手架 create-neat](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fxun082%2Freact-cli "https://github.com/xun082/react-cli")
- [在线代码协同编辑器](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fxun082%2Fonline-cooperative-edit "https://github.com/xun082/online-cooperative-edit")

这两个项目都会一直维护的,如果你也喜欢,欢迎 star 🥰🥰🥰
