---
title: flex + margin 布局
date: 2024-11-26T16:31:42+08:00
updated: 2024-11-26T16:48:32+08:00
dg-publish: false
---

## 居中

```css
.container {
  display: flex;
  height: 300px;
}

.content {
  margin: auto;  /* 会吃掉剩余空间 */
}
```

## 单独一个到右边

```css
.container {
  display: flex;
}

.content:nth-last-of-type(1) {  /* 倒数第一个 */
  margin-left: auto;
}
```

## 最后两个到右边

```css
.container {
  display: flex;
}

.content:nth-last-of-type(2) {  /* 倒数第二个 */
  margin-left: auto;
}
```

## 最后一个到右边，倒数第二个到中间

```css
.container {
  display: flex;
}

.content:nth-last-of-type(2) {  /* 倒数第二个 */
  margin: 0 auto;
}
```

## 平均分布

```css
.container {
  display: flex;
  flex-wrap: wrap;
}

.content {
  --w: 50px;
  --n: 3;
  --rest: calc(100% - var(--w) * var(--n));
  margin: 10px calc(var(--rest) / var(--n) / 2);
}
```
