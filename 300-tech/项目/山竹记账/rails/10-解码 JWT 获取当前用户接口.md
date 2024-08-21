---
title: 10-解码 JWT 获取当前用户接口
date: 2024-05-27T05:23:00+08:00
updated: 2024-08-21T10:33:32+08:00
dg-publish: false
---

## 写测试用例

路由之前已经创建好了，就是在 `config/routes.rb` 里的 `resource :me, only: [:show]`

命令创建 controller

```sh
bin/rails g controller api/v1/mes_controller
```

创建了 controller 文件和 rspec 测试文件

`spec/requests/api/v1/mes_spec.rb`

```rb
require 'rails_helper'

RSpec.describe "Me", type: :request do
  describe "获取当前用户" do
    it "登录后成功获取" do
      # 先创建并登录
      user = User.create email: 'fangyinghang@foxmail.com'
      post '/api/v1/session', params: {email: 'fangyinghang@foxmail.com', code: '123456'}
      json = JSON.parse response.body
      jwt = json['jwt']

      # 再获取
      get '/api/v1/me', headers: {'Authorization': "Bearer #{jwt}"} # get 时设置 header
      expect(response).to have_http_status(200)
      json = JSON.parse response.body
      expect(json['resource']['id']).to eq user.id
    end
  end
end
```

约定 Token 前面加 `Bearer `

测试

```sh
rspec
```

## 实现

`app/controllers/api/v1/mes_controller.rb`

```rb
class Api::V1::MesController < ApplicationController
  def show
    # 获得 jwt ←  Bearer jwt
    header = request.headers["Authorization"]
    jwt = header.split(' ')[1] rescue ''
    # 解密
    payload = JWT.decode jwt, Rails.application.credentials.hmac_secret, true, { algorithm: 'HS256' } rescue nil
    return head 400 if payload.nil?
    user_id = payload[0]['user_id'] rescue nil
    user = User.find user_id
    return head 404 if user.nil?
    render json: { resource: user }
  end
end
```

> rails 中 `rescue ''` 如果出错了就返回 `''`，类似 try-catch 的 catch 中处理
