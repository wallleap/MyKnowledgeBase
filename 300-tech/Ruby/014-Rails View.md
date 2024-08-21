---
title: 014-Rails View
date: 2024-08-06T06:16:00+08:00
updated: 2024-08-21T10:32:31+08:00
dg-publish: false
---

## View 的原理

- ActionView：actionview gem
- ActionController：actionpack gem

actionview 和 actionpack 都可以单独应用于任何 Ruby 项目

Model 是 activerecord

View 的查找：

- `app/views/` 下和路由的同名文件
- 其他自定义目录：`append_view_path method`

View 的分类：

- Template：`index.html.erb` 模板
- Partial：`_user.html.erb` 片段/局部模板（以 `_` 开头），可以由其他 View 引入
- Layouts：`application.html.erb` 布局模板

View 的解析：

- ERB
- HAML
- Builder：json builder、xml builder 等

都是完成 Ruby 语法到字符串的解释

### ERB 解析

```erb
<%  %>  # 没有输出，用于循环、遍历、复制等操作
<$=  %> # 输出内容
```

例如

```erb
<% @users.each do |user| %>
  <p><%= user.username %></p>
<% end -%>  # 在这里用 -%> 可以让 end 不输出空行
```

赋值

```erb
<% a = 1 %>
<%= a %> # 1
```

注释

```erb
<%# 注释 %>
```

### View 的命名

index.html.erb

- index：action 对应的名称/partial 名称
- html：输出文件类型（mime type）
- erb：解释引擎

index.xml.builder

```xml
xml.rss("version" => "2.0", "xmlns:dc" => "...")
  xml.channel do
    xml.title(@feed_title)
  end
end
```

## Render

作用：

- 生成 HTTP response
- 渲染和解释子视图（sub-view）

### 在 Controller 中

修改 action 的查找 view 的行为，组建当前 request 的 response

```rb
def index
  # ...
end

def search
  @users = User.where("username like ?", "%#{params[:username]}%")
    .page(params[:page] || 1).per_page(params[:per_page] || 10).order("id DESC")
  
  render action: :index # 可以接着跟相应码等 ,status: 200
end
```

还能接收

```rb
render text: 'ok'  # 文本
render json: @users  # 将对象转成 JSON
render xml: @users
render file: 'app/views/users/index'
render partial: 'app/views/users/search'  # 这里不需要加上它本身的 _

render html: "<h1>hello</h1>".html_safe  # 类似于 v-html，不调用是不会解析成 HTML 的
render :edit  # 下面的省略写法
render action: :edit
render "edit"  # 也可以传全名
render "edit.html.erb"
render action: "edit"
render action: edit.html.erb
render "books/edit"
render "books/edit.html.erb"
render template: "books/edit"
render template: "books/edit.html.erb"
render "/path/to/rails/app/views/books/edit"
render "/.../books/edit.html.erb"
render file: "/path/.../books/edit"
render file: "/path/.../books/edit.html.erb"

render json: @user, content_type: 'application/json', location: user_url(@user), status: 200  # :ok

render layout: 'application'
```

HTTP status code in Rails 状态码

| Response Class        | HTTP Status Code | Symbol                   |
| --------------------- | ---------------- | ------------------------ |
| 1xx Informational     |                  |                          |
| Continue              | 100              | `:continue`              |
| Switching Protocols   | 101              | `:switching_protocols`   |
| Processing            | 102              | `:processing`            |
| 2xx Success           |                  |                          |
| OK                    | 200              | `:ok`                    |
| Created               | 201              | `:created`               |
| Accepted              | 202              | `:accepted`              |
| No Content            | 204              | `:no_content`            |
| Reset Content         | 205              | `:reset_content`         |
| Partial Content       | 206              | `:partial_content`       |
| Multi-Status          | 207              | `:multi_status`          |
| Already Reported      | 208              | `:already_reported`      |
| Im Used               | 226              | `:im_used`               |
| 3xx Redirection       |                  |                          |
| Multiple Choices      | 300              | `:multiple_choices`      |
| Moved Permanently     | 301              | `:moved_permanently`     |
| Found                 | 302              | `:found`                 |
| Not Modified          | 304              | `:not_modified`          |
| 4xx Client Error      |                  |                          |
| Bad Request           | 400              | `:bad_request`           |
| Unauthorized          | 401              | `:unauthorized`          |
| Forbidden             | 403              | `:forbidden`             |
| Not Found             | 404              | `:not_found`             |
| 5xx Server Error      |                  |                          |
| Internal Server Error | 500              | `:internal_server_error` |
| Bad Gateway           | 502              | `:bad_gateway`           |

