---
title: CSS 知识点及技巧整理
date: 2023-08-04T04:11:00+08:00
updated: 2024-08-21T10:32:48+08:00
dg-publish: false
aliases:
  - CSS 知识点及技巧整理
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
  - CSS
  - web
url: //myblog.wallleap.cn/post/1
---

这些个知识点是我个人认为的，下面我们就来看看这些个知识点。

## 1.怎么让一个不定宽高的 DIV，垂直水平居中?

**使用 Flex**

- 只需要在父盒子设置：display: flex; justify-content: center;align-items: center;

**使用 CSS3 transform**

- 父盒子设置:`display:relative`
- Div 设置: `transform: translate(-50%，-50%);position: absolute;top: 50%;left: 50%;`

**使用 display:table-cell 方法**

- 父盒子设置:`display:table-cell; text-align:center;vertical-align:middle`;
- Div 设置: `display:inline-block;vertical-align:middle`;

## 2.position 几个属性的作用

### 2.1 relative，absolute，fixed，static

position 的常见四个属性值: relative，absolute，fixed，static。一般都要配合 "left"、"top"、"right" 以及 "bottom" 属性使用。

- **static: 默认位置。** 在一般情况下，我们不需要特别的去声明它，但有时候遇到继承的情况，我们不愿意见到元素所继承的属性影响本身，从而可以用 Position:static 取消继承，即还原元素定位的默认值。设置为 static 的元素，它始终会处于页面流给予的位置 (static 元素会忽略任何 top、 bottom、left 或 right 声明)。一般不常用。
- **relative: 相对定位。** 相对定位是相对于元素默认的位置的定位，它偏移的 top，right，bottom，left 的值都以它原来的位置为基准偏移，而不管其他元素会怎么 样。注意 relative 移动后的元素在原来的位置仍占据空间。
- **absolute: 绝对定位。** 设置为 absolute 的元素，如果它的 父容器设置了 position 属性，并且 position 的属性值为 absolute 或者 relative，那么就会依据父容器进行偏移。如果其父容器没有设置 position 属性，那么偏移是以 body 为依据。注意设置 absolute 属性的元素在标准流中不占位置。
- **fixed: 固定定位。** 位置被设置为 fixed 的元素，可定位于相对于浏览器窗口的指定坐标。不论窗口滚动与否，元素都会留在那个位置。它始终是以 body 为依据的。 注意设置 fixed 属性的元素在标准流中不占位置。

### 2.2 sticky

position 新增的属性“sticky”： 设置了 sticky 的元素，在屏幕范围（viewport）时该元素的位置并不受到定位影响（设置是 top、left 等属性无效），当该元素的位置将要移出偏移范围时，定位又会变成 fixed，根据设置的 left、top 等属性成固定位置的效果。

sticky 属性有以下几个特点：

- 该元素并不脱离文档流，仍然保留元素原本在文档流中的位置。
- 当元素在容器中被滚动超过指定的偏移值时，元素在容器内固定在指定位置。亦即如果你设置了 top: 50px，那么在 sticky 元素到达距离相对定位的元素顶部 50px 的位置时固定，不再向上移动。

比较蛋疼的是这个属性的兼容性还不是很好，目前仍是一个试验性的属性，并不是 W3C 推荐的标准。它之所以会出现，也是因为监听 scroll 事件来实现粘性布局使浏览器进入慢滚动的模式，这与浏览器想要通过硬件加速来提升滚动的体验是相悖的。

> 详见：[blog.csdn.net/sinat\_37390…](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Fsinat_37390744%2Farticle%2Fdetails%2F77479239 "https://blog.csdn.net/sinat_37390744/article/details/77479239")

## 3.浮动与清除浮动

### 3.1 浮动相关知识

**float 属性的取值：**

- left：元素向左浮动。
- right：元素向右浮动。
- none：默认值。元素不浮动，并会显示在其在文本中出现的位置。

**浮动的特性：**

- 浮动元素会从普通文档流中脱离，但浮动元素影响的不仅是自己，它会影响周围的元素对齐进行环绕。
- 不管一个元素是行内元素还是块级元素，如果被设置了浮动，那浮动元素会生成一个块级框，可以设置它的 width 和 height，因此 float 常常用于制作横向配列的菜单，可以设置大小并且横向排列。

**浮动元素的展示在不同情况下会有不同的规则：**

