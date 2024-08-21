---
title: 002-全局模版设置及 User 模型开发
date: 2024-08-11T12:53:00+08:00
updated: 2024-08-21T10:32:41+08:00
dg-publish: false
---

## 创建 Welcome 并设置为首页

```sh
$ bin/rails g controller welcome index
```

`app/controllers/welcome_controller.rb`

```rb
class WelcomeController < ApplicationController
  def index
    @greeting = "Hello World!"
  end
end
```

`config/routes.rb`

```rb
Rails.application.routes.draw do
  # Define your application routes per the DSL in https://guides.rubyonrails.org/routing.html

  # Defines the root path route ("/")
  root "welcome#index"
end
```

`app/views/welcome/index.html.erb`

```rb
<h1><%= @greeting %></h1>
```

## Rails 7 中 JavaScript

在 Rails 7 中，JavaScript 的使用可以通过以下几个步骤来实现：

1. **默认 JavaScript 入口**：
	 Rails 7 默认使用 esbuild 或 importmap 来管理 JavaScript。通常，入口文件是 `app/javascript/application.js`。

2. **引入 JavaScript 文件**：
	 确保在 `app/javascript/application.js` 中导入你的 JavaScript 文件。例如：

	 ```javascript
   import { Turbo } from "@hotwired/turbo-rails";
   import "./controllers";
   ```

3. **使用 importmap**：
	 如果你使用 importmap (Rails 7 的默认配置)，可以在 `config/importmap.rb` 中注册外部 JavaScript 库。例如：

	 ```ruby
   pin "application", preload: true
   pin "bootstrap", to: "https://cdn.jsdelivr.net/npm/bootstrap@5.1.0/dist/js/bootstrap.bundle.min.js"
   ```

4. **添加自定义 JavaScript**：
	 你可以在 `app/javascript` 文件夹中创建自定义文件，并在 `application.js` 中导入它们：

	 ```javascript
   import "./my_custom_script";
   ```

5. **使用 Stimulus**：
	 Rails 7 默认支持 Stimulus 作为前端框架。在 `app/javascript/controllers` 文件夹中创建控制器，例如：

	 ```javascript
   // app/javascript/controllers/hello_controller.js
   import { Controller } from "@hotwired/stimulus";

   export default class extends Controller {
     static targets = ["output"];

     greet() {
       this.outputTarget.textContent = "Hello, Stimulus!";
     }
   }
   ```

	 并在视图中使用它：

	 ```html
   <div data-controller="hello">
     <button data-action="click->hello#greet">Greet</button>
     <span data-hello-target="output"></span>
   </div>
   ```

6. **Turbo**：
	 Rails 7 默认使用 Turbo 来处理页面导航和表单提交。Turbo 的相关功能可以通过在 `app/javascript/application.js` 中导入来使用：

	 ```javascript
   import { Turbo } from "@hotwired/turbo-rails";
   ```

确保在应用中按需引入和配置 JavaScript 库，以便于功能的实现和维护。

**对于现在的项目只需要运行两个命令：**

```sh
$ rails importmap:install
Add Importmap include tags in application layout
      insert  app/views/layouts/application.html.erb
Create application.js module as entrypoint
      create  app/javascript/application.js
Use vendor/javascript for downloaded pins
      create  vendor/javascript
      create  vendor/javascript/.keep
Ensure JavaScript files are in the Sprocket manifest
      append  app/assets/config/manifest.js
Configure importmap paths in config/importmap.rb
      create  config/importmap.rb
Copying binstub
      create  bin/importmap
$ rails turbo:install stimulus:install
Import Turbo
      append  app/javascript/application.js
Pin Turbo
      append  config/importmap.rb
Run turbo:install:redis to switch on Redis and use it in development for turbo streams
Create controllers directory
      create  app/javascript/controllers
      create  app/javascript/controllers/index.js
      create  app/javascript/controllers/application.js
      create  app/javascript/controllers/hello_controller.js
Import Stimulus controllers
      append  app/javascript/application.js
Pin Stimulus
Appending: pin "@hotwired/stimulus", to: "stimulus.min.js"
      append  config/importmap.rb
Appending: pin "@hotwired/stimulus-loading", to: "stimulus-loading.js"
      append  config/importmap.rb
Pin all controllers
Appending: pin_all_from "app/javascript/controllers", under: "controllers"
      append  config/importmap.rb
```

## 全局模版修改

`app/views/layouts/application.html.erb`

```rb
<!DOCTYPE html>
<html>
  <head>
    <title>柚子商城</title>
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>

    <%= stylesheet_link_tag "application", "data-turbo-track": "reload" %>
    <%= yield :stylesheets %>
    <%= javascript_importmap_tags %>
  </head>

  <body>
    <%= render 'layouts/menu' %>
    <div class="container">
      <%= yield %>
    </div>

    <%= javascript_include_tag "application", "data-turbo-track": "reload", defer: true %>
    <%= yield :javascripts %>
  </body>
</html>

```

`app/views/layouts/_menu.html.erb`

```rb
<nav class="navbar navbar-expand-lg navbar-light bg-light fixed-top">
  <div class="container-fluid">
    <a class="navbar-brand fw-bold text-primary" href="#">
      柚子商城
    </a>
    <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarNav" aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
      <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse" id="navbarNav">
      <ul class="navbar-nav ms-auto">
        <li class="nav-item">
          <a class="nav-link" href="#">链接 1</a>
        </li>
        <li class="nav-item">
          <a class="nav-link" href="#">链接 2</a>
        </li>
        <li class="nav-item">
          <a class="nav-link" href="#">链接 3</a>
        </li>
        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown" aria-expanded="false">
            Dropdown
          </a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="#">Action</a></li>
            <li><a class="dropdown-item" href="#">Another action</a></li>
            <li><hr class="dropdown-divider"></li>
            <li><a class="dropdown-item" href="#">Something else here</a></li>
          </ul>
        </li>
        <form class="d-flex" role="search">
          <input class="form-control me-2" type="search" placeholder="Search" aria-label="Search">
          <button class="btn btn-outline-primary" type="submit">Search</button>
        </form>
      </ul>
    </div>
  </div>
</nav>
```

修改主题色 `app/assets/stylesheets/application.scss`

```scss
$primary: #ff8308;
@import "bootstrap";
@import "font-awesome";
```

## User 模型

`app/models/user.rb`

```rb
class User < ApplicationRecord
  authenticates_with_sorcery!

  # 校验
  attr_accessor :password, :password_confirmation

  validates_presence_of :email, message: "邮箱不能为空"
  validates :email, uniqueness: true, message: "邮箱已被使用"
  validates :email, format: { with: /\A[^@\s]+@([^@\s]+\.)+[^@\s]+\z/, message: "邮箱格式不正确" }

  validates_presence_of :password, message: "密码不能为空", if: :need_validate_password?
  validates_presence_of :password_confirmation, message: "确认密码不能为空", if: :need_validate_password?
  validates_confirmation_of :password, message: "两次密码不一致", if: :need_validate_password?
  validates_length_of :password, minimum: 6, message: "密码长度不能小于6位", if: :need_validate_password?

  private
  def need_validate_password?
    self.new_record? ||
      (!self.password.nil? || !self.password_confirmation.nil?)
  end
end
```