在一个 action 内部只能执行一次 render 或者 redirect_to

```rb
def search
  @users = User.where("username like ?", "%#{params[:username]}%")
    .page(params[:page] || 1).per_page(params[:per_page] || 10).order("id DESC")

  if @users
    render json: @users
    return  # 进入条件，这里添加了 return 就不会再往下执行了
  end

  render text: 'Not found'  # 如果上面没有 return 那就会报错 Can only render or redirect once per action
end
```

### 在 View 中

渲染子视图

```erb
# app/views/users/index.html.erb
<%= render "shored/menu" %>  # app/views/shared/_menu.html.erb
<h1>Products</h1>
<p>...</p>
```

参数

```erb
<%= render "shared/menu" %>
<%= render partial: "shared/menu" %>

<%= render "shared/menu", locals: { var_1: 'value_1' } %>
```

遍历输出

```erb
<% @users.each do |user| %>
  <%= render partial: "one_user", locals: { user: user } %>
<% end -%>

# 还可以
<%= render "one_user", collection: @users, as: :user %>
```

`_one_user.html.erb` 文件

```erb
<p><%= user.username %></p>  # 没有 as 就可以直接用 one_user.username
```

sub-view 能够访问当前 view 或 action 的所有变量

## Layout

定义了 view 的父视图或者布局视图

```erb
# app/views/layout/application.html.erb
<body>
  ...
  <%= yield %>  # 所有子视图都会插入在这里
  ...
</body>
```

Controller 中使用父模版

```rb
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  layout 'application'  # 默认就是 application 可以省略，可以修改成其他的父模板
  # layout: 'user'
  # layout: false

  def index
    # ...
  end
end
```

layout 运行时逻辑，layout 参数的数据类型必须是 Symbol

```rb
class Admin::UsersController <ApplicationController
  layout :generate_layout
  
  def index
    # ...
    @current_user = :normal
  end
  def new
    # ...
    @current_user = :admin
  end

  private
  def generate_layout
    @current_user == :normal ? 'application' : 'admin'
  end
end
```

layout 继承

```rb
class Admin::BaseController < ApplicationController
  layout 'admin'
end

class Admin::BlogsController < Admin::BaseController
  # 默认继承 'admin' layou
end

class Admin::UserController < Admin::BaseController
  layout 'user'  # 可以修改
end
```

## render layout

```rb
class Admin::UsersController <ApplicationController
  layout 'admin'
  def index
    # ...
  end
  def new
    # ...
    render layout: 'application'
  end
  def edit
    # ...
    render layout: false
  end
end
```

## redirect_to

重定向

```rb
def destroy
  @blog = current_student.blogs.find(params[:id])
  if @blog.destroy
    flash[:notice] = "删除成功"
    redirect_to learn_blogs_path, status: 301
  else
    flash[:notice] = "删除失败"
    redirect_to :back, status: 302  # 之前的页面
  end
end
```

status 参数

- 301 permanent 永久不可访问
- 302 temporary 临时跳转，下次还可以访问

render vs. redirect_to

- redirect_to 只有重定向，会重新发请求

shell 的 curl 命令

```rb
class WelcomeController < ApplicationController
  def index
    redirect_to new_session_path
    # @user = if session[:user_id]
    #   User.find(session[:user_id])
    # else
    #   nil
    # end
  end
end
```

