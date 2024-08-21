---
title: 025-Rails Test
date: 2024-08-09T06:04:00+08:00
updated: 2024-08-21T10:32:31+08:00
dg-publish: false
---

## 软件测试

- 保证软件的质量和可用性，尽可能减少 bug 和性能问题
- 保证软件的可持续集成
- 潜在帮助分析和完善产品需求
- 给团队带来信心

方式

- 黑盒测试（功能实现方面）
- 白盒测试（代码实现方面）

内容划分

- Unit Test **单元测试** → 开发者自己进行
- Integration Test 集成测试
- Smoke Test 冒烟测试 / Regression Test 回归测试

白盒测试常用框架/工具

- 单元测试：RSpec /Minitest / UnitTest
- 集成测试：Cucumber / Selenium / Capybara

测试基础概念

- Test Case 测试用例
- Edge Case 边缘用例
- Coverage Rate 测试覆盖率
- Mock 伪造

## RSPec

<https://rspec.info>

### 安装

```
# Gemfile
group :development, :test do
  gem 'rspec-rails'
end
```

安装

```sh
$ bundle install
$ bin/rails generate rspec:install
```

### Rails 测试环境

- development
- production
- **test**

## 配置

`.rspec` 默认参数

```
--color
--require spec_helper
-f d
```

在 RSpec 测试框架中，`spec/rails_helper.rb` 和 `spec/spec_helper.rb` 是用于配置测试环境和设置测试用例的关键文件。以下是对这两个文件的说明以及它们的常见配置方式。

### `spec/rails_helper.rb`

`spec/rails_helper.rb` 是一个专门为 Rails 应用程序配置 RSpec 测试环境的文件。它通常包含对 Rails 相关配置的设置和导入。该文件通常会自动由 `rails generate rspec:install` 命令生成。

#### 主要内容

1. **加载 Rails 环境**：

	 ```ruby
   ENV['RAILS_ENV'] ||= 'test'
   require_relative '../config/environment'
   ```

	 这行代码确保在运行测试时加载 Rails 应用程序的环境。

2. **加载 RSpec 和其他测试库**：

	 ```ruby
   require 'rspec/rails'
   ```

	 这行代码加载 RSpec 的 Rails 支持库。

3. **配置数据库**：

	 ```ruby
   ActiveRecord::Migration.maintain_test_schema!
   ```

	 确保在测试中维护数据库的结构与迁移一致。

4. **配置 RSpec**：

	 ```ruby
   RSpec.configure do |config|
     # 设置RSpec的配置
     config.fixture_path = "#{::Rails.root}/spec/fixtures"
     config.use_transactional_fixtures = true
     config.infer_spec_type_from_file_location!
     config.filter_rails_from_backtrace!
   end
   ```

	 这些配置选项包括：

	 - `fixture_path`: 设定测试用的 fixture 文件路径。
	 - `use_transactional_fixtures`: 启用事务来管理测试中的数据库状态。
	 - `infer_spec_type_from_file_location!`: 根据文件位置推断测试类型（例如模型、控制器等）。
	 - `filter_rails_from_backtrace!`: 从错误堆栈中移除 Rails 库的路径，以便更清晰地看到问题。

### `spec/spec_helper.rb`

`spec/spec_helper.rb` 是一个用于配置 RSpec 的通用设置文件。这个文件通常用于设置不依赖于 Rails 的配置项，或者提供通用的 RSpec 配置。

#### 主要内容

1. **加载 RSpec**：

	 ```ruby
   require 'rspec'
   ```

	 这行代码加载 RSpec 库。

2. **配置 RSpec**：

	 ```ruby
   RSpec.configure do |config|
     config.disable_monkey_patching!
     config.expect_with :rspec do |c|
       c.syntax = :expect
     end
     config.mock_with :rspec do |c|
       c.verify_partial_doubles = true
     end
     # 其他配置项
   end
   ```

	 常见的 RSpec 配置包括：

	 - `disable_monkey_patching!`: 禁用 RSpec 的猴子补丁，以避免污染全局命名空间。
	 - `expect_with :rspec`: 设置使用 RSpec 的期望语法。
	 - `mock_with :rspec`: 设置 RSpec 的 mock 库，并验证部分 double（mock 对象）。

### 实际使用

在实际的 RSpec 测试中，你通常会在 `rails_helper.rb` 中配置与 Rails 相关的内容，在 `spec_helper.rb` 中配置通用的 RSpec 设置。通常情况下，你会在测试文件中引入 `rails_helper.rb`，因为它已经包含了 `spec_helper.rb` 的内容，并添加了 Rails 相关的配置。

