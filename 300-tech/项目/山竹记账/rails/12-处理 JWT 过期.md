---
title: 12-处理 JWT 过期
date: 2024-05-28T02:05:00+08:00
updated: 2024-08-21T10:33:32+08:00
dg-publish: false
---

JWT 过期时间：2~24 小时，之前有说 payload 中有个 exp 字段，它是秒数

## 添加 JWT 过期时间

`app/models/user.rb`

```rb
class User < ApplicationRecord
  validates :email, presence: true

  def generate_jwt
    payload = { user_id: self.id, exp: (Time.now + 2.hours).to_i } # 2小时 转成整数
    JWT.encode payload, Rails.application.credentials.hmac_secret, 'HS256'
  end

  def generate_auth_header
    {Authorization: "Bearer #{self.generate_jwt}"}
  end
end
```

`lib/auto_jwt.rb`

```rb
class AutoJwt
  def initialize(app)
    @app = app
  end
  def call(env)
    # jwt 跳过以下路径
    return @app.call(env) if ['/api/v1/session','/api/v1/validation_codes'].include? env['PATH_INFO']

    header = env['HTTP_AUTHORIZATION']
    jwt = header.split(' ')[1] rescue ''
    begin # 高级 rescue
      payload = JWT.decode jwt, Rails.application.credentials.hmac_secret, true, { algorithm: 'HS256' } 
    rescue JWT::ExpiredSignature # 签名过期
      return [401, {}, [JSON.generate({reason: 'token expired'})]]
    rescue  # 其他错误
      return [401, {}, [JSON.generate({reason: 'token invalid'})]]
    end
    env['current_user_id'] = payload[0]['user_id'] rescue nil
    @status, @headers, @response = @app.call(env)
    [@status, @headers, @response]
  end
end
```

`spec/requests/api/v1/me_spec.rb`

```rb
require 'rails_helper'
require 'active_support/testing/time_helpers' # 时间相关的库

RSpec.describe "Me", type: :request do
  include ActiveSupport::Testing::TimeHelpers # 这一行
  describe "获取当前用户" do
    it "登录后成功获取" do
      user = User.create email: 'fangyinghang@foxmail.com'
      post '/api/v1/session', params: {email: 'fangyinghang@foxmail.com', code: '123456'}
      expect(response).to have_http_status(200)
      json = JSON.parse response.body
      jwt = json['jwt']
      get '/api/v1/me', headers: {'Authorization': "Bearer #{jwt}"}
      expect(response).to have_http_status(200)
      json = JSON.parse response.body
      expect(json['resource']['id']).to eq user.id
    end
    it "jwt过期" do
      travel_to Time.now - 3.hours # 时间倒回3小时前
      user1 = User.create email: '1@qq.com'
      jwt = user1.generate_jwt

      travel_back
      get '/api/v1/me', headers: {'Authorization': "Bearer #{jwt}"}
      expect(response).to have_http_status(401)
    end
    it "jwt没过期" do
      travel_to Time.now - 1.hours
      user1 = User.create email: '1@qq.com'
      jwt = user1.generate_jwt

      travel_back
      get '/api/v1/me', headers: {'Authorization': "Bearer #{jwt}"}
      expect(response).to have_http_status(200)
    end
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
      user1 = User.create email: '1@qq.com'
      user2 = User.create email: '2@qq.com'
      11.times { Item.create amount: 100, user_id: user1.id }
      11.times { Item.create amount: 100, user_id: user2.id }

      get '/api/v1/items', headers: user1.generate_auth_header
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
    it "按时间筛选（边界条件2）" do
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
    it "按时间筛选（边界条件3）" do
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
    xit "can create an item" do 
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

现在不想测试，可以用 `xit`，将关闭这个测试

`spec/acceptance/sessions_spec.rb`

```rb
require 'rails_helper'
require 'rspec_api_documentation/dsl'

resource "会话" do
  post "/api/v1/session" do
    parameter :email, '邮箱', required: true
    parameter :code, '验证码', required: true
    response_field :jwt, '用于验证用户身份的 token'
    let(:email) { '1@qq.com' }
    let(:code) { '123456' }
    example "登录" do
      do_request
      expect(status).to eq 200
      json = JSON.parse response_body
      expect(json['jwt']).to be_a String
    end
  end
end
```

`spec/requests/api/v1/sessions_spec.rb`

```rb
require 'rails_helper'

RSpec.describe "Sessions", type: :request do
  describe "会话" do
    it "登录（创建会话）" do
      User.create email: 'fangyinghang@foxmail.com'
      post '/api/v1/session', params: {email: 'fangyinghang@foxmail.com', code: '123456'}
      expect(response).to have_http_status(200)
      json = JSON.parse response.body
      expect(json['jwt']).to be_a(String)
    end
    it "首次登录" do 
      post '/api/v1/session', params: {email: 'fangyinghang@foxmail.com', code: '123456'}
      expect(response).to have_http_status(200)
      json = JSON.parse response.body
      expect(json['jwt']).to be_a(String)
    end
  end
end
```

`app/controllers/api/v1/sessions_controller.rb`

```rb
require 'jwt'
class Api::V1::SessionsController < ApplicationController
  def create 
    if Rails.env.test?
      return render status: :unauthorized if params[:code] != '123456'
    else
      canSignin = ValidationCodes.exists? email: params[:email], code: params[:code], used_at: nil
      return render status: :unauthorized unless canSignin
    end
    # 找不到就创建
    user = User.find_or_create_by email: params[:email]
    render status: :ok, json: { jwt: user.generate_jwt }
  end
end
```

## JWT 续期

Refresh Token

过期时间会比 jwt 更长，这样在这段时间内都可以重新申请新的 jwt

内容：

1. 随机数
2. 也可以是另外一个 jwt

随机数得用张表记录，创建时间，用户，随机数

另一个 jwt 就不用，用 type: refresh 标识，无状态