测试

```sh
$ curl -i http://localhost:4001/  # -i 显示头，-L 显示跳转后页面
HTTP/1.1 302 Found
X-Frame-Options: SAMEORIGIN
X-XSS-Protection: 0
X-Content-Type-Options: nosniff
X-Download-Options: noopen
X-Permitted-Cross-Domain-Policies: none
Referrer-Policy: strict-origin-when-cross-origin
Location: http://localhost:4001/sessions/new
Content-Type: text/html; charset=utf-8
Cache-Control: no-cache
X-Request-Id: 0a941013-fe81-4a7c-a5e4-c0dea0c3c90c
X-Runtime: 0.128726
Server-Timing: start_processing.action_controller;dur=0.14, redirect_to.action_controller;dur=0.28, process_action.action_controller;dur=1.16
Transfer-Encoding: chunked

<html><body>You are being <a href="http://localhost:4001/sessions/new">redirected</a>.</body></html>%
```

## 后台管理页面模版

新建 `app/views/layouts/admin.html.erb`

```erb
<!DOCTYPE html>
<html>
  <head>
    <title>Circles 后台管理</title>
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
            <a class="navbar-brand custom-logo-link" href="/admin">
              Circles 后台管理
            </a>
          </div>

          <!-- Collect the nav links, forms, and other content for toggling -->
          <div  class="collapse navbar-collapse" id="navbarNav">
            <ul class="navbar-nav">
              <li class="nav-item">
                <a class="nav-link" href="<%= admin_users_path %>">用户管理</a>
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

在 `app/controllers/admin/users_controller.rb` 使用该模版

```rb
class Admin::UsersController < ApplicationController
  layout "admin"

  def index
    @users = User.page(params[:page] || 1).per_page(params[:per_page] || 10).order("id DESC")
  end

  def search
    @users = User.where("username like ?", "%#{params[:username]}%")
      .page(params[:page] || 1).per_page(params[:per_page] || 10).order("id DESC")

    render action: :index
  end
end
```

## Other Helper Methods

ActionView 提供的

[ActionView Helpers](https://api.rubyonrails.org/classes/ActionView/Helpers.html)

### image

```erb
image_tag("rails.png")  # <img src="/assets/rails.png" alt="Rails"
 />
image_path("rails.png")  # /assets/rails.png
```

### javascript

```erb
javascript_include_tag("application")

javascript_include_tag(:all)
```

### stylesheet

```erb
stylesheet_link_tag "application"
```

### benchmark

```erb
<% benchmark "Process data files" do %>
  <%= expensive_files_operation %>
<% end %>
```

将会添加一些形如 "Process data files (0.34523)" 的到 log 中

指定代码运行效率

### cache

View 层面 Fragment Cache

```erb
<% cache do %>
  <%= render "shared/footer" %>
<% end %>
```

Controller 层面提供的缓存策略 Page Cache（整个页面）

### capture

```erb
<% @greeting = capture do %>  # 也可以不用实例变量
  <p>Date and time is <%= Time.now %></p>
<% end %>

<%= @greeting %>
```

### 时间操作

```erb
distance_of_time_in_words(Time.now, Time.now + 15.seconds)  # less than a minute

distance_of_time_in_words(Time.now, Time.now + 15.seconds, include_seconds: true)  # less than 20 seconds
```

### number helper

```erb
number_to_currency(1234567890,50) # $1,234,567,890.50

number_to_human_size(1234) # 1.2 kb
number_to_human_size(1234567) # 1.2MB

number_to_percentage(100, precision: 0) # 100%

number_to_phone(1235551234) # 123-555-1234

number_with_delimiter(12345678) # 12,345,678

number_with_precision(111.2345) # 111.235
number_with_precision(111.2345, 2) # 111.23 
```

### sanitize helper

```erb
strip_links("<a href="">链接</a>") # 输出纯字符串

