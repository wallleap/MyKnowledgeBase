---
title: Rails 7 前端方案前瞻
date: 2024-08-16T10:02:00+08:00
updated: 2024-08-21T10:32:31+08:00
dg-publish: false
source: //ruby-china.org/topics/41699
---

原文链接： [https://geeknote.net/Rei/posts/286](https://geeknote.net/Rei/posts/286)

---

自 Rails 6 引入了 Webpacker 以来，Rails 的前端部分就引发了很多吐槽。吐槽主要分两类：

- 未接触过前端深坑的人看到 Webpacker 引入的几百个依赖感到恐惧。
- 已经了解 Webpack 的人需要额外学习 Webpacker 的配置。

有关 Webpacker 的安装和使用问题，资深 Rails 开发者也不一定懂解决。遇到这些问题都建议先跳过 Webpacker。一个号称开发者友好的框架，却要在一开始跳过某个模块，这着实劝退了一些人。

不过这个问题有望在 Rails 7 解决。本文就来前瞻一下 Rails 7 的前端方案。

## 前端问题的产生

首先要了解一个问题，为什么前端需要构建工具？

前端构建工具是为了处理以下问题：

- **编译**。原生 CSS/JavaScript 的语法不够强大，需要借助 SCSS 和 ES6，JSX 等工具编译前端代码。
- **打包**。大量零碎的前端文件会影响网站载入速度，需要打包成一个大文件。
- **哈希**。为了减少网络传输，静态文件一般会设置浏览器缓存。又为了能正确更新前端文件，需要给文件名加上文件的哈希值。

Rails 的 Asset Pipeline 就是为了处理这些问题而诞生。首次引入 Asset Pipeline 的 Rails 3.1 发布于 2011 年 8 月 31 日。相比之下，NPM 发布于 2010 年 1 月 12 日，当时的打包方式还没形成共识。而 Webpack 发布于 2014 年 2 月 19，比 Asset Pipeline 晚了两年半。当时 Asset Pipeline 设计是领先的。

从那以后，前端领域有了很多发展。一个是打包方式的确立，从 AMD、CMD、UMD 到 ESM；一个是 ES6 语法的普及，同时又多了 TypeScript；另外还有浏览器原生支持了 ES6，HTTP/2 实现了多文件并行下载等。

相比之下，Asset Pipeline 的打包就跟不上变化了。它基本上是把源码编译之后，进行字符串拼接。因为不是基于语义层面进行打包，所以它不理解各种打包规范的语法，也不能进行代码裁剪。

这时候一种方案是用 JavaScript 社区的打包工具替换掉 Asset Pipeline，Webpack 就是其中一个。Webpack 是前端社区打包工具的集大成者，通过插件机制可以实现各种前端代码编译打包和拆分。Rails 基于 Webpack 加上自己选择的插件和配置就是 Webpacker。

但 Webpack 也有一些问题，最主要的就是它的配置过于复杂。为了什么都能兼顾，也就什么都要配置。Rails 引入 Webpacker 之后，虽然解决了如何打包复杂前端代码的问题，但它本身也是个问题，它和 Asset Pipeline 并存也让不少原先的 Rails 开发者感到困惑。

## Rails 7 的新方案

在吸取了 Rails 6 的教训后，Rails 7 将会在前端管理方案做出改变。DHH 最近写了一篇文章描述他的设计：《[Rails 7 will have three great answers to JavaScript in 2021+](https://world.hey.com/dhh/rails-7-will-have-three-great-answers-to-javascript-in-2021-8d68191b)》。

概括来说，Rails 7 提供了三个前端方案：

1. Asset Pipeline + ImportMap
2. cssbundling && jsbundling
3. API Mode

下面分别介绍这三个方案。

## Asset Pipeline + ImportMap

这是 Rails 7 的默认方案。

Asset Pipeline 功能跟之前的作用类似，不同的是默认不对 CSS 和 JavaScript 进行编译和打包。不编译和打包的前提是现在的浏览器都已经原生支持 ES6，可以把 ES6 直接发送给浏览器。同时由于 HTTP/2 协议实现了多路复用，零碎的前端文件不再是问题，还可以得到细粒度缓存的好处。

不过无 bundle 方案有一个问题要解决，就是如何处理加载路径。例如对于代码：

```
import "stimulus"
```

浏览器会加载 `stimulus.js` 这个文件。但是为了缓存，`stimulus.js` 这个文件的文件名会加上一串哈希值，变成 `stimulus-abcdef.js`。为了让浏览器找到需要加载的文件，出现了 ImportMap 这个标准。

ImportMap 的原理是在 HTML 的头部插入一段 json 数据，描述 ES6 模块和文件名的对应关系：

```
<script type="importmap">{
  "imports": {
    "stimulus": "/assets/stimulus-abcdef.js"
  }
}</script>
```

这样浏览器在执行 `import "stimulus"` 的时候，就知道要加载 `/assets/stimulus-abcdef.js` 这个文件，而开发者无感知。

这个方案是否能成为以后的主流，我认为还需要观察。首先 ImportMap 还是草案标准（[Import Maps](https://wicg.github.io/import-maps/)），Chrome 和 Edge 以外的浏览器需要额外的库补充。其次有一些前端的库是以开发环境有打包器为前提开发的，一个库拆成了很多库分发，形成依赖的依赖。而 ImportMap 处理不了依赖的依赖。

我觉得 ImportMap 最大作用是去掉 Rails 默认栈的 Node.js 依赖，不用安装 Node 就可以基于 Rails 的默认栈进行开发。如果需要用到比较多的第三方库，就需要下面的 bundling 方案。

## cssbundling && jsbundling

`cssbundling-rails` 和 `jsbundling-rails` 是 Rails 新增加的两个 gem 包，用于引入外部的前端编译/打包工具。

以前的 Asset Pipeline 是一个编译/打包/哈希一条龙的工具，但如前面提到过，编译/打包方式已经赶不上前端的变化，所以新的方案是把这部分工作交还给前端工具处理，Sprockets 只处理最后的哈希这一步。

[](https://l.ruby-china.com/photo/Rei/ed29c236-517f-4b3e-84ce-43ea9976b610.png!large)

[![](https://l.ruby-china.com/photo/Rei/ed29c236-517f-4b3e-84ce-43ea9976b610.png!large)

](<https://l.ruby-china.com/photo/Rei/ed29c236-517f-4b3e-84ce-43ea9976b610.png!large>)

cssbundling 默认支持 `tailwind|bootstrap|postcss|sass` 等 CSS 处理前端，jsbundling 默认支持 `esbuild|rollup|webpack` 等 js 处理前端。

没有看到需要的处理前端也没关系，其实阅读这个两个 gem 的代码可以发现，这两个 gem 内部并没有运行时代码，只是一些安装配置。自己仿照这些配置就可以任意引入其他处理器，只要把构建好的文件放入 `app/assets/builds` 文件夹就行。

这个方案适合多数现有需要编译前端代码的项目。相比原先的 Asset Pipeline，外置构建工具可以获得完整的前端生态支持。和 Webpacker 方案相比，这个方案把选择权交还给了开发者，不会一上来把人搞懵。

## API mode

最后一个方案是 API mode，前后端分离，甚至前端部分独立一个项目，Rails 只负责 API。

API mode 其实从 Rails 5 就支持了，创建的命令是：

```
$ rails new myapp --api
```

使用 API mode，前端怎么做就与 Rails 无关了。

## 总结

Rails 7 的前端方案是对过往的经验总结，找出一套渐进式的前端方案。一方面降低了新建项目的门槛，不用一上来就面对 webpack 这样的巨无霸，另一方面在引入外部编译工具的时候给开发者充分选择权。

现在 [Rails 7 alpha 版已经发布](https://weblog.rubyonrails.org/2021/9/15/Rails-7-0-alpha-1-released/)，有兴趣的可以进行测试，遇到问题及时向官方反馈。

让 Rails 再次简单。

---

感觉很长一段时间内，还是要以 Webpacker 为主，它距离主流前端生态更为接近

---

Snowpack 和 Vite 的模式未来可能会在前端占有一席之地

可以跟 webpacker 一样通过额外的 gem 支持，例如 vite-ruby。进阶选项应该让开发者自己选择。

---

看了 Rei 说“gem 内部并没有运行时代码，只是一些安装配置”之后，我昨天帮大家加了一个 Bulma 支持

虽然已经很久没用 Bulma 了，最近一直在用 Bootstrap，Bootstrap 自带的 JS 的支持，比如 modal、tooltip 之类的还挺好用的

---

其实之前我就是自己写的

[https://ruby-china.org/topics/39542](https://ruby-china.org/topics/39542)

rails 换前端方案也不难 照着写就好了

ps：esbuild 是好东西 就是还不敢上生产

---

esbuild 的确是个好东西，vite 也没用在 prod 上。可以考虑开发的时候为了节约时间用 esbuild，部署走 pipeline 直接 tsc 好了，机器的时间可以忽略不计。

---

Asset Pipeline 和 Webpacker 这两个当时确实把我搞懵逼了....

---

在开发阶段使用 esbuild 可以提高效率，但在生产环境中，直接使用 [head soccer unblocked](https://headsoccer.pro/) tsc 编译可能更合适。虽然机器时间可以忽略不计，但采用这种方式可以确保代码质量和可维护性。
