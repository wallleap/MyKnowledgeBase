---
title: 渐进式迁移到现代前端开发模式 — 用 Webpacker 替代 Sprockets 实现前端资源的管理
date: 2024-08-16T11:40:00+08:00
updated: 2024-08-21T10:32:33+08:00
dg-publish: false
source: //ruby-china.org/topics/38214
---

这是我在 [[北京] [2019 年 3 月 9 日] Ruby Saturday](https://ruby-china.org/topics/38181) 的讲稿

## Webpacker 4

Webpacker 是 Rails 官方团队提出的一套在 Rails 上集成 Webpack 的方案

### 项目动机

- Webpack 是目前前端社区资源打包工具的主流选择
- Webpack 很难配置
- Rails 需要接纳现代的前端技术
- 需要在视图中引用 Webpack 管理的资源

### 提供的功能

- 包装了 webpack-server 监听前端资源的变化，按需编译
- 提供了一套 Webpack 默认配置，集成了如 file-loader 等常见插件
- 提供了一组 Rails 视图 Helpers，用于引用 Webpack 管理的资源

## 从应用中剥离 Sprockets

### 为什么

慢、不支持最新的 ES 标准、接入现代风格的 NPM 包困难

### 怎么做

对于新创建的应用，可以使用 `--skip-sprockets`

对于旧应用，将 `config/application.rb` 中默认的 `require 'rails/all'` 修改成

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
  sprockets/railtie
).each do |railtie|
  begin
    require railtie
  rescue LoadError
  end
