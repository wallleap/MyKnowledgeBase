---
title: 016-Rails Model
date: 2024-08-07T02:59:00+08:00
updated: 2024-08-21T10:32:31+08:00
dg-publish: false
---

## CoC

Conventions

### table name

```rb
class User < ActiveRecord::Base
end
```

- User → users
- Person → people
- Blog → blogs
- BlogTag → blog_tags

### columns

| Column         | Column Type                                       |
| -------------- | ------------------------------------------------- |
| **id**         | int(11) NOT NULL AUTO_INCREMENT                   |
| username       | varchar(255) COLLATE utf8_unicode_ci DEFAULT NULL |
| password       | varchar(255) COLLATE utf8_unicode_ci DEFAULT NULL |
| **created_at** | datetime NOT NULL                                 |
| **updated_at** | datetime NOT NULL                                 |

对应数据类型

| Rails 数据类型 | MySQL 对应类型                    | PostgreSQL 对应类型             | SQLite 对应类型                 |
|----------------|----------------------------------|--------------------------------|--------------------------------|
| string         | varchar(255) / text               | character varying / text       | varchar / text                  |
| integer        | int                              | integer                        | integer                        |
| float          | float                            | double precision               | float                          |
| boolean        | tinyint(1)                       | boolean                        | boolean                        |
| datetime       | datetime                         | timestamp                      | datetime                       |
| text           | text                             | text                           | text                           |
| timestamp      | timestamp                        | timestamp                      | timestamp                      |

**修改表名**

```rb
class User < ActiveRecord::Base
  self.table_name = "user"  # users → user
end
```

`migrate` 下迁移文件也需要修改对应的表名

## CRUD 基础操作

```rb
class User < ActiveRecord::Base
end
```

有了模型就可以对其进行操作

```rb
user = User.new username: "test", password: "123456"
user.save  # 成功返回 user 对象，失败返回 false

user2 = User.create username: "test2", password: "123456"

user3 = User.new
user3.username = "test3"
user3.password = "123456"
user3.save
```

查询 基本 SQL 语句支持的关键字都有

```rb
user = User.where(username: "test").first
user = User.find_by(username: "test")
user = User.first

users = User.where(["username like ?", "%test%"]).order("id desc").limit(2)
```

更新

```rb
user = User.find_by(username: "test")
user.username = "test1"
user.save

User.update_all "status = 1, must_change_password = 'true'"
```

删除

```rb
user = User.find_by(username: "test")
user.destroy
```

## Associations

表间关联关系

- `has_many`：一对多、多对多
- `has_one`：一对一
- `belongs_to`：一对一
- `has_and_belongs_to_many`：多对多

```rb
# 这是 user.rb 文件
class User < ApplicationRecord
  has_many :blogs
end

# 这是 blog.rb 文件
class Blog < ApplicationRecord
  belongs_to :user
end
```

| model | table | column  | key type       |
| ----- | ----- | ------- | -------------- |
| User  | users | id      | primary key    |
| Blog  | blogs | user_id | foreign_key 外键 |

外键命名：一对多或属于，所有模型小写加下划线 id

## 项目中添加 blogs 功能

### 创建模型

```sh
$ bin/rails g model blog             
      invoke  active_record
      create    db/migrate/20240807080531_create_blogs.rb
      create    app/models/blog.rb
      invoke    test_unit
      create      test/models/blog_test.rb
      create      test/fixtures/blogs.yml
```

修改生成的迁移文件 `db/migrate/20240807080531_create_blogs.rb`

```rb
class CreateBlogs < ActiveRecord::Migration[7.0]
  def change
    create_table :blogs do |t|
      t.string :title
      t.text :content
      t.boolean :is_public, default: false
      # t.integer :user_id
      t.belongs_to :user, null: false, foreign_key: true  # 这样会自己创建user_id字段

      t.timestamps
    end

    # add_index :blogs, [:user_id]  # 索引，加快数据库查找
  end
end
```

数据库层面也可以先加验证

运行命令创建数据库表

```sh
$ bin/rake db:migrate   
== 20240807080531 CreateBlogs: migrating ======================================
-- create_table(:blogs)
   -> 0.0210s
== 20240807080531 CreateBlogs: migrated (0.0211s) =============================
```

模型验证和关联 `app/models/blog.rb`

```rb
class Blog < ApplicationRecord

  validates :title, presence: { message: "标题不能为空" }
  validates :content, presence: { message: "内容不能为空" }
  validates :user_id, presence: { message: "用户不能为空" }

  belongs_to :user

end
```

`app/models/user.rb`

```rb
class User < ApplicationRecord

  # ...
  has_many :blogs

end
```

### 创建控制器

```sh
$ bin/rails g controller blogs       
      create  app/controllers/blogs_controller.rb
      invoke  erb
      create    app/views/blogs
      invoke  test_unit
      create    test/controllers/blogs_controller_test.rb
      invoke  helper
      create    app/helpers/blogs_helper.rb
      invoke    test_unit
```

路由 `config/routes.rb`

```rb
resources :blogs
```

`app/views/layouts/application.html.erb` 添加上博客链接

```rb
<li class="nav-item"><%= link_to "博客", blogs_path, class: "nav-link" %></li>
```

`app/controllers/blogs_controller.rb`

```rb
class BlogsController < ApplicationController

  def index
    @blogs = Blog.page(params[:page] || 1).per_page(params[:per_page] || 10).order("id DESC")
  end

end
```

### 创建对应视图

`app/views/blogs/index.html.erb`