- 浮动元素在浮动的时候，其 margin 不会超过包含块的 padding。*PS：如果想要元素超出，可以设置 margin 属性*
- 如果两个元素一个向左浮动，一个向右浮动，左浮动元素的 marginRight 不会和右浮动元素的 marginLeft 相邻。
- 如果有多个浮动元素，浮动元素会按顺序排下来而不会发生重叠的现象。
- 如果有多个浮动元素，后面的元素高度不会超过前面的元素，并且不会超过包含块。
- 如果有非浮动元素和浮动元素同时存在，并且非浮动元素在前，则浮动元素不会高于非浮动元素
- 浮动元素会尽可能地向顶端对齐、向左或向右对齐

**重叠问题**

- 行内元素与浮动元素发生重叠，其边框，背景和内容都会显示在浮动元素之上
- 块级元素与浮动元素发生重叠时，边框和背景会显示在浮动元素之下，内容会显示在浮动元素之上

**clear 属性** clear 属性：确保当前元素的左右两侧不会有浮动元素。clear 只对元素本身的布局起作用。 取值：left、right、both

### 3.2 父元素高度塌陷问题

**为什么要清除浮动，父元素高度塌陷** 解决父元素高度塌陷问题：一个块级元素如果没有设置 height，其 height 是由子元素撑开的。对子元素使用了浮动之后，子元素会脱离标准文档流，也就是说，父级元素中没有内容可以撑开其高度，这样父级元素的 height 就会被忽略，这就是所谓的高度塌陷。

### 3.3 清除浮动的方法

**方法 1：给父级 div 定义 高度** 原理：给父级 DIV 定义固定高度（height），能解决父级 DIV 无法获取高度得问题。 优点：代码简洁 缺点：高度被固定死了，是适合内容固定不变的模块。（不推荐使用）

**方法二：使用空元素，如 `<div class="clear"></div> (.clear{clear:both})`** 原理：添加一对空的 DIV 标签，利用 css 的 clear:both 属性清除浮动，让父级 DIV 能够获取高度。 优点：浏览器支持好 缺点：多出了很多空的 DIV 标签，如果页面中浮动模块多的话，就会出现很多的空置 DIV 了，这样感觉视乎不是太令人满意。（不推荐使用）

**方法三：让父级 div 也一并浮起来** 这样做可以初步解决当前的浮动问题。但是也让父级浮动起来了，又会产生新的浮动问题。 不推荐使用

**方法四：父级 div 定义 `display:table**` 原理：将 div 属性强制变成表格 优点：不解 缺点：会产生新的未知问题。（不推荐使用）

**方法五：父元素设置 `overflow：hidden、auto；**` 原理：这个方法的关键在于触发了 BFC。在 IE6 中还需要触发 hasLayout（zoom：1） 优点：代码简介，不存在结构和语义化问题 缺点：无法显示需要溢出的元素（亦不太推荐使用）

**方法六：父级 div 定义 伪类 `:after` 和 zoom**

```css
.clearfix:after{
    content:'.';
    display:block;
    height:0;
    clear:both;
    visibility: hidden;
}
.clearfix {zoom:1;}
```

原理：IE8 以上和非 IE 浏览器才支持 `:after`，原理和方法 2 有点类似，`zoom`(IE 转有属性) 可解决 ie6,ie7 浮动问题 优点：结构和语义化完全正确,代码量也适中，可重复利用率（建议定义公共类） 缺点：代码不是非常简洁（极力推荐使用）

**经益求精写法**

```css
.clearfix:after {
    content:”\200B”; 
    display:block; 
    height:0; 
    clear:both;
 }
.clearfix { *zoom:1; }  /* 照顾IE6，IE7就可以了 */
```

