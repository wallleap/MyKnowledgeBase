---
title: CSS 中实现元素居中的七种方法总结
date: 2023-08-07T09:39:00+08:00
updated: 2024-08-21T10:32:48+08:00
dg-publish: false
aliases:
  - CSS 中实现元素居中的七种方法总结
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

在前端开发中，经常需要将元素**居中显示**，CSS 提供了多种技术方法来实现元素的居中，在不同场景下有不同的使用方法、不同的效果，需要特别记住它们的应用场景才能够正常的居中。这篇文章就大致总结一下 CSS 中的居中方法。

## 一、元素分类

在 CSS 中，元素大致可以分为以下几种：

### 1.块级元素（Block-level Elements）

这些元素以块的形式显示在页面上，每个块级元素会独占一行（除非通过其他 CSS 属性进行修改）。 块级元素可以设置宽度、高度、内边距和外边距。 一些常见的块级元素包括 `<div>, <p>, <h1>-<h6>, <ul>, <li>, <section>, <footer>` 等。

### 2.行内元素（Inline Elements）

**行内**元素也称为**内联**元素，这些元素以行内的形式显示在页面上，它们不会独占一行，而是在同一行上与其他元素并排显示。 行内元素的宽度和高度默认由其内容决定，无法设置宽度和高度。 一些常见的行内元素包括 `<span>, <a>, <strong>, <em>, <img>, <input>` 等。

### 3.行内块元素（Inline-block Elements）

这些元素以行内块的形式显示在页面上，具有行内元素的特性，但可以设置宽度、高度、内边距和外边距。 行内块元素会在同一行上显示，但它们之间会保留空白间隔。 一些常见的行内块元素包括 `<button>, <label>, <select>, <textarea>, <img>` 等。

## 二、使用 text-align: center 居中

使用 text-align: center; 可以在 CSS 中实现内联元素的水平居中。这个技术利用了 CSS 的 text-align 属性，通过对元素的文本对齐方式进行调整来实现居中效果。**注：只展示主要代码。**

```
<div class="container">
  <span>检测居中效果</span><br>
  <img src="1.jpg" alt=""><br>
  <input type="text" value="检测居中效果">
</div>
```

```
.container {
   text-align: center;
} 
```

在上述示例中，将容器的 text-align 属性设置为 center，使容器内的文本水平居中显示。由于内联元素的默认宽度与内容宽度一致，所以通过调整文本的对齐方式，元素就可以在容器中水平居中。

需要注意的是，这种方法**适用于内联元素，而不适用于块级元素**。对于块级元素，可以将其包裹在一个容器中，并对容器应用 text-align: center; 实现块级元素的水平居中。

这是一种简单而常用的方法，特别适用于文本、按钮、图标等内联元素的水平居中。然而，它**只能实现水平居中**，对于垂直居中需要采用其他的布局方法。若元素是单行文本, 则可设置 **line-height 等于父元素高度**来实现垂直居中。

## 三、使用 margin: 0 auto 居中

要将块级元素水平居中，可以使用 margin 属性将左右边距设置为 auto。

```
.container {
   width: 300px; /* 设置容器的宽度 */
   margin: 0 auto; /* 水平居中 */
}
```

在上述示例中，将容器的宽度设置为一个固定值，然后使用 margin: 0 auto; 将左右外边距设置为 "auto"，实现元素的水平居中。由于左右外边距都设置为 "auto"，浏览器会自动将剩余的空间均匀分配给两侧的外边距，从而使元素居中显示，这种方法适用于具有固定宽度的块级元素。

## 四、使用 Flexbox 居中元素

Flex 弹性布局，通过将容器的 display 属性设置为 flex，并使用 justify-content 和 align-items 属性分别进行水平和垂直居中设置，元素将在容器中居中显示。

```
.container {
   display: flex;
   justify-content: center; /* 水平居中 */
   align-items: center;     /* 垂直居中 */
} 
```

Flexbox 还提供了其他属性，如 flex-direction、flex-wrap、align-content 等，可以根据具体需求进行进一步的布局调整。使用 Flexbox 可以轻松实现各种居中效果，并且具有很好的浏览器兼容性。

## 五、使用 Grid 居中元素

网格布局 Grid 是另一种强大的布局模型，也可以用于实现元素的居中布局。通过将容器的 display 属性设置为 grid，并使用 place-items 属性设置为 center，元素将在容器中居中显示。

