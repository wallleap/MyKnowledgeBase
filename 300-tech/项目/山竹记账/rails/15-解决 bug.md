---
title: 15-解决 bug
date: 2024-05-29T08:35:00+08:00
updated: 2024-08-21T10:33:32+08:00
dg-publish: false
---

## 发送验证码没有检验邮箱

`spec/requests/api/v1/validation_codes_spec.rb`

```rb
require 'rails_helper'
RSpec.describe "ValidationCodes", type: :request do
  describe "验证码" do
    it "发送太频繁就会返回 429" do
      post '/api/v1/validation_codes', params: {email: 'fangyinghang@foxmail.com'}
      expect(response).to have_http_status(200)
      post '/api/v1/validation_codes', params: {email: 'fangyinghang@foxmail.com'}
      expect(response).to have_http_status(429)
    end
    it "邮件不合法就返回 422" do
      post '/api/v1/validation_codes', params: {email: '1'}
      expect(response).to have_http_status(422)
      json = JSON.parse response.body
      expect(json['errors']['email'][0]).to eq('is invalid')
    end
  end
end
```

`app/models/validation_code.rb`

```rb
class ValidationCode < ApplicationRecord
  validates :email, presence: true
  # email 必须是合法的邮箱地址
  validates :email, format: {with: /\A.+@.+\..+\z/}

  before_create :generate_code
  after_create :send_email
  enum kind: { sign_in: 0, reset_password: 1 }
  def generate_code
    self.code = SecureRandom.random_number.to_s[2..7]
  end
  def send_email
    UserMailer.welcome_email(self.email).deliver
  end
end
```

`app/controllers/api/v1/validation_codes_controller.rb`

```rb
class Api::V1::ValidationCodesController < ApplicationController
  def create
    if ValidationCode.exists?(email: params[:email], kind:'sign_in',created_at: 3.seconds.ago..Time.now)
      render status: :too_many_requests
      return
    end
    validation_code = ValidationCode.new email: params[:email], kind: 'sign_in'
    if validation_code.save
      render status: 200
    else
      render json: {errors: validation_code.errors}, status: 422
    end
  end
end
```

## 报错信息国际化 i18n

使用 i18n(internationallization)

[Rails Internationalization (I18n) API — Ruby on Rails Guides](https://guides.rubyonrails.org/i18n.html)

`config/application.rb`

```rb
Bundler.require(*Rails.groups)
module Mangosteen1
  class Application < Rails::Application
    config.load_defaults 7.0
    config.api_only = true
    config.middleware.use AutoJwt
    config.i18n.default_locale = 'zh-CN' # 默认用中文
  end
end
```

`config/locales/zh-CN.yml` 和上面的一模一样

```yml
zh-CN:
  activerecord:
    errors:
      messages:
        invalid: 格式不正确
      models:
        validation_code:
          attributes:
            email:
              invalid: 邮件地址格式不正确
```

`spec/requests/api/v1/validation_codes_spec.rb`

```rb
require 'rails_helper'
RSpec.describe "ValidationCodes", type: :request do
  describe "验证码" do
    it "发送太频繁就会返回 429" do
      post '/api/v1/validation_codes', params: {email: 'fangyinghang@foxmail.com'}
      expect(response).to have_http_status(200)
      post '/api/v1/validation_codes', params: {email: 'fangyinghang@foxmail.com'}
      expect(response).to have_http_status(429)
    end
    it "邮件不合法就返回 422" do
      post '/api/v1/validation_codes', params: {email: '1'}
      expect(response).to have_http_status(422)
      json = JSON.parse response.body
      expect(json['errors']['email'][0]).to eq('邮件地址格式不正确')
    end
  end
end
```