```rb
<h1 class="mb-4">博客列表 <%= link_to '去写博客', new_blog_path, class: "btn btn-primary" %></h1>

<div class="blog-list">
  <% @blogs.each do |blog| %>
    <div class="card mb-3">
      <div class="card-body">
        <h2 class="card-title"><%= link_to blog.title, blog_path(blog), class: "text-dark" %></h2>
        <p class="card-text">作者: <%= blog.user.username %> · 博客 id：<%= blog.id %></p>
        <p class="card-text">发布时间: <%= blog.created_at.strftime('%b %d, %Y') %></p>
      </div>
    </div>
  <% end %>
</div>
```

### 新建博客

由于需要校验用户登录了才能创建博客，所以把之前校验的代码移到 `app/controllers/application_controller.rb`

```rb
class ApplicationController < ActionController::Base
  protect_from_forgery wddith: :exception

  include UserSession

  private
  def authenticate_user!
    unless logged_in?
      flash[:notice] = "请先登录"
      redirect_to :back
    end
  end
end
```

`app/controllers/blogs_controller.rb`

```rb
class BlogsController < ApplicationController

  before_action :authenticate_user!, except: [:index, :show]

  def index
    @blogs = Blog.page(params[:page] || 1).per_page(params[:per_page] || 10).order("id DESC")
  end

  def new
    @blog = Blog.new
  end

  def create
    @blog = Blog.new(blog_params)
    @blog.user = current_user  # current_user 是 UserSession 模块定义的，可以获取当前登录的用户
    if @blog.save
      flash[:notice] = "博客创建成功"
      redirect_to blogs_path
    else
      render action: :new, status: 422
    end
  end

  private
  def blog_params
    params.require(:blog).permit(:title, :content)
  end

end
```

View

```rb
<h1>创建博客</h1>

<div class="row">
  <div class="col-sm-5">
    <%= form_for @blog, url: blogs_path, method: :post do |f| %>
      <div class="dialog alert alert-danger" style="display: <%= @blog.errors.any? ? 'block' : 'none' %>">
        <ul class="list-unstyled">
          <% @blog.errors.messages.values.flatten.each do |error| %>
            <li><i class="fa fa-exclamation-circle"></i> <%= error %></li>
          <% end -%>
        </ul>
      </div>

      <div class="form-group">
        <%= f.label :title, "标题" %>
        <%= f.text_field :title, class: "form-control", placeholder: "标题" %>
      </div>

      <div class="form-group">
        <%= f.label :content, "内容" %>
        <%= f.text_area :content, class: "form-control", placeholder: "内容" %>
      </div>

      <%= f.submit "发布", class: "btn btn-primary" %> 
    <% end -%>
  </div>

  <div class="col-sm-7">
  </div>
</div>
```

### 展示单篇博客

`app/controllers/blogs_controller.rb`

```rb
def show
  @blog = Blog.find(params[:id])
end
```

`app/views/blogs/show.html.erb`

```rb
<div class="container">
  <div class="row">
    <div class="col-md-8 offset-md-2">
      <div class="card mt-4">
        <div class="card-header">
          <h2><%= @blog.title %></h2>
        </div>
        <div class="card-body">
          <p class="card-text"><%= @blog.content.html_safe %></p>
        </div>
        <ul class="list-group list-group-flush">
          <li class="list-group-item"><strong>Blog ID:</strong> <%= @blog.id %></li>
          <li class="list-group-item"><strong>Created At:</strong> <%= @blog.created_at.strftime('%B %e, %Y') %></li>
          <li class="list-group-item"><strong>Author:</strong> <%= @blog.user.username %></li>
        </ul>
        <div class="card-footer">
          <%= link_to 'Edit', edit_blog_path(@blog), class: 'btn btn-primary' %>
          <%= link_to 'Back', blogs_path, class: 'btn btn-secondary' %>
        </div>
      </div>
    </div>
  </div>
</div>
```

### 展示我的博客

`config/routes.rb`

```rb
resources :users do
  get :blogs, on: :member
end
```

修改 `app/controllers/users_controller.rb`

```rb
def blogs
  @blogs = current_user.blogs.page(params[:page] || 1).per_page(params[:per_page] || 10).order("id DESC")
end
```

添加 `app/views/users/blogs.html.erb`

```rb
<h1 class="mb-4">
  <%= current_user.username %>的博客
  <%= link_to '去写博客', new_blog_path, class: "btn btn-primary" %>
</h1>

<div class="blog-list">
  <% @blogs.each do |blog| %>
    <div class="card mb-3">
      <div class="card-body">
        <h2 class="card-title"><%= link_to blog.title, blog_path(blog), class: "text-dark" %></h2>
        <p class="card-text">发布时间: <%= blog.created_at.strftime('%b %d, %Y') %></p>
      </div>
    </div>
  <% end %>
</div>
```

在 `app/views/blogs/index.html.erb` 添加一个链接

```rb
<% if logged_in? %>
  <%= link_to '我的博客', blogs_user_path(current_user), class: "btn btn-default" %>
<% end %>
```

创建了 users::blogs 这样的话在 blogs create 就可以这样写

```rb
@blog = current_user.blogs.new(blog_params)
# current_user.blogs << Blog.new(blog_params)  # 追加进入
```

### 博客标签

has_and_belongs_to_many 一篇博客会有很多个标签，一个标签能匹配到很多博客

- blogs
- tags
- 还需要一张关联关系表 blogs_tags

has_many 当我们需要访问关联关系表 blogs_tags，且在关联关系表 blogs_tags 添加一些自定义的字段、回调、属性检查等 `has_many :through`

#### 创建模型

```sh
$ bin/rails g model tag
      invoke  active_record
      create    db/migrate/20240807093135_create_tags.rb
      create    app/models/tag.rb
      invoke    test_unit
      create      test/models/tag_test.rb
      create      test/fixtures/tags.yml
```

