---
title: 012-Rails Model 基础
date: 2024-08-05T01:39:00+08:00
updated: 2024-08-21T10:32:31+08:00
dg-publish: false
---

ORM 概念：Object Relational Model

数据映射 Mapping

- Database <-> Interface
- Virtual Data <-> Interface

Java: XML、Ruby style: ActiveRecord

## Rails 对数据库的支持

- 关系型数据库 RDBMS：SQLite（Rails 默认，文本型数据库）、MySQL、PostgreSQL、Oracle、SQLServer 等 → **ActiveRecord**
- 非关系型数据库 NoSQL：MongoDB（文档型数据库）、Redis/HBase 等 → Mongoid/MongoMapper 等

`config/database.yml` 中可以看到配置

## 修改数据库为 MySQL

SQLite 适合于小型的数据存储

自行安装 MySQL 数据库 5.7

```sh
brew install mysql@5.7
brew services start mysql@5.7
mysql -u root -p
```

因为之前创建项目使用的默认的 SQLite 数据库，所以现在需要到 `Gemfile` 中修改

```
# Use sqlite3 as the database for Active Record
# gem "sqlite3", "~> 1.4"
gem "mysql2" # 把上面那行改成这个
```

修改配置 `config/database.yml`

```yml
# MySQL.  Versions 5.0+ are recommended.
#
# Install the MYSQL driver
#   gem install mysql2
#
# Ensure the MySQL gem is defined in your Gemfile
#   gem 'mysql2'
#
# And be sure to use new-style password hashing:
#   http://dev.mysql.com/doc/refman/5.0/en/old-client.html
#
default: &default
  adapter: mysql2
  encoding: utf8mb4
  charset: utf8mb4
  collation: utf8mb4_unicode_ci
  pool: 5
  host: 127.0.0.1
  username: root
  password:

development:
  <<: *default
  database: circles_development

# Warning: The database defined as "test" will be erased and
# re-generated from your development database when you run "rake".
# Do not set this db to the same as development or production.
test:
  <<: *default
  database: circles_test

# As with config/secrets.yml, you never want to store sensitive information,
# like your database password, in your source code. If your source code is
# ever seen by anyone, they now have access to your database.
#
# Instead, provide the password as a unix environment variable when you boot
# the app. Read http://guides.rubyonrails.org/configuring.html#configuring-a-database
# for a full rundown on how to provide these environment variables in a
# production deployment.
#
# On Heroku and other platform providers, you may have a full connection URL
# available as an environment variable. For example:
#
#   DATABASE_URL="mysql2://myuser:mypass@localhost/somedatabase"
#
# You can use this database configuration with:
#
#   production:
#     url: <%= ENV['DATABASE_URL'] %>
#

production:
  <<: *default
  host: 127.0.0.1
  database: circles_production
  username: circles
  password: circles_p_roduction
```

在 MySQL 中，“utf8”编码只支持每个字符最多三个字节，而标准的 UTF-8 编码是每个字符最多四个字节。

在 MySQL5.53 中，mysql 更新了 utf8mb4 这个系统编码，mb4 就是 most bytes 4 的意思，支持了**特殊符号和繁体字**，专门用来兼容四字节的 unicode。utf8mb4 是 utf8 的超集，除了将编码改为 utf8mb4 外不需要做其他转换。

`utf8mb4_unicode_ci` 是基于标准的 Unicode 来排序和比较，能够在各种语言之间精确排序 ；`utf8mb4_general_ci` 没有实现 Unicode 排序规则，在遇到某些特殊语言或字符是，排序结果可能不是所期望的。

正常创建数据库（现在不执行）

```sql
create database DBNAME default character set utf8mb4 collate utf8mb4_unicode_ci;
```

使用 rake 创建

```sh
bin/rake db:create
```

输出如下内容就是创建好了本地开发和测试数据库

```sh
Created database 'circles_development'
Created database 'circles_test'
```

使用 `rake -T` 命令可以查看所有操作数据库的命令

## 建立 User Model

- 用户名/邮箱
- 密码：这里先用明文存储

```sh
# rails help
bin/rails g model user
```

输出

