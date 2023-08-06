---
title: 2023 年 8 个最佳 React UI 组件库和框架
date: 2023-08-03 17:54
updated: 2023-08-03 18:00
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - 2023 年 8 个最佳 React UI 组件库和框架
rating: 1
tags:
  - React
  - web
category: web
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: #
url: //myblog.wallleap.cn/post/1
---

> 将展示八个最好的 React UI 组件库和框架，如下表所示：

- **React Bootstrap**：一个与 Bootstrap 框架集成的实用的 React UI 库。
- **Grommet**：如果您想在设计中实现可访问性，这个 React UI 组件库非常有用。
- **Blueprint**：对于桌面 React 应用程序，您需要查看 Blueprint 的产品。
- **Ant Design**：这个库的设计重点是与用户的联系：这是每个设计师都想实现的。
- **Onsen UI React**：如果你想构建一个移动 React 应用程序，这个库将涵盖 UI 设计。
- **Rebass**：这个库的独特之处在于你可以在你的代码中使用样式化的道具，而不需要打开第二个样式表。
- **Semantic UI React**：顾名思义，这个 React 库集成了语义 UI 开发框架。
- **MUI**：为了实现类似 Google 的 Material Design 的外观，MUI 是一个很好的选择——特别是考虑到它的实现简洁明了。

> 事实上，库的数量远不止八个。但是，我们认为这些代表了市场上最高的质量。更重要的是，它们没有任何顺序——所以请随意阅读每一个，看看它们是如何比较的。

## 1.React Bootstrap

首先，我们有 React Bootstrap。这是较旧的 React UI 库之一，这意味着您为 UI 设计奠定了良好的基础。

