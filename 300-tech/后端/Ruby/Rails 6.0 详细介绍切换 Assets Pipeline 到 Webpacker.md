---
title: Rails 6.0 详细介绍切换 Assets Pipeline 到 Webpacker
date: 2024-08-16T11:34:00+08:00
updated: 2024-08-21T10:32:31+08:00
dg-publish: false
source: //ruby-china.org/topics/38832
---

> 开篇说明：本文力争成为介绍和开始使用 webpacker 最好用的中文入门文档，欢迎各位提出改进建议。

Rails 6 默认启用了 webpacker 作为 Javascript 文件的管理方案，但使用起来与 assets pipeline 颇有不同，但 webpacker 的优点非常明显，所以强烈建议各位 Railser 切换，故此这里详细介绍。

在文尾，我们 dao42 团队准备了开箱即用的初始化新项目方案，有兴趣动手的朋友可以研究一番。

## 为什么要使用 webpacker

assets pipeline 从技术角度非常先进，一直以来是作为前端打包最佳实践被推荐的。但 assets pipeline 也有几个痛点问题由于架构的原因没能解决的很好。

1. assets pipeline 在大的项目中，开发过程非常慢。以我们最近的一个项目，引入了十多个前端库，它的首次编译时间已经超过了 **40s**，改动一个 css 导致的重新编译也有 **15s** 之久。这个对于重视开发体验的 Rails 来说已经无法接受了。
2. assets pipeline 要求前端库针对它作少量的适配。这导致我们往往必须打一个 gem 包才能使用这个这个前端库，否则就要自己进行“魔改”，难度较大，容易出错。如果 Gem 包的作者没能及时更新，我们只能自己 fork 再修改，想想每一个前端库都要这么做，成本是很高的。
3. assets pipeline 对 ES6 的支持有问题，随着 ES6 取代 coffeescript 成为前端界的标准，这个问题变的突出了。

随着 nodejs 生态的发展，尤其是 webpack 的长足发展，webpack 已经成为大多前端库安装的标准，它不仅在功能上与 assets pipeline 媲美，同样具备前端打包的最佳实践，更还有 HMR 模块热加载等酷炫功能。webpack4.0 的发布，解决了它被诟病的配置复杂的问题。所以让 webpack 成为 Rails 中前端打包的标准，让 Rails 的开发效率更进一步，已经刻不容缓了。

webpacker 是 webpack 的 封装，让我们能够简化配置，采用 Rails 惯例进行开发。现支持 webpack4.0 最新版本。webpacker 解决了以下痛点。（以下，我们不区分 webpacker 与 webpack，统一叫 webpacker）。

1. webpacker 有 `webpack-dev-server` 工具，轻松解决开发环境编译慢的问题，经测试，之前几十秒的编译过程，现在连 `1s` 都不到。
2. webpacker 是目前前端界标准事实的打包工具，从此引用 JS 库不用找 gem 包了。
3. webpacker 让你写 JS 更规范，可以直接用 ES6。

那么，我们开始研究 webpacker 的使用吧。

## webpacker 的安装

经过研究，迁移旧的 assets pipeline 成本与风险很大，比如 jQuery 1.x 根本不支持 webpacker，大量旧的库无法很好支持 webpacker，所以升级项目时，建议两者都保留，但后续可采用 webpacker 方式进行开发。

