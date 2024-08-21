---
title: 011-Rails 快速上手
date: 2024-08-04T10:58:00+08:00
updated: 2024-08-21T10:32:30+08:00
dg-publish: false
---

目标：

- 创建 Circles 项目
- MVC 设计模式和 Rails 目录结构
- Controller 基础学习
- 引入 Bootstrap

## 创建项目

由于运行下面命令最后会自动下载依赖（国外网站），所以最好先翻墙（否则需要修改安装源）

```sh
rails new circles
```

后面常用的参数

- `-h` 帮助
- `--database=` 指定数据库 mysql/postgresql/sqlite3/oracle/sqlserver/jdbcmysql/jdbcsqlite3/jdbcpostgresql/jdbc（默认 sqlite3）
- `--skip-test` 跳过测试文件
- `--api` API 模式

启动 server

```sh
rails s -p 4001 # server 缩写为 s
```

创建 welcome 的 controller

```sh
rails g controller welcome index # generate 缩写为 g
# 名字 Action
```

输出内容

```sh
      create  app/controllers/welcome_controller.rb
       route  get 'welcome/index'
      invoke  erb
      create    app/views/welcome
      create    app/views/welcome/index.html.erb
      invoke  test_unit
      create    test/controllers/welcome_controller_test.rb
      invoke  helper
      create    app/helpers/welcome_helper.rb
      invoke    test_unit
```

修改文件 `app/views/welcome/index.html.erb`

```erb
<h1>This is Circle's Home Page!</h1>
```

修改首页 `config/routes.rb`

```rb
Rails.application.routes.draw do
  # get 'welcome/index'
  # Define your application routes per the DSL in https://guides.rubyonrails.org/routing.html

  root "welcome#index"
end
```

## MVC 设计模式

Model 数据、View 用户界面、Controller 应用逻辑

目录结构

```
circles/
├── Gemfile  # 所有依赖的 Gems
├── Gemfile.lock
├── README.md
├── Rakefile  # rake 任务
├── app/
    ├── assets/
    │   ├── config/
    │   │   └── manifest.js
    │   ├── images/
    │   └── stylesheets/
    │       └── application.css
    ├── channels
    │   └── application_cable
    │       ├── channel.rb
    │       └── connection.rb
    ├── controllers/
    │   ├── application_controller.rb
    │   ├── concerns
    │   └── welcome_controller.rb
    ├── helpers/
    │   ├── application_helper.rb
    │   └── welcome_helper.rb
    ├── jobs/
    │   └── application_job.rb
    ├── mailers/
    │   └── application_mailer.rb
    ├── models/
    │   ├── application_record.rb
    │   └── concerns
    └── views/
        ├── layouts/
        │   ├── application.html.erb
        │   ├── mailer.html.erb
        │   └── mailer.text.erb
        └── welcome/
            └── index.html.erb
├── bin/
    ├── rails
    ├── rake
    └── setup
├── config/
    ├── application.rb
    ├── boot.rb
    ├── cable.yml
    ├── credentials.yml.enc
    ├── database.yml
    ├── environment.rb
    ├── environments
    │   ├── development.rb
    │   ├── production.rb
    │   └── test.rb
    ├── initializers
    │   ├── assets.rb
    │   ├── content_security_policy.rb
    │   ├── filter_parameter_logging.rb
    │   ├── inflections.rb
    │   └── permissions_policy.rb
    ├── locales
    │   └── en.yml
    ├── master.key
    ├── puma.rb
    ├── routes.rb
    └── storage.yml
├── config.ru
├── db/
    ├── development.sqlite3
    └── seeds.rb
├── lib/
    ├── assets
    └── tasks
├── log/
    └── development.log
├── public/
    ├── 404.html
    ├── 422.html
    ├── 500.html
    ├── apple-touch-icon-precomposed.png
    ├── apple-touch-icon.png
    ├── favicon.ico
    └── robots.txt
├── storage/
├── test/
    ├── application_system_test_case.rb
    ├── channels
    │   └── application_cable
    │       └── connection_test.rb
    ├── controllers
    │   └── welcome_controller_test.rb
    ├── fixtures
    │   └── files
    ├── helpers
    ├── integration
    ├── mailers
    ├── models
    ├── system
    └── test_helper.rb
├── tmp/
└── vendor/
```

## Controller 使用

`app/controllers/welcome_controller.rb`

继承 `WelcomeController < ApplicationController < ActionController::Base`

```rb
class WelcomeController < ApplicationController
  def index # 会先在 app/views/welcome 下查找同名文件 index.html.erb
  end
end
```

手动创建，例如 users

`app/controllers/users_controller.rb`

```rb
class UsersController < ApplicationController
  def new
  end
end
```

`app/views/users/new.html.erb`

```erb
<h1>User Signup</h1>
```

`config/routes.rb`

```rb
Rails.application.routes.draw do
  # get 'welcome/index'
  # Define your application routes per the DSL in https://guides.rubyonrails.org/routing.html

  root "welcome#index"

  resources :users
end
```

重新启动下服务 `bin/rails s -p 4001`

访问 <http://127.0.0.1:4001/users/new>

## 引入 Bootstrap

[Download · Bootstrap v5.3 (getbootstrap.com)](https://getbootstrap.com/docs/5.3/getting-started/download/)

可以下载 CSS 文件放到 `app/assets/stylesheets` 再到 `views/layouts/application.html.erb` 引入或者下载 Gem

编写 Gemfile

```ruby
gem 'sassc-rails' # 看 bootstrap 文档，也可以下载其他处理 scss 的
gem 'bootstrap', '~> 5.3.3'
```

运行

```sh
bundle
```

修改 `app/assets/stylesheets` 下文件后缀为 `scss`

在 `app/assets/stylesheets/application.scss` 文件中添加

```scss
@import "bootstrap";
```

接着在 `app/views/users/new.html.erb` 中加入

```html
<button type="button" class="btn btn-primary">Primary</button>
```

重新启动服务刷新页面查看是否成功
