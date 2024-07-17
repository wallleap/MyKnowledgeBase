---
title: 001-Ruby on Rails
date: 2024-06-06 16:22
updated: 2024-06-06 16:40
---

官网：<https://rubyonrails.org/>

教程：<https://guides.rubyonrails.org/>

GitHub：<https://github.com/rails/rails/>

Ruby on Rails — A web-app framework that includes everything needed to create database-backed web applications according to the Model-View-Controller (MVC) pattern.

全栈框架 **full-stack framework.**

默认使用的数据库为 SQLite

使用 Rails 前需要先安装 Ruby 和 SQLite3

安装 RVM 用于管理 Ruby 版本

```sh
brew install rvm
```

通过 RVM 安装使用 Ruby 相应的版本

```sh
rvm install 3.0.0
rvm use 3
ruby --version
# ruby 3.0.0p0
```

安装 SQLite3

```sh
brew install sqlite3
sqlite3 --version
```

使用 Gem 下载 Rails

```sh
gem install rails
```

查看 Rails 版本

```sh
rails --version
# Rails 7.0.8.1
```

RESTful 关键词

index 列表 show 单个

new create

edit update

destroy

## 创建项目

```sh
rails new <app-name>
cd myapp
```

| File/Folder               | Purpose                                                                                                                                                                                                                                                                        |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| app/                      | Contains the controllers, models, views, helpers, mailers, channels, jobs, and assets for your application. You'll focus on this folder for the remainder of this guide.                                                                                                       |
| bin/                      | Contains the `rails` script that starts your app and can contain other scripts you use to set up, update, deploy, or run your application.                                                                                                                                     |
| config/                   | Contains configuration for your application's routes, database, and more. This is covered in more detail in [Configuring Rails Applications](https://guides.rubyonrails.org/configuring.html).                                                                                 |
| config.ru                 | Rack configuration for Rack-based servers used to start the application. For more information about Rack, see the [Rack website](https://rack.github.io/).                                                                                                                     |
| db/                       | Contains your current database schema, as well as the database migrations.                                                                                                                                                                                                     |
| Dockerfile                | Configuration file for Docker.                                                                                                                                                                                                                                                 |
| Gemfile  <br>Gemfile.lock | These files allow you to specify what gem dependencies are needed for your Rails application. These files are used by the Bundler gem. For more information about Bundler, see the [Bundler website](https://bundler.io/).                                                     |
| lib/                      | Extended modules for your application.                                                                                                                                                                                                                                         |
| log/                      | Application log files.                                                                                                                                                                                                                                                         |
| public/                   | Contains static files and compiled assets. When your app is running, this directory will be exposed as-is.                                                                                                                                                                     |
| Rakefile                  | This file locates and loads tasks that can be run from the command line. The task definitions are defined throughout the components of Rails. Rather than changing `Rakefile`, you should add your own tasks by adding files to the `lib/tasks` directory of your application. |
| README.md                 | This is a brief instruction manual for your application. You should edit this file to tell others what your application does, how to set it up, and so on.                                                                                                                     |
| storage/                  | Active Storage files for Disk Service. This is covered in [Active Storage Overview](https://guides.rubyonrails.org/active_storage_overview.html).                                                                                                                              |
| test/                     | Unit tests, fixtures, and other test apparatus. These are covered in [Testing Rails Applications](https://guides.rubyonrails.org/testing.html).                                                                                                                                |
| tmp/                      | Temporary files (like cache and pid files).                                                                                                                                                                                                                                    |
| vendor/                   | A place for all third-party code. In a typical Rails application this includes vendored gems.                                                                                                                                                                                  |
| .dockerignore             | This file tells Docker which files it should not copy into the container.                                                                                                                                                                                                      |
| .gitattributes            | This file defines metadata for specific paths in a git repository. This metadata can be used by git and other tools to enhance their behavior. See the [gitattributes documentation](https://git-scm.com/docs/gitattributes) for more information.                             |
| .gitignore                | This file tells git which files (or patterns) it should ignore. See [GitHub - Ignoring files](https://help.github.com/articles/ignoring-files) for more information about ignoring files.                                                                                      |
| .ruby-version             | This file contains the default Ruby version.                                                                                                                                                                                                                                   |
| app/                      | Contains the controllers, models, views, helpers, mailers, channels, jobs, and assets for your application. You'll focus on this folder for the remainder of this guide.                                                                                                       |
| bin/                      | Contains the rails script that starts your app and can contain other scripts you use to set up, update, deploy, or run your application.                                                                                                                                       |
| config/                   | Contains configuration for your application's routes, database, and more. This is covered in more detail in Configuring Rails Applications.                                                                                                                                    |
| config.ru                 | Rack configuration for Rack-based servers used to start the application. For more information about Rack, see the Rack website.                                                                                                                                                |
| db/                       | Contains your current database schema, as well as the database migrations.                                                                                                                                                                                                     |
| Dockerfile                | Configuration file for Docker.                                                                                                                                                                                                                                                 |
| Gemfile                   |                                                                                                                                                                                                                                                                                |
| Gemfile.lock              | These files allow you to specify what gem dependencies are needed for your Rails application. These files are used by the Bundler gem. For more information about Bundler, see the Bundler website.                                                                            |
| lib/                      | Extended modules for your application.                                                                                                                                                                                                                                         |
| log/                      | Application log files.                                                                                                                                                                                                                                                         |
| public/                   | Contains static files and compiled assets. When your app is running, this directory will be exposed as-is.                                                                                                                                                                     |
| Rakefile                  | This file locates and loads tasks that can be run from the command line. The task definitions are defined throughout the components of Rails. Rather than changing Rakefile, you should add your own tasks by adding files to the lib/tasks directory of your application.     |
| README.md                 | This is a brief instruction manual for your application. You should edit this file to tell others what your application does, how to set it up, and so on.                                                                                                                     |
| storage/                  | Active Storage files for Disk Service. This is covered in Active Storage Overview.                                                                                                                                                                                             |
| test/                     | Unit tests, fixtures, and other test apparatus. These are covered in Testing Rails Applications.                                                                                                                                                                               |
| tmp/                      | Temporary files (like cache and pid files).                                                                                                                                                                                                                                    |
| vendor/                   | A place for all third-party code. In a typical Rails application this includes vendored gems.                                                                                                                                                                                  |
| .dockerignore             | This file tells Docker which files it should not copy into the container.                                                                                                                                                                                                      |
| .gitattributes            | This file defines metadata for specific paths in a git repository. This metadata can be used by git and other tools to enhance their behavior. See the gitattributes documentation for more information.                                                                       |
| .gitignore                | This file tells git which files (or patterns) it should ignore. See GitHub - Ignoring files for more information about ignoring files.                                                                                                                                         |
| .ruby-version             | This file contains the default Ruby version.                                                                                                                                                                                                                                   |

### 启动 Web Server

```sh
bin/rails server
```

### 添加路由

To get Rails saying "Hello", you need to create at minimum a _route_, a _controller_ with an _action_, and a _view_. 

- A route maps a request to a controller action.
- A controller action performs the necessary work to handle the request, and prepares any data for the view. 
- A view displays data in a desired format.

添加路由到 `config/routes.rb` 文件中

```rb
Rails.application.routes.draw do
  get "/articles", to: "articles#index" # 声明 GET /articles 请求映射到 ArticlesController 的 index action

  # For details on the DSL available within this file, see https://guides.rubyonrails.org/routing.html
end
```

命令 `bin/rails routes` 查看所有的路由

使用命令创建 Controller（一般都带 `s`）

```sh
bin/rails generate controller Articles index --skip-routes
```

使用命令创建 Model（不带 `s`）

```sh
bin/rails generate model Article title:string body:text
```

数据库

```sh
bin/rails db:migrate
```

## 使用 Rails 控制台

```sh
bin/rails console
```
