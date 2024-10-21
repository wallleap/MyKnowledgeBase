---
title: 纯 CSS 实现波纹效果
date: 2024-09-23T10:34:12+08:00
updated: 2024-09-23T11:17:06+08:00
dg-publish: false
---

```scss
@use "sass:math";

.box {
  $r: 50;
  $p: 45;
  $x: 2 * math.sqrt($r * $r - $p * $p);
  width: 100vw;
  height: 200px;
  margin: 5px;
  border: 1px solid;
  background-image: radial-gradient(#{$r}px at 50%  #{$r + $p}px, red 100%, transparent 101%),
    radial-gradient(#{$r}px at 50% #{-$p}px, transparent 100%, red 101%);
  background-position: calc(50% - #{$x}px) 0, 50% #{$r}px;
  background-size: #{2 * $r}px 100%;
  background-repeat: repeat-x;
}
```
