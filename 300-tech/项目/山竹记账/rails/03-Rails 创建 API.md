---
title: 032-Rails 创建 API
date: 2023-05-03 22:07
updated: 20243-510- 20:4907 11:33
---

Rails 路由，namespace

```rb
# config/routes.rb
Rails.application.routes.draw do
	namespace :api do
		namespace :v1 do
			# /api/v1 开头
			resources :validation_codes, only: [:create]
			resources :session, only: [:create, :destroy]
			resources :me, only: [:show]
			resources :items
			resources :tags
		end
	end
end
```

运行下面命令能自动生成

```sh
bin/rails routes
```

## 生成获取验证码数据库

生成数据库表

```sh
bin/rails g model ValidationCode email:string kind:int code:string used:bool used_at: datetime
```

生成的 `***_create_validation_codes.rb` ，修改一下 kind 字段，给定一个默认值，修改 code 字段，限制长度（最好不要给当前的 6，留个余地）

```rb
……
	t.integer :kind, default: 1, null: false
	t.string :code, limit: 64
……
```

运行创建表

```sh
bin/rails db:migrate
```

(如果错了，可以先回滚然后修改之后再创建)

运行命令创建 Controller

```sh
bin/rails g controller validation_codes create
```

修改生成的 `app/controllers/validation_codes_controller.rb`

```rb
class Api::V1::ValidationCodeController < ApplicationController
  def create
    head 201
  end
end
```

在 `controllers` 下新建目录 `api/v1`，并将上面的文件移动到这个目录下（由于之前不知道需要控制版本，后面的就使用正确的命令）

启动服务

```sh
bin/rails s
```

在另一个命令行窗口测试

```sh
curl -X POST http://127.0.0.1:3000/api/v1/validation_codes -v
```

可以看到状态码为 201，修改 `head 202` 之后就可以看到了

## 创建 items

创建数据库

```sh
bin/rails g model item user_id:bigint amount:integer notes:text tags_id:bigint happen_at:datetime 
```

修改生成的文件中的字段

```rb
#...
t.bigint :user_id
t.integer :amount
t.text :note
t.bigint :tags_id, array: true
t.datetime :happen_at
#...
```

运行

```sh
bin/rails db:migrate
```

创建 Controller

```sh
bin/rails g controller Api::V1::Items
```

修改这个生成的文件 `items_controller.rb`

```rb
class Api::V1::ItemsController < ApplicationController
  def index
    #Item.all #获取所有
  end
  def create
    item = Item.new amount: 1
    if item.save
      render json: {resource: item}
    else
      render json: {errors: item.errors}
    end
  end
end
```

**如何实现分页**

方案 1: 使用 page 和 per_page 参数，见 kaminari 或 pagy 库

方案 2: 使用 start_id 和 limit 参数，需要 id 是自增数字

在 GitHub 搜到相应的库，查看使用方法，这里使用 kaminari

修改根目录下文件 Gemfile

在 group 的上方添加

```
gen 'kaminari'
```

运行

```sh
bundle install
# 可以省略 install，即 bundle
# 罗嗦模式 bundle --verbose
```

运行 `bin/rails s` 重新启动服务这样上面那个文件就可以直接使用 page

```rb
def index
  items = Item.page params[:page]
  #如果想在这里约束每页可以 items = Item.page(params[:page]).per(100)
  render json: {resources:items}
end
```

运行 `bin/rails g kaminari:config` 生成配置，并对生成的文件进行修改

之后运行命令查看获取结果

```sh
curl http://127.0.0.1:3000/api/v1/items\?page\=2
```

需要总条数

```rb
def index
  items = Item.page(params[:page]).per(100)
  render json: {resouces: items, pager: {
    page: params[:page],
    per_page: 100,
    count: Item.count
  }}
end
```