end
```

（实际上就是 `rails/all` 的源码）

然后删除 `sprockets/railtie`

搜索 `config` 目录下，文本搜索所有 `assets` 关键字，注释或删除

## 怎样通过 Webpacker 完成 Sprockets 的工作

### 复用 Rails 的 assets 目录

参考 `config/webpacker.yml` 的注释，将 `resolved_paths: []` 修改成 `resolved_paths: ['app/assets']`

以此将 Rails 的 `app/assets` 目录加入到 Webpack 的查找路径

### 自动构建图片、字体等静态资源

参考 Webpack 文档 [Dependency Management](https://webpack.js.org/guides/dependency-management/#context-module-api) 中提到的 `require.context` 函数，从其文档中摘抄

```
const importAll = (r) => r.keys().map(r)
importAll(require.context('images', false, /\.(png|jpe?g|svg)$/i));
```

到 `javascript/application.js` 即可

### 引入样式文件

用 Webpack 提供的 `import`，如果迁移自 Sprockets，样式文件在 `assets/stylesheets` 的话，代码形如 `import "stylesheets/application"`

### 用 Webpacker 提供的 Helper 替换 Sprockets 的

- `stylesheet_tag` => `stylesheet_pack_tag`
- `javascript_tag` => `javascript_pack_tag`
- `asset_path` => `asset_pack_path`
- `image_tag` => `image_pack_tag`
- `asset_url` => `asset_pack_url`

Webpacker 额外提供了 [stylesheet_packs_with_chunks_tag](https://github.com/rails/webpacker/blob/master/lib/webpacker/helper.rb#L125) 用于暴露 Webpack 的 [SplitChunksPlugin](https://webpack.js.org/plugins/split-chunks-plugin/)

## 扩展：禁用 ActionView 对模型验证失败时的 HTML 包装

Rails 会把模型验证失败的字段的 `<input>` 用 `<div class="field_with_errors">` 包裹，这会破坏一些前端框架的行为

这个行为是 `ActionView::Base.field_error_proc` 的默认实现，可以覆盖他屏蔽掉这个行为，参考：

[https://github.com/jasl/cybros_core/blob/master/config/initializers/action_view.rb](https://github.com/jasl/cybros_core/blob/master/config/initializers/action_view.rb)

## 扩展：增强 Gem 中代码的行为

一个来自官方指南 Rails Guide 的隐藏技巧

[https://edgeguides.rubyonrails.org/engines.html#overriding-models-and-controllers](https://edgeguides.rubyonrails.org/engines.html#overriding-models-and-controllers)

### TL;DR

利用 `class_eval` 特性实现的 Override pattern

### 应用：为 form helpers 增加当字段验证失败时增加 Bootstrap 渲染表单控件错误所需的 class

参考

[https://github.com/jasl/cybros_core/blob/master/app/overrides/action_view/helpers/form_builder_override.rb](https://github.com/jasl/cybros_core/blob/master/app/overrides/action_view/helpers/form_builder_override.rb)

这段代码增强了 form helpers 的行为，增加了 `class_for_error` 参数，当字段验证失败时渲染指定的 `class`，并且增加了 `error_message` helper 当字段错误时渲染错误信息。

这样强化过的 form helpers 使得 `simple_form` 在通常情况下不在需要（当然我在示例中没有做太重的封装），并且让代码保持干爽。

效果形如：[https://github.com/jasl/cybros_core/blob/master/app/views/admin/users/_form.html.erb](https://github.com/jasl/cybros_core/blob/master/app/views/admin/users/_form.html.erb)

## 示例应用

[https://github.com/jasl/cybros_core](https://github.com/jasl/cybros_core)

## 扩展：集成前端库的坑

### 存在不能引入的库

一些几年没有更新的库，有可能 import 后为 null 的情况，需要检查这个库是否使用 js 的 export 进行导出，如果不是则无法用在 webpack 中。

### jQuery 导出到 `window` 对象

在 webpack 的 entry 中引入的库，都无法在外部可见，这会影响大多数 jQuery 插件， 因为很多时候我们需要在 HTML 中插入 `<script>` 所以你需要通过这种方式将 jQuery 导出到 `window` 对象，以便 HTML 中的 `<script>` 工作正常

`window.$ = window.jQuery = require("jquery");`

### 涉及引用静态资源的库

涉及引用静态资源的库，需要做一些处理，才能正确的让 webpack 包含这些静态资源， 比如示例应用中包含了 web font 引入的 Font Awesome，参考

[https://github.com/jasl/cybros_core/blob/master/app/assets/stylesheets/application.scss#L5](https://github.com/jasl/cybros_core/blob/master/app/assets/stylesheets/application.scss#L5)

遇到这种库，需要分析一下其源码，如果其路径不可定制，则可能无法引入到 webpack 中，这里不下定论，需要具体问题具体分析

### 一些库与 Turbolinks 冲突

作为初级前端，我仍然推崇 Turbolinks，因为无需特别处理，即可达到前后端分离的无闪屏页面切换体验，但是，一些新的库会与其冲突。

#### 例子 1：Font Awesome 的 JS 方式引入

Font Awesome 的 JS 引入方式的效果是其 JS 会监视 DOM 变化，发现是图标的标签则将其替换成 SVG（这里不讨论这样设计的意义）， 这种方式在 Turbolinks 下，有可能造成闪屏，建议使用传统 Web font 方式引入。

#### 例子 2：React

在 Turbolinks 下直接使用 React 时，页面切换会导致 React 的状态不重置并且重复执行，有一些解决相关问题的库，如 [https://github.com/renchap/webpacker-react](https://github.com/renchap/webpacker-react)

---

因为我倾向 Gitlab、Github 的传统方式前端和 React/Vue 的混合前端技术栈，传统前端这边我仍然认为 Bootstrap 是生态最好的，并且我调研过一些 React 的库（我的一些项目需要的组件），仍然提供的是 Bootstrap 标准的 CSS（比如 [https://github.com/mozilla-services/react-jsonschema-form](https://github.com/mozilla-services/react-jsonschema-form)）

此外，我想要一个类似 Gitlab 的带左右分栏的布局，这种方式的布局我觉得最适合后台管理，所以调研过一波，开源且相对质量靠谱的只有 CoreUI，所以做了这样的选择

这些插件都开源的你可以自己集成啊，无非就是记得用他们提供 bootstrap 的 css 而已，这就是用 Bootstrap 的好处了，直接换 bootstrap 标准版本的，通常显示效果就是正确的了。

CoreUI 收费就是把他们帮你预先集成好，外加微调了一下样式

---

其实我现场还分享了一些别的观点，我认为 GraphQL 流行后，我就会转换到前后端分离方式去了，并且我会认为在那个时候，Rails 就可以进入历史书了。不过我个人对 GQL 的研究还很初级，暂时就不分享这部分了，现场我也只是说这是我的一部分私货

---

混合前端技术栈我同意，现下应该是比较平衡的一种方式。 但是 CoreUI 一些比较常用的组件都是收费的，比如 select2 基于 bootstrap 的 ui 库还有很多，比如 adminLTE 应该比 CoreUI 名气大吧，而且也提供更多的免费组件

另外不用 AdminLTE 的原因是他还停留在 Bootstrap 3，我还是希望尽量用最新版本的组件的，包括将来 Bootstrap 5 要去掉 jQuery 依赖，就算是传统风格的前端架构，也还是要跟上潮流，不然就很难补上了，这才是我标题提到的渐进式迁移到现代前端

一个是 CoreUI 是去年的项目比较新，另一个，Boostrap 3 包括一干老式的 jQuery 插件很多不能直接集成进 Webpack 中，这跟那些维护不周的项目仍在使用老式 JS 模块加载方式有关。

所以这里我做出这个选择，我希望保持传统 Web MVC 的开发体验，同时甩掉过去的包袱，不然大可直接用 Sprockets，没必要迁移到 Webpacker 去了

---

哈哈 在 medium 也看过一篇 [https://medium.com/@coorasse/goodbye-sprockets-welcome-webpacker-3-0-ff877fb8fa79](https://medium.com/@coorasse/goodbye-sprockets-welcome-webpacker-3-0-ff877fb8fa79)

只不过 rails 6 就默认 webpacker 了 x