编写迁移文件 `db/migrate/20240807093135_create_tags.rb`

```rb
class CreateTags < ActiveRecord::Migration[7.0]
  def change
    create_table :tags do |t|
      t.string :title

      t.timestamps
    end

    create_table :blogs_tags do |t|  # 由于多对多关系，需要创建一个中间表，而且不需要使用它的模型，所以就直接在这创建关联关系表就行了
      t.integer :blog_id
      t.integer :tag_id

      # t.references :blog, null: false, foreign_key: true
      # t.references :tag, null: false, foreign_key: true
    end
  end
end
```

`app/models/blog.rb` 添加多对多的语句

```rb
class Blog < ApplicationRecord

  validates :title, presence: { message: "标题不能为空" }
  validates :content, presence: { message: "内容不能为空" }
  validates :user_id, presence: { message: "用户不能为空" }

  belongs_to :user
  has_and_belongs_to_many :tags

end
```

`app/models/tag.rb`

```rb
class Tag < ApplicationRecord

  has_and_belongs_to_many :blogs

end
```

执行命令迁移数据库

```sh
$ bin/rake db:migrate         
== 20240807093135 CreateTags: migrating =======================================
-- create_table(:tags)
   -> 0.0356s
-- create_table(:blogs_tags)
   -> 0.0348s
== 20240807093135 CreateTags: migrated (0.0706s) ==============================
```

#### 重构 View

之前的我的博客和博客展示页面可以合并的

创建 partial 模版 `app/views/blogs/_blogs.html.erb`

```rb
<div class="blog-list">
  <% @blogs.each do |blog| %>
    <div class="card mb-3">
      <div class="card-body">
        <h2 class="card-title"><%= link_to blog.title, blog_path(blog), class: "text-dark" %></h2>
        <p class="card-text">作者: <%= blog.user.username %> · 博客 id：<%= blog.id %></p>
        <p class="card-text">发布时间: <%= blog.created_at.strftime('%b %d, %Y') %></p>
      </div>
    </div>
  <% end %>
</div>

<%= will_paginate @blogs %>
```

在 `app/views/blogs/index.html.erb` 引入

```rb
<h1 class="mb-4">博客列表
  <%= link_to '去写博客', new_blog_path, class: "btn btn-primary" %>
  <% if logged_in? %>
    <%= link_to '我的博客', blogs_user_path(current_user), class: "btn btn-default" %>
  <% end %>
</h1>

<%= render 'blogs' %>  # 同级目录直接写
```

在 `app/views/users/blogs.html.erb` 引入

```rb
<h1 class="mb-4">
  <%= current_user.username %>的博客
  <%= link_to '去写博客', new_blog_path, class: "btn btn-primary" %>
</h1>

<%= render 'blogs/blogs' %>  # 不同页面以 view 为根目录写全
```

#### 添加上标签相关

`app/views/blogs/new.html.erb`

```rb
<div class="form-group">
  <label for="article_tags">标签</label>
  <input type="text" name="article[tags]" class="form-control" placeholder="标签（以 , 分隔）" >
</div>
```

`app/models/tag.rb` 模型校验

```rb
class Tag < ApplicationRecord

  validates :title, uniqueness: { message: "标签已存在" }

  has_and_belongs_to_many :blogs

end
```

`app/controllers/blogs_controller.rb`

```rb
  def create
    @blog = Blog.new(blog_params)  # 也可以 @blog = current_user.blogs.new(blog_params)
    # current_user.blogs << Blog.new(blog_params)
    @blog.user = current_user  # current_user 是 UserSession 模块定义的，可以获取当前登录的用户
    if @blog.save
    
      tags = params[:article][:tags]
      tags.split(",").each do |tag|
        @blog.tags << Tag.find_or_create_by(title: tag.strip)
        # 等同下面三行
        # one_tag = Tag.find_by(title: tag)
        # one_tag = Tag.new(title: tag) unless one_tag
        # @blog.tags << one_tag
      end

      flash[:notice] = "博客创建成功"
      redirect_to blogs_path
    else
      render action: :new, status: 422
    end
  end
```

在 view 中展示

```rb
<ul class="list-inline">
    <% blog.tags.each do |tag| %>
      <li class="badge rounded-pill badge-primary text-bg-secondary"><%= tag.title %></li>
    <% end %>
</ul>
```

### 操作关联关系表

改用 has_many

直接新建 `app/models/blogs_tags.rb`

```rb
class BlogsTags < ApplicationRecord

  self.table_name = "blogs_tags"

  validates_uniqueness_of :blog_id, scope: [:tag_id] # 同一个博客只能关联一个标签

  belongs_to :blog
  belongs_to :tag

end
```

`app/models/blog.rb`

```rb
class Blog < ApplicationRecord

  validates :title, presence: { message: "标题不能为空" }
  validates :content, presence: { message: "内容不能为空" }
  validates :user_id, presence: { message: "用户不能为空" }

  belongs_to :user
  has_many :blogs_tags, class_name: "BlogsTags"
  has_many :tags, through: :blogs_tags

end
```

`app/models/tag.rb`

```rb
class Tag < ApplicationRecord

  validates :title, uniqueness: { message: "标签已存在" }

  has_many :blogs_tags, class_name: "BlogsTags"
  has_many :blogs, through: :blogs_tags

end
```

### 关联关系定制

需要返回只有是公开状态下的博客数据记录 has_many proc

`app/views/blogs/new.html.erb`

