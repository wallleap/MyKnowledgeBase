---
title: 11-把 JWT 做成中间件及实现 items 接口
date: 2024-05-28 10:23
updated: 2024-05-28 14:05
---

JWT 那几行代码，只要那个接口需要获取 user_id 就都要写

```rb
header = request.headers["Authorization"]
jwt = header.split(' ')[1] rescue ''
payload = JWT.decode jwt, Rails.application.credentials.hmac_secret, true, { algorithm: 'HS256' } rescue nil
return head 400 if payload.nil?
user_id = payload[0]['user_id'] rescue nil
```

是否能在进入 Controller 之前就获取好

中间件：

- express 函数
- koa 异步函数
- Spring 类

![](https://cdn.wallleap.cn/img/pic/illustration/202405281039729.png)

## Rails 中间件

[Rails on Rack — Ruby on Rails Guides](https://guides.rubyonrails.org/rails_on_rack.html#action-dispatcher-middleware-stack)

```sh
bin/rails middleware
```

获取到现在所有的中间件

```
use ActionDispatch::HostAuthorization
use Rack::Sendfile
use ActionDispatch::Static
use ActionDispatch::Executor
use ActionDispatch::ServerTiming
use ActiveSupport::Cache::Strategy::LocalCache::Middleware
use Rack::Runtime
use Rack::MethodOverride
use ActionDispatch::RequestId
use ActionDispatch::RemoteIp
use Sprockets::Rails::QuietAssets
use Rails::Rack::Logger
use ActionDispatch::ShowExceptions
use WebConsole::Middleware
use ActionDispatch::DebugExceptions
use ActionDispatch::ActionableExceptions
use ActionDispatch::Reloader
use ActionDispatch::Callbacks
use ActiveRecord::Migration::CheckPending
use ActionDispatch::Cookies
use ActionDispatch::Session::CookieStore
use ActionDispatch::Flash
use ActionDispatch::ContentSecurityPolicy::Middleware
use Rack::Head
use Rack::ConditionalGet
use Rack::ETag
use Rack::TempfileReaper
run MyApp::Application.routes
```

自己添加中间件 `config/application.rb`

```rb
require_relative "boot"
require "rails"
# Pick the frameworks you want:
require "active_model/railtie"
require "active_job/railtie"
require "active_record/railtie"
require "active_storage/engine"
require "action_controller/railtie"
require "action_mailer/railtie"
require "action_mailbox/engine"
require "action_text/engine"
require "action_view/railtie"
require "action_cable/engine"
# require "rails/test_unit/railtie"
require_relative '../lib/auto_jwt' # 手动引用

Bundler.require(*Rails.groups)
module Mangosteen1
  class Application < Rails::Application
  
    config.load_defaults 7.0
    
    config.api_only = true
    config.middleware.use AutoJwt # 使用中间件
  end
end
```

`lib/auto_jwt.rb`

```rb
class AutoJwt
  def initialize(app)
    @app = app
  end
  def call(env) # 中间件被调用时执行的代码
    header = env['HTTP_AUTHORIZATION']
    jwt = header.split(' ')[1] rescue ''
    payload = JWT.decode jwt, Rails.application.credentials.hmac_secret, true, { algorithm: 'HS256' } rescue nil
    env['current_user_id'] = payload[0]['user_id'] rescue nil
    @status, @headers, @response = @app.call(env)
    # 必须返回的三个内容 状态码 响应头 响应体
    [@status, @headers, @response]
  end
end
```

`app/controllers/api/v1/mes_controller.rb`

```rb
class Api::V1::MesController < ApplicationController
  def show
    user_id = request.env['current_user_id'] rescue nil # 获取到 user_id
    user = User.find user_id
    return head 404 if user.nil?
    render json: { resource: user }
  end
end
```

> env 就是个 hash，保存了请求的时候的内容，可读写

测试指定用例

```sh
rspec -e "登录后成功获取"
```

## 实现 get /api/v1/items

`config/routes.rb` 路由 `resources :items` 所有动作

```sh
bin/rails g controller items
```

之前使用这个测试了分页，所以实际上不需要创建 Controller

为了方便测试 `app/models/user.rb` 定义两个方法，到时候直接调用方法就可以携带请求头

```rb
class User < ApplicationRecord
  validates :email, presence: true

  # 生成 jwt 的方法
  def generate_jwt
    payload = { user_id: self.id } # self 当前
    JWT.encode payload, Rails.application.credentials.hmac_secret, 'HS256'
  end

  # 生成请求头的方法
  def generate_auth_header
    {Authorization: "Bearer #{self.generate_jwt}"}
  end
end
```

`spec/requests/api/v1/items_spec.rb`

```rb
require 'rails_helper'

RSpec.describe "Items", type: :request do
  describe "获取账目" do
    it "分页，未登录" do
      user1 = User.create email: '1@qq.com'
      user2 = User.create email: '2@qq.com'
      11.times { Item.create amount: 100, user_id: user1.id }
      11.times { Item.create amount: 100, user_id: user2.id }
      get '/api/v1/items'
      expect(response).to have_http_status 401
    end
    it "分页" do
      # 用户鉴定
      user1 = User.create email: '1@qq.com'
      user2 = User.create email: '2@qq.com'
      11.times { Item.create amount: 100, user_id: user1.id }
      11.times { Item.create amount: 100, user_id: user2.id }

      get '/api/v1/items', headers: user1.generate_auth_header # 有这个头就认为是登录了，不需要每次都 post /session 获取到 jwt
      expect(response).to have_http_status 200
      json = JSON.parse(response.body)
      expect(json['resources'].size).to eq 10
      get '/api/v1/items?page=2', headers: user1.generate_auth_header
      expect(response).to have_http_status 200
      json = JSON.parse(response.body)
      expect(json['resources'].size).to eq 1
    end
    it "按时间筛选" do
      user1 = User.create email: '1@qq.com'
      item1 = Item.create amount: 100, created_at: '2018-01-02', user_id: user1.id
      item2 = Item.create amount: 100, created_at: '2018-01-02', user_id: user1.id
      item3 = Item.create amount: 100, created_at: '2019-01-01', user_id: user1.id

      get '/api/v1/items?created_after=2018-01-01&created_before=2018-01-03', 
        headers: user1.generate_auth_header
      expect(response).to have_http_status 200
      json = JSON.parse(response.body)
      expect(json['resources'].size).to eq 2
      expect(json['resources'][0]['id']).to eq item1.id
      expect(json['resources'][1]['id']).to eq item2.id
    end
    it "按时间筛选（边界条件）" do
      user1 = User.create email: '1@qq.com'
      item1 = Item.create amount: 100, created_at: '2018-01-01', user_id: user1.id

      get '/api/v1/items?created_after=2018-01-01&created_before=2018-01-02',
        headers: user1.generate_auth_header
      expect(response).to have_http_status 200
      json = JSON.parse(response.body)
      expect(json['resources'].size).to eq 1
      expect(json['resources'][0]['id']).to eq item1.id
    end
    it "按时间筛选（边界条件 没有结束时间）" do
      user1 = User.create email: '1@qq.com'
      item1 = Item.create amount: 100, created_at: '2018-01-01', user_id: user1.id
      item2 = Item.create amount: 100, created_at: '2017-01-01', user_id: user1.id
      get '/api/v1/items?created_after=2018-01-01', 
        headers: user1.generate_auth_header
      expect(response).to have_http_status 200
      json = JSON.parse(response.body)
      expect(json['resources'].size).to eq 1
      expect(json['resources'][0]['id']).to eq item1.id
    end
    it "按时间筛选（边界条件 没有开始时间）" do
      user1 = User.create email: '1@qq.com'
      item1 = Item.create amount: 100, created_at: '2018-01-01', user_id: user1.id
      item2 = Item.create amount: 100, created_at: '2019-01-01', user_id: user1.id

      get '/api/v1/items?created_before=2018-01-02', 
        headers: user1.generate_auth_header
      expect(response).to have_http_status 200
      json = JSON.parse(response.body)
      expect(json['resources'].size).to eq 1
      expect(json['resources'][0]['id']).to eq item1.id
    end
  end
  describe "create" do
    it "can create an item" do 
      expect {
        post '/api/v1/items', params: {amount: 99}
      }.to change {Item.count}.by 1
      expect(response).to have_http_status 200
      json = JSON.parse response.body
      expect(json['resource']['id']).to be_an(Numeric)
      expect(json['resource']['amount']).to eq 99
    end
  end
end
```

执行特定文件的指定行

```sh
rspec spec/requests/api/v1/items_spec.rb:5 # 第5行
```

时区造成的测试失败，手动指定时区 `created_at: Time.new(2018, 1, 1, 0, 0, 0, "+00:00")` 第七个参数也可以 `"Z"`，要么不确定时区（直接用字符串）

`spec/acceptance/items_spec.rb`

```rb
require 'rails_helper'
require 'rspec_api_documentation/dsl'

resource "账目" do
  get "/api/v1/items" do
    authentication :basic, :auth # 基础验证 值在下面
    parameter :page, '页码'
    parameter :created_after, '创建时间起点（筛选条件）' 
    parameter :created_before, '创建时间终点（筛选条件）' 
    with_options :scope => :resources do
      response_field :id, 'ID'
      response_field :amount, "金额（单位：分）"
    end
    let(:created_after) { '2020-10-10'}
    let(:created_before) { '2020-11-11'}
    let(:current_user) { User.create email: '1@qq.com' }
    let(:auth) { "Bearer #{current_user.generate_jwt}" }
    example "获取账目" do
      11.times do Item.create amount: 100, created_at: '2020-10-30', user_id: current_user.id end
      do_request
      expect(status).to eq 200
      json = JSON.parse response_body
      expect(json['resources'].size).to eq 10
    end
  end
end
```

更新 API 文档

```sh
bin/rake docs:generate
npx http-server doc/api
```

`app/controllers/api/v1/items_controller.rb`

```rb
class Api::V1::ItemsController < ApplicationController
  def index
    # 登录用户才能获取
    current_user_id = request.env['current_user_id']
    return head :unauthorized if current_user_id.nil?
    items = Item.where({user_id: current_user_id})
      .where({created_at: params[:created_after]..params[:created_before]})
      .page(params[:page])
    render json: { resources: items, pager: {
      page: params[:page] || 1, # 分页修复
      per_page: Item.default_per_page,
      count: Item.count
    }}
  end
  def create
    item = Item.new amount: params[:amount]
    if item.save
      render json: {resource: item}
    else
      render json: {errors: item.errors}
    end
  end
end
```
