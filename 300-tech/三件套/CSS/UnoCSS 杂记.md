---
title: UnoCSS 杂记
date: 2023-08-04 13:49
updated: 2023-08-04 14:34
---
---title: UnoCSS 杂记
date: 2023-08-04 13:49
updated: 2023-08-04 13:49
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
	- UnoCSS 杂记
rating: 1
tags:
	- CSS
	- web
category: web
keywords:
	- 关键词 1
	- 关键词 2
	- 关键词 3
description: 文章描述
source: #
url: //myblog.wallleap.cn/post/1
---

官网：[UnoCSS: The instant on-demand Atomic CSS engine](https://unocss.dev/)

## 安装和使用

```sh
pnpm add @unocss/reset
```

`main.js` 中添加以下重置样式表之一：

```js
// normalize.css https://github.com/csstools/normalize.css
import '@unocss/reset/normalize.css'

// sanitize.css https://github.com/csstools/sanitize.css
import '@unocss/reset/sanitize/sanitize.css'
import '@unocss/reset/sanitize/assets.css'

// Eric Meyer https://meyerweb.com/eric/tools/css/reset/index.html
import '@unocss/reset/eric-meyer.css'

// Tailwind
import '@unocss/reset/tailwind.css'

// Tailwind compat
// 基于 Tailwind 重置，但是去除了按钮背景色透明
import '@unocss/reset/tailwind-compat.css'
```

安装

```sh
pnpm add -D unocss
```

## 预设和配置

配置文件：`uno.config.ts`

```ts
// uno.config.ts
import { defineConfig } from 'unocss'

export default defineConfig({
  rules: [
    // ['m-1', { margin: '1px' }],
    [/^m-([\.\d]+)$/, ([_, num]) => ({ margin: `${num}px` })], // 可以直接使用正则
  ],
})
```

在上面自定义了一条规则，可以在 class 中使用

```html
<div class="m-1">Hello</div>
```

并且它会生成如下 CSS：

```css
.m-1 { margin: 1px; }
```

也可以把规则抽离到预设中

```ts
// my-preset.ts
import { Preset } from 'unocss'

export const myPreset: Preset = {
  name: 'my-preset',
  rules: [
    [/^m-([\.\d]+)$/, ([_, num]) => ({ margin: `${num}px` })],
    [/^p-([\.\d]+)$/, ([_, num]) => ({ padding: `${num}px` })],
  ],
  variants: [/* ... */],
  shortcuts: [/* ... */]
  // ...
}
```

然后在配置文件中使用

```ts
// uno.config.ts
import { defineConfig, presetAttributify, presetUno } from 'unocss'
import { myPreset } from './my-preset'

export default defineConfig({
  presets: [
    presetAttributify({ /* preset options */}), 
    presetUno(),
    // ...custom presets
    myPreset // your own preset
  ],
})
```

So similarly, we provided a few [official presets](https://unocss.dev/presets/) for you to start using right away, and you can also find many interesting [community presets](https://unocss.dev/presets/community).

官方预设和社区中找 ↑

功能齐全的配置文件

```ts
// uno.config.ts
import {
  defineConfig, presetAttributify, presetIcons,
  presetTypography, presetUno, presetWebFonts,
  transformerDirectives, transformerVariantGroup
} from 'unocss'

export default defineConfig({
  variants: [ 
    // hover: 
    (matcher) => { 
      if (!matcher.startsWith('hover:'))
        return matcher
      return { // slice `hover:` prefix and passed to the next variants and rules
        matcher: matcher.slice(6),
        selector: s => `${s}:hover`, 
      }
    }
  ],
  rules: [
    [/^m-(\d)$/, ([, d]) => ({ margin: `${d / 4}rem` })],
  ],
  shortcuts: [
    // you could still have object style 
    { btn: 'py-2 px-4 font-semibold rounded-lg shadow-md', }, 
    // dynamic shortcuts 
    [/^btn-(.*)$/, ([, c]) => `bg-${c}-400 text-${c}-100 py-2 px-4 rounded-lg`],
    // btn-green btn-red
  ],
  theme: {
    colors: {
      'veryCool': '#0000ff', // class="text-very-cool"
      'brand': {
        'primary': 'hsla(var(--hue, 217), 78%, 51%)', //class="bg-brand-primary"
      }
    },
    breakpoints: {
      sm: '320px', // sm:
      md: '640px', // md:
    },
  },
  presets: [
    presetUno(),
    presetAttributify(),
    presetIcons(),
    presetTypography(),
    presetWebFonts({
      fonts: {
        // ...
      },
    }),
  ],
  transformers: [
    transformerDirectives(),
    transformerVariantGroup(),
  ],
})
```

一般会自动查找配置文件，也可以手动指定，如 Vite 中：

```ts
// vite.config.ts
import { defineConfig } from 'vite'
import UnoCSS from 'unocss/vite'

export default defineConfig({
  plugins: [
    UnoCSS({
      // configFile: '../my-uno.config.ts',
      mode: 'vue-scoped', // 默认 global
    })
  ]
})
```

在 `main.ts` 中添加

```ts
// main.ts
import 'virtual:uno.css'
```

## 插件

VSCode 插件：[Install in Marketplace 安装到市场](https://marketplace.visualstudio.com/items?itemName=antfu.unocss)

得益于 UnoCSS 提供的灵活性，我们可以在其基础上尝试很多创新功能，例如

- [Pure CSS icons 纯 CSS 图标](https://unocss.dev/presets/icons)
- [Attributify Mode 属性化模式](https://unocss.dev/presets/attributify)
- [Variant Groups 变体组](https://unocss.dev/transformers/variant-group)
- [Shortcuts 捷径](https://unocss.dev/config/shortcuts)
- [Tagify 标签](https://unocss.dev/presets/tagify)
- [Web fonts 网络字体](https://unocss.dev/presets/web-fonts)
- [CDN Runtime CDN 运行时间](https://unocss.dev/integrations/runtime)
- [Inspector 检查员](https://unocss.dev/tools/inspector)

规则查询：[UnoCSS Interactive Docs](https://unocss.dev/interactive/)
