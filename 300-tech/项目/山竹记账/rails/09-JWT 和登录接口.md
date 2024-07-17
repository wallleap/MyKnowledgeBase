---
title: 09-JWT 和登录接口
date: 2024-05-27 15:52
updated: 2024-05-28 10:21
---

[[你会做 Web 上的用户登录功能吗]]

Token：加密**字符串**

认证方式

## Session + Cookie

需要保存到内存或文件中，会占用服务器资源

### Cookie

HTTP

- 服务器给客户端发的一段 Token（Set-Cookie: Token）
- 它会在后续的**每次**同域名请求的请求头（Cookie: Token）中**自动**携带

### Session

一般基于 Cookie 实现的技术

服务器给不同客户端发送标识（例如 uid），客户端访问的时候都会自动携带这个标识，如果使用明文，黑客直接修改 uid 的值，就可以伪造对应的客户端——**用户身份伪造**

使用密文，先进行加密，将客户端对应的标识存储在内存或文件中，一对一对应关系（Session）

前端请求 post/session 自动 Set-Cookie，get /user 就自动携带 Cookie

## JWT

JSON Web Token，读类似 jot

服务器不存对应关系了，将 Token 放在客户端，服务器直接进行解密就行

三部分组成（每部分 base64 后用 `.` 分隔）

- Header 标记加密算法和类型 `{ "alg": "HS256", "typ": "JWT" }`（typ 类型 登录 认证等）
- Payload 重要数据 JSON `{ "uid": 1, "exp": 145553332 }`
- Signature 密文 `加密(私钥, base64(header), base64(payload))`

前端请求 post /jwt，服务器发送一个 jwt（`h.p.s`），前端手动存到一个地方（例如 localStorage），设置之后的每个请求都携带 jwt（不携带会报 403），之后 get /user，服务器端检测到请求头中有 jwt，进行解密，s 解出 p 发现没对应上就是篡改了

疑问：为什么不直接放到 Cookie 里？

Cookie 是 HTTP 的内容，可能不使用 HTTP，这样 jwt 就携带不上（例如 ws、TCP、UDP 等）

但是可以这样做，因为前端通常就和 HTTP 打交道

## 给登录写测试用例

路由之前已经创建好了，就是在 `config/routes.rb` 里的 `resource :session, only: [:create, :destroy]`

命令创建 controller

```sh
bin/rails g controller api/v1/sessions_controller
```

创建了 controller 文件和 rspec 测试文件

`spec/requests/api/v1/sessions_spec.rb`

```rb
require 'rails_helper'

RSpec.describe "Sessions", type: :request do
  describe "会话" do
    it "登录（创建会话）" do
      # 提前到数据库创建好这个邮箱
      User.create email: 'fangyinghang@foxmail.com'
      post '/api/v1/session', params: {email: 'fangyinghang@foxmail.com', code: '123456'}
      expect(response).to have_http_status(200)
      json = JSON.parse response.body
      # p json['jwt'] # 打印看下 jwt
      expect(json['jwt']).to be_a(String)
    end
  end
end
```

测试

```sh
rspec
```

## 实现登录逻辑

[jwt/ruby-jwt: A ruby implementation of the RFC 7519 OAuth JSON Web Token (JWT) standard. (github.com)](https://github.com/jwt/ruby-jwt)

Gemfile 中添加

```
gem jwt
```

安装

```sh
bundle
```

创建密钥

```sh
EDITOR="code --wait" rails credentials:edit
```

现在应该有三个密钥了

```
# 随便到本地生成的随机数 uuid generate
secret_key_base: xxx
email_password: xxx
hmac_secret: xxx
```

`app/controllers/api/v1/sessions_controller.rb`

```rb
require 'jwt' # 引入库
class Api::V1::SessionsController < ApplicationController
  def create 
    if Rails.env.test?
      # 测试环境把 code 写死
      return render status: :unauthorized if params[:code] != '123456'
    else
      # 发过验证码并且未使用过
      canSignin = ValidationCodes.exists? email: params[:email], code: params[:code], used_at: nil
      return render status: :unauthorized unless canSignin
      # if !canSignin render return 的省写 if not 又可以省写为 unless
    end
    user = User.find_by_email params[:email]
    # 找不到用户
    if user.nil?
      render status: :not_found, json: {errors: '用户不存在'}
    # 用户存在，生成 jwt 并渲染
    else
      payload = { user_id: user.id }
      # 加密 第二个参数是密钥 第三个参数是加密算法
      token = JWT.encode payload, Rails.application.credentials.hmac_secret, 'HS256'
      render status: :ok, json: { jwt: token }
    end
  end
end
```

状态码可以用数字也可以用对应的字符串