```rb
<div class="form-group">
  <label for="article_tags">是否公开</label>
  <% [[true, "公开"], [false, "不公开"]].each do |public| %>
    <input type="radio" name="blog[is_public]" value="<%= public[0] %>" <%= "checked" if public[0] == @blog.is_public %>> <%= public[1] %>
  <% end %>
</div>
```

`app/views/blogs/_blogs.html.erb`

```rb
<% if blog.user == current_user %>
  <%= blog.is_public ? "公开" : "私密" %>
<% end %>
```

`app/controllers/blogs_controller.rb`

```rb
private
def blog_params
  params.require(:blog).permit(:title, :content, :is_public)
end
```

**只显示公开的博客** 通过查询语句

`app/models/user.rb`

```rb
has_many :public_blogs, -> { where(is_public: true) }, class_name: "Blog"
```

`app/controllers/blogs_controller.rb`

```rb
def index
  @blogs = Blog.where(is_public: true).page(params[:page] || 1).per_page(params[:per_page] || 10)
    .order("id DESC")
end
```

### 编辑博客

`app/controllers/blogs_controller.rb` 添加 edit 和 update，抽离 update_tags 方法

```rb
class BlogsController < ApplicationController

  before_action :authenticate_user!, except: [:index, :show]

  def index
    @blogs = Blog.page(params[:page] || 1).per_page(params[:per_page] || 10).order("id DESC")
  end

  def new
    @blog = Blog.new
  end

  def create
    @blog = Blog.new(blog_params)  # 也可以 @blog = current_user.blogs.new(blog_params)
    # current_user.blogs << Blog.new(blog_params)
    @blog.user = current_user  # current_user 是 UserSession 模块定义的，可以获取当前登录的用户
    if @blog.save
      update_tags

      flash[:notice] = "博客创建成功"
      redirect_to blogs_path
    else
      flash[:notice] = "博客创建失败"
      render action: :new, status: 422
    end
  end

  def show
    @blog = Blog.find(params[:id])
  end

  def edit
    @blog = Blog.find(params[:id])
    render action: :new
  end

  def update
    @blog = Blog.find(params[:id])
    @blog.attributes = blog_params
    if @blog.save
      @blog.tags.clear
      update_tags

      flash[:notice] = "博客更新成功"
      redirect_to blog_path(@blog)
    else
      flash[:notice] = "博客更新失败"
      render action: :new, status: 422
    end
  end

  private
  def blog_params
    params.require(:blog).permit(:title, :content, :is_public)
  end

  def update_tags
    tags = params[:article][:tags]
    tags.split(",").each do |tag|
      @blog.tags << Tag.find_or_create_by(title: tag.strip)
      # 等同下面三行
      # one_tag = Tag.find_by(title: tag)
      # one_tag = Tag.new(title: tag) unless one_tag
      # @blog.tags << one_tag
    end
  end

end
```

`app/views/blogs/new.html.erb` 通过 `@blog.new_record?` 可以判断是否是新建的

```rb
<h1><%= @blog.new_record? ? "创建" : "修改" %>博客</h1>

<div class="row">
  <div class="col-sm-5">
    <%= form_for @blog, url: (@blog.new_record? ? blogs_path : blog_path(@blog)), method: (@blog.new_record? ? :post : :put) do |f| %>
      <div class="dialog alert alert-danger" style="display: <%= @blog.errors.any? ? 'block' : 'none' %>">
        <ul class="list-unstyled">
          <% @blog.errors.messages.values.flatten.each do |error| %>
            <li><i class="fa fa-exclamation-circle"></i> <%= error %></li>
          <% end -%>
        </ul>
      </div>

      <div class="form-group">
        <%= f.label :title, "标题" %>
        <%= f.text_field :title, class: "form-control", placeholder: "标题" %>
      </div>

      <div class="form-group">
        <%= f.label :content, "内容" %>
        <%= f.text_area :content, class: "form-control", placeholder: "内容", rows: 10 %>
      </div>

      <div class="form-group">
        <label for="article_tags">标签</label>
        <input type="text" name="article[tags]" class="form-control" placeholder="标签（以 , 分隔）" 
          value="<%= @blog.tags.map(&:title).join(', ') %>"
        >
      </div>

      <div class="form-group">
        <label for="article_tags">是否公开</label>
        <% [[true, "公开"], [false, "不公开"]].each do |public| %>
          <input type="radio" name="blog[is_public]" value="<%= public[0] %>" <%= "checked" if public[0] == @blog.is_public %>> <%= public[1] %>
        <% end %>
      </div>

      <%= f.submit (@blog.new_record? ? "发布" : "修改"), class: "btn btn-primary" %> 
    <% end -%>
  </div>

  <div class="col-sm-7">
  </div>
</div>
```

`app/views/blogs/show.html.erb` 在单篇博客页面显示编辑按钮

```rb
<div class="card-footer">
  <% if current_user == @blog.user %>
    <%= link_to 'Edit', edit_blog_path(@blog), class: 'btn btn-primary' %>
  <% end %>
  <%= link_to 'Back', blogs_path, class: 'btn btn-secondary' %>
</div>
```

## CRUD 更多操作

### 查找

- `all` 全部
- `find(1)` 只能接收 id，没找到抛出异常
- `find_by(id: 1)` 接收 Hash，没找到返回 nil
- `find_by!` 会抛出异常（Ruby 中 `!` 方法修改数据自身，Rails 中带叹号说明有可能会抛出异常）
- `find_by_sql` 可以指定 SQL 语句查询，SQL 语句复杂的时候，返回数组，接收字符串形式，模型和后面的表可以不对应，但是这样没意义（会封装成模型的数据），多表查到同字段会返回第一张表的字段
- `where` 查询，返回数组，查询对象（还没有真正向数据库查数据）
	- 后面传递的参数对应每一个问号
	- Hash