```sh
      invoke  active_record
      create    db/migrate/20240805065826_create_users.rb
      create    app/models/user.rb
      invoke    test_unit
      create      test/models/user_test.rb
      create      test/fixtures/users.yml
```

Rails 针对模型命名统一使用单数（user），数据库内部映射的表为复数（users）

`db/migrate/20240805065826_create_users.rb`

```rb
class CreateUsers < ActiveRecord::Migration[7.0]
  def change
    create_table :users do |t|
      t.string :username
      t.string :password
      t.timestamps
    end
  end
end
```

string 对应数据库中 varchar

运行

```sh
bin/rake db:migrate
```

它会自动执行没有执行的操作

创建表成功

```sh
== 20240805065826 CreateUsers: migrating ======================================
-- create_table(:users)
   -> 0.0173s
== 20240805065826 CreateUsers: migrated (0.0174s) =============================
```

查看生成的 `app/models/user.rb`

```rb
class User < ApplicationRecord

end
```

`User < ApplicationRecord < ActiveRecord::Base` 这样就可以进行增删改查操作

对象到数据库的映射，所以可以直接针对 User 这个对象进行操作

```sh
bin/rails c # 控制台
```

create、destroy 等方法

```sh
3.0.0 :001 > User.create username: 'luwang', password: '123456'
  TRANSACTION (30.1ms)  BEGIN
  User Create (8.0ms)  INSERT INTO `users` (`username`, `password`, `created_at`, `updated_at`) VALUES ('luwang', '123456', '2024-08-05 07:09:22.838038', '2024-08-05 07:09:22.838038')
  TRANSACTION (4.1ms)  COMMIT


3.0.0 :002 > User.all
  User Load (0.9ms)  SELECT `users`.* FROM `users`
 => 
[#<User:0x000000014d4c60a0
  id: 1,
  username: "luwang",
  password: "[FILTERED]",
  created_at: Mon, 05 Aug 2024 07:09:22.838038000 UTC +00:00,
  updated_at: Mon, 05 Aug 2024 07:09:22.838038000 UTC +00:00>]


3.0.0 :003 > user = User.first
  User Load (1.3ms)  SELECT `users`.* FROM `users` ORDER BY `users`.`id` ASC LIMIT 1
 => 
#<User:0x000000013acd9f48
... 
3.0.0 :004 > user.id
 => 1 
3.0.0 :005 > user.username
 => "luwang"
3.0.0 :006 > user.username = 'Tom'
 => "Tom" 
3.0.0 :007 > user.save
  TRANSACTION (0.8ms)  BEGIN
  User Update (33.5ms)  UPDATE `users` SET `users`.`username` = 'Tom', `users`.`updated_at` = '2024-08-05 07:12:02.331354' WHERE `users`.`id` = 1
  TRANSACTION (3.0ms)  COMMIT
 => true


3.0.0 :008 > user.destroy
  TRANSACTION (3.0ms)  BEGIN
  User Destroy (5.8ms)  DELETE FROM `users` WHERE `users`.`id` = 1
  TRANSACTION (1.2ms)  COMMIT
 => 
#<User:0x000000013acd9f48
 id: 1,
 username: "Tom",
 password: "[FILTERED]",
 created_at: Mon, 05 Aug 2024 07:09:22.838038000 UTC +00:00,
 updated_at: Mon, 05 Aug 2024 07:12:02.331354000 UTC +00:00>
```

<kbd>Ctrl</kbd> + <kbd>d</kbd> 退出

## 用户注册登录功能

### erb 语法

里面放 Ruby 变量和语法

```erb
<% if flash[:notice] %>
  <div class="alert alert-info"><%= flash[:notice] %></div>
<% end -%> 输出内容

<%= yield %> 只判断、遍历
```

view 中可用的 Rails 方法都是 helper 方法

### 查看已有路由

`bin/rails routes` 可以查看当前所有可用的路由

```
Prefix Verb  URI Pattern       Controller#Action
root GET 	  /                   welcome#index
users GET	  /users(.:format)    users#index
POST	      /users(.:format)    users#create
new_user GET		/users/new(.:format)  users#new
edit_user GET 	/users/:id/edit(.:format)  users#edit
user GET  /users/:id(.:format)  users#show
PATCH     /users/:id(.:format)  users#update 
PUT       /users/:id(.:format)  users#update
DELETE    /users/:id(.:format)  users#destroy
```

