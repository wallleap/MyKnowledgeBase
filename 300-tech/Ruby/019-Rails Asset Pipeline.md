---
title: 019-Rails Asset Pipeline
date: 2024-08-09 12:33
updated: 2024-08-09 14:58
---

Rails 管理和部署静态资源的一种方式

由 sprockets-rails gem 来完成的，默认添加了支持

`app/assets`、`lib/assets`、`vendor`、`app/javascript`

`config/application.rb` 可以配置

例如配置 `rails g` 时候的行为

```rb
config.generators do |generator|
  generator.assets false
  generator.view_specs false
  generator.test_framework false
end
```

`config` 放项目的配置，例如 `application.rb` 总体的配置，`database.yml` 数据库的配置，其他 gem 对应的配置等

现在 rails 默认不支持 scss，如果需要用可以在 Gemfile 中引入 `gem "sassc-rails"` 等 Gem

如果需要其他的支持，也可以引入对应的 Gem

`application.js` 中引入其他 js（同级目录下 js 或包提供的）

```js
//= require welcome
```

在 `application.html.erb` 中通过 `<%= javascript_include_tag 'application' %>` 引入

CSS 通过 `<%= stylesheet_link_tag 'application', media: 'all' %>`

图片 `<%= image_tag '子目录/图片.png' %>`

资源目录 `<%= asset_path "文件" %>` 输出的是相对地址

资源目录 `<%= asset_url "文件" %>` 完整的地址，如果生产环境需要指定域名可以 `config/enviroments/production.rb` 中添加 `config.action_controller.asset_host = "https://...."`

CSS 中背景图片也可以使用 erb 语法的

```css
.a {
  background: url(<%= asset_path "bg.png" %>)
}
```

`config/initializers` 目录中的只在项目启动运行一次，如果有额外的文件引入（不通过 application 对应格式引入）需要在 `assets.rb` 中指定

```rb
Rails.application.config.assets.precompile += %w(
  search.js
  welcome.css
)
```

里面还有一个配置需要的资源路径

```rb
Rails.application.config.assets.paths << Emoji.images_path
```

`config/enviroments/production.rb` 之中能配置生产环境需要的配置项，例如压缩、合并等

```rb
config.assets.js_compressor = :uglifier
config.assets.css_compressor = :sass

config.assets.digest = true  # 对 css 和 js 文件添加随机字符串
```

手动编译静态文件

```sh
rake assets:precompile
```