```sh
$ bin/rails c        
Loading development environment (Rails 7.0.8.4)
3.0.0 :001 > u = User.find 2
  User Load (7.5ms)  SELECT `users`.* FROM `users` WHERE `users`.`id` = 2 LIMIT 1
 => 
#<User:0x000000011d656a30
... 
3.0.0 :002 > u1 = User.find_by id: 2
  User Load (35.1ms)  SELECT `users`.* FROM `users` WHERE `users`.`id` = 2 LIMIT 1
 => 
#<User:0x0000000109968908
...
3.0.0 :005 > u1 = User.find_by username: "luwang"
  User Load (0.8ms)  SELECT `users`.* FROM `users` WHERE `users`.`username` = 'luwang' LIMIT 1
 => 
#<User:0x0000000128bbe710
...

3.0.0 :004 > u = User.find 200
  User Load (0.6ms)  SELECT `users`.* FROM `users` WHERE `users`.`id` = 200 LIMIT 1
(irb):4:in `<main>': Couldn't find User with 'id'=200 (ActiveRecord::RecordNotFound)
3.0.0 :003 > u1 = User.find_by id: 200
  User Load (1.0ms)  SELECT `users`.* FROM `users` WHERE `users`.`id` = 200 LIMIT 1
 => nil 

3.0.0 :001 > u = User.find_by! username: "luwang"
  User Load (0.6ms)  SELECT `users`.* FROM `users` WHERE `users`.`username` = 'luwang' LIMIT 1
 => 
#<User:0x000000010880f5d0
3.0.0 :002 > u = User.find_by! username: ""
  User Load (0.8ms)  SELECT `users`.* FROM `users` WHERE `users`.`username` = '' LIMIT 1
(irb):2:in `<main>': Couldn't find User with [WHERE `users`.`username` = ?] (ActiveRecord::RecordNotFound)
...

3.0.0 :004 > u = User.find_by_sql "select * from users where id = 2"
  User Load (1.8ms)  select * from users where id = 2
 => 
[#<User:0x00000001056b81a8
...
3.0.0 :005 > u.first.id
 => 2

3.0.0 :007 > User.where("id = 2")
  User Load (1.4ms)  SELECT `users`.* FROM `users` WHERE (id = 2)
 => 
[#<User:0x00000001056097e8
  id: 2,
  username: "luwang",
  password: "[FILTERED]",
  created_at: Mon, 05 Aug 2024 08:38:03.046254000 UTC +00:00,
  updated_at: Mon, 05 Aug 2024 08:38:03.046254000 UTC +00:00>] 
3.0.0 :009 > u = User.where(["id = ?", 3])
  User Load (1.6ms)  SELECT `users`.* FROM `users` WHERE (id = '3')
 => 
[#<User:0x0000000107c18e20
...
3.0.0 :010 > u = User.where(["id = ? and username = ?", 3, "admin"])
  User Load (1.5ms)  SELECT `users`.* FROM `users` WHERE (id = '3' and username = 'admin')
 => 
[#<User:0x0000000107feb638
...
3.0.0 :012 > u = User.where(id: 3, username: "admin").first
  User Load (1.0ms)  SELECT `users`.* FROM `users` WHERE `users`.`id` = 3 AND `users`.`username` = 'admin' ORDER BY `users`.`id` ASC LIMIT 1
 => 
#<User:0x000000011e833cf8
...
```

**n + 1 查询**

例如博客列表中，先查询当前用户，然后查询博客，查询博客的标签（每篇博客都需要查询，因此会有博客 id 一直在加后查询，用户也是）

```sh
  User Load (0.4ms)  SELECT `users`.* FROM `users` WHERE `users`.`id` = 2 LIMIT 1
  ↳ app/controllers/concerns/user_session.rb:31:in `current_user'
  Blog Load (1.1ms)  SELECT `blogs`.* FROM `blogs` WHERE `blogs`.`is_public` = TRUE ORDER BY id DESC LIMIT 10 OFFSET 0
  ↳ app/views/blogs/_blogs.html.erb:2
  Tag Load (0.7ms)  SELECT `tags`.* FROM `tags` INNER JOIN `blogs_tags` ON `tags`.`id` = `blogs_tags`.`tag_id` WHERE `blogs_tags`.`blog_id` = 19
  ↳ app/views/blogs/_blogs.html.erb:7
  CACHE User Load (0.0ms)  SELECT `users`.* FROM `users` WHERE `users`.`id` = 2 LIMIT 1
  ↳ app/views/blogs/_blogs.html.erb:11
  Tag Load (1.4ms)  SELECT `tags`.* FROM `tags` INNER JOIN `blogs_tags` ON `tags`.`id` = `blogs_tags`.`tag_id` WHERE `blogs_tags`.`blog_id` = 18
  ↳ app/views/blogs/_blogs.html.erb:7
  CACHE User Load (0.0ms)  SELECT `users`.* FROM `users` WHERE `users`.`id` = 2 LIMIT 1
  ↳ app/views/blogs/_blogs.html.erb:11
  Tag Load (5.0ms)  SELECT `tags`.* FROM `tags` INNER JOIN `blogs_tags` ON `tags`.`id` = `blogs_tags`.`tag_id` WHERE `blogs_tags`.`blog_id` = 1
  ↳ app/views/blogs/_blogs.html.erb:7
  CACHE User Load (0.0ms)  SELECT `users`.* FROM `users` WHERE `users`.`id` = 2 LIMIT 1
```

在项目中需要避免这种关联关系查询，使用 `#includes`

优化 `blogs#index`

