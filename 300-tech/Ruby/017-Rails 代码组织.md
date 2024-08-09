---
title: 017-Rails 代码组织
date: 2024-08-08 22:36
updated: 2024-08-09 11:49
---

- concerns 目录
- lib 目录
- Fat Model

## concerns 目录

共用逻辑，跨控制器/模型共享代码

- `controllers/concerns`
- `models/concerns`

例如 `app/controllers/concerns/user_session.rb` 定义好方法后，在 `app/controllers/application_controller.rb` 中引入 `include UserSession`

- 放置重复代码
- 独立功能模块

没有 concerns 使用

- presenters
- services

自己手动在 `app` 下建立目录，然后在 rails 环境中进行配置

## lib 目录

通常放置通用业务或业务相关的代码

## Fat Model

- 让 controller 轻量化
- 把业务逻辑放在 model 中

针对数据层的逻辑

## 密码加密

之前移植文件中定义密码字段用的是明文存储，应当让它存储加密后的 hash 值

新建移植文件

```sh
$ bin/rails g migration add_users_crypted_password_column
      invoke  active_record
      create    db/migrate/20240809020505_add_users_crypted_password_column.rb
```

修改生成的移植文件，添加两个字段

```rb
class AddUsersCryptedPasswordColumn < ActiveRecord::Migration[7.0]
  def change
    add_column :users, :crypted_password, :string, null: false
    add_column :users, :salt, :string, null: false  # 混淆

    remove_column :users, :password
  end
end
```

1. **`crypted_password`**：存储经过加密处理的用户密码。原始密码不会直接存储，增强安全性。
2. **`salt`**：用于生成加密密码的随机值，增加密码的复杂性，防止相同密码生成相同加密值，避免彩虹表攻击。

运行命令进行数据库迁移

```sh
$ bin/rake db:migrate                                    
== 20240809020505 AddUsersCryptedPasswordColumn: migrating ====================
-- add_column(:users, :crypted_password, :string, {:null=>false})
   -> 0.1273s
-- add_column(:users, :salt, :string, {:null=>false})
   -> 0.1135s
-- remove_column(:users, :password)
   -> 0.1278s
== 20240809020505 AddUsersCryptedPasswordColumn: migrated (0.3688s) ===========
```

接着查看 `db/schema.rb` 就能看到这两个字段添加进去了

```rb
create_table "users", charset: "utf8mb4", collation: "utf8mb4_unicode_ci", force: :cascade do |t|
  t.string "username"
  t.datetime "created_at", null: false
  t.datetime "updated_at", null: false
  t.string "crypted_password", null: false
  t.string "salt", null: false
end
```

现在修改一下注册表单，需要三个参数：用户名、密码、确认密码

```rb
<%= form_for @user, url: users_path, method: :post do |f| %>
  <div class="dialog alert alert-danger" style="display: <%= @user.errors.any? ? 'block' : 'none' %>">
    <ul class="list-unstyled">
      <% @user.errors.messages.values.flatten.each do |error| %>
        <li><i class="fa fa-exclamation-circle"></i> <%= error %></li>
      <% end -%>
    </ul>
  </div>

  <div class="form-group">
    <%= f.label :username, "用户名" %>
    <%= f.text_field :username, class: "form-control", placeholder: "用户名" %>
  </div>

  <div class="form-group">
    <%= f.label :password, "密码" %>
    <%= f.password_field :password, class: "form-control", placeholder: "密码" %>
  </div>

  <div class="form-group">
    <%= f.label :password_confirmation, "确认密码" %>
    <%= f.password_field :password_confirmation, class: "form-control", placeholder: "再次输入密码" %>
  </div>

  <%= f.submit "注册", class: "btn btn-primary" %> 
  <%= link_to "登录", new_session_path %>
<% end -%>
```

加密和校验在 model 中完成 `app/models/user.rb`