![](https://cdn.wallleap.cn/img/pic/illustration/202308031755495.png)

> 该库具有以下特点：

1. 使用 TypeScript；
2. 使用 Bootstrap 样式表，将能够将它与 Bootstrap 主题一起使用。

> 你可以通过如下指令快速引入该包

```sh
npm install react-bootstrap bootstrap
```

> React Bootstrap 使用“variants”来创建不同的元素：

```jsx
function TypesExample() {
  return (
    <>
      <Button variant="primary">Primary</Button>{' '}
      <Button variant="secondary">Secondary</Button>{' '}
      <Button variant="success">Success</Button>{' '}
      <Button variant="warning">Warning</Button>{' '}
      <Button variant="danger">Danger</Button>{' '}
      <Button variant="info">Info</Button>{' '}
      <Button variant="light">Light</Button>{' '}
      <Button variant="dark">Dark</Button>
      <Button variant="link">Link</Button>
    </>
  );
}

export default TypesExample;
```

此示例将创建一系列的样式按钮：

![](https://cdn.wallleap.cn/img/pic/illustration/202308031755496.png)

总的来说，React Bootstrap 使用起来很直观，并且会帮助创建看起来很棒的 UI 元素。

## 2.Grommet

> 在我们最好的 React UI 组件库和框架列表中，下一个是 Grommet。这提倡简化的方法，如果将它与 React Bootstrap 进行比较，它会提供更多的功能。

![](https://cdn.wallleap.cn/img/pic/illustration/202308031755497.png)

> 该框架不需要很长的设计周期就可以开始工作。例如，您有 Grommet Themer 来帮助您将组件库与您的配色方案相匹配。更重要的是，您拥有专用的 Grommet Designer，它使用简单的构建器来创建您的组件设计。

使用 npm 或 Yarn 可以轻松安装：

```sh
npm install grommet grommet-icons styled-components --save
```

> 可以使用以下方式创建一个简单的光滑按钮

```jsx
export default () => (
  <SandboxComponent>
    <Button label='Submit' onClick={() => {}} />
  </SandboxComponent>
);
```

![](https://cdn.wallleap.cn/img/pic/illustration/202308031755498.png)

总而言之，我们喜欢 Grommet 的酷炫默认设计、可访问性特性和其他设计工具。

## 3.Blueprint UI

> 如果您想要一个简洁又实用的 React UI 组件库，Blueprint UI 可能是适合您的工具包。

![](https://cdn.wallleap.cn/img/pic/illustration/202308031755499.png)

> 你不会希望在移动优先的应用程序中使用 Blueprint。这是一种开发在浏览器中运行的桌面应用程序的方法: 复杂度越高越好！

> 你可以通过如下指令快速引入该包

```sh
npm add @blueprintjs/core react react-dom
```

> 通过以下代码可以创建一个按钮：

```jsx
<Button intent="success" text="button content" onClick={incrementCounter} />
```

![](https://cdn.wallleap.cn/img/pic/illustration/202308031755500.png)

我们认为 Blueprint UI 是一种简单易用的工具，非常适合快速启动设计。

## 4.Ant Design

> Ant Design 自称是世界上第二受欢迎的 React UI 框架。即便如此，它仍然是你项目的首选。

![](https://cdn.wallleap.cn/img/pic/illustration/202308031755501.png)

> Ant Design 的设计理念就是清晰。您可以在其默认设计和工具包含中均可以看到这一点。例如，你有一个前端主题工具，还有丰富而现代的组件，看起来很漂亮。

> 你可以通过如下指令快速引入该包

```sh
npm install antd
```

> 要创建一个按钮，您只需要最少的行数：

```jsx
import { Button } from 'antd';

const App = () => (
  <>
    <Button type="primary">PRESS ME</Button>
  </>
);
```

结果是一个直观、简单且简洁的按钮：

![](https://cdn.wallleap.cn/img/pic/illustration/202308031755502.png)

总的来说，Ant Design 可以帮助你创建现代设计，在我们看来，它是最好的 React UI 组件库和框架之一。

## 5\. Onsen UI React

> Onsen UI React 是移动应用程序的组件库。

![](https://cdn.wallleap.cn/img/pic/illustration/202308031755503.png)

> 它同时支持 Android 和 iOS，这意味着您拥有适用于 Material 和 Flat 设计的专用组件。更好的是，Onsen UI React 将自动检测您的设计适用于哪个平台并相应地进行调整。

> 通过以下指令将引入该包

```sh
npm install onsenui react-onsenui --save
```

> 您将使用 `VOns<element> ` 组件和修饰符来创建元素：

```jsx
<v-ons-button>Normal</v-ons-button>
<v-ons-button modifier="quiet">Quiet</v-ons-button>
<v-ons-button modifier="outline">Outline</v-ons-button>
<v-ons-button modifier="cta">Call to action</v-ons-button>
<v-ons-button modifier="large">Large</v-ons-button>
```

加上一些样式，您将有一些漂亮的按钮可以添加到您的项目中：

![](https://cdn.wallleap.cn/img/pic/illustration/202308031755504.png)

对于移动应用程序，您找不到许多容易使用的库：所以强烈推荐使用该 React UI 组件库。

## 6\. Rebass

> 样式对于任何 UI 设计来说显然都非常重要。Rebass 试图使用 styled props，以便您可以将其编码到 React UI 中。

![](https://cdn.wallleap.cn/img/pic/illustration/202308031755505.png)

> Rebass 这个 React UI 库的设计理念是尽量减少需要编写的 CSS 代码。这样可以让开发者在开发过程中更接近最终的设计效果，而不需要进行第二轮的 CSS 样式调整。此外，Rebass 提供了一组原始组件，这些组件也具有良好的外观效果，并且整个库非常轻量级。因此，Rebass 非常灵活、可扩展，而且容易与您的项目集成。

> 通过以下指令引入该包：

```sh
npm i rebass
```

> 通过以下方式添加一个按钮：

```jsx
<Button variant='primary' mr={2}>Primary</Button>
```

![](https://cdn.wallleap.cn/img/pic/illustration/202308031755506.png)

Rebass for React 类似于 Bootstrap for CSS，但具有更好的标记和一流的设计选项。该库将帮助您进行组件设计，而不是 HTML 和 CSS 设计。

## 7\. Semantic UI React

> 与 React Bootstrap 非常相似，Semantic UI React 是其父开发框架的扩展。

![](https://cdn.wallleap.cn/img/pic/illustration/202308031755507.png)

> 如果你选择使用 Semantic UI，这将是最好的 React UI 组件库和框架之一。更好的是，您将能够快速集成它，并且使用起来很直观。此外，Semantic UI 还提供了一些强大的功能，例如增强、简写属性和自动控制状态。这意味着您可以创建组件，并且它们将自我管理其状态，无需您的输入。

> 通过以下指令引入该包：

```sh
npm install semantic-ui-react semantic-ui-css
```

> 简单使用

```jsx
const ButtonExampleButton = () => <Button>Click Here</Button>
```

![](https://cdn.wallleap.cn/img/pic/illustration/202308031755508.png)

如果您选择使用 Semantic UI，这个 React UI 组件库将最适合您的项目。

## 8.MUI

> 几年前，Google's Material Design 曾出现在公众视野中一段时间。这个想法是将布局和设计选择标准化,以达到 Google 认为的“corrent”。在不讨论这种方法的优缺点的情况下，MUI 是最好的 React UI 组件库和框架之一，可以帮助您以这种风格进行创建。

![](https://cdn.wallleap.cn/img/pic/illustration/202308031755509.png)

> MUI 是一个工具箱而不是一个简单的库。例如，您有 MUI Core，还有用于高级用例的 MUI X。当您需要树视图、数据选择器、数据网格等时，这就是您要使用的工具。此外，还提供了 UI 布局模板和设计工具包可以帮助您完成项目。

> 引入所需要的包：

```sh
npm install @mui/material @emotion/react @emotion/styled
```

> 简单使用

```jsx
<Button variant="contained">Contained</Button>
```

![](https://cdn.wallleap.cn/img/pic/illustration/202308031755510.png)

考虑到可用性、实现和结果的综合因素，选择 MUI 是没有错的。如果您需要将应用程序与 Google 的设计风格进行整合，那么 MUI 是理想的选择。

## 总的来说

> React 是用来帮助你加快编码速度的的一个库，React UI 能够进一步提速，如果你基于 Google 的设计原则进行设计可以使用 MUI；如果开发面向移动设备的 UI 设计可以使用 Onsen UI React；如果开发桌面应用程序可以首选 Blueprint UI。库虽然很多，但是大部分人首选还是 Antd。

参考文档：<https://www.designbombs.com/best-react-ui-component-libraries-frameworks/>
