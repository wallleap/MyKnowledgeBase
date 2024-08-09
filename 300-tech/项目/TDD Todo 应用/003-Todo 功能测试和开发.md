---
title: 003-Todo 功能测试和开发
date: 2024-08-08 23:06
updated: 2024-08-08 23:06
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




