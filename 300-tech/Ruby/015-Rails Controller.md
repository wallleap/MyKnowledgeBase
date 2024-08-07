---
title: 015-Rails Controller
date: 2024-08-07 12:09
updated: 2024-08-07 12:31
---

## Controller

- actionpack gem
- ActionController::Base

## 使用

- `app/controller` 目录
- 命名规则
- 支持命名空间，以 module 的方式组织

继承

```rb
class ApplicationController < ActionController::Base
end

class UsersController < ApplicationController
end

class BaseController < ApplicationController
end
class BlogsController < BaseController
end
```

模块化 modulized controller

```rb
# app/controllers/admin/base_controller
class Admin::BaseController < ApplicationController
  layout 'admin'
end

# app/controllers/admin/blogs_controller
class Admin::BlogsController < Admin::BaseController
end
```

## 实例方法

（不能在 Model 中使用），很多方法都提供给 View 中使用

- params
	- 获取 HTTP 请求中 GET 和 POST 的参数
	- 可以使用 Symbol 和字符串的方式访问，比如 `params[:user], params["user"]`
- session
- cookies
- render
- redirect_to
- send_data 返回二进制数据
- send_file
- request
	- `request.fullpath`
	- `request.get?`
	- `request.headers`
	- `request.ip`
	- `request.body`
- response
	- `response.location`
	- `response.response_code`
	- `response.body=`

```rb
def download
  send_file "/files/to/download.pdf"
end

def blob
  send_data image.data, type: image.content_type
end
```

## 类方法

Filters 过滤器

- before_action
- after_action
- around_action

CSRF

- protect_from_forgery

helper_method

```rb
class UsersController < ApplicationController
  helper_method :current_user

  protected
  def current_user
    if user = User.find(session[:user_id])
      user
    else
      nil
    end
  end
end
```


## 异常处理


日志

- `Rails.logger.info "debug info"`
- `Rails.logger.info "Exception: ..."`

Exception 异常

rescue_from

```rb
class ApplicationController < ActionController::Base
  rescue_from ActiveRecord::RecordNotFound, with :record_not_found

  protected
  def record_not_find
    render plain: "404 Not Found", status: 404
  end
end
```

例如比较友好的处理，发送邮件

```rb
rescue_from Exception do |exception|
  if Rails.env.production? or Rails.env.staging? and ![ActionController::RoutingError].include?(exception.class)
    Rails.logger.info "Send Exception Email!"
    SystemMailer.web_exception_notify(Appconfig.app_notify_receiver, {
      request: request, 
      params: params,
      current_user: current_user,
      exception: exception
    }).deliver_now
    redirect_to '/?response_code=505'
  else
    rails exception
  end
end
```

## 添加用户 session 管理

注：**共用方法、通用逻辑**封装代码可以放在 `app/controllers/concerns` 目录下，在其他 Controller 中直接 include

新建 `app/controllers/concerns/user_session.rb`

```rb
module UserSession
  extend ActiveSupport::Concern

  # 当这个模块被包含（include）到控制器中时，其中的代码就会被执行
  included do
    helper_method :logged_in?
    helper_method :current_user
  end

  # def self.included base
  #   base.class_eval do
  #     helper_method :logged_in?
  #     helper_method :current_user
  #   end
  # end

  def signin_user user
    session[:user_id] = user.id
  end

  def logout_user
    session[:user_id] = nil
  end

  def logged_in?
    !!session[:user_id]
  end

  def current_user
    if logged_in?
      @current_user ||= User.find_by(id: session[:user_id])
    else
      nil
    end
  end
end
```

在新版本中不再需要 `Concerns::User_session` 这样的命名空间，直接写 module 就可以了

在 `app/controllers/application_controller.rb` 中引入

```rb
class ApplicationController < ActionController::Base
  protect_from_forgery wddith: :exception

  include UserSession  # 引入自定义的模块，在所有控制器和 View 中都可以使用
end
```

`app/controllers/sessions_controller.rb` 中就可以使用

```rb
class SessionsController < ApplicationController
  def create
    @user = User.find_by(username: params[:username], password: params[:password])
    if @user
      signin_user @user
      flash[:notice] = "登录成功"
      redirect_to root_path
    else
      flash[:notice] = "用户名或密码错误"
      render action: :new, status: 422
    end
  end

  def destroy
    logout_user if logged_in?
    flash[:notice] = "退出成功"
    redirect_to root_path
  end

  def new
  end
end
```

在 View 中也能直接使用

```rb
<% if logged_in? %>
  <li class="nav-item"><%= link_to "#{current_user.username}, 退出", session_path(session[:user_id]), data: {
      turbo_method: :delete,
      turbo_confirm: "确定退出登录吗?"
    }, class: "nav-link" %></li>
<% else %>
  <li class="nav-item"><%= link_to "登录", new_session_path, class: "nav-link" %></li>
<% end -%>
```

