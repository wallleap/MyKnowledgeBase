---
title: 013-Rails Routes
date: 2024-08-06T04:13:00+08:00
updated: 2024-08-21T10:32:31+08:00
dg-publish: false
---

## Routes

- 规定了特定格式的 URL 请求到后端 Controller 的 Action 的分发规则
- 配置文件：`config/routes.rb`，优先采用上面定义的路由
- 使用指定的方法和参数来生成路由

例如 GET /users/2

```rb
get '/users/:id', to: 'users#show'
# 或
get '/users/:id' => 'users#show'
```

`:id` 是个变量，可以通过 action 的方法获取 `params[:id]`

### 命名路由

Named routes

```rb
get '/users/:id', to: 'users#show', as 'user'
```

使用关键字 `as` 取个名，这样就有 `user` 这个方法了

在 View 中就可以

```rb
user_path(user) #例如 /users/2
user_path(user.id)

user_url # http://127.0.0.1/users/2
```

### RESTful 资源设计

- GET
- POST
- PUT/PATCH
- DELETE

**定义**每个请求 URL 的请求方式，同一个 URL 可以拥有不同的请求方式

```rb
resources :users
```

rails 生成了七个路由，可以通过命令 `rails routes` 查看

| Prefix    | Verb   | URI Pattern     | Controller#Action | 说明                  |
| --------- | ------ | --------------- | ----------------- | ------------------- |
| users     | GET    | /users          | users#index       | 返回创建用户的 HTML 页面列表信息 |
| users     | POST   | /users          | users#create      | 表单提交的目标动作           |
| new_user  | GET    | /users/new      | users#new         | 返回 HTML 页面以创建用户     |
| edit_user | GET    | /users/:id/edit | users#edit        | 返回一个用户的 HTML 页面以编辑  |
| user      | GET    | /users/:id      | users#show        | 展示一个用户信息            |
| user      | PATCH  | /users/:id      | users#update      |                     |
| user      | PUT    | /users/:id      | users#update      | 更新用户                |
| user      | DELETE | /users/:id      | users#destroy     | 删除用户                |

- RESTful 只是约束，让我们的代码设计更统一
- 专注于产品实现，而不是代码设计本身

**定义多个资源**

```rb
resources :users, :sessions
```

**单数资源**，有的时候不需要传递 id，比如用户自身的信息编辑页面

```rb
get 'profile', to 'users#show'
```

Rails 还提供了单数 resource

```rb
resource :users
```

| Verb   | URI Pattern | Controller#Action | Explanation                                             |
| ------ | ----------- | ----------------- | ------------------------------------------------------- |
| GET    | /user/new   | users#new         | Displays a form to create a new user.                   |
| POST   | /user       | users#create      | Create a new user                                       |
| GET    | /user       | users#show        | Shows details of a specific user.                       |
| GET    | /user/edit  | users#edit        | Displays a form to edit the details of a specific user. |
| PATCH  | /user       | users#update      | Update  a specific                                      |
| PUT    | /user       | users#update      |                                                         |
| DELETE | /user       | users#destroy     | Deletes a specific user.                                |

Controller namespace **命名空间**

```rb
namespace :admin do
  resources :users
end
```

| Prefix          | Verb   | URI Pattern           | Controller#Action   | Named Helper              |
| --------------- | ------ | --------------------- | ------------------- | ------------------------- |
| admin_users     | GET    | /admin/users          | admin/users#index   | admin_users_path          |
|                 | POST   | /admin/users          | admin/users#create  | admin_users_path          |
| new_admin_user  | GET    | /admin/users/new      | admin/users#new     | new_admin_user_path       |
| edit_admin_user | GET    | /admin/users/:id/edit | admin/users#edit    | edit_admin_user_path(:id) |
| admin_user      | GET    | /admin/users/:id      | admin/users#show    | admin_user_path(:id)      |
|                 | PATCH  | /admin/users/:id      | admin/users#update  | admin_user_path(:id)      |
|                 | PUT    | /admin/users/:id      | admin/users#update  | admin_user_path(:id)      |
|                 | DELETE | /admin/users/:id      | admin/users#destroy | admin_user_path(:id)      |