前面有单词的就代表可以直接在 erb 中使用 `xxx_path` 获取路径，例如 `new_user_path` 对应的路径是 `/users/new` 对应的动作为 `users#new`

或者 `xxx_url` 输出的绝对地址

有 `:` 开头的 URL 都是需要传递参数的 `/sessions/:id`

### Session 和 Cookie

HTTP 短连接（keep-alive 设置存活时间，当前网页关闭就断开）、无状态

HTTP 通过 Cookies 保存状态

Response `Set-Cookie: _circles_session=QBmUPt5...`

Request `Cookie: _circles_session=79woRnP8%2FEnzl2LWqbVYS...`

Session 是后端实现、Cookie 是浏览器的实现

Session 的 key 存储基于 Cookie，Session 的数据存储可以基于 Cookie 也可以基于后端存储，比如数据库等

Session 支持存储后端的数据结构，比如对象，Cookie 只支持字符串

Session 没有大小限制，Cookie 有大小限制（尽量不要存数据到 Cookie）

Cookie 操作

- `cookie` 方法
- cookie 有效期、域名等信息

### 注册功能

确保有三个 Gem

```rb
gem "importmap-rails"
gem "turbo-rails"
gem "stimulus-rails"
```

`bundle install` 后安装

```sh
rails importmap:install
rails turbo:install stimulus:install
```

有这些 Gems 了，但好像有些文件没有弄进去（具体影响了哪些文件可以通过 git 查看）

`app/controllers/users_controller.rb`

```rb
class UsersController < ApplicationController
  def new
    @user = User.new # 传递到 View 的数据
  end

  def create
    # 验证表单数据
    @user = User.new(params.require(:user).permit(:username, :password))
    if @user.save
      flash[:notice] = "注册成功，请登录"  # flash 是 Rails 提供的临时数据存储，一次访问后自动销毁
      redirect_to new_session_path  # 跳转（重定向）到登录页面
    else
      render action: :new, status: 422  # 如果验证失败，重新渲染注册页面
    end
  end
end
```

校验 `app/models/user.rb`

```rb
class User < ApplicationRecord

  # 模型验证
  validates :username, presence: { message: "用户名不能为空" }
  validates :username, uniqueness: { message: "用户名已存在" }
  validates :password, presence: { message: "密码不能为空" }
  validates :password, length: { minimum: 6, message: "密码长度不能小于6位" }

end
```

`app/views/layouts/application.html.erb`

```erb
<!DOCTYPE html>
<html>
  <head>
    <title>Circles</title>
    <meta http-equiv="content-type" content="text/html; charset=utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    
    <link rel="stylesheet" href="https://cdn.staticfile.org/font-awesome/4.7.0/css/font-awesome.css">
    <%= stylesheet_link_tag    'application', media: 'all' %>
    <%= javascript_include_tag "application", "data-turbo-track": "reload", defer: true %>
    <%= csrf_meta_tags %>
    <%= javascript_importmap_tags %>
  </head>
  <body>
    <header name="top">
      <nav class="navbar navbar-expand-lg bg-body-tertiary" role="navigation" id="nav">
        <div class="container">
          <!-- Brand and toggle get grouped for better mobile display -->
          <div class="navbar-header nav-color">
            <button id="nav_collapse" type="button" class="navbar-toggle toggle-btn" data-toggle="collapse" data-target="#bs-navbar-collapse-1" aria-controls="bs-navbar-collapse-1">
              <span class="fa fa-bars fa-lg"></span>
            </button>
            <a class="navbar-brand custom-logo-link" href="/">
              Circles
            </a>
          </div>

          <!-- Collect the nav links, forms, and other content for toggling -->
          <div  class="collapse navbar-collapse" id="navbarNav">
            <ul class="navbar-nav">
              <li class="nav-item">
                <a class="nav-link" href="/">Home</a>
              </li>
              <% if session[:user_id] %>
                <li class="nav-item"><%= link_to "#{User.find(session[:user_id]).username}, 退出", session_path(session[:user_id]), data: {
                    turbo_method: :delete,
                    turbo_confirm: "确定退出登录吗?"
                  }, class: "nav-link" %></li>
              <% else %>
                <li class="nav-item"><%= link_to "登录", new_session_path, class: "nav-link" %></li>
              <% end -%>
            </ul>
          </div>
          <!-- /.navbar-collapse -->
        </div>
        <!-- /.container-->
      </nav>
    </header>

    <div class="container body-offset">
      <% if flash[:notice] %>
        <div class="alert alert-info"><%= flash[:notice] %></div>
      <% end -%>
      
      <%= yield %>
    </div>
  </body>
</html>
```