```rb
def index
  @blogs = Blog.where(is_public: true).page(params[:page] || 1).per_page(params[:per_page] || 10)
    .order("id DESC").includes(:tags, :user)  # 如果还有可以接着关联用户模型关联关系，例如头像 .includes(:tags, [:user, :avatar])
end
```

现在查询的时候就用的 `IN (19, 18, 1)`

```sh
  User Load (1.4ms)  SELECT `users`.* FROM `users` WHERE `users`.`id` = 2 LIMIT 1
  ↳ app/controllers/concerns/user_session.rb:31:in `current_user'
  Blog Load (1.0ms)  SELECT `blogs`.* FROM `blogs` WHERE `blogs`.`is_public` = TRUE ORDER BY id DESC LIMIT 10 OFFSET 0
  ↳ app/views/blogs/_blogs.html.erb:2
  BlogsTags Load (19.3ms)  SELECT `blogs_tags`.* FROM `blogs_tags` WHERE `blogs_tags`.`blog_id` IN (19, 18, 1)
  ↳ app/views/blogs/_blogs.html.erb:2
  Tag Load (4.9ms)  SELECT `tags`.* FROM `tags` WHERE `tags`.`id` IN (9, 3, 10)
  ↳ app/views/blogs/_blogs.html.erb:2
  User Load (4.1ms)  SELECT `users`.* FROM `users` WHERE `users`.`id` = 2
```

### 更新

- `update` 批量修改
- `update_attribute` 字段和内容
- `attributes`
- `changed?` 是否有修改没有保存到数据库
- `changed_attributes` 哪些字段修改了没有保存，输出的是修改前的值

```sh
3.0.0 :014 > b = Blog.first
  Blog Load (0.6ms)  SELECT `blogs`.* FROM `blogs` ORDER BY `blogs`.`id` ASC LIMIT 1
 => 
#<Blog:0x0000000108834588
... 
3.0.0 :015 > b.title
 => "我的第一篇博客" 
3.0.0 :016 > b.title = "First Blog I Changed"
 => "First Blog I Changed" 
3.0.0 :017 > b.save
  TRANSACTION (0.5ms)  BEGIN
  User Load (0.5ms)  SELECT `users`.* FROM `users` WHERE `users`.`id` = 2 LIMIT 1
  Blog Update (3.4ms)  UPDATE `blogs` SET `blogs`.`title` = 'First Blog I Changed', `blogs`.`updated_at` = '2024-08-08 09:22:19.870773' WHERE `blogs`.`id` = 1
  TRANSACTION (1.6ms)  COMMIT
 => true 
3.0.0 :018 > b.title
 => "First Blog I Changed"

3.0.0 :020 > b.update title: "First Blog", content: "Blog Content"
  TRANSACTION (0.5ms)  BEGIN
  Blog Update (0.9ms)  UPDATE `blogs` SET `blogs`.`title` = 'First Blog', `blogs`.`content` = 'Blog Content', `blogs`.`updated_at` = '2024-08-08 09:25:00.671100' WHERE `blogs`.`id` = 1
  TRANSACTION (0.9ms)  COMMIT
 => true

3.0.0 :021 > b.update_attribute :title, "First Blog by luwang"
  TRANSACTION (0.6ms)  BEGIN
  Blog Update (1.1ms)  UPDATE `blogs` SET `blogs`.`title` = 'First Blog by luwang', `blogs`.`updated_at` = '2024-08-08 09:26:32.248804' WHERE `blogs`.`id` = 1
  TRANSACTION (1.5ms)  COMMIT
 => true

3.0.0 :023 > b.attributes
 => 
{"id"=>1,
 "title"=>"First Blog by luwang",
 "content"=>"Blog Content",
 "is_public"=>true,
 "user_id"=>2,
 "created_at"=>Wed, 07 Aug 2024 08:45:18.952964000 UTC +00:00,
 "updated_at"=>Thu, 08 Aug 2024 09:26:32.248804000 UTC +00:00}

3.0.0 :022 > b.changed?
 => false
```

## ActiveRecord 中的 ! 方法

会抛出异常

- `save!` 模型校验不通过 ActiveRecord::RecordInvalid
- `create!`
- `update!`

## 异常

- `ActiveRecord::RecordNotFound`
- `ActiveRecord::UnknownAttributeError` 未知属性错误，例如 new 的时候给没有的字段
- `ActiveRecord::RecordInvalid` 验证没通过，并使用 `!`
- `ActiveRecord::Rollback`

## model 自定义属性

现在处理标签是在 controller 下定义了一个方法，标签从 `tags = params[:article][:tags]` 中获取的

可以在 `app/models/blog.rb` 添加自定义属性/方法

```rb
class Blog < ApplicationRecord

  validates :title, presence: { message: "标题不能为空" }
  validates :content, presence: { message: "内容不能为空" }
  validates :user_id, presence: { message: "用户不能为空" }

  belongs_to :user
  has_many :blogs_tags, class_name: "BlogsTags"
  has_many :tags, through: :blogs_tags

  # 自定义属性/方法
  # 这个方法用于处理从表单提交过来的标签字符串并将其转换成 Tag 对象的集合
  def tags_string=(one_tags)
    transaction do  # 用到了事务，后面讲
      self.tags.clear  # 清空当前关联的所有标签，重新设定，self 就还是 blog

      one_tags.split(",").each do |tag|
        tag_obj = Tag.find_or_create_by!(title: tag.strip)
        self.tags << tag_obj
      rescue ActiveRecord::RecordInvalid => e
        errors.add(:tags_string, "Invalid tag: #{e.record.title}")
        raise ActiveRecord::Rollback
      end
    end
  end
