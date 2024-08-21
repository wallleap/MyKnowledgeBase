---
title: 005-重构 Todo 代码
date: 2024-08-08T11:07:00+08:00
updated: 2024-08-21T10:32:43+08:00
dg-publish: false
---

## 重构 Todo 测试用例

`spec/features/todos_spec.rb`

```rb
require 'rails_helper'

feature "todo" do
  feature "not logged in" do
    scenario "visit todos page should failed" do
      visit todos_path
      expect(page).to have_current_path(new_session_path)
    end
  end

  feature "logged in" do
    background do
      @user = create_user
      login_user @user
    end

    scenario "visit todos page should successfully" do
      visit todos_path
      expect(page).to have_css 'h1', text: 'Todos list'
      expect(page).to have_css 'a', text: 'Create a Todo'
    end

    scenario "should open new page successfully" do
      visit new_todo_path
      expect(page).to have_content "Create a Todo"
    end

    scenario "should crete todo successfully" do
      visit new_todo_path
      within('#todo-form') do
        fill_in 'todo[title]', with: 'My first todo'
        click_button 'Save'
      end
    end

    expect(page).to have_current_path(todos_path)
    expect(page).to have_css 'ul li', text: 'My first todo'
  end
end
```

使用第三方 gem

```
# Gemfile
group :development, :test do
  gem 'rack_session_access'
end
```

`config/environments/test.rb` 注入

```rb
config.middleware.use RackSessionAccess::Middleware
```

`spec/rails_helper.rb`

```rb
require 'rakc_session_access/capybara'
```

`spec/support/user_helper.rb`

```rb
def login_user user
  page.set_rack_session(user_id: user.id)
end
```

## 业务逻辑重构

`app/controllers/application_controller.rb`

```rb
private
def auth_user
  unless logged_in?
    redirect_to new_session_path
  end
end

def logged_in?
  !session[:user_id].blank?
end

def current_user
  if logged_in?
    @current_user ||= User.find(session[:user_id])
  else
    nil
  end
end
```

`app/controllers/todos_controller.rb`

```rb
before_action :auth_user
```

创建移植文件

```sh
$ bin/rails g migration add_user_id_to_todos
```

打开移植文件

```rb
add_column :todos, :user_id, :integer
```

运行 `rails db:migrate`

`app/models/todo.rb`

```rb
belongs_to :user
```

`app/models/user.rb`

```rb
has_many :todos
```

`app/controllers/todos_controller.rb`

```rb
def index
  @todos = current_user.todos.order("id desc")
end

def create
  @todo = current_user.todos.new(params.require(:todo).permit(:title))
  if @todo.save
    redirect_to todos_path
  else
    render action: :new
  end
end
```