strip_tags("<script>console.log(1)</script>") # 去掉所有标签
```

Rails 不信任任何 view 中的字符串，它将直接把它原样输出

```erb
<% a = "<b>Hello</b>" %>
<%= a %> # <b>Hello</b>  # 不会被解释
<%= a.html_safe %> # 这样浏览器就能将它当成标签
```

### form Helper

- `form_tag` 生成一个 HTML form，一般用于 **非模型相关** 的操作，比如搜索、登录功能等
	- `text_field_tag`
	- `password_field_tag`
	- `email_field_tag`
	- `file_field_tag`
	- `submit_tag`
- `form_for` 生成一个 HTML form，用于 **模型相关** 的操作，比如用户注册、商品创建等
	- `text_field`
	- `password_field`
	- `email_field`
	- `file_field`
	- `submit`
	- `fields_for`
	- `options_for_select`

method

- 目前浏览器只支持 GET/POST 的请求方式
- Rails 的 PUT/DELETE/PATCH 请求实际是通过 POST 请求的方式，然后再添加 `_method` 参数来区分的，`_method` 的值为 `put`/`delete`/`patch`

**form_tag**

```rb
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
```

例如用户登录页面的进行修改

```rb
<%= form_tag sessions_path, method: "post" do %>
  <div class="form-group">
    <%= label_tag :username, "用户名" %>
    <%= text_field_tag :username, nil, placeholder: "用户名...", class: "form-control" %>
  </div>
  <div class="form-group">
    <%= label_tag :password, "密码" %>
    <%= password_field_tag :password, nil, placeholder: "密码...", class: "form-control" %>
  </div>
  <%= submit_tag "登录", class: "btn btn-primary" %>
  <%= link_to "注册", new_user_path %>
<% end %>
```

对应 Controller 中登录 create 方法

```rb
class SessionsController < ApplicationController
  def create
    @user = User.find_by(username: params[:username], password: params[:password])
    if @user
      session[:user_id] = @user.id
      flash[:notice] = "登录成功"
      redirect_to root_path
    else
      flash[:notice] = "用户名或密码错误"
      render action: :new, status: 422
    end
  end
end
```

**form_for**

- 和 resource 绑定
- 支持模型关联（一对多、一对一）
- 自动表单数据设置
- 支持表单数据命名空间

```rb
<%= form_for @user, url: users_path, method: :post do |f| %>
  <div class="dialog alert alert-danger" style="display: <%= @user.errors.any? ? 'block' : 'none' %>">
    <ul class="list-unstyled">
      <% @user.errors.messages.values.flatten.each do |error| %>
        <li><i class="fa fa-exclamation-circle"></i> <%= error %></li>
      <% end -%>
    </ul>
  </div>

  <div class="form-group">
    <%= f.label :username, "用户名" %>
    <%= f.text_field :username, class: "form-control", placeholder: "用户名" %>
  </div>

  <div class="form-group">
    <%= f.label :password, "密码" %>
    <%= f.password_field :password, class: "form-control", placeholder: "密码" %>
  </div>

  <%= f.submit "注册", class: "btn btn-primary" %> 
  <%= link_to "登录", new_session_path %>
<% end -%>
```

Controller 中 create 方法

```rb
class UsersController < ApplicationController
  def create
    # 验证表单数据
    @user = User.new(user_params)
    if @user.save
      flash[:notice] = "注册成功，请登录"  # flash 是 Rails 提供的临时数据存储，一次访问后自动销毁
      redirect_to new_session_path  # 跳转（重定向）到登录页面
    else
      render action: :new, status: 422  # 如果验证失败，重新渲染注册页面
    end
  end
  
  def user_params
    params.require(:user).permit(:username, :password)  # 允许哪些参数，没有的参数会自动忽略
  end
