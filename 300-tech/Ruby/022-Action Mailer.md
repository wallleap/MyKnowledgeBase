---
title: 022-Action Mailer
date: 2024-08-09 17:47
updated: 2024-08-09 17:48
---

- 用来发送 Email
- SMTP 协议
- 像使用 controller/view 一样来发送邮件

## 配置

### SMTP 信息

- SMTP 原理
- 在项目中设置 SMTP 的账户信息：`config/application.rb`

```rb
config.action_mailer.delivery_method = :smtp
config.action_mailer.smtp_settings = {
  address:         'smtp.gmail.com',
  port:            587,
  domain:          'example.com',
  user_name:       '<username>',
  password:        '<password>',
  authentication:  'plain',
  enable_starttls: true,
  open_timeout:    5,
  read_timeout:    5
}
```

### controller

`app/mailers` 下 `application_mailer.rb` 如果还需要更多例如邮箱验证、验证码、重置密码等都可以新建文件

```rb
class ApplicationMailer < ActionMailer::Base
  default from: "from@example.com"
  layout "mailer"  # 对应的 layouts 下文件
end
```

Rails 邮件附件

```rb
def send_log log_file
  attachments['access.log'] = File.read(log_file)
  mail(to: 'qa@abc.com', subject: 'Error logs')
end
```

### view

`app/views/layouts` 下创建对应的 `mailer.html.erb`

```html
<!DOCTYPE html>
<html>
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <style>
      /* Email styles need to be inline */
    </style>
  </head>

  <body>
    <%= yield %>
  </body>
</html>

```

生成 mailer 命令

```sh
rails g mailer user_mailer
```

view 注意事项

```rb
# config/application.rb 或 config/environments/*
config.action_mailer.default_url_options = { host: 'example.com' }

# view 中只能是绝对地址 xxx_url 而不是 xxx_path
link_to 'welcome', welcome_url
```

Email 类型

遵循 view 的命名规则，根据 view 的后缀来决定

- Text 类型 signup.text.erb
- HTML 类型 signup.html.erb



## 发送 Email

```rb
UserMailer.signup(@user).deliver_now

UserMailer.signup(@user).deliver_later
```



