---
title: 手撸原生 CSS 变量，给 Vant2 实现动态主题与暗黑模式
date: 2023-08-04 09:42
updated: 2023-08-04 09:47
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - 手撸原生 CSS 变量，给 Vant2 实现动态主题与暗黑模式
rating: 1
tags:
  - CSS
  - web
category: web
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: #
url: //myblog.wallleap.cn/post/1
---

在需求开发中如果需要大规模替换样式，例如：将主题颜色从蓝色改为橙色或绿色，亦或是需要添加日间、夜间主题模式，以供用户动态切换。这个时候将其一个个覆盖起来也许并不是一个好主意。

目前前端实现定制主题有三种主流方式：

1. 通过 sass-loader 或 less-loader 覆盖 less 或 sass 变量，重新编译；
2. 通过 CSS 变量设置；
3. CSS-in-JS

巧的是，本人接手的老项目用的是 Vant2.0，只支持第一种方案，而目前 Vant4.0 已支持第二种方案，所以我们刚好借此看看方案一是如何操作的，并且让 Vant2.0 也能支持方案二。

## 方案一：通过 Less 变量

组件库会使用 Less 或 SCSS 对样式进行预处理，并内置一些样式变量，所以通过替换样式变量即可定制自己需要的主题。

### 查找 less 变量文件

其实不光 Vant，Element、Antd（5.0 以上除外） 等主流组件库都有自己默认的样式变量，我们去它的配置文件中就能看到：

- 👉 [Vant2.0 样式配置文件](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fyouzan%2Fvant%2Fblob%2F2.x%2Fsrc%2Fstyle%2Fvar.less "https://github.com/youzan/vant/blob/2.x/src/style/var.less")
- 👉 [Element-plus 样式配置文件](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Felement-plus%2Felement-plus%2Fblob%2Fdev%2Fpackages%2Ftheme-chalk%2Fsrc%2Fcommon%2Fvar.scss "https://github.com/element-plus/element-plus/blob/dev/packages/theme-chalk/src/common/var.scss")
- 👉 [Antd 样式配置文件](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fant-design%2Fant-design%2Fblob%2F4.x-stable%2Fcomponents%2Fstyle%2Fthemes%2Fdefault.less "https://github.com/ant-design/ant-design/blob/4.x-stable/components/style/themes/default.less")

第一步要做的就是从文件中找到自己想要变更的颜色，这里我们尝试将按钮的默认绿色改为红色，主要针对以下三种变量：

```js
{
    'button-primary-color': '#fff';
    'button-primary-background-color': '#E4004F';
    'button-primary-border-color': '#E4004F';
}
```

### 配置 less-loader

