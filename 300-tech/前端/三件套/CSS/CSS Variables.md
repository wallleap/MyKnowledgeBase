---
title: CSS Variables
date: 2024-08-22T01:41:02+08:00
updated: 2024-08-22T01:41:09+08:00
dg-publish: false
---

## CSS Variables

### CSS Variables (CSS 变量)

关于 [CSS 自定义变量](https://drafts.csswg.org/css-variables/) 新的草案介绍可以通过自定义变量进行 CSS 的赋值。

___

#### 声明

变量名前面要加两根连词线 (--), 变量名大小写敏感

```css
body { --foo: #7F583F; --bar: #F7EFD2; }
```

#### var() 函数

> var() 函数用于读取变量，也可用于变量声明，只能作为属性值，不能用作属性名。

```css
a { color: var(--foo); text-decoration-color: var(--bar); } :root { --primary-color: red; --logo-text: var(--primary-color); }
```

> var 函数第二个参数，表示变量的默认值。参数可以带逗号和空格。

```css
color: var(--foo, #7F583F); var(--font-stack, "Roboto", "Helvetica"); var(--pad, 10px 15px 20px);
```

#### 变量值的类型

> 字符串：可以与其它字符串进行拼接。

```css
--bar: 'hello'; --foo: var(--bar)' world';
```

> 数值：不能与数值单位直接连用，必须使用 calc() 函数。如果变量值带有单位，不能写成字符串，直接写：--foo: 20px;

```css
.foo { --gap: 20; /* 无效 */margin-top: var(--gap)px; } .foo { --gap: 20; margin-top: calc(var(--gap) * 1px); }
```

### 作用域

同一个 CSS 变量，可以在多个选择器内声明。读取的时候，优先级最高的声明生效。这与 CSS 的“层叠”规则是一致的。

```css
<style> :root { --color: blue; } div { --color: green; } #alert { --color: red; } * { color: var(--color); } </style> <p>蓝色</p> <div>绿色</div> <div id="alert">红色</div>
```

变量的作用域就是它所在的选择器的有效范围，全局的变量通常放在根元素 :root 里面，确保任何选择器都可以读取。

#### 响应式布局

可以在 media 命令里声明变量，使得不同的屏幕宽度有不同的变量值。

```css
body { --primary: #7F583F; --secondary: #F7EFD2; } a { color: var(--primary); text-decoration-color: var(--secondary); } @media screen and (min-width: 768px) { body { --primary: #F7EFD2; --secondary: #7F583F; } }
```

#### 兼容性处理

不支持 CSS 变量的浏览器，采用以下写法

```css
a { color: #7F583F; color: var(--primary); } 也可以使用@supports命令进行检测。 @supports ( (--a: 0)) { /* supported */ } @supports ( not (--a: 0)) { /* not supported */ }
```

JS 操作

检测浏览器是否支持 CSS 变量。

```css
const isSupported = window.CSS && window.CSS.supports && window.CSS.supports('--a', 0); if (isSupported) { /* supported */ } else { /* not supported */ }
```

操作 CSS 变量。

```css
// 设置变量 document.body.style.setProperty('--primary', '#7F583F'); // 读取变量 document.body.style.getPropertyValue('--primary').trim(); // '#7F583F' // 删除变量 document.body.style.removeProperty('--primary');
```

JS 可以将任意值存入样式表，也可以读取任意 CSS 变量值。CSS 变量提供了一种 JS 与 CSS 通信的一种途径。

### tips

比如我们如果当算设置主题色，我们需要在样式表的很多地方用到这样的颜色。我们可以将这些颜色设置为变量，然后在其他地方引用，而不是直接进行赋值。(写过 less sass 的应该可以体会到它的好处)

```
:root {
  --theme-colour: cornflowerblue;

}

h1 { color: var(--theme-colour); }
a { color: var(--theme-colour); }
strong { color: var(--theme-colour); }
```

当然这种特性 less 和 sass 已经很早支持了，但是 css 实现这个支持也有一定得好处，比如我们可以通过 JS 实现颜色值得统一更新。

```
const rootEl = document.documentElement;
rootEl.style.setProperty('--theme-colour', 'plum');
```

[CSS Variables的兼容性](https://www.caniuse.com/#feat=css-variables)
