---
title: 006-Todo 状态管理功能
date: 2024-08-08T11:07:00+08:00
updated: 2024-08-21T10:32:43+08:00
dg-publish: false
---

## 测试用例

`spec/features/complete_todo_spec.rb`

```rb
require 'rails_helper'

feature "complete todo" do
  background do
    @user = create_user
    login_user @user
    @todo_1 = @user.todos.create(title: 'my first todo')
  end

  scenario "todo should be in incomoplete" do
    visit todos_path
    expect(page).to have_css "li#todo-#{@todo_1.id} span.incomplete"
    expect(find("li#todo-#{@todo_1.id}")).to have_link "mark as complete"
  end

  scenario "todo should be in comoplete" do
    visit todos_path
    within("li#todo-#{@todo_1.id}") do
      click_link "mark as complete"
    end
    expect(page).to have_css "li#todo-#{@todo_1.id} span.complete"
    expect(find("li#todo-#{@todo_1.id}")).to have_link "mark as incomplete"
  end
end
```

factorygirl gem 批量创建测试模型

## 功能开发

```sh
$ bin/rails g migration add_state_to_todos
```

移植文件

```rb
add_column :todos, :state, :interger, default: Todo.states["incomplete"]
```

`app/models/todo.rb`

```rb
enum state: %i[incomplete complete]
```

运行 `rails db:migrate`

`app/views/todos/index.html.erb`

```rb
<li id="todo-<%= todo.id %>">
  <span class="<%= todo.complete? ? 'complete' : 'incomplete' %>">
    当前状态：<%= todo.complete? '已完成' : '未完成' %>
  </span>
  <%= todo.title %>
  <%= link_to "#{todo.complete? ? 'mark as incomplete' : 'mark as complete'}", todo_path(todo), method: :put %>
</li>
```

`app/controllers/todos_controller.rb`

```rb
def update
  @todo = current_user.todos.find(params[:id])
  @todo.incomplete? ? @todo.complete! : @todo.incomplete!
  redirect_back(fallback_location: todos_path)
end
```