```
.container {
  display: grid;
  place-items: center; /* 水平和垂直居中 */
}
```

在上面的代码示例中，`place-items: center` 是水平和垂直居中，如果只想水平居中可以用 `justify-items: center`。如果只想垂直居中可以用 `align-items: center`。

## 六、使用定位和负边距居中

首先将容器的左边距设置为 50%（相对于父容器），然后使用 `transform: translateX(-50%)`; 将元素向左平移 50% 的宽度，从而实现了水平居中。

```
.container {
   position: absolute;
   left: 50%;
   transform: translateX(-50%);
} 
```

下面是水平和垂直居中的示例。

```
.container {
   position: absolute;
   top: 50%;
   left: 50%;
   transform: translate(-50%, -50%);
} 
```

将要居中的元素的定位属性设置为 absolute。通过将元素的 top 和 left 属性都设置为 50%，元素的左上角将位于容器的中心。最后，通过 transform 属性和 translate 函数将元素向上和向左平移自身宽度和高度的一半，从而实现垂直居中的效果。

使用绝对定位和负边距可以适用于不同类型的元素，包括块级元素和内联元素。这是一种简洁而有效的方法，可以快速实现水平居中布局。

## 七、使用 calc() 函数居中

calc() 函数通过执行简单的数学运算，并返回计算结果作为 CSS 属性值。使用 calc() 函数可以根据具体的需求进行灵活的计算和布局，实现元素在水平或垂直方向的居中。

对于水平居中，可以使用 calc() 函数结合百分比和像素值来计算元素的左右外边距。通过将 50%（容器的一半宽度）减去 150 像素（元素宽度的一半）来计算得到。

```
.container {
   width: 300px;
   margin-left: calc(50% - 150px);
   margin-right: calc(50% - 150px);
   /* background-color: blue; */
} 
```

对于垂直居中，可以使用 calc() 函数结合百分比、像素值和视口单位（如 vh）来计算元素的上下外边距。通过将 50vh（视口高度的一半）减去 200 像素（元素高度的一半）来计算得到的。

```
.container {
   height: 400px;
   margin-top: calc(50vh - 200px);
   margin-bottom: calc(50vh - 200px);
} 
```

请注意，calc() 函数的兼容性良好，但在使用时需要确保计算表达式正确并考虑浏览器的兼容性。

## 八、使用 table 居中

使用表格布局（Table Layout）可以实现元素的居中布局。虽然表格布局在现代响应式布局中不常用，但在某些特定情况下仍然可以作为一种解决方案。

要使用表格布局居中元素，需要创建一个包含一个单元格的表格，并将元素放置在该单元格中。

```
.container {
   display: table;
   width: 100%;
}
.content {
   display: table-cell;
   text-align: center;
}
```

```
<div class="container">
   <div class="content">
     <div>检测居中效果</div>
     <p>检测居中效果</p>
     <input type="text" value="检测居中效果">
   </div>
</div>
```

在上述示例中，容器的宽度被设置为 100% 以使其填充父容器的宽度。父容器设置为 display: table，子容器设置为 display: table-cell，并使用 text-align: center 将元素水平居中。

```
<div class="box">
    <div class="son"></div>
</div>

<style>
  .box{
     width: 200px;
     height: 200px;
     background-color: aqua;
     display: table-cell;
     vertical-align: middle;
     text-align: center;
   }
  .son{
     width: 50px;
     height: 50px;
     background-color: blueviolet;
     display: inline-block;
   }
</style>
```

需要注意的是，使用表格布局可能会影响文档的语义性，因此仅在适用的情况下使用。在现代的 CSS 布局中，使用 Flexbox 或 Grid 布局更为推荐，因为它们提供更灵活和语义化的布局选项。

## 九、总结

本文介绍了在 CSS 中实现元素居中的几种常用技术方法，主要介绍的是水平居中，根据具体需求和布局，选择适合的方法实现元素的居中效果即可。这些方法可以单独使用或结合使用，取决于布局和设计要求。同时，还可以使用其他 CSS 属性和技术来进一步优化和调整居中效果。

能力一般，水平有限，本文可能存在纰漏或错误，如有问题欢迎指正，感谢你阅读这篇文章，如果你觉得写得还行的话，不要忘记点赞、评论、收藏哦！祝生活愉快！
