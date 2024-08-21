---
title: 003-Todo 功能测试和开发
date: 2024-08-08T11:06:00+08:00
updated: 2024-08-21T10:32:43+08:00
dg-publish: false
---

## 产品功能

由产品提供，生成了原型图，功能点

## 测试用例

`spec/features/todos_spec.rb`

```rb
require 'rails_helper'

feature "todo" do
  scenario "visit todos page should successfully" do
    visit todos_path
    expect(page).to have_css 'h1', text: 'Todos list'
    expect(page).to have_css 'a', text: 'Create a Todo'
  end

  feature "new todo" do
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

修改 `.rspec` 添加一行，让输出更友好

```
-f d
```

## Todo 功能

`config/routes.rb`

```rb
resources :todos
```

关闭自动生成测试文件

```rb
# config/application.rb
config.generators do |generator|
  generator.test_framework false
end
```

创建模型

```sh
$ bin/rails g model todo title:string
```

创建数据库

```sh
$ bin/rails db:create
```

手动新建 `app/controller/todos_controller.rb`

```rb
class TodosController < ApplicationController
  def index
    @todos = Todo.order("id desc")
  end

  def new
    @todo = Todo.new
  end

  def create
    @todo = Todo.new(params.require(:todo).permit(:title))
    if @todo.save
      redirect_to todos_path
    else
      render action: :new
    end
  end
end
```

实现 view `app/view/todos/index.html.erb`

```rb
<h1>Todos List</h1>

<%= link_to "Create a Todo", new_todo_path %>

<ul>
  <% @todos.each do |todo| %>
    <li><%= todo.title %></li>
  <% end %>
</ul>
```

`app/view/todos/new.html.erb`

```rb
<h1>Create a Todo</h1>

<%= form_for @todo, url: todos_path, html: { di: 'todo-form' } doo |f| %>
  <%= f.text_field :title %>
  <%= f.submit "Save" %>
<% end %>
```