end
```

**CSRF Token**

跨站伪造攻击：攻击者在客户端利用图片或链接等能够放链接或触发事件的标签中，放入一些敏感链接或操作（转账等）

Rails 应对措施

- 使用正确的 HTTP 请求方式
- `protect_form_forgery`（application_controller.rb 中） / View 中 Helper `csrf_meta_tags`
- 表单的 `authenticity_token` 参数

链接等 Get 方式不能请求敏感的 API

使用了 csrf_meta_tags 后生成两个 meta 标签

```rb
<meta name="csrf-param" content="authenticity_token">
<meta name="csrf-token" content="n-voCdSQFb7yRzbX-74PFK8cO8QqizR8gyHaHX0eHxblaPpeUkIcji3UNw9AR1OYQ27pawG-eqIHZeMdMSLxzA">
```

form helper 表单页面中多生成了一个隐藏 input

```html
<input type="hidden" name="authenticity_token" value="N9x5qiPfh4liJxQ5njGucMDQuMy-Y4wdJagM5Uo5JZ9bsGbbcnPSTth3e7yPVggK5MF7N3SyVbps48OWUaqLTg" autocomplete="off">
```

## AJAX

Gemfile

```sh
gem 'jquery-rails'
```

application.js

```js
//= require jquery
//= require jquery-ujs
```

在链接或表单中添加 `remote: true`

### link_to

```rb
<%= link_to "删除", blog_path(@blog), remote: true, method: 'delete', class: "remove_blog_btn" %>

<script>
  $('.remove_blog_btn').on('ajax:success', function(event, data) {
    $(this).parent().remove()
  })
</script>
```

link_to confirm

```rb
<%= link_to "删除", blog_path(@blog), remote: true, method: 'delete', class: "remove_blog_btn", data: { confirm: '确定要删除吗？' } %>
```

要是没有生效可以加上 turbo 试下 `data-turbo-method/confirm`

### form

```rb
<%= form_for @user, url: users_path, remote: true, method: 'post', html: { class: "signup_form" } do |f| %>
  # ...
<% end %>

<script>
  $('.signup_form').on('ajax:success', function(event, data) {
    // ...
  })
</script>
```

### 不使用 jquery

还和上面一样，但是不写 script

1. **使用 `remote: true` 选项**： 在处理 AJAX 请求时，Rails 提供了一个便捷的方式，即在表单或链接中使用 `remote: true` 选项。这样会自动将请求转换为 AJAX 请求，而不是传统的同步请求。

```erb

<%= form_with(model: @model, remote: true) do |form| %>

<!-- Form fields -->

<% end %>

```

或者在链接中：

```

<%= link_to 'Delete', model_path(@model), method: :delete, remote: true %>

```

1. **使用 `respond_to` 块**： 在控制器中，通过使用 `respond_to` 块可以根据请求类型（如 HTML、JS 等）来响应不同类型的请求。这使得可以轻松地处理 AJAX 请求和非 AJAX 请求。

	```

	def create

    @model = Model.new(model_params)

    respond_to do |format|

      if @model.save

        format.html { redirect_to @model, notice: 'Model was successfully created.' }

        format.json { render :show, status: :created, location: @model }

        format.js   # 默认会寻找 create.js.erb 文件

      else

        format.html { render :new }

        format.json { render json: @model.errors, status: :unprocessable_entity }

        format.js   # 默认会寻找 create.js.erb 文件

      end

    end

	end

	```

	在这个例子中，如果是通过 AJAX 请求创建模型，Rails 会渲染 `create.js.erb` 模板，以便于在页面上进行动态更新。

2. **使用 UJS 和 Turbolinks**： Rails 7 继续支持 UJS（Unobtrusive JavaScript）和 Turbolinks，它们可以帮助优化 AJAX 请求的体验。Turbolinks 可以加速页面切换，并且不需要重新加载整个页面。

	```

	import { Turbo } from "@hotwired/turbo-rails"

	Turbo.navigator.delegateToElement = true

	Turbo.navigator.render(this.nextHTML, Turbo.navigator.currentURL)

	```

### 文件上传

- AJAX 的本质是采用 JS 发送请求的
- JS 不支持文件操作
- 需要采用 iframe 的形式，设置表单 target 为 iframe

采用已有的 JS 第三方插件，比如 AjaxUpload