所幸的是，Rails 6.0 正是这样想的，它已经默认安装了 webpacker，但同时也支持 assets pipeline，所以我建议暂时不要动任何 assets pipeline 的代码。当你按照 Rails 6 的升级指南（在这里：[https://edgeguides.rubyonrails.org/6_0_release_notes.html](https://edgeguides.rubyonrails.org/6_0_release_notes.html)）后，即可获得使用 webpacker 的能力。

如果你想完全移除 assets pipeline，或全新构建一个没有 assets pipeline 的应用，你可以往下翻到最后一节，研究一下我为大家准备的 dao42/rails-template 模板。

升级项目后，我们还要做以下步骤来启用全功能的 webpacker。

第一步，在 `app/javascript` 目录分别创建 `styles/`, `fonts/`, `js`, `images` 目录。

第三步，解掉 `app/javascript/packs/application.js` 中的 images 打包注释：

```
const images = require.context('../images', true)
const imagePath = (name) => images(name, true)
```

第四步，将 `app/views/layouts/application.html.erb` 中的 `stylesheet_include_tag` 标签下面增加 `stylesheet_pack_tag` 和 `javascript_pack_tag`。

```
<%= stylesheet_link_tag 'application', media: 'all', 'data-turbolinks-track': 'reload' %>
<%= javascript_include_tag 'application', 'data-turbolinks-track': 'reload' %>
<%= stylesheet_pack_tag 'application', media: 'all', 'data-turbolinks-track': 'reload' %>
<%= javascript_pack_tag 'application', 'data-turbolinks-track': 'reload' %>

```

最后，在开发时不要忘了启动 `./bin/webpack-dev-server`，这个工具可以实时编译资源，并帮你刷新浏览器。

## webpacker 下如何写我们的 JS 与 CSS

上述安装配置后，我们新增了一个专门编写 webpacker 下 JS 与 CSS 的地方，现在我们的目录布局如下：

`app/javascript/packs/`: 放在 packs 里面的 js 文件，最终会打包出来供 `javascript_pack_tag` 引用。

`app/javascript/js/`: 在这里写我们自己的 JS 逻辑。

`app/javascript/styles/`: 在这里写我们自己的 CSS 逻辑。推荐直接写 SCSS。

`app/javascript/images/`: 放项目用到的图片资源。`fonts/` 目录类同。

### 思维调整

我们从思维上要发生一个根本变化，即：`一切皆 JS`。所以我们的 CSS 也需要在 application.js 中引入：

```
// app/javascript/packs/application.js
import 'styles/application'
```

### 模块隔离

webpacker 会将各文件中的对象都隔离，所以无须担心互相影响，也没有所谓的全局变量。所以写一个组件也变得很简单：

```
// app/javascript/js/a.js
var a = 1
export default a
```

```
// app/javascript/js/b.js
import a from './a'
console.log(a)
```

### 引入前端库

引入前端库变的简单了：

先用 yarn 安装一下

```
$ yarn add bootstrap
```

然后在 `app/javascript/packs/application.js` 中引用

```
import 'bootstrap/dist/js/bootstrap'
```

CSS 中是这样的，`app/javascript/styles/application.scss` 引用

```
@import 'bootstrap/scss/bootstrap';
```

以上路径可以直接去相关前端库的安装说明中找即可。

注意：引用相对路径时一定要加上 `./`，否则不生效并且不一定报错。

## JS 调试

webpacker 默认已经将 map 文件装载，所以调试起来仍然相当方便。见下图

[](https://daotestimg.dao42.com/ipic/130239.jpg)

[![](https://daotestimg.dao42.com/ipic/130239.jpg)

](<https://daotestimg.dao42.com/ipic/130239.jpg>)

同时，也可以用 `debugger` 指令来强制下断点。

### View 中引用资源的方法

`stylesheet_link_tag` 与 `javascript_include_tag` 不再使用，与此类似，webpacker 提供了几个类似的方法。

现在常用的几个

1.`stylesheet_pack_tag`: 加载 CSS 资源

用法：

`stylesheet_pack_tag 'application', media: 'all', 'data-turbolinks-track': 'reload'`

2.`javascript_pack_tag`: 加载 JS 资源

用法

`javascript_pack_tag 'application', 'data-turbolinks-track': 'reload'`

3.`asset_pack_path`: 类似于 `asset_path`，加载图片、字体等资源

用法 ( 注意 **media/images** 这个路径，要加上 media 这个前缀，而且不能使用相对路径 )

`asset_pack_path('media/images/favicon.ico')`

4.`image_pack_tag`: 取代 `image_tag`，更明确的加载图片

用法

`image_pack_tag 'media/images/calendar.png'`

或 ( 两者等价 )

`image_pack_tag 'calendar.png'`

## webpacker 的原理简单说明

webpacker 是包装自 webpack 并且基本做到零配置开箱即用。配置文件在 config/webpacker.yml，无须做任何调整。

webpack 分为 Entry, Output, Loaders, Plugins 几个关键概念。最新的大版本是 v4。

webpack 会将所有的资源都进行依赖分析，生成依赖树，并最终打成一个或多个包。

## 示例：常用库的安装

### jQuery

```
$ yarn add jquery@3.3.1
```

添加一个 webpack 插件，确保 $, jQuery, 'window.jQuery' 可以在 JS 库中使用。

```
// config/webpack/environment.js

const webpack = require('webpack')
environment.plugins.append(
  'Provide',
  new webpack.ProvidePlugin({
    $: 'jquery',
    jQuery: 'jquery',
    'window.jQuery': 'jquery',
    Popper: ['popper.js', 'default']
  })
)
```

开始使用

```
// app/javascript/packs/application.js
import 'jquery'
```

### Bootstrap

```
$ yarn add bootstrap@4.3.1 popper.js
```

在 app/javascript/styles/ 中创建 `bootstrap_custom.scss`

```
// app/javascript/styles/bootstrap_custom.scss
@import 'bootstrap/scss/bootstrap';
```

```
// app/javascript/styles/application.scss
@import 'bootstrap_custom'
```

最后，引用 JS：

```
// app/javascript/packs/application.js
import 'bootstrap/dist/js/bootstrap'
```

## RoadMap

- [*] 处理 assets pipeline 切换的兼容

## rails-template

是不是仍然感觉有点复杂，没关系， `dao42/rails-template` 已经为你的新项目初始化做好了开箱即用。

`dao42/rails-template` 已经经历过 2 年半的持续升级维护，做好了新项目很多方面的开箱即用。

1. `dao42/rails-template` 全面拥抱 webpacker。
2. `rails-template` 集成最新的 `bootstrap4`, `font-awesome5`，无须额外任何配置。
3. `rails-template` 集成 mina 发布套件，一键发布项目。
4. `rails-template` 集成了 `adminlte 3` 最新版，自带管理后台。

点击这里开始体验 webpacker 的便利之处：[dao42/rails-template](https://github.com/dao42/rails-template)。

## 附：如何完全移除 assets pipeline

除非你打算全新构建你的 Rails 6 应用，并采用完全的前后端分离方案（如 vue，react 等）。否则强烈建议保留 assets pipeline。因为还有众多 Rails Engine 还未支持最新的 webpacker。

新项目

```
$ rails new my_app --webpack --skip-sprockets
```

旧项目，将 `config/application.rb` 的 `require 'rails/all' 替换为：

```
require "rails"

%w(
  active_record/railtie
  active_storage/engine
  action_controller/railtie
  action_view/railtie
  action_mailer/railtie
  active_job/railtie
  action_cable/engine
  action_mailbox/engine
  action_text/engine
  rails/test_unit/railtie
  #sprockets/railtie
).each do |railtie|
  begin
    require railtie
  rescue LoadError
  end
end
```

第二步，删除 `app/assets` 目录，在 `app/javascript` 目录分别创建 `styles/`, `fonts/`, `js`, `images` 目录。

第三步，解掉 app/javascript/packs/application.js 中的 images 打包注释：

```
const images = require.context('../images', true)
const imagePath = (name) => images(name, true)
```

第四步，将 `app/views/layouts/application.html.erb` 中的 `stylesheet_include_tag` `javascript_include_tag` 分别替换为 `stylesheet_pack_tag` 和 `javascript_pack_tag`。

最后，在开发时不要忘了启动 `bin/webpack-dev-server`，这个工具可以实时编译资源，并帮你刷新浏览器。

## 实践一段时间后的还存在的待处理的事宜

目前 webpacker 使用中还有一些问题：

1. 开发环境 CSS 是异步加载，无法保证在 DOMLoaded 后各 div 的尺寸大小。这个要小心你的 JS 代码。（可以修改 webpacker.yml 中 development 选项的 extract_css，改为 true）
2. 开发环境 Turbolinks 集成有些问题，load 事件无法保证正确加载，像 `turbolinks:before-render` 事件也不触发，要等后续更新处理。( 已 fixed）

其他的坑已经由 [dao42/rails-template](https://github.com/dao42/rails-template) 填好了。大家可以参考处理。

## 参考文稿

- [From the Asset Pipeline to Webpack](https://medium.com/michelada-io/from-the-asset-pipeline-to-webpack-ce5a4bc323a9)
- [Using Webpacker in Ruby on Rails applications](https://www.lugolabs.com/articles/using-webpacker-in-ruby-on-rails-applications)
- [渐进式迁移到现代前端开发模式 — 用 Webpacker 替代 Sprockets 实现前端资源的管理](https://ruby-china.org/topics/38214)

---

好，不过感觉离开篇说明的目标有些距离，讲的还是有些不详细，旧项目比如 5.0 能不能将 assets 用 webpacker 替换，替换的好处在哪，具体怎么替换，必须要升级到 rails6 吗，文中介绍时候建议用截图对比方式将 webpacker 和 assets 目录结构，js css 文件的引入使用作一介绍，是不是 gemfile 里不用再添加 gem 'bootstrap' 等了？文章介绍使用 jQuery 时候要添加一个 webpacker 插件，bootstrap 却不用，那到底用哪些需要添加哪些不需要，怎么判断呢？个人从小白阅读参考试用角度提以上问题和建议，如有不妥请指正

---

用了一下，通过 css loader 加载的 css 在 html 中不带 data-turbolinks-track 的属性，页面切换的时候 css 加载会慢半拍。没有仔细看 css loader 文档不知道有没有方便加标签属性的方法……

---

Assets Pipeline 暂时不用急着去掉，不想用自己开发的时候不要往那里加文件就好了。很多 Rails Engine 可能带了 Assets 需要依赖这个。

---

`我们从思维上要发生一个根本变化，即：一切皆 JS。所以我们的 CSS 也需要在 application.js 中引入` `stylesheet_pack_tag: 加载 CSS 资源`

配置的时候有点疑惑，css 文件需要在 application.js 中引入， `stylesheet_pack_tag` 这个是用来做什么的呢？

好问题，可以这样理解：

在 packs/application.js 中 import 的是各种资源 ( 图片，css, 字体 ), 这些内容在开发调试的时候由 webpack 用 socket 传给前端方便调试。

在生产环境， `stylesheet_pack_tag` 可以将 `packs/application.js` import 的 css 解析成具体的 link 标签。

而 `javascript_pack_tag` 仅仅将 `packs/application.js` 中的 js 打包解析成具体的 script 标签。

了解了，我开发的时候没加 `stylesheet_pack_tag` 也能用，看样子得我部署了，才能发现这一点

非常好的建议，现在全篇已经按这个思路做了调整，并经过多次真实项目测试。

---

实践一段时间后的还存在的待处理的事宜

目前 webpacker 使用中还有一些问题：

1. 开发环境 CSS 是异步加载，无法保证在 DOMLoaded 后各 div 的尺寸大小。这个要小心你的 JS 代码。
2. 开发环境 Turbolinks 集成有些问题，load 事件无法保证正确加载，像 `turbolinks:before-render` 事件也不触发，要等后续更新处理。

其他的坑已经由 [dao42/rails-template](https://github.com/dao42/rails-template) 填好了。大家可以参考处理。

---

目前 webpacker 最大的缺陷就是 不能像 assets pipeline 那样便捷使用 engine 里的 js/css 文件，官方给的方案也麻烦的很。

[https://github.com/rails/webpacker/blob/master/docs/engines.md](https://github.com/rails/webpacker/blob/master/docs/engines.md)

---

关于 engine 与 webpacker 的使用，我最近实践出了一个比较好的方案。[https://ruby-china.org/topics/39025](https://ruby-china.org/topics/39025)
