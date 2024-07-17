---
title: 06-使用 Rails + QQ 邮箱发邮件
date: 2024-05-25 21:17
updated: 2024-05-25 22:09
---

后端代码：[https://github.com/FrankFang/OEzuTHAqbftv/commits/master](https://github.com/FrankFang/OEzuTHAqbftv/commits/master)

拿到代码后，你可能需要重新创建密钥：[https://docs.qq.com/doc/DU3ByQXZWbFpQRHND](https://docs.qq.com/doc/DU3ByQXZWbFpQRHND) 。这一步是可选的，但如果你遇到 ActiveSupport::MessageEncryptor::InvalidMessage 报错，那么你就必须重新创建密钥了。

使用邮箱发验证码

由于手机验证码需要钱，而且接入第三方会被做短信轰炸，所以使用邮箱

## 尝试使用 Action Mailer

Rails 文档中直接搜 Action Mailer Base

[Action Mailer Basics — Ruby on Rails Guides](https://guides.rubyonrails.org/action_mailer_basics.html)

先根据文档操作，创建 Mailer

```sh
$ bin/rails generate mailer User
create  app/mailers/user_mailer.rb
create  app/mailers/application_mailer.rb
invoke  erb
create    app/views/user_mailer
create    app/views/layouts/mailer.text.erb
create    app/views/layouts/mailer.html.erb
invoke  test_unit
create    test/mailers/user_mailer_test.rb
create    test/mailers/previews/user_mailer_preview.rb
```

修改文件

```
# app/mailers/application_mailer.rb
class ApplicationMailer < ActionMailer::Base
  default from: "from@example.com"
  layout 'mailer'
end
```

同目录下 user_mailer

```rb
class UserMailer < ApplicationMailer
  def welcome_email
    mail(to: '990083804@qq.com', subject: 'Welcome to My Awesome Site')
  end
end
```

Create a file called `welcome_email.html.erb` in `app/views/user_mailer/`. 邮件模板

```erb
    <h1>Welcome to example.com</h1>
    <p>Thanks for joining and have a great day!</p>
```

修改 `config/environments/development.rb` 中一行，让发送邮件错误时报错

```rb
config.action_mailer.raise_delivery_errors = true
```

运行命令进入控制台

```sh
bin/rails console
```

运行命令进行测试

```sh
UserMailer.welcome_email.deliver
```

	报错 `(irb):1:in <main>': Connection refused - connect(2) for "localhost" port 25 (Errno::ECONNREFUSED)`

## 配置邮箱服务器

这里使用的是 smtp 协议，任意一个支持的都可以，例如使用 QQ 邮箱，需要在帐号设置中开启并生成一个授权码

添加密码的命令是什么？

```
EDITOR="code --wait" bin/rails credentials:edit
```

在新文件中写入 `email_password: xxx`，

然后在 Rails 中使用 `Rails.application.credentials.email_password` 获取对应的值。

同时给生产环境的也配置一下

```sh
EDITOR="code --wait" bin/rails credentials:edit --environment production
```

在刚刚修改的 `config/environments/development.rb` 及对应的 `production.rb` 文件那下面添加

```rb
config.action_mailer.delivery_method = :smtp
config.action_mailer.smtp_settings = {
  address:         'smtp.qq.com',
  port:            587,
  domain:          'smtp.qq.com',
  user_name:       'wallleap@foxmail.com',
  password:        Rails.application.credentials.email_password,
  authentication:  'plain',
  enable_starttls: true,
  open_timeout:    10,
  read_timeout:    10
}
```

之后重新测试发送邮件，显示绿色就代表成功，501 是收件邮箱错误，换一个试试

## 修改发送信息

`app/mailers/user_mailer.rb`

```rb
class UserMailer < ApplicationMailer
  def welcome_email
    @code = '123456'
    @appname = '多购记账'
    # mail(to: 'dogojz.qq.com', subject: 'Welcome to My Awesome Site')
    mail(to: 'luwang@uicop.com', subject: 'Welcome to My Awesome Site')
  end
end
```

`app/views/user_mailer/welcome_email.html.erb`

```erb
<h1>邮箱验证码</h1>
<p>您正在登录 <%= @appname %>，验证码为</p>
<strong style="font-size: 2rem; font-weight: bold;"><%= @code %></strong>
<p>验证码的有效期为 10 分钟。为了您的账户安全，请勿向他人泄露验证码信息。</p>
```

重新测试




