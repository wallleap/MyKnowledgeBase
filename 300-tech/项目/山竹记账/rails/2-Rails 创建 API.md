Rails 路由，namespace

```rb
# routes.rb
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

运行
```zsh
bin/rails routes
```

如何实现分页
先生成数据库表
```zsh
bin/rails g model ValidationCode email:string kind:int used:bool used_at: datetime
```

生成的 `***_create_validation_codes.rb` ，修改一下 kind 字段，给定一个默认值
```rb
……
	t.integer :kind, default: 1, null: false
……
```