**Scope**

不想在 URL 中显示 `/admin` 前缀

```rb
scope module: 'admin' do
  resources :users
end
```

或者

```rb
resources :users, module: 'admin'
```

反之想在 URL 中显示前缀，而不想把代码放在 admin 的命名空间中

```rb
scope '/admin' do
  resources :users
end
```

| Verb   | URI Pattern           | Controller#Action | Named Helper        |
| ------ | --------------------- | ----------------- | ------------------- |
| GET    | /admin/users          | users#index       | users_path          |
| POST   | /admin/users          | users#create      | users_path          |
| GET    | /admin/users/new      | users#new         | new_user_path       |
| GET    | /admin/users/:id/edit | users#edit        | edit_user_path(:id) |
| GET    | /admin/users/:id      | users#show        | user_path(:id)      |
| PATCH  | /admin/users/:id      | users#update      | user_path(:id)      |
| PUT    | /admin/users/:id      | users#update      | user_path(:id)      |
| DELETE | /admin/users/:id      | users#destroy     | user_path(:id)      |

Nested resources **嵌入式路由**

```rb
resources :users do
  resources :blogs
end
```

| Verb   | URI Pattern                    | Controller#Action |
| ------ | ------------------------------ | ----------------- |
| GET    | /users/:user_id/blogs          | blogs#index       |
| POST   | /users/:user_id/blogs          | blogs#create      |
| GET    | /users/:user_id/blogs/new      | blogs#new         |
| GET    | /users/:user_id/blogs/:id/edit | blogs#edit        |
| GET    | /users/:user_id/blogs/:id      | blogs#show        |
| PATCH  | /users/:user_id/blogs/:id      | blogs#update      |
| PUT    | /users/:user_id/blogs/:id      | blogs#update      |
| DELETE | /users/:user_id/blogs/:id      | blogs#destroy     |

两个嵌套已经很长了，所以不要再接着嵌套（例如 /users/:user_id/blogs/:id/comments）

**排除**不需要的 action 和请求方式

```rb
resources :sessions, only: [:new, :create, :destroy]
```

添加**自定义** RESTful 路由

- 需要传 id 的称为成员 member 路由
- 不需要传 id 的称为集合 collection 路由

```rb
resources :users do
  member do
    post :status
  end

  collection do
    get :online
  end
end
```

| Verb | URI Pattern       | Controller#Action | Route Helper |
| ---- | ----------------- | ----------------- | ------------ |
| POST | /users/:id/status | users#status      | status_user  |
| GET  | /users/online     | users#online      | online_users |

还可以

```rb
resources :users do
  post :status, on: :member
  get :online, on: :collection
end
```

**Non-Resourcesful Routes** 非资源路由

```rb
# /photos/show/1
# /photos/show
# /phontos  # => index action
get ':controller(/:action(/:id))'  # :action 和 :id 可选

# /photos/show/1/2
get ':controller/:action/:id/:user_id'

match 'photos', to: 'photos#show', via: [:get, :post]
```

**Segment Constraints** 约束

```rb
# /photos/A12345
get 'photos/:id', to: 'photos#show', constraints: { id: /[A-z]\d{5}/ }
```

Redirection **重定向**

```rb
get '/stories', to: redirect('/articles')
```

Mount

拆分模块

```rb
mount AdminApp, at: '/admin'
```

AdminApp 必须遵循 Rack 的设计模式

**根路由** `root_path`

```rb
# /
root to: 'pages#main'
rout 'pages#main'
```

**controller**

```rb
controller :welcome do
  get '/welcome/hello'
end
```

定义了一个 `WelcomeController` 控制器，并且指定了一个路由，当访问 `/welcome/hello` 路径时，会执行 `WelcomeController` 的 `hello` 动作来响应请求

## Circle 管理后台实现

创建控制器，命名空间

