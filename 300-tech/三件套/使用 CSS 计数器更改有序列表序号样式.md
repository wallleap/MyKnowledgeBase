
**CSS 计数器**可让你根据内容在文档中的位置调整其显示的外观。例如，你可以使用计数器自动为网页中的标题编号，或者更改有序列表的编号。

文档：[CSS Counter Styles - CSS（层叠样式表） | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Counter_Styles)

## 使用计数器

例如：生成 HTML，四个 Section，包裹着 `h3` 和 `p`

```html
article>section*4>h2{Section}+(h3{话题}+p{段落111111111}+p{段落aaaaaaaaaaaaaa})*4
```

![HTML](https://cdn.wallleap.cn/img/pic/illustrtion/202210121755148.png)

### 1、使用 [`counter-reset`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/counter-reset) 属性初始化计数器的值

在使用计数器之前，需要使用 [`counter-reset`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/counter-reset) 属性来初始化它的值。这个属性也可用于指定计数器的初始值。

#### 正序计数器



#### 反向计数器

反向计数器可以通过在 [`counter-reset`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/counter-reset) 属性中将计数器的名称使用 `reversed()` 函数包裹来创建



> 计数器只能在可以生成盒子的元素中使用（设值或重设值、递增）。例如，如果一个元素被设置为了 `display: none`，那么在这个元素上的任何计数器操作都会被忽略。

### 2、计数器可通过 [`counter-increment`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/counter-increment) 属性指定其值为递增或递减



### 3、当前计数器的值通过 [`counter()`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/counter) 或 [`counters()`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/counters) 函数显示出来

这通常会在[伪元素](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Pseudo-elements)的 [`content`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/content) 属性中使用。