```rb
require 'securerandom'

class User < ApplicationRecord

  # 模型验证
  validates :username, presence: { message: "用户名不能为空" }
  validates :username, uniqueness: { message: "用户名已存在" }
  # validates :password, presence: { message: "密码不能为空" }
  # validates :password, length: { minimum: 6, message: "密码长度不能小于6位" }

  has_many :blogs
  has_many :public_blogs, -> { where(is_public: true) }, class_name: "Blog"

  # 声明访问器 读写属性
  attr_accessor :password
  attr_accessor :password_confirmation

  # 自定义验证
  validate :validate_password, on: :create

  # 在创建之前执行
  before_create :set_password 

  # 定义一些方法
  def self.authenticate(username, password)
    user = User.find_by(username: username)
    if user && user.valid_password?(password)
      user
    else
      nil
    end
  end

  def valid_password? password
    self.crypted_password == encrypt(password, self.salt)
  end

  private
  def validate_password
    if self.password.blank?
      errors.add(:password, "密码不能为空")
      return false  # 验证不通过
    end

    unless self.password == self.password_confirmation
      errors.add(:password, "两次密码不一致")
      return false
    end

    return true  # 验证通过
  end

  def set_password
    self.salst = SecureRandom.hex(16)  # 随机 并保证长度足够
    self.crypted_password = encrypt(self.password, self.salt)
  end

  def encrypt(password, salt)
    # 自定义加密方法，通常使用散列算法（如 SHA-256）和 salt
    Digest::SHA256.hexdigest(password + salt)
  end

end
```

**密码加密**：散列算法，例如 SHA-256

salt 随机数生成（随机性和唯一性）：适合长度、不使用伪随机生成

- 使用随机数生成器 `require 'securerandom'`，`salt = SecureRandom.hex(16)` 生成一个 32 字节的十六进制字符串
- 使用加密库 bcrypt 提供了生成 salt

在 controller 中 `app/controllers/users_controller.rb`

参数多传了一个，加上

```rb
def user_params
  params.require(:user).permit(:username, :password, :password_confirmation)
end
```

登录的时候

```rb
def create
  @user = User.authenticate(params[:username], params[:password])
  if @user
    signin_user @user
    
    render json: {
      status: 'ok',
      msg: {
        redirect_url: root_path
      }
    }
  else
    render json: {
      status: 'error',
      msg: "用户名或密码不正确"
    }
  end
end
```

由于我们直接删除了之前的 password 字段，现在是登录不上的，正常解决方案是需要联系用户重置密码。之后，考虑以下步骤：

1. **创建重置密码机制**：设计一个用户友好的重置密码流程。确保系统通过电子邮件或短信通知用户，帮助他们重设密码。
2. **为现有用户生成新的密码**：如果无法让用户重置密码，考虑为他们生成随机密码，并通过安全渠道（如电子邮件）发送给他们。
3. **存储加密密码**：确保新的密码使用安全的加密算法存储。例如，使用 `bcrypt` 或 `argon2` 进行密码加密。
4. **更新数据库模型**：在数据库中添加加密密码字段和 `salt` 字段（如果使用）。
5. **实施数据保护措施**：验证所有用户数据的安全性，确保今后不再存储明文密码。

这里就不直接操作数据库，给一个临时解决方案

`app/models/user.rb`

```rb
require 'securerandom'
require 'digest'

class User < ApplicationRecord
  validates :username, presence: { message: "用户名不能为空" }
  validates :username, uniqueness: { message: "用户名已存在" }

  has_many :blogs
  has_many :public_blogs, -> { where(is_public: true) }, class_name: "Blog"

  attr_accessor :password
  attr_accessor :password_confirmation

  validate :validate_password, on: :create

  before_create :set_password 

  def self.authenticate(username, password)
    user = User.find_by(username: username)
    return { user: nil, message: "用户不存在" } if user.nil?
    if user.crypted_password.blank? && user.salt.blank?
      user.salt = SecureRandom.hex(16)
      user.crypted_password = encrypt_password('123456', user.salt)
      user.save
      return { user: user, message: "密码已重置为 '123456'，请修改密码" }
    end
    if user.valid_password?(password)
      { user: user, message: "登录成功" }
    else
      { user: nil, message: "密码错误" }
    end
  end

  def valid_password?(password)
    self.crypted_password == self.class.encrypt_password(password, self.salt)
  end

  private

  def validate_password
    if self.password.blank?
      errors.add(:password, "密码不能为空")
      return false
    end
    unless self.password == self.password_confirmation
      errors.add(:password, "两次密码不一致")
      return false
    end
    true
  end

  def set_password
    self.salt = SecureRandom.hex(16)
    self.crypted_password = self.class.encrypt_password(self.password, self.salt)
  end

  def self.encrypt_password(password, salt)
    Digest::SHA256.hexdigest(password + salt)
  end
end
```

`app/controllers/sessions_controller.rb`

```rb
def create
  result = User.authenticate(params[:username], params[:password])
  if result
    if result[:message]
      flash[:notice] = result[:message]
    end
    if result[:user]
      @user = result[:user]
      signin_user @user
      redirect_to root_path
    else
      render action: :new, status: 422
    end
  end
end
```