```sh
bin/rails g controller admin::users
```

输出内容

```sh
      create  app/controllers/admin/users_controller.rb
      invoke  erb
      create    app/views/admin/users
      invoke  test_unit
      create    test/controllers/admin/users_controller_test.rb
      invoke  helper
      create    app/helpers/admin/users_helper.rb
      invoke    test_unit
```

添加路由 `config/routes.rb`

```rb
Rails.application.routes.draw do
  # get 'welcome/index'
  # Define your application routes per the DSL in https://guides.rubyonrails.org/routing.html

  root "welcome#index"

  resources :users
  resources :sessions, only: [:new, :create, :destroy]

  namespace :admin do
    resources :users
  end
end
```

运行 `rails routes` 查看

| Verb   | URI Pattern           | Controller#Action   | Named Helper              |
| ------ | --------------------- | ------------------- | ------------------------- |
| GET    | /admin/users          | admin/users#index   | admin_users_path          |
| POST   | /admin/users          | admin/users#create  |                           |
| GET    | /admin/users/new      | admin/users#new     | new_admin_user_path       |
| GET    | /admin/users/:id/edit | admin/users#edit    | edit_admin_user_path(:id) |
| GET    | /admin/users/:id      | admin/users#show    | admin_user_path(:id)      |
| PATCH  | /admin/users/:id      | admin/users#update  |                           |
| PUT    | /admin/users/:id      | admin/users#update  |                           |
| DELETE | /admin/users/:id      | admin/users#destroy |                           |

### 用户列表

`app/controllers/admin/users_controller.rb`

```rb
class Admin::UsersController < ApplicationController
  def index
    @users = User.page(params[:page] || 1).per_page(params[:per_page] || 10).order("id DESC")
  end
end
```

新建 `app/views/admin/users/index.html.erb`

```erb
<h1>管理后台 - 用户列表</h1>

<div class="row">
  <div class="col-md-12">
    <table class="table table-bordered table-striped">
      <thead>
        <tr>
          <th>ID</th>
          <th>用户名</th>
          <th></th>
        </tr>
      </thead>
      <tbody>
        <% @users.each do |user| %>
          <tr>
            <td><%= user.id %></td>
            <td><%= user.username %></td>
            <td></td>
          </tr>
        <% end %>
      </tbody>
    </table>

    <%= will_paginate @users %>
  </div>
</div>
```

访问 `http://127.0.0.1:4001/admin/user` 可以看到后台管理页面

我们想要 `/admin` 就能看到用户列表，可以在 routes.rb 中添加

```rb
namespace :admin do
  root "users#index"

  resources :users
end
```

### 搜索用户

routes.rb 添加

```rb
namespace :admin do
  root "users#index"

  resources :users do
    collection do
      get :search
    end
  end
end
```

controller 中模糊匹配

```rb
def search
  @users = User.where("username like ?", "%#{params[:username]}%")
    .page(params[:page] || 1).per_page(params[:per_page] || 10).order("id DESC")

  render action: :index
end
```

View 中

```erb
<h1>管理后台 - 用户列表</h1>

<div class="row">
  <div class="col-md-12">
    <form class="form-inline" action="<%= search_admin_users_path %>" method="get">
      <div class="input-group mb-3">
        <input type="text" name="username" value="<%= params[:username] %>" class="form-control" placeholder="输入用户名" aria-label="Recipient's username" aria-describedby="button-addon2">
        <button class="btn btn-primary" type="submit" id="button-addon2">搜索</button>
      </div>
    </form>

    <table class="table table-bordered table-striped">
      <thead>
        <tr>
          <th>ID</th>
          <th>用户名</th>
          <th></th>
        </tr>
      </thead>
      <tbody>
        <% @users.each do |user| %>
          <tr>
            <td><%= user.id %></td>
            <td><%= user.username %></td>
            <td></td>
          </tr>
        <% end %>
      </tbody>
    </table>

    <%= will_paginate @users %>
  </div>
</div>
```