end
```

然后在 view 中使用 ``

```rb
<div class="form-group">
  <label for="blog_tags_string">标签</label>
  <input type="text" name="blog[tags_string]" class="form-control" placeholder="标签（以 , 分隔）" 
    value="<%= @blog.tags.map(&:title).join(', ') %>"
>
</div>
```

这样 tags_string 就和其他参数一样，放在了 blog 里面了

`app/controllers/blogs_controller.rb` 中就可以把 update_tags 方法删除，然后直接在 blog 参数中获取

```rb
def create
  @blog = Blog.new(blog_params)
  @blog.user = current_user
  if @blog.save
    flash[:notice] = "博客创建成功"
    redirect_to blogs_path
  else
    flash[:notice] = "博客创建失败"
    render action: :new, status: 422
  end
end

def update
  @blog = Blog.find(params[:id])
  @blog.attributes = blog_params
  if @blog.save
    flash[:notice] = "博客更新成功"
    redirect_to blog_path(@blog)
  else
    flash[:notice] = "博客更新失败"
    render action: :new, status: 422
  end
end

private
def blog_params
  params.require(:blog).permit(:title, :content, :is_public, :tags_string)
end
```

如果是已有属性，将会进行覆盖默认行为

```rb
class Blog < ApplicationRecord
  def content= one_content
    write_attribute :content, one_content * 2 # 不使用 self 就不会一直循环递归
  end

  attr_accessor :custom_field  # 声明自定义属性
  
end
```

**声明自定义属性** 读操作

例如汉语拼音，先 Gemfile 中添加 `gem 'chinese_pinyin`

```rb
class Blog < ActiveRecord::Base
  def title_pinyin
    Pinyin.t self.title
  end
end
```

## transaction 事务

数据库中的事务通过事务管理器进行管理和控制，确保数据的完整性和可靠性。

比如支付状态的成功、添加积分等

`save`、`destroy` 被自动封装在一个事务中，包括他们所对应的回调

事务是基于 database 层级的，不是针对一个表

```rb
Blog.transaction do  # 模型 Blog、实例 @blog
  # 增删改查，无所谓对那个模型/实例操作
  blog.save!
end

Blog.transaction do
  rails 'error'
end
```

内部抛出异常，这个事务就会整体进行回滚 rollback，异常仍会继续抛出，需要自己处理

```rb
def create
  begin
    ActiveRecord::Base.transaction do
      @blog = Blog.new(blog_params)
      @blog.user = current_user
      @blog.save!  # 使用 save! 方法，如果保存失败会抛出异常 ActiveRecord::RecordInvalid

      flash[:notice] = "博客创建成功"
      redirect_to blogs_path
    end
  rescue ActiveRecord::RecordInvalid => e
    flash[:notice] = "博客创建失败: #{e.message}"
    render action: :new, status: 422
  end
end
```

可以显式地回滚

```rb
def create
  ActiveRecord::Base.transaction do
    @blog = Blog.new(blog_params)
    @blog.user = current_user

    if @blog.save
      flash[:notice] = "博客创建成功"
      redirect_to blogs_path
    else
      flash[:notice] = "博客创建失败"
      render action: :new, status: 422
      raise ActiveRecord::Rollback  # 回滚事务
    end
  end
end
```

## 模型验证

```rb
class User < ApplicationRecord

  # 模型验证
  validates :username, presence: { message: "用户名不能为空" }
  # validates_presence_of :username, message: "..."
  validates :username, uniqueness: { message: "用户名已存在" }
  # validates_uniqueness_of :username, message: "..."
  validates :password, presence: { message: "密码不能为空" }
  validates :password, length: { minimum: 6, message: "密码长度不能小于6位" }
  validates :email, presence: true, format: { with: URI::MailTo::EMAIL_REGEXP }
  validates :age, presence: true, numericality: { only_integer: true, greater_than: 0 }

end
```

实际不需要在这里写死 message，可以配置国际化

activemodel 提供的 .validates（之前是 activerecord gem）

validates 和 validates_xxx_of 作用是一样的

- acceptance 数据格式
- confirmation 密码重复校验
- format
- inclusion
- length
- numericality
- presence

`#valid?` 和 `#errors`

```rb
user = User.new(name: "Tom")
user.valid? # false
user.errors # [password: error...]
```

之前用户创建 view 中有用到

**自定义 validate**

```rb
class User < ActiveRecord::Base
  validate :shold_start_with_letter

  private
  def shold_start_with_letter
    unless self.username =~ /^[a-z]/i
      errors.add(:username, "should start with letter")
    end
  end
end
```

封装成模块

```rb
class EvilUserValidator < ActiveModel::Validator
  def validate(record)
    if record.username == "Evil"
      record.errors[:base] << "This person is Evil"
    end
  end
end

class User < ActiveRecord::Base
  validates_with EvilUserValidator
end
```

**validations 触发时间**：默认情况下 `save` 触发，指定 on 参数

```rb
class User < ActiveRecord::Base
  validate_uniqueness_of :username, message: "...", on: :create # on: :update
end
```

- create
- update

**跳过 validations**，如果数据库层面做了校验也还是会失败

```rb
user = User.new(name: "Tom")
user.save validate: false
```

这些都不会触发 validations

- `decrement!`
- `decrement_counter`
- `increment!`
- `increment_counter`
- `toggle!`
- `touch`
- `update_all`
- `update_attribute`
- `update_column`
- `update_columns`
- `update_counters`

## Scopes 查询范围

指定默认的查询参数，限定查询的范围

例如已上线的商品 `Product.online.where()`