```ruby
# spec/models/user_spec.rb

require 'rails_helper'

RSpec.describe User, type: :model do
  # 测试代码
end
```

通过正确配置这两个文件，可以确保 RSpec 测试环境的稳定性和一致性。

## 目录结构

`spec` 目录，所有的测试代码都放在这里面

新建 `controllers`、`models` 目录，根据对应功能代码新建测试文件

新建 `support` 目录，放通用的方法

## 测试出发点

合法情况、不合法情况、边缘情况、异常情况

例如 edit

- 用户未登录，期待跳转到了登录页面
- 用户登录了且参数合法，期待渲染的是 new 模版页面

## 测试用例

### 测试 edit 和 update

`spec/controllers/blogs_controller_spec.rb`

```rb
require 'rails_helper'

describe BlogsController do # 也能显式告诉它是测试的控制器 BlogsController, type: :controller

  context "not logged in" do
    it "should be redirect to login page" do
      get :edit, id: 1
      expect(response).to redirect_to(new_session_path)
    end
  end

  context "logged in" do
    before(:each) do
      @user = create_user
      signin_user @user

      @blog = @user.blogs.create(title: "test blog", content: "test content")
    end

    it "#edit" do
      get :edit, id: @blog.id
      expect(response).to render_template(:new)
    end

    context '#update' do
      it "should fail" do
        put :update, id: @blog.id, blog: { title: "" }
        expect(assigns(:blog).valid?).to be false
        expect(response).to render_template(:new)
      end

      it "should success" do
        put :update, id: @blog.id, blog: { title: "new title", content: "new content" }
        expect(assigns(:blog).valid?).to be true
        expect(response).to redirect_to(blogs_path)
      end 
    end

    context "#update by correct way" do
      it "should fail" do
        expect(@blog).to receive(:save).and_return(false)
        expect(controller).to receive_message_chain(:current_user, :blogs, :find).and_return(@blog)

        put :update, id: @blog.id, blog: { title: "test" }
        expect(response).to render_template(:new)
      end

      it "should success" do
        expect(@blog).to receive(:save).and_return(true)
        expect(controller).to receive_message_chain(:current_user, :blogs, :find).and_return(@blog)

        put :update, id: @blog.id, blog: { title: "" }
        expect(response).to redirect_to(blogs_path)
      end
    end
  end

end
```

`describe` 可以传测试的类的名字/字符串描述，所有的测试代码就放后面代码块里面

`context` 相当于一个命名空间，描述正在测试的内容

`it` 测试用例，描述具体测试的功能或情况

能直接使用 http 的动作，直接进行请求，请求完 `response` 就能直接使用

`expect` 进行断言

进行测试

```sh
$ rspec spec/controllers/blogs_controller_spec.rb:6
```

上面指定文件和测试用例所在行号

`before` 会在下面测试用例进行前执行，要想在下面的测试用例中获得变量，这里需要使用实例变量

在 `support` 中新建 `helpers.rb` 通用方法

```rb
module Helpers

  def create_user attrs = {}
    User.create({username: "test", password: "111111", password_confirmation: "111111"}.merge(attrs))
  end

  def signin_user user
    session[:user_id] = user.id
  end

end
```

正确的测试用例：

- 测试当前 controller / model 中的功能，不要在测试 controller 的时候测模型
- 每个功能点都需要测试

实际上：只要测试自己注重的功能点

### 测试 user 模型中 save 和 authenticate

`spec/models/user_spec.rb`

```rb
require 'rails_helper'

describe User do

  context "#save" do
    it "should fail" do
      user = User.new(username: "test")
      expect(user.valid?).to be false
    end

    it "should fail with password invalid" do
      user = User.new(username: "test", password: "111111", password_confirmation: "222222")
      expect(user.valid?).to be false
      expect(user.errors.messages).to have_key(:password_confirmation)
    end

    it "should success" do
      user = User.new(username: "test", password: "111111", password_confirmation: "111111")
      expect(user.valid?).to be true
      expect(user.save).to be true
    end
  end

  it ".authenticate" do
    user = User.create(username: "test", password: "111111", password_confirmation: "111111")
    
    expect(User.authenticate('test', '111111')).not_to be nil
    expect(User.authenticate('test', '111111').id).to be user.id
  end

end
```

## TDD

Test Driven Development

一种开发思想，测试驱动开发，先写单元测试用例再写逻辑代码