> 详细关于浮动的知识请参看这篇文章： [http://luopq.com/2015/11/08/CSS-float/](https://link.juejin.cn/?target=http%3A%2F%2Fluopq.com%2F2015%2F11%2F08%2FCSS-float%2F "http://luopq.com/2015/11/08/CSS-float/")

## 4.BFC 相关知识

**定义**：BFC(Block formatting context) 直译为 " 块级格式化上下文 "。它是一个独立的渲染区域，只有 Block-level box 参 与， 它规定了内部的 Block-level Box 如何布局，并且与这个区域外部毫不相干。

**BFC 布局规则** BFC 就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素。反之也如此。

- BFC 这个元素的垂直方向的边距会发生重叠，垂直方向的距离由 `margin` 决定，取最大值
- BFC 的区域不会与浮动盒子重叠（`清除浮动原理`）。
- 计算 BFC 的高度时，浮动元素也参与计算。

**哪些元素会生成 BFC**

- 根元素
- float 属性不为 none
- position 为 absolute 或 fixed
- display 为 inline-block， table-cell， table-caption， flex， inline-flex
- overflow 不为 visible

## 5.box-sizing 是什么

设置 CSS 盒模型为标准模型或 IE 模型。标准模型的宽度只包括 `content`，二 IE 模型包括 `border` 和 `padding` box-sizing 属性可以为三个值之一：

- content-box，默认值，只计算内容的宽度，border 和 padding 不计算入 width 之内
- padding-box，padding 计算入宽度内
- border-box，border 和 padding 计算入宽度之内

## 6.px，em，rem 的区别

**px** 像素 (Pixel)。绝对单位。像素 px 是 `相对于显示器屏幕分辨率` 而言的，是一个虚拟长度单位，是计算 机系统的数字化图像长度单位，如果 px 要换算成物理长度，需要指定精度 DPI。

**em** 是相对长度单位，`相对于当前对象内文本的字体尺寸`。如当前对行内文本的字体尺寸未被人为设置， 则相对于浏览器的默认字体尺寸。它会继承父级元素的字体大小，因此并不是一个固定的值。

**rem** 是 CSS3 新增的一个相对单位 (root em，根 em)，使用 rem 为元素设定字体大小时，仍然是相对大小， 但 `相对的只是 HTML 根元素`。

## 7.CSS 引入的方式有哪些? link 和@import 的区别是?

有四种：内联 (元素上的 style 属性)、内嵌 (style 标签)、外链 (link)、导入 (`@import`) link 和 `@import` 的区别：

- `link` 是 XHTML 标签，除了加载 CSS 外，还可以定义 RSS 等其他事务；`@import` 属于 CSS 范畴，`只能加载CSS`。
- `link` 引用 CSS 时，在 `页面载入时同时加载`；`@import需要页面网页完全载入以后加载`。
- `link` 是 XHTML 标签，`无兼容问题`；`@import` 是在 CSS2.1 提出的，`低版本的浏览器不支持`。
- `link` 支持使用 Javascript 控制 DOM 去 `改变样式`；而 `@import` 不支持。

## 8.流式布局与响应式布局的区别

**流式布局** 使用非固定像素来定义网页内容，`也就是百分比布局`，通过盒子的宽度设置成百分比来根据屏幕的宽度来进 行伸缩，不受固定像素的限制，内容向两侧填充。

**响应式开发** 利用 CSS3 中的 Media Query(媒介查询)，通过查询 screen 的宽度来指定某个宽度区间的网页布局。

- 超小屏幕 (移动设备) 768px 以下
- 小屏设备 768px-992px
- 中等屏幕 992px-1200px
- 宽屏设备 1200px 以上

由于响应式开发显得繁琐些，一般使用第三方响应式框架来完成，比如 bootstrap 来完成一部分工作，当然也 可以自己写响应式。

**区别**

| \- | 流式布局 | 响应式开发 |
| --- | --- | --- |
| 开发方式 | 移动 Web 开发 +PC 开发 | 响应式开发 |
| 应用场景 | 一般在已经有 PC 端网站，开发移动的的时候只需要单独开发移动端 | 针对一些新建的网站，现在要求适配移动端，所以就一套页面兼容各种终端 |
| 开发 | 正对性强，开发效率高 | 兼容各种终端，效率低 |
| 适配 | 只适配移动设备，pad 上体验相对较差 | 可以适配各种终端 |
| 效率 | 代码简洁，加载快 | 代码相对复杂，加载慢 |

## 9.渐进增强和优雅降级

关键的区别是他们所侧重的内容，以及这种不同造成的工作流程的差异

- **优雅降级**一开始就构建完整的功能，然后再针对低版本浏览器进行兼容。。
- **渐进增强**针对低版本浏览器进行构建页面，保证最基本的功能，然后再针对高级浏览器进行效果、交互等改进和追加功能达到更好的用户体验。

区别：

- 优雅降级是从复杂的现状开始，并试图减少用户体验的供给
- 渐进增强则是从一个非常基础的，能够起作用的版本开始，并不断扩充，以适应未来环境的需要
- 降级（功能衰减）意味着往回看；而渐进增强则意味着朝前看，同时保证其根基处于安全地带

## 10.CSS 隐藏元素的几种方式及区别

**display:none**

- 元素在页面上将彻底消失，元素本来占有的空间就会被其他元素占有，也就是说它会导致浏览器的重排和重绘。
- 不会触发其点击事件

**visibility:hidden**

- 和 `display:none` 的区别在于，`元素在页面消失后，其占据的空间依旧会保留着`，所以它 `只会导致浏览器重绘` 而不会重排。
- 无法触发其点击事件
- 适用于那些元素隐藏后不希望页面布局会发生变化的场景

**opacity:0**

- 将元素的透明度设置为 0 后，在我们用户眼中，元素也是隐藏的，这算是一种隐藏元素的方法。
- 和 `visibility:hidden` 的一个共同点是元素隐藏后依旧占据着空间，但我们都知道，设置透明度为 0 后，元素只是隐身了，它依旧存在页面中。
- 可以触发点击事件

**设置 height，width 等盒模型属性为 0**

- 简单说就是将元素的 `margin`，`border`，`padding`，`height` 和 `width` 等影响元素盒模型的属性设置成 `0`，如果元素内有子元素或内容，还应该设置其 `overflow:hidden` 来隐藏其子元素，这算是一种奇技淫巧。
- 如果元素设置了 border，padding 等属性不为 0，很显然，页面上还是能看到这个元素的，触发元素的点击事件完全没有问题。如果全部属性都设置为 0，很显然，这个元素相当于消失了，即无法触发点击事件。
- 这种方式既不实用，也可能存在着着一些问题。但平时我们用到的一些页面效果可能就是采用这种方式来完成的，比如 jquery 的 slideUp 动画，它就是设置元素的 `overflow:hidden` 后，接着通过定时器，不断地设置元素的 height，margin-top，margin-bottom，border-top，border-bottom，padding-top，padding-bottom 为 0，从而达到 slideUp 的效果。

**其他脑洞方法**

- 设置元素的 position 与 left，top，bottom，right 等，将元素移出至屏幕外
- 设置元素的 position 与 z-index，将 z-index 设置成尽量小的负数

## 11.消除图片底部间隙的方法

- 图片块状化 - 无基线对齐：`img { display: block; }`
- 图片底线对齐：`img { vertical-align: bottom; }`
- 行高足够小 - 基线位置上移：`.box { line-height: 0; }`

## 12."nth-child" 和 "nth-of-type" 的区别

“nth-child”选择的是父元素的子元素，这个子元素并没有指定确切类型，同时满足两个条件时方能有效果：其一是子元素，其二是子元素刚好处在那个位置；“nth-of-type”选择的是某父元素的子元素，而且这个子元素是指定类型。[参考](https://juejin.cn/post/6844903495854735374 "https://juejin.cn/post/6844903495854735374")

## 13.CSS 中的颜色体系有哪些？

![](https://cdn.wallleap.cn/img/pic/illustration/202308041612807.jpeg) [参考](https://link.juejin.cn/?target=http%3A%2F%2Fwww.cnblogs.com%2Fcoco1s%2Fp%2F5622534.html "http://www.cnblogs.com/coco1s/p/5622534.html")

## 14\. CSS 流体布局下的宽度分离原则

所谓“宽度分离”原则，就是 CSS 中的 width 属性不与影响宽度的 pading/border（有时候包括 margin）属性共存，width 独立占用一层标签，而 padding、border、margin 利用流动性在内部自适应呈现。

举例：元素边框内有 20px 的留白问题

```css
.father { width:102px; }
.son {
    border:1px solid;
    padding:20px;
}
```

## 15\. !important 问题

超越 `!important`：max-width 会覆盖 width，而且这种覆盖是超级覆盖，比 `!important` 的权重还要高

超越最大：min-width 覆盖 max-width，此规则发生在 min-width 和 max-width 冲突的时候，如下：

```css
.container{
    min-width:1400px;
    max-width:1200px;
}
```

## 16\. 任意高度元素的展开收起动画

使用 height + overflow:hidden 实现会比较生硬。好的方式是使用 `max-height`。

```css
.element{
    max-height:0;
    overflow:hidden;
    transition: height .25s;
}
.element.active {
    max-height: 666px; /* 一个足够大的最大高度值 */
}
```

## 17\. 基于伪元素的图片内容生成技术

需求：图片还没加载时就把 `alt` 信息呈现出来。

实现：图片没有 `src` ，因此，`::before` 和 `::after` 可以生效，我们可以通过 `content` 属性呈现 alt 属性值。

```css
img::after{
    /* 生成 alt 信息 */
    content: attr(alt);
    /* 尺寸和定位 */
    postion:absolute; bottom: 0;
    width:100%;
    background-color:rgba(0,0,0,.5);
    transform: translateY(100%);
    transition: transform .2s;
}
img:hover::after{
    transform: translateY(0);
}    

```

当我们给图片添加 src 属性时图片从普通元素变成替换元素，原本还支持的 `::before` 和 `::after` 此时全部无效，此时再 hover 图片，是不会有任何信息出现的。

## 18\. 轻松实现 hover 图片变成另外一张图片

```css
img:hover{
    content: url(laugh-tear.png);
}
```

content 改变的仅仅是视觉呈现，当我们鼠标右键或其他形式保存这张图片时，所保存的还是原来 src 对应的图片。这种方法还可以用在官网标志上。

由于使用 conetnt 生成图片无法设置图片的尺寸，要想在移动端使用该技术，建议使用 SVG 图片。

## 19\. content 属性的特点

我们使用 content 生成的文本是无法选中、无法复制的，好像设置了 `user-select:none` 声明一般，而普通元素的文本可以被轻松选中。content 生成的文本无法被屏幕阅读设备读取，也无法被搜索引擎抓取，因此，千万不要自以为是地把重要的文本信息使用 content 属性生成， 因为这对可访问性和 SEO 都不友好，content 属性只能用来生成一些无关紧要的内容，如装饰性图形或者序号之类；同样，也不要担心原本重要的文字信息会被 content 替换，替换的仅仅是视觉层。

content 动态生成值无法获取。content 是一个非常强大的 CSS 属性，其中一个强大之处就是计数器效果，可以自动累加数值。如：

```css
.total::after{
    content: counter(icecream);
}
```

示例：[jsrun.net/6VyKp](https://link.juejin.cn/?target=https%3A%2F%2Fjsrun.net%2F6VyKp "https://jsrun.net/6VyKp")

## 20\. content 换行符与打点 loading 效果

```html
正在加载中<dot>...</dot>
```

```css
dot {
    display: inline-block; 
    height: 1em;
    line-height: 1;
    text-align: left;
    vertical-align: -.25em;
    overflow: hidden;
}

dot::before {
    display: block;
    content: '...\A..\A.';
    white-space: pre-wrap;
    animation: dot 3s infinite step-start both;
}
@keyframes dot {
    33% { transform: translateY(-2em); }
    66% { transform: translateY(-1em); }
}
```

[在线demo](https://link.juejin.cn/?target=https%3A%2F%2Fdemo.cssworld.cn%2F4%2F1-9.php "https://demo.cssworld.cn/4/1-9.php")

## 21\. margin:auto 的应用

`margin:auto` 的填充规则：

1. 如果一侧定值，一侧 auto，则 auto 为剩余空间大小
2. 如果两侧均是 auto，则平分剩余空间

块级元素垂直方向居中：

使用 writing-mode 改变文档流方向

```css
.father{ height200px; writing-mode:vertical-lr; } .son{ height:100px; margin:auto; } 
```

绝对定位下的 margin:auto 居中

```css
.father{ width:300px; height:150px; position: relative; } .son{ position:absolute; top:0; right:0; bottom:0; left:0; width:200px; height:100px; margin:auto; } 
```

在线效果：[demo.cssworld.cn/4/3-5.php](https://link.juejin.cn/?target=https%3A%2F%2Fdemo.cssworld.cn%2F4%2F3-5.php "https://demo.cssworld.cn/4/3-5.php")

## 22\. border-width 支持的关键字

- thin：薄薄的，等同于 1px
- medium：（默认值）薄厚均匀，等同于 3px
- thick：厚厚的，等同于 4px

## 23.border-color 和 color

border-color 默认颜色就是 color 色值，就是当没有指定 border-color 颜色值的时候，会使用当前元素的 color 计算值作为边框色，如：

```css
.box{
    border: 10px solid;
    color: red;
}
```

此时，.box 元素的 10px 边框颜色就是红色。
