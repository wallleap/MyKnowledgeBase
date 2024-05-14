---
title: 04-后端单元测试 RSpec
date: 2024-05-13 20:51
updated: 2024-05-13 22:16
---

一般开发流程：

1. 前端组、后端组约定
2. 分别开发
3. 前后端联调（发现 bug）

之前测试 API：

- 软件，例如 Postman、APIFox 等
- curl 命令
	- GET: `curl http://127.0.0.1:3000/api/v1/tags?page=1` 需要看响应头就加 `-v` 参数
	- POST: `curl -X POST http://127.0.0.1:3000/api/v1/tags`
		- 添加请求头 `-H 'Content-Type: application.json'`
		- 添加消息体 `-d '{"amount":99}'`
	- 其他请求: `curl -X PATCH ...`

查找 Ruby 相关工具：[Ruby Toolbox](https://www.ruby-toolbox.com)

现在需要的是单元测试，找到测试框架

- respec
	- BDD（行为驱动开发）
	- 用起来会更爽一点
	- 更像 DSL（领域设计语言）
- minitest（rails 自带的）

单元测试

- 目前只测试 Controller，因为 Model 和 View 都很简单
- 不测这些
	- rails 自带的功能，因为官方已经测试过了（即使没测也不测这个功能本身），例如之前的校验
	- 不测第三方功能，例如发送验证码

## 安装 RSpec

[rspec/rspec-rails: RSpec for Rails 6+ (github.com)](https://github.com/rspec/rspec-rails)

把这个 `gem 'rspec-rails', '~> 6.1.0'` 放到 Gemfile 的这个组里

```
group :development, :test do
  # See https://guides.rubyonrails.org/debugging_rails_applications.html#debugging-with-the-debug-gem
  gem 'rspec-rails', '~> 6.1.0'
  gem "debug", platforms: %i[ mri mingw x64_mingw ]
end
```

然后运行 `bundle install` 或 `bundle` 进行安装，`--verbose` 啰嗦模式

初始化

```sh
bin/rails generate rspec:install
```

创建了四个文件 `.rspec`、`spec`、`spec/spec_helper.rb`、`spec/rails_helper.rb`

创建测试文件

```sh
rails generate rspec:model user
```

创建了 `spec/models/user_spec.rb`

```rb
require 'rails_helper'

RSpec.describe User, type: :model do
  it 'has email' do
    user = User.new email: 'tom@qq.com'
    expect(user.email).to eq 'tom@qq.com'
  end
end
```

BDD 需要先描述行为，例如描述 xxx，它怎样怎样，举个例子

注意：rails 里面万物皆对象

- `to be` 是完全相同，地址也要相同
- `to eq` 值相同，一般使用这个

运行测试之前需要创建测试用数据库，命令行运行

```sh
# 创建数据库
RAILS_ENV=test bin/rails db:create
# migrate 创建表
RAILS_ENV=test bin/rails db:migrate
```

配置一下测试用数据库，修改 `config/database.yml` 的 `test` 字段

```yml
test:
  <<: *default
  database: magosteen_test
  username: mangosteen
  password: 123456
  host: db-for-mangosteen
```

运行测试

```sh
bundle exec rspec
```

测试成功的话会显示绿色的点，一个点代表一个测试用例

## 测试请求

使用 RSpect 的 request

```sh
bin/rails generate rspec:request items
```

自动创建一个 `spec/requests/items_spec.rb`

```rb
require 'rails_helper'

RSpec.describe "Items", type: :request do
  describe "index by page" do
    it "work"
      11.times do
        Item.create amount: 100
      end
      # 缩写
      # 11.times { Item.create amount: 100 } 
      expect(Item.count).to eq 11
      get '/api/v1/items'
      expect(response).to have_http_status 200
      json = JSON.page response.body
      expect(json.['resources'].size).to eq 10
      get '/api/v1/items?page=2'
      expect(response).to have_http_status 200
      json = JSON.page response.body
      expect(json.['resources'].size).to eq 1
    end
  end
  describe "create" do
    it "can create an items" do
      # expect(Item.count).to eq(0) # 每个测试用例之间不互相影响
      # post '/api/v1/items', params: {amount: 99}
      # expect(Item.count).to eq(1)
      expect {
        post '/api/v1/items', params: {amount: 99}
      }.to change {Item.count}.by(+1)
      expect(response).to have_http_status 200
      json = JSON.page response.body
      expect(json.['resources']['amount']).to be_an(Numeric)
      expect(json.['resources']['amount']).to eq 99
    end
  end
end
```

运行测试

```sh
bundle exec rspec
```

## 测试发送验证码

```sh
bin/rails generate rspec:request validation_codes
```

修改生成的文件

```rb
require 'rails_helper'

RSpec.describe "ValidationCodes", type: :request do
  describe "验证码" do
    it "可以被发送"
      post 'api/v1/validation_codes', params: {email: 'tom.qq.com'}
      expect(response).to have_http_status 200
    end
  end
end
```

写验证码的 Controller

```rb
def create
  code = SecureRandom.random_number.to_s[2..7]
  validation_code = ValidationCode.new email: params[:email], kind: 'sign_in', code: code
  if validation_code.save
    head 200
  else
    render json: {errors: validation_code.errors}
end
```

运行测试

```sh
bundle exec rspec
```

注：`db/sckema.rb` 中可以看字段

`bin/rails console` 或 `bin/rails c` 可以进入控制台，运行测试代码，Ctrl + D 退出

生成随机串

- rand
- SecureRandom.hex(6)
- has_secure_token 24
- SecureRandom.random_number