```rb
class User < ActiveRecord::Base
  scope :evils, -> { where(style: "Evil") }

  # 和定义方法效果一样，只是语法上更方便
  def self.evils
    where(style: "Evil")
  end
end

User.evils.first
User.evils.new  # 指定 style 默认值
```

## Associations 参数

### proc

```rb
class User < ActiveRecord::Base
  has_many :blogs
  has_many :public_blogs, -> { where(is_public: true) },
    class_name: "Blog"
end
```

### has_one

```rb
class User < ActiveRecord::Base
  has_one :latest_blog, -> { order("id desc") },
    class_name: :Blog  # has_one => limit 1
end
```

### self join

```rb
class User < ActiveRecord::Base
  has_many :staffs, class_name: "User", foreign_key: "manager_id"  # 员工的管理者（领导）
  belongs_to :manager, class_name: "User"
end
```

### 约束

```rb
class User < ActiveRecord::Base
  has_many :blogs
end
```

- blogs → Blog
- User → user_id
- User primary key → id
- Blog foreign_key → user_id

### 指定关联参数

```rb
class User < ActiveRecord::Base
  has_many :blogs, class_name: :Blog, primary_key: :id, foreign_key: :user_id
end
```

### dependent

```rb
class User < ActiveRecord::Base
  has_many :blogs, dependent: :destroy
end

user = User.first
user.destroy  # 用户删除了，他之下的博客也会删除
```

### 其他参数

- inverse_of
- validate
- source

### 其他概念

- Polymorphic 多态
- Cache 缓存，看日志
- Join 查询

## Migrations

数据库移植/迁移

activerecord gem 提供

作用

- 采用 Ruby DSL（特定语义语言）的方式来管理数据库的设计模式
- 采用 RDB 模式管理，方便在不同数据库之间使用
- 支持版本管理和团队协作
- 支持数据库回滚 rollback

使用

```sh
rails g model  # 生成 model 时会在 db/migrate 下新建 create 迁移文件 
rails g migration  # 独立生成迁移文件，其他对数据库操作时可使用
```

生成的迁移文件，以时间戳开头，每次执行 `rake db:migrate` 命令 rails 都会自动检测并且执行没有执行过的创建和迁移数据库操作

```sh
$ rails g migration add_users_style_column  # 文件名，有意义即可
      invoke  active_record
      create    db/migrate/20240808134529_add_users_style_column.rb
```

移植文件内容

```rb
class AddUsersStyleColumn < ActiveRecord::Migration[7.0]
  def change
    add_column :users, :style, :string
    # 也能操作 Model 等，例如
    # User.all... # 遍历之后对它进行操作
  end
end
```

执行 `rake db:migrate` 命令

其他命令 `rake --tasks`（或 `-T`）查看

```sh
rake db:migrate  # 让迁移文件保持最新
rake db:rollback  # 回滚一个版本
rake db:rollback STEP=4 # 回滚4次
rake db:migrate:status  # 查看迁移状态（down 是没执行，up 是执行过了）
rake db:version  # 最新移植文件对应时间戳
```

自定义迁移和回滚（对于一些可能猜不到回滚会怎样的操作）

```rb
class MigrateFile < ActiveRecord::Migration[7.0]

  def on
  end

  def down
  end

end
```

*永远不要修改已经提交的 migrations*，执行过后它不会再执行了

db/schema.rb rails 自动维护的，映射的所有的 migrate 目录下移植文件，不能去修改

## Callback 回调

在增删改查的操作上添加的回调事件，在执行增删改查的时候会同步触发这些逻辑

回调触发分类

- creating an object（新建记录，而不是只能调用 create 方法 new + save 也行，下面同理）
- updating an object
- destroying an object
- finding an object

### create 回调

- before_validation
- after_validation
- before_save
- around_save
- before_create
- around_create
- after_create
- after_save
- after_commit/after_rollback → 成功/异常

会依次按照这些进行执行

validations 在回调之前执行，可以 `before_validation` 在 validations 运行前修复数据

`before_save`

```rb
class User < ActiveRecord::Base
  # before_save do
  #  self.username = self.username.downcase
  #end
  before_save :update_username  # 或者这样做
  # before_save ... # 一样的回调就依次执行

  private
  def update_username
    self.username = self.username.downcase
  end
end
```

回调方法区分

- before_save 保存数据
- before_create 记录创建
- before_update 修改记录

在 after_save 中调用 `self.save` 这样会造成死循环

### update 回调

- before_validation
- after_validation
- before_save
- around_save
- before_update
- around_update
- after_update
- after_save
- after_commit/after_rollback

### destroy 回调

- before_destroy
- around_destroy
- after_destroy
- after_commit/after_rollback

### find 回调

- after_find
- after_initialize

### 触发 callback 的方法

- create
- create!
- decrement!
- destroy
- destroy!
- destroy_all
- increment!
- save
- save!
- save(validate: false)
- toggle!
- update_attribute
- update
- update!
- valid?

### 触发 after_find 的方法

- all
- first
- find
- find_by
- find_by_xxx
- find_by_xxx!
- find_by_sql
- last

### 跳过 callback 的触发方法

- decrement
- decrement_counter
- delete
- delete_all
- increment
- increment_counter
- toggle
- touch
- update_column
- update_columns
- update_all
- update_counters

### callback 参数

```rb
class User < ActiveRecord::Base
  before_save :update_username, unless: proc { |user| user.evil? }
  # before_save :update_username, if: proc { |user| !user.evil? }
end
```

### transaction

所有的回调都被自动放在了事务中

如果一个 `before_xxx` 回调 **返回值** 为 `false` 或认为抛出异常那么当前事务就会自动进行回滚（`nil` 不会，after 因为已经进入数据库了，所以只有认为抛出异常才会回滚）
