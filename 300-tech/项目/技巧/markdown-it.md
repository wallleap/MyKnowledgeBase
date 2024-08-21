---
title: markdown-it
date: 2024-03-25T09:27:00+08:00
updated: 2024-08-21T10:32:44+08:00
dg-publish: false
---

Markdown-it 是一个 javascript markdown 解析器，导出一个函数，可以生成纯 HTML，人们可以随意使用。如 GitHub 页面所述，它还具有大量插件和功能。

用它制作一个 Vue 组件非常容易，唯一的要求是在设置一些参数后安装和导入 `markdown-it` 函数并使用该 `render` 方法。

## **要求和插件**

首先要做的事情 - 安装依赖项。

最重要的是 markdown-it 本身：

```sh
npm i markdown-it
```

或者

```sh
yarn add markdown-it
```

推荐其他更有用的插件，因为 markdown-it 默认不支持某些功能：

- markdown-it-abbr - 缩写
- markdown-it-anchor - 锚点的自动 ID，可用于为页面的某些部分添加书签。
- markdown-it-footnote
- markdown-it-highlightjs - 对语法高亮非常有用
- markdown-it-sub - 下标
- markdown-it-sup - 上标
- markdown-it-task-lists - 复选框的格式化列表

## **编写组件**

导入 `markdown-it` 包及其插件后，可以使用 render 方法生成输出：

```python
import MarkdownIt from "markdown-it";

const markdown = new MarkdownIt();

markdown.render("# Header");
```

因此，它应该产生以下结果：

```python
<h1>Header</h1>
```

你也可以把之前提到的插件放进去：

```python
const markdown = new MarkdownIt()
  .use(MarkdownItAbbr)
  .use(MarkdownItAnchor)
  .use(MarkdownItFootnote)
  .use(MarkdownItHighlightjs)
  .use(MarkdownItSub)
  .use(MarkdownItSup)
  .use(MarkdownItTasklists);
```

剩下的只是定义一个字符串属性，组件应该使用它来显示所需的文本并使用常规的 Vue 语法：

```python
<template>
  <div v-html="markdown.render(source)" />
</template>

<script setup lang="ts">
import MarkdownIt from "markdown-it";
import MarkdownItAbbr from "markdown-it-abbr";
import MarkdownItAnchor from "markdown-it-anchor";
import MarkdownItFootnote from "markdown-it-footnote";
import MarkdownItHighlightjs from "markdown-it-highlightjs";
import MarkdownItSub from "markdown-it-sub";
import MarkdownItSup from "markdown-it-sup";
import MarkdownItTasklists from "markdown-it-task-lists";
import MarkdownItTOC from "markdown-it-toc-done-right";

const markdown = new MarkdownIt()
  .use(MarkdownItAbbr)
  .use(MarkdownItAnchor)
  .use(MarkdownItFootnote)
  .use(MarkdownItHighlightjs)
  .use(MarkdownItSub)
  .use(MarkdownItSup)
  .use(MarkdownItTasklists)
  .use(MarkdownItTOC);

defineProps({
  source: {
    type: String,
    default: ""
  }
});
</script>
```

并这样使用：

```python
<MarkdownRenderer :source="'# Hi!!'" />
```

## 自定义语法

1. [markdown-it 源码分析及插件编写：parse 和 token（1/3）](https://zhuanlan.zhihu.com/p/400036665)
2. [markdown-it 源码分析及插件编写：render（2/3）](https://zhuanlan.zhihu.com/p/401550182)
3. markdown-it 源码分析及插件编写：Plugin 插件编写（3/3）

首先强调，最简单的方法永远是模仿：你可以在 [NPM~search:markdown-it-plugin](https://www.npmjs.com/search?q=keywords:markdown-it-plugin) 上找到很多插件，然后模仿它们的实现。

回顾前两篇文章，我们可以知道 markdown-it 如下的执行流程：

- Parse 的 核心规则链
- Block 规则链
- List 规则
- Fence 规则
- 等等
- Inline
- 图片规则
- 强调规则
- 等等等
- 等等
- Render

由于 markdown-it 插件并没有特定的规则，所以我们需要发挥自己的想象力来实现自己想要的插件。

从上面的流程可以看出，我们至少可以在四个思路来实现插件：

1. 修改现有的规则
2. 添加新的规则
3. 修改现有的 render
4. 添加新的 render

## 修改现有规则

修改现有规则使用 `Ruler.prototype.at` 方法，定义在 `lib\ruler.js` 中。这个方法很简单，留给读者作为习题了。

## 添加新的规则

在添加新的规则前，我们首先需要明确一点，我们需要把规则添加到哪个规则链中。

从最上面的结构我们可以看到，我们有很多规则链，分别是：

- core
- block
- inline（游前文可知，包含 `ruler` 和 `ruler2`）

我们以 core 为例：

```js
[
  [ 'normalize',      require('./rules_core/normalize')      ],
  [ 'block',          require('./rules_core/block')          ],
  [ 'inline',         require('./rules_core/inline')         ],
  [ 'linkify',        require('./rules_core/linkify')        ],
  [ 'replacements',   require('./rules_core/replacements')   ],
  [ 'smartquotes',    require('./rules_core/smartquotes')    ]
];
```

上面的规则定义在 `lib\parse_core.js` 中，我们可以看到，这个规则链中包含了六个规则。

假设我们想在规则 `block` 前添加一个名为 `'jks-github-task-lists'` 规则，我们可以这样实现：

```ts
  md.core.ruler.before('inline', 'jks-github-task-lists', function (state: StateCore): boolean {
      const tokens = state.tokens;
      // ...
      // Just make the type checking happy
      return true;
  });
```

- 注 1：上面例子节选自真实的插件：参见 [https://github.com/jks-liu/markdown-it-zhihu-common/blob/master/src/index.ts](https://github.com/jks-liu/markdown-it-zhihu-common/blob/master/src/index.ts) 。
- 注 2：上面例子用的是 TypeScript 的语法，请自行类比到 JS。

同理，可以使用 `md.core.ruler.after` 添加规则到某个规则的后面。

## 修改/添加 `render` 方法

修改或添加 `render` 方法的实例你可以在 [上篇文章](https://zhuanlan.zhihu.com/p/401550182) 的最后找到。

再强调一下重点

> 然后再看一个类型有没有特殊规则：`typeof rules[type] !== 'undefined'`。如果 rule 存在（即不为 `undefined`）,则使用特定的规则。

修改/添加 `render` 方法，就是修改/添加上面所说的特殊规则。添加/修改规则，需要知道 token 的类型（`type`）,你可以在 [https://markdown-it.github.io/](https://markdown-it.github.io/) 中找到查看，当然更快的方法是打 log 看看。

我们以 [第一篇文章](https://zhuanlan.zhihu.com/p/400036665) 中的一个 token 为例，`type` 的名字其实是很有规律的：

> {
> type: "strong_open",
> tag: "strong",
> nesting: 1,
> markup: "**",
> },

要为它添加一个特殊的 render 规则，我们可以这样实现：

```js
md.renderer.rules.strong_open = function(tokens, options, env) {
    // do something
    return 'some html text';
}
```

> 本文使用 [WPL/s](https://zhuanlan.zhihu.com/p/390528313) 发布 [@GitHub](https://github.com/jks-liu/WPL-s)
