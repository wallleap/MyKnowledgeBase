---
title: 002-创建项目并设置 RSpec 和 Capybara
date: 2024-08-08T11:05:00+08:00
updated: 2024-08-21T10:32:43+08:00
dg-publish: false
---

## 创建项目

```sh
rails new tdd_todo --skip-test-unit --skip-bundle
code tdd_todo
```

## 安装两个 gem 并配置

打开 Gemfile 修改 source，添加两个 Gem

```
source 'https://gem.ruby-china.com'


group :development, :test do
  # See https://guides.rubyonrails.org/debugging_rails_applications.html#debugging-with-the-debug-gem
  gem "debug", platforms: %i[ mri mingw x64_mingw ]

  gem "rspec-rails"
  gem "capybara"
end
```

运行 `bundle` 安装 gem

运行 `rails g` 查看可使用的命令

运行命令初始化 rspec

```sh
rails g rspec:install
```

## 第一个测试用例

集成测试文件夹 `spec/feature`

新建 `spec/feature/visit_homepage_spec.rb`

```rb
require 'rails_helper'

feature "visit homepage" do
  scenario "should successfully" do
    visit root_path
    expect(page).to have_content 'hello world'
  end
end
```

运行 `rake` 自动进行测试，发现不通过，因为还没有定义 root_path

路由 `config/routes.rb`

```rb
Rails.application.routes.draw do
  root to: 'welcome#index'
end
```

接着运行 `rake`，不通过，没有 WelcomeController

新建 `app/controller/welcome_controller.rb`

```rb
class WelcomeController < ApplicationController
  def index
  end
end
```

运行 `rake`，不通过，找不到对应 View 视图

新建 `app/view/welcome/index.html.erb`

运行 `rake`，不通过，没有内容

```rb
<h1>hello world</h1>
```

运行 `rake` 就通过了

这样就是测试驱动开发，测试报错，然后去处理错误