Less 提供 [modifyVars](https://link.juejin.cn/?target=https%3A%2F%2Flesscss.org%2Fusage%2F%23using-less-in-the-browser-modify-variables "https://lesscss.org/usage/#using-less-in-the-browser-modify-variables")，该方法可以在不重新加载 less 文件的情况下，重新编译 less 文件，达到修改运行时中 less 变量的目的。

我们在 webapck 文件中修改 `less-loader` 的配置：

```js
module.exports = {
  rules: [
    {
      test: /\.less$/,
      use: [
        // ...其他 loader 配置
        {
          loader: 'less-loader',
          options: {
            // 若 less-loader 版本小于 6.0，请移除 lessOptions 这一级，直接配置选项。
            lessOptions: {
              modifyVars: {
                    'button-primary-color': '#fff';
                    'button-primary-background-color': '#E4004F';
                    'button-primary-border-color': '#E4004F';
              },
            },
          },
        },
      ],
    },
  ],
};
```

重启项目编译后，会发现绿色变成了我们想要的红色。

![](https://cdn.wallleap.cn/img/pic/illustration/202308040943637.png)

如果要修改的颜色较多，也可以单独写在一个文件里：

```less
// src/style/theme/vant-vars.less
@button-primary-color: #fff;
@button-primary-background-color: #E4004F;
@button-primary-border-color: #E4004F;
```

然后将之前的属性值换成文件路径，名字随便取（比如就叫 'hack'）。

```js
{
  loader: 'less-loader',
  options: {
    lessOptions: {
      modifyVars: { hack: 'true; @import "@/style/theme/vant-vars.less"' },
    },
  },
},
```

值得一提的是，这种写法其实在 Less 官方文档里并不存在，但是却出现在了 Vant2.0 的 [官方文档](https://link.juejin.cn/?target=https%3A%2F%2Fvant-contrib.gitee.io%2Fvant%2Fv2%2F%23%2Fzh-CN%2Ftheme%23bu-zou-er-xiu-gai-yang-shi-bian-liang "https://vant-contrib.gitee.io/vant/v2/#/zh-CN/theme#bu-zou-er-xiu-gai-yang-shi-bian-liang") 中，有意思的是路径中的 `true` 也可以忽略不写，但是路径前的 `;` 却不能省略，否则会报错。

究其原因，这种写法应该是利用了 `modifyVars` 的特性，通过创建一个假的变量 "hack"，在编译的样式中任意注入导入的主题，有兴趣的同学可以查看 [less 的解析器](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fless%2Fless.js%2Fblob%2Fmaster%2Fpackages%2Fless%2Fsrc%2Fless%2Fparser%2Fparser.js "https://github.com/less/less.js/blob/master/packages/less/src/less/parser/parser.js")

### 优缺点

这种方案比较简单，对于需要大批更换主题色的需求，此方案已绰绰有余，只需每次通过修改 SCSS 文件并重新编译就能实现。

但也正因如此，这种方案只能做到一次性修改，如果想继续覆盖，只能再重新编译，而且每次只能固定展示一种颜色，无法做到动态切换。

## 方案二：通过 CSS 变量

CSS 变量是一个非常有用的功能，几乎所有浏览器都支持。 （IE：啊这？)

这意味着你可以动态的改变组件内的个别变量，以便更好地自定义组件样式，而不需要修改 SCSS 文件重新编译。

打开 Vant4.0 的官网，找到 HTML 根标签，可以看到已声明的 CSS 变量（或者查看 [源码样式文件](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fyouzan%2Fvant%2Fblob%2Fmain%2Fpackages%2Fvant%2Fsrc%2Fstyle%2Fcss-variables.less "https://github.com/youzan/vant/blob/main/packages/vant/src/style/css-variables.less")）：

![](https://cdn.wallleap.cn/img/pic/illustration/202308040943638.png)

先来介绍下其中的 `:root` 和 CSS 变量。

### :root

`:root` 是 CSS 伪类元素，用来匹配文档树的根元素，对 HTML 来说，`:root` 就代表着 `<html>` 元素，但是它的优先级要更高一些，所以在声明全局的 CSS 变量时，`:root` 就会很有用。

你甚至可以连写两个 root，就像这样：

```css
/* 添加这段样式后，Primary Button 会变成红色 */
:root:root {
  --van-button-primary-background: red;
}
```

这是因为 Vant 中的主题变量也是在 `:root` 下声明的，所以在有些情况下会由于优先级的问题无法成功覆盖。通过 `:root:root` 可以显式地让你所写内容的优先级更高一些，从而确保主题变量的成功覆盖。

### CSS Variables 和 var() 函数

CSS 变量是由开发者自己定义的，属性名必须带有前缀 `--` 。比如 `--main-color: black`，表示的是带有值为 `black` 的自定义属性，可以通过 [var() 函数](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FCSS%2Fvar "https://developer.mozilla.org/zh-CN/docs/Web/CSS/var")**在全文档范围内复用**。

CSS 变量受级联的约束，并从其父级继承其值，每一个 CSS 变量可以多次出现，并且变量的值将会借助级联算法和自定义属性值运算出来。

```css
:root {
  --text-color: #488cff;
}

p {
  color: var(--text-color);
}
```

在上述示例中，如果我想修改 `<p>` 标签内字体的颜色，只需要修改变量 `--text-color` 就可以了，比如刚刚提到的连写 root 提高优先级。

```css
:root:root {
    --text-color: pink;
}
```

如果你只想自定义一个特定的组件，只需为这些组件单独添加**内联样式**。

```html
<p style="--text-color: red">蓝屏的钙</p>
```

出于性能原因，对于单个特定组件，更加推荐在类名下添加自定义 css 变量，而不是在全局的 `:root` 下。

```css
p {
    --text-color: blue;
    color: var(--text-color)
}
```

这三种方法都能达到覆盖 CSS 变量的目的。

### step 1：配置 CSS 变量

接下来，我们着手来实现自己的 CSS 变量。首先新建一个 `css-vars.css` 文件：

```css
// src/style/theme/css-vars.css
:root {
  /* Color Palette */
  --g-white-1: #fff;
  --g-red-1: #e4004f;

  /* Component Colors */
  --g-primary-color: var(--g-red-1);
  
}
```

这里面定义了两种变量 —— “基础变量” 和 “组件变量”。其中，组件变量继承基础变量。这样的好处是既可以直接修改基础变量影响所有相关组件，也可以有针对地修改某个组件变量而不造成全局变量污染。

### step 2：替换 Vant2.0 组件

Vant2.0 组件并没有使用 CSS 变量，所以我们要将组件中的颜色统统替换成自己的 CSS 变量，新建一个 `vant.scss` 文件：

```scss
// src/style/vant.scss
.van-button--primary {
    color: var(--g-white-1) !important;
    background-color: var(--g-primary-color) !important;
}
```

在 `main.js` 入口文件中导入这两个文件，并重启服务。

```js
// main.js
import './style/theme/css-vars.css';
import './style/vant.scss';
```

![](https://cdn.wallleap.cn/img/pic/illustration/202308040943639.png)

可以看到，按钮的颜色已经变成了我们自己的变量了！通过这种方式，我们可以将更多的组件加进来，设置成自己喜欢的颜色。

### step 3：setProperty 动态控制 CSS 变量

其实在介绍 CSS 变量时，已经提及了一种控制 CSS 变量的方法，就是在类名下直接自定义 CSS 变量进行覆盖。你可以使用一个布尔值去控制类名的添加与移除，来达到这个目的。

还有一种方式就是通过 JS 控制 CSS 变量，使用 `setProperty` 方法直接设置一个新的值。

```vue
<template>
  <van-button block @click="changeColor('blue')"> 变蓝 </van-button>
  <van-button block @click="changeColor('green')"> 变绿 </van-button>

  <van-button id="login-btn" type="primary" block> 登录</van-button>
</template>

export default {
  methods: {
    changeColor(color) {
        const dom = document.getElementById('login-btn');
        dom.style.setProperty('--g-primary-color', color);
    }
  }
}
```

![](https://cdn.wallleap.cn/img/pic/illustration/202308040943640.gif)

这里演示的是对单个组件的针对性修改，打开控制台你会发现 `--g-primary-color` 变量只在这个按钮身上发生了变化，而全局的变量依旧是红色，这样我们就可以将颜色动态切换的精度控制在单个组件内。

![](https://cdn.wallleap.cn/img/pic/illustration/202308040943641.png)

如果你想修改所有 primary 按钮组件的颜色，那就直接给 HTML（[documentElement](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FAPI%2FDocument%2FdocumentElement "https://developer.mozilla.org/zh-CN/docs/Web/API/Document/documentElement")）修改即可。

```vue
// app.vue
document.documentElement.style.setProperty('--g-primary-color', color);
```

大到动态控制全局组件样式，小到自定义单个组件样式，CSS 变量都能灵活胜任，还是很香的。

## 添加暗黑主题模式

了解了 CSS 变量的使用，我们接下来给项目提供日间和暗黑（夜间）两种主题，供用户动态切换。

所谓模式，就是一套配色。比如日间就是白底 + 黑字，但是到了夜间就得黑底 + 白字。所以 `background-color` 和 `color` 这俩属性就构成了一套最简单的配色，用户在一键点击时，就需要同时更换这两个属性。

目前主流的做法就是单独写一套暗黑配色，通过类名 `dark` 直接挂到 HTML 元素上，所以切换主题就是在切换 `dark` 类名。

![](https://cdn.wallleap.cn/img/pic/illustration/202308040943642.png)

### step 1：准备 drak 配色

先准备一套暗黑风格的配色（颜色你们自己定义，或者直接问 UI 要），覆盖默认的 CSS 变量：

```scss
// src/style/theme/dark.scss
html.dark {
    --g-background: var(--g-black-2);
    --g-text-title-color: var(--g-white-2);
    --g-input-background: var(--g-black-3);
    --g-danger-color: #ff4d4f;
    --g-text-gray-color: var(--g-gray-2);
    --g-text-primary-color: var(--g-gray-2);

    // 覆盖用到的 Vant2 组件的默认样式
    .van-popup>button.van-share-sheet__cancel::before {
        background-color: var(--g-black-1)
    }
}
```

在 `main.js` 入口文件中添加。

```js
// main.js
import './style/theme/css-vars.css';
import './style/vant.scss';
import './style/theme/dark.scss';
```

### step 2：classList 切换 dark 类名

处于安全考虑，推荐使用 [document.documentElement.classList](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FAPI%2FElement%2FclassList "https://developer.mozilla.org/zh-CN/docs/Web/API/Element/classList") 的内置方法给 HTML 动态添加和移除 `dark` 类名，而不是直接通过捕获 HTML 修改其 `className`，因为 HTML 上可能还有其他的类名，如果直接使用 `className` 就会将之前的类名全部覆盖，这一点需要注意。

```js
darkThemeChange(theme) {
    if (theme === 'dark') {
        document.documentElement.classList.add('dark');
    } else {
        document.documentElement.classList.remove('dark');
    }
},
```

![](https://cdn.wallleap.cn/img/pic/illustration/202308040943643.gif)

## Antd 的 CSS-in-JS

最后一种方案就是 `CSS-in-JS`， Antd5.0 版本就是采用的这种方式。不过本人能力有限，还没完全吃透这个技术方案，就不深入分析了，这里就简单和大家聊聊，等在下研究透了再来和大家分享😬

### 什么是 CSS-in-JS

> 在 5.0 版本的 Ant Design 中，我们提供了一套全新的定制主题方案。不同于 4.x 版本的 less 和 CSS 变量，有了 CSS-in-JS 的加持后，动态主题的能力也得到了加强。
> 
> 1.  支持动态切换主题；
> 2.  支持同时存在多个主题；
> 3.  支持针对某个/某些组件修改主题变量；
> 4.  ....

看上去很高大上有木有？其实 CSS-in-JS 本身只是一种操作方式，比如在 React 中，我们就是在用 JavaScript 写 HTML 和 CSS。

```js
const style = {
  color: 'red',
  fontSize: '46px'
}
```

上述方式就是 React 对 CSS 的一种简单封装。但是由于 CSS 的封装非常弱，或者说功能不够强大，于是就出现了一系列的第三方库，用来加强 React 的 CSS 操作。它们统称为 `CSS in JS`。意思就是使用 JS 语言去写 CSS。比如 `styled` 的写法，就是由 [styled-components](https://link.juejin.cn/?target=https%3A%2F%2Fstyled-components.com%2F "https://styled-components.com/") 首创：

```js
import styled from 'styled-component';
import { List } from 'xxx';

// 创建样式组件
const StyledList = styled(List)`
  border: 1px solid ${(p) => p.theme.colorPrimary};
  border-radius: 2px;
  box-shadow: 0 8px 20px ${(p) => p.theme.colorShadow};
`;

const App: FC = ({ list }) => {
  return (
    // 引用组件
    <StyledList dataSource={list} />
  );
};
```

主流 CSSinJS 库基本都支持这种写法。

### Ant Design Style

> 由于 CSS in JS 的写法过多，所以我们需要给出一种最佳实践的写法，能兼容 V5 的 Token System、自定义主题、较低的研发心智和良好的扩展性。

在 Antd 看来，社区目前主流的几款 CSS in JS 多多少少都存在一些弊端，于是便推出了自己的最佳解决方案 —— [Ant Design Style](https://link.juejin.cn/?target=https%3A%2F%2Fant-design.github.io%2Fantd-style "https://ant-design.github.io/antd-style")。

antd-style 内置了 `@emotion/styled` 作为 styled 语法的样式引擎，选择了 `emotion` 作为 css 语法的样式引擎，有兴趣的小伙伴可以看看它的 [设计理念与实施策略](https://link.juejin.cn/?target=https%3A%2F%2Fant-design.github.io%2Fantd-style%2Fguide%2Fstrategy "https://ant-design.github.io/antd-style/guide/strategy")。

antd-style 通过在容器组件 `ThemeProvider` 上修改 `apperance props`，即可实现主题切换，这是也是动态主题最简单的使用方式。

```js
import { ThemeProvider } from 'antd-style';

export default () => {
  return (
    // 自动变为暗色模式
    <ThemeProvider apperance={'dark'}>
      <App />
    </ThemeProvider>
  );
};
```

## 总结

文本主要分享如何定制主题，以及定制主题的两种主流方案。

第一种方案通过配置 less-loader 修改 less 变量以达到修改主题色的目的，但是无法做到动态切换；

第二种方案就是使用原生 CSS 变量。通过修改 CSS 变量，我们实现了动态修改颜色，并且针对不同的修改对象，可以控制修改的覆盖范围，大到全局组件，小到单个组件，都能精准控制。

随后，我们给出了目前主流的暗黑（夜间）模式解决方案，即通过 HTML 切换类名的方式去匹配一整套的暗黑配色。

最后，我们简单聊了聊 CSS-in-JS 技术方案，关于这部分内容，打算后续深入了解后再和大家分享，敬请期待哈~😃

## 参考资料

- [定制主题 | Vant](https://link.juejin.cn/?target=https%3A%2F%2Fvant-contrib.gitee.io%2Fvant%2Fv2%2F%23%2Fzh-CN%2Ftheme "https://vant-contrib.gitee.io/vant/v2/#/zh-CN/theme")
- [Modify Variables | Less](https://link.juejin.cn/?target=https%3A%2F%2Flesscss.org%2Fusage%2F%23using-less-in-the-browser-modify-variables "https://lesscss.org/usage/#using-less-in-the-browser-modify-variables")
- [:root | MDN](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FCSS%2F%3Aroot "https://developer.mozilla.org/zh-CN/docs/Web/CSS/:root")
- [使用 CSS 自定义属性（变量）| MDN](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FCSS%2FUsing_CSS_custom_properties "https://developer.mozilla.org/zh-CN/docs/Web/CSS/Using_CSS_custom_properties")
- [var() | MDN](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FCSS%2Fvar "https://developer.mozilla.org/zh-CN/docs/Web/CSS/var")
- [documentElement | MDN](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FAPI%2FDocument%2FdocumentElement "https://developer.mozilla.org/zh-CN/docs/Web/API/Document/documentElement")
- [Element.classList | MDN](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FAPI%2FElement%2FclassList "https://developer.mozilla.org/zh-CN/docs/Web/API/Element/classList")
- [CSS in JS 简介 | 阮一峰](https://link.juejin.cn/?target=https%3A%2F%2Fwww.ruanyifeng.com%2Fblog%2F2017%2F04%2Fcss_in_js.html "https://www.ruanyifeng.com/blog/2017/04/css_in_js.html")
- [CSS in JS 快速入门 | Ant Design Style](https://link.juejin.cn/?target=https%3A%2F%2Fant-design.github.io%2Fantd-style%2Fguide%2Fcss-in-js-intro "https://ant-design.github.io/antd-style/guide/css-in-js-intro")
