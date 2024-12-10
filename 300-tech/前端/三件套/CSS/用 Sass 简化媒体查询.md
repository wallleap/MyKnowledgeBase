---
title: 用 Sass 简化媒体查询
date: 2024-11-26T16:19:48+08:00
updated: 2024-11-26T16:31:15+08:00
dg-publish: false
---

```scss
$breakpoints: (
  'phone': (320px, 480px),
  'pad': (481px, 768px),
  'notebook': (769px, 1024px),
  'desktop': (1025px, 1200px),
  'tv': 1201px
);
@mixin respondTo($breakname) {
  $bp: map-get($breakpoints, $breakname)
  @if type-of($bp) == 'list' {
    $min: nth($bp, 1)
    $max: nth($bp, 2)
    @media (min-width: $min) and (max-width: $max) {
      @content;
    }
  }
  @else {
    @media (min-width: $bp) {
      @content;
    }
  }
}
```

其他地方直接使用

```scss
.header {
  width: 100%;
  @include respondTo('phone') {
    height: 50px;
  }
  @include respondTo('pad') {
    height: 60px;
  }
  @include respondTo('notebook') {
    height: 70px;
  }
}
```