现在由于没有 session 暂时会报错

对应 new  action 的 View `app/views/users/new.html.erb`

```erb
<h1>用户注册</h1>

<div class="row">
  <div class="col-sm-5">
    <%= form_for @user, url: users_path, method: :post do |f| %>
      <div class="form-group">
        <ul class="list-unstyled">
          <% @user.errors.messages.values.flatten.each do |error| %>
            <li><i class="fa fa-exclamation-circle"></i> <%= error %></li>
          <% end -%>
        </ul>
      </div>

      <div class="form-group">
        <%= f.text_field :username, class: "form-control", placeholder: "用户名" %>
      </div>

      <div class="form-group">
        <%= f.password_field :password, class: "form-control", placeholder: "密码" %>
      </div>

      <%= f.submit "注册", class: "btn btn-primary" %> 
      <%= link_to "登录", new_session_path %>
    <% end -%>
  </div>

  <div class="col-sm-7">
  </div>
</div>
```

表单页面

View 中所有可用的方法，Rails 在 View 中的 helper 方法

通过 `form_for` 和 `form` 生成表单

`form_for` 的第一个参数是当前需要建立模型对应的实例变量，Controller 向 View 传递变量一个方法就是在方法中声明实例变量（`@` 开头），局部变量就只能在 Controller 中使用；第二个参数是 Hash（大括号能省略），url 对应 form 标签的 action，method

`users_path post` 对应的 `create` action

`<%= link_to "登录", new_session_path %>` 直接默认 Get 方式前往对应路径

### 登录功能

使用 Session

创建 sessions controller

```sh
bin/rails g controller sessions create destroy new
```

输出

```sh
      create  app/controllers/sessions_controller.rb
       route  get 'sessions/create'
              get 'sessions/destroy'
              get 'sessions/new'
      invoke  erb
      create    app/views/sessions
      create    app/views/sessions/create.html.erb
      create    app/views/sessions/destroy.html.erb
      create    app/views/sessions/new.html.erb
      invoke  test_unit
      create    test/controllers/sessions_controller_test.rb
      invoke  helper
      create    app/helpers/sessions_helper.rb
      invoke    test_unit
```

自动创建了多余的内容，先删除 `app/views/sessions/create.html.erb`、`app/views/sessions/destroy.html.erb`

路由中只保留需要的内容 `config/routes.rb`

```rb
Rails.application.routes.draw do
  # get 'welcome/index'
  # Define your application routes per the DSL in https://guides.rubyonrails.org/routing.html

  root "welcome#index"

  resources :users # 默认会建立七个路由
  resources :sessions, only: [:new, :create, :destroy]
end
```

查看路由信息 `bin/rails routes`

```
Prefix      Verb    URI Pattern     Controller#Action
root        GET     /               welcome#index
users       GET     /users          users#index
            POST    /users          users#create
new_user    GET     /users/new      users#new
edit_user   GET     /users/:id/edit users#edit
user        GET     /users/:id      users#show
            PATCH   /users/:id      users#update
            PUT     /users/:id      users#update
            DELETE  /users/:id      users#destroy
sessions    POST    /sessions       sessions#create
new_session GET     /sessions/new   sessions#new
session     DELETE  /sessions/:id   sessions#destroy
```

`app/controllers/sessions_controller.rb`

```rb
class SessionsController < ApplicationController
  def create
    @user = User.find_by(username: params[:username], password: params[:password])
    if @user
      session[:user_id] = @user.id  # 记录用户登录状态，session 是 Rails 提供的临时存储数据的方式
      flash[:notice] = "登录成功"
      redirect_to root_path
    else
      flash[:notice] = "用户名或密码错误"
      render action: :new, status: 422 # status: :unprocessable_entity      
    end
  end

  def destroy
    session[:user_id] = nil
    flash[:notice] = "退出成功"
    redirect_to root_path
  end

  def new
  end
end
```

