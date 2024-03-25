---
title: svg
date: 2024-03-11 23:53
updated: 2024-03-11 23:54
---

SVG 绘图的七种主要形状：

```
- 圆圈<circle>

- 椭圆<ellipse>

- 线<line>

- 小路<path>

- 多边形<polygon>

- 折线<polyline>
```

SVG 元素：

```
- <svg>

- <symbol>

- <g>

- <path>
```

SVG 属性：

- d
- fill
- stroke
- stroke-width

## **2. 如何绘制 SVG 图标**

下面通过 rect 分别绘制矩形和圆角矩形进行讲解：

**step1：绘制 SVG 画布**

- 绘制一个 500x500px 的矩形作为容器，增加 1px 的黑色（#000000） 描边；

SVG 可以使用 CSS 和行内样式设置宽高。

![绘制SVG画布](https://cms.pixso.cn/images/designskills/2022/sixskills/svgtubiao02.png)

**step2：绘制矩形**

- 矩形基于 SVG 画布左上角的 xy 坐标。x 坐标为 200，y 坐标为 100；
- 设置矩形的宽高都为 100；填充（fill）为紫色；描边（stroke）为黄色，也可以用十六进制的颜色代码；

![绘制矩形](https://cms.pixso.cn/images/designskills/2022/sixskills/svgtubiao03.png)

**step3：绘制圆角矩形**

- 圆角矩形的 x 坐标和 y 坐标都为 300；宽高都为 100；填充（fill）为天空蓝；
- 设置矩形的圆角 rx='10' 、ry='20'；

![绘制圆角矩形](https://cms.pixso.cn/images/designskills/2022/sixskills/svgtubiao04.png)

**注意：**如果 rx 与 ry 的值相等，可以只设置一个，如果只写了 rx，那么 ry 也会引用 rx 的值。
