---
title: 004-添加 Todo 的用户功能
date: 2024-08-08T11:06:00+08:00
updated: 2024-08-21T10:32:43+08:00
dg-publish: false
---

## 测试用例

`spec/features/user_login_spec.rb`

```rb
require 'rails_helper'

feature "user login" do
  scenario "should open login page successfully" do
    visit new_session_path
    expect(page).to have_content 'Please login'
  end

  scenario "shohuld login failed" do
    visit new_session_path
    within("form#login-form") do
      fill_in "username", with: "test_username"
      fill_in "password", with: "test_password"
      click_button "Login"
    end

    expect(page).to have_css "div.error"
  end

  scenario "shohuld login successfully" do
    create_user  # 后面需要新建
    visit new_session_path
    within("form#login-form) do
      fill_in "username", with: "test_username"
      fill_in "password", with: "test_password"
      click_button "Login"
    end

    expect(page).to have_current_path(todos_path)
  end
end
```

`spec/support/user_helper.rb`

```rb
module UserHelper
  def create_user opts = {}
    User.create({username: 'test_username', password: 'test_password'}.merge(opts))
  end
end
```

`spec/rails_helper.rb`

```rb
Dir[Rails.root.join('spec/support/**/*.rb')].each { |f| require f }

# ....
config.include UserHelper
```

## 用户登录功能

```sh
$ bin/rails g model user username:string password:string
```

移植数据库

```sh
$ bin/rails db:migrate
```

`config/routes.rb`

```rb
resources :sessions, only: [:new, :create]
```

`app/controller/sessions_controller.rb`

```rb
class SessionsController < ApplicationController
  def new
  end
end
```

`app/views/sessions/new.html.erb`

```rb
<h1>Please login</h1>

<%= form_tag sessions_path, method: :post, id: 'login-form' do %>
  <p><%= text_field_tag :username %></p>
  <p><%= password_field_tag :password %></p>
  <p><%= submit_tag "Login" %></p>
<% end %>
```

`app/controller/sessions_controller.rb`

```rb
class SessionsController < ApplicationController
  def new
  end

  def create
    @user = User.find_by(username: params[:username], password: params[:password])
    if @user
      session[:user_id] = @user.id
      redirect_to todos_path
    else
      flash[:notice] = "invalid username or password"
      render action: :new
    end
  end
end
```

`app/views/sessions/new.html.erb`

```rb
<h1>Please login</h1>

<% unless flash[:notice].blank? %>
  <div class="error"><%= flash[:notice] %></div>
<% end %>

<%= form_tag sessions_path, method: :post, id: 'login-form' do %>
  <p><%= text_field_tag :username %></p>
  <p><%= password_field_tag :password %></p>
  <p><%= submit_tag "Login" %></p>
<% end %>
```

只想跑一个测试

```sh
$ rspec spec/features/user_login_spec.rb
```