- new action 打开登录界面
- create action 页面提交，登录成功
- destroy action 用户退出

最好把用户的 uuid 存到 session 中

目前是默认将 session 存储在 cookie 中

可以直接操作 cookie 例如：`cookies[:user_id] = @user.id`（明文）

`app/views/sessions/new.html.erb`

```rb
<h1>用户登录</h1>

<div class="row">
  <div class="col-sm-4">
    <%= form_tag sessions_path, method: "post" do %>
      <div class="form-group">
        <input type="text" name="username" placeholder="用户名..." class="form-control" id="form_username" title="用户名6-12个字符">
      </div>
      <div class="form-group">
        <input type="password" name="password" placeholder="密码..." class="form-control" id="form_password">
      </div>
      <input type="submit" class="btn btn-primary" value="登录" /> 
      <%= link_to "注册", new_user_path %>
    <% end %>
  </div>
  <div class="col-sm-8">
  </div>
</div>```

### Log out

之前的 application.erb 中已经写了退出，session 中 destroy 方法中只需要将用户 session id 设为空即可

```erb
<%= javascript_include_tag "application", "data-turbo-track": "reload", defer: true %>
<%= javascript_importmap_tags %>

<li class="nav-item"><%= link_to "#{User.find(session[:user_id]).username}, 退出", session_path(session[:user_id]), data: {
                    turbo_method: :delete,
                    turbo_confirm: "确定退出登录吗?"
                  }, class: "nav-link" %></li>
```

### 分页

在 Gemfile 中添加

```
gem 'will_paginate', '~> 4.0'
```

运行 `bundle` 之后重新启动 server

**基本使用**

```ruby
## perform a paginated query:
@posts = Post.paginate(page: params[:page])

# or, use an explicit "per page" limit:
Post.paginate(page: params[:page], per_page: 30)

## render page links in the view:
<%= will_paginate @posts %>
```

And that's it! You're done. You just need to add some CSS styles to [make those pagination links prettier](http://mislav.github.io/will_paginate/).

You can customize the default "per_page" value:

```ruby
# for the Post model
class Post
  self.per_page = 10
end

# set per_page globally
WillPaginate.per_page = 10
```

New in Active Record 3:

```ruby
# paginate in Active Record now returns a Relation
Post.where(published: true).paginate(page: params[:page]).order(id: :desc)

# the new, shorter page() method
Post.page(params[:page]).order(created_at: :desc)
```

`app/controllers/users_controller.rb` 中添加 index action

```rb
class UsersController < ApplicationController
  def index
    @users = User.page(params[:page] || 1).per_page(params[:per_page] || 10).order("id DESC")
  end
  #...
end
```

### 用户列表

新建对应的 View `app/views/users/index.html.erb`

```erb
<h1>用户列表</h1>

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

在 `` 中添加一个链接

```erb
<li class="nav-item">
  <a class="nav-link" href="<%= users_path %>">用户列表</a>
</li>
```

### Controller Filter

使用场景

- 用户登录校验
- 用户请求追踪

Filters

- `before_action` action 之前
- `around_action`
- `after_action`

```rb
class UsersController < ApplicationController
  before_action :authenticate_user!, only: [:index]  # 只希望 index 页面需要登录

  def index
    @users = User.page(params[:page] || 1).per_page(params[:per_page] || 10).order("id DESC")
  end

  def new
    @user = User.new # 传递到 View 的数据
  end

  def create
    # 验证表单数据
    @user = User.new(params.require(:user).permit(:username, :password))
    if @user.save
      flash[:notice] = "注册成功，请登录"  # flash 是 Rails 提供的临时数据存储，一次访问后自动销毁
      redirect_to new_session_path  # 跳转（重定向）到登录页面
    else
      render action: :new, status: 422  # 如果验证失败，重新渲染注册页面
    end
  end

  private # 私有方法，只有 UsersController 才能调用，不希望被其他控制器调用
  def authenticate_user!
    unless session[:user_id]
      flash[:notice] = "请先登录"
      redirect_to new_session_path
    end
  end
end
```
