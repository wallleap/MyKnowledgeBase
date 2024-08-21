---
title: 010-Rails 简介
date: 2024-08-04T08:05:00+08:00
updated: 2024-08-21T10:32:30+08:00
dg-publish: false
---

## Ruby on Rails

[Ruby on Rails — A web-app framework that includes everything needed to create database-backed web applications according to the Model-View-Controller (MVC) pattern.](https://rubyonrails.org/)

- Web application framework Web 开发框架
- Written in Ruby
- 2004/07
- Shipped with OSX Leopard
- Written by DHH

## 谁在用

- Hulu
- 暴走漫画
- KICKSTARTER
- GroupOn
- FreeWheel
- Yotta
- 友盟
- Twitch
- 无数的创业公司以及国内很多大公司的新项目

## 全栈框架

full-stack framework

Rendering HTML templates, updating databases, sending and receiving emails, maintaining live pages via WebSockets, enqueuing jobs for asynchronous work, storing uploads in the cloud, providing solid security protections for common attacks. Rails does it all and so much more.

- Model → Active Records
- Controllers → Action Controller、Action Dispatch
- View → Action Views
- Route → Action Dispatch

## Rails 信条

[Ruby on Rails — The Rails Doctrine](https://rubyonrails.org/doctrine)

## 快速开始

- 指南/教程：[Ruby on Rails Guides](https://guides.rubyonrails.org/)
- 仓库：[rails/rails: Ruby on Rails](https://github.com/rails/rails)
- API 文档：[Ruby on Rails API](https://api.rubyonrails.org/)
- 更新日志：[Ruby on Rails — Blog](https://rubyonrails.org/blog/)
- 默认前端框架：[HTML Over The Wire | Hotwire (hotwired.dev)](https://hotwired.dev/)

其他教程

- [關於 Rails 實戰聖經 - Rails 實戰聖經 (ihower.tw)](<https://ihower.tw/rails/>
- [《应用 Rails 进行敏捷 Web 开发》](http://book.douban.com/subject/1839273/) 中文已更新至第四版，使用 Rails 3.0。 英文版本已经更新到 Agile Web Development with Rails 5，使用 Rails 5.0，书中部分内容已经重写，英语好的还是建议阅读 Rails 5.0 英文版。
- [《Rails 实践》](http://rails-practice.com/)
- [《Ruby on Rails 实战圣经 Rails 3 Edition》](http://ihower.tw/rails3/index.html)
- [《Ruby on Rails 实战圣经 Rails 4 + Ruby 2.1 Edition》](http://ihower.tw/rails4/) 在线中文资料，作者 [](https://twitter.com/ihower)[_@_ihower](https://ruby-china.org/ihower "@ihower")，[博客](http://ihower.tw/blog/)。
- [《Ruby on Rails Tutorial》](http://ruby.railstutorial.org/) [在线中文版](http://railstutorial-china.org/)，翻译：[](https://twitter.com/andor_chen)[_@_andor_chen](https://ruby-china.org/andor_chen "@andor_chen")
- [《Objects on Rails》](http://objectsonrails.com/)

## 快速创建一个 Rails 项目

```sh
rails -v # 确保已经安装 rails
rails new testproject
```

最后会运行 `bundle install`，国内网络可能安装不了，<kbd>Ctrl</kbd> + <kbd>c</kbd> 中断，然后修改下 `Gemfile`

```
source 'https://ruby.taobao.org'
```

接着重新运行 `bundle` 安装依赖

启动服务

```sh
rails server -p 3000
```

## 设计哲学

- Follow web standards and HTTP 遵循网络标准和 HTTP 标准
- CoC (Convention over Configuration) 约束大于配置原则
- MVC (Model-View-Controller) 模型 - 视图 - 业务逻辑（怎么组织代码）
- DRY 不要重复造轮子
- Software Engineering Patterns
- Assets 静态资源放置
- Restful 一种 API 设计风格
- Testing 测试
- Deployment 部署
- Automation 自动化

## 敏捷开发

Agile Development

提升开发效率

快速迭代

创意或想法 → 线上 → 修改 → 线上 → 下次迭代

从本地开发到线上

- 开发环境：Development 本地开发、Test 测试、Production 线上生产环境、Staging
- 测试
- 配置
- 部署

页面和资源

- 动态页面：
	- 路由 Routes
		- HTTP 请求方式：GET/POST/PUT/DELETE
		- 自定义路由
		- 易于重构 reactor
	- Restful
	- 继承 Inheritance：代码重用
		- Filter 过滤器
			- before_filter：例如登录验证
			- around_filter
			- after_filter：例如记录日志
	- Cache 缓存
		- Page Cache 页面级缓存
		- Fragment Cache 片段缓存
	- Response 相应格式
		- HTML
		- JSON
		- XML
- 静态资源：
	- Asset Pipeline 管道
	- Minify、Combine
	- Webpack、Babel

**ActiveRecord**

Powerful ORM（映射）

```rb
User.find(2)
User.find_by(email: 'luwang@oicode.cn')
User.where(membership: 'Gold').page(2)
  .per_page(10).order("id desc").limit(10)
user.blogs
# etc.
```

Triggers 触发器（回调）

- `before_save`
- `after_save`
- `after_destroy`
- `after_create`
- etc.

从开发到测试

- 单元测试：rspec、minitest
- 冒烟测试、集成测试：cucumber、selenium

脚本和自动化

```sh
rails c # console
rails g
rails s
```

rake 任务

```sh
rake routes
rake db:create
rake db:migrate
```

自定义 rake 任务

库

- gem
- bundle
- rails 是很多 gems 的组合
- 创建自己的 gem
