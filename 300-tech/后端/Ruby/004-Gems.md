---
title: 004-Gems
date: 2024-07-16T03:41:00+08:00
updated: 2024-08-21T10:32:30+08:00
dg-publish: false
---

## Gems

- Ruby 管理工具，类似于 Linux 的 rpm、deb 等
- 社区中 Gem 包涵盖了各方面开发所需要的工具和组件
- 通过 `gem` 命令安装

```sh
# 列出本地安装的 Gems 和版本
gem list
gem install
gem uninstall
gem sources
```

可以切换镜像地址 RubyChina

## bundle gem

- gem 本身会有版本依赖问题
- bundle 是一个 gem，被用来管理其他 gems
- bundle 类似于 Linux 的 yum、apt 等，是一个 gems 关系管理工具

```sh
gem install bundler
vim Gemfile
bundle install
bundle update
bundle show
bundle list
```

### bundle exec

`rake -T` 执行后发现安装版本和依赖版本不一致，并提示需要先执行 `bundle exec`

`which rake` 查看在哪里，然后使用编辑器打开查看

发现最后 load 了 `Gem.bin_path`，接着在 `rubygems.rb` 中找到

它会查找可执行程序位置

这样运行就没问题

```sh
bundle exec rake -T
```

运行

```sh
bundle exec env
```

查看环境变量，它添加了 `RUBYOPT` 它会引入 `rbundler/setup` 模块

### 独立使用 bundler

```sh
mkdir ruby_project
cd ruby_project
bundle init
vim Gemfile
```

修改文件内容

```
source "https://ruby.taobao.org"

gem "activesupport"
```

运行 `bundle` 或 `bundle install`

```sh
vim boot.rb
```

内容

```rb
ENV['BUNDLE_GEMFILE'] ||= File.expand_path('../Gemfile', __FILE__)

require 'bundler/setup'
```

```sh
vim main.rb
```

内容

```rb
require File.expand_path('../boot', __FILE__)
require 'active_support/all'

p 1.days
```

运行 `ruby main.rb` 执行

## 创建测试发布自己的 Gem

### 创建

```sh
bundle gem gem_test
cd gem_test
```

目录

```
|-bin/
  |-console
  |-setup
|-lib/
  |-gem_test
    |-version.rb
  |-gem_test.rb
|-gem_test.gemspec # 修改信息 例如包名、描述等
|-Gemfile
|-Rakefile
```

在 `lib/gem_test` 下新建 `hi.rb`

```rb
module GemTest
  class Hi
    def self.hi
      'hi'
    end
  end
end
```

编辑 `lib/gem_test.rb`

```rb
require "gem_test/hi"
```

### 在 irb 中测试 Gem 和重新加载

```sh
irb
$: << File.expand_path('./lib')
require 'gem_test'
GemTest::Hi.hi
```

修改文件之后

```sh
def reload(gem)
  $".grep(Regexp.new(gem)).each { |file| load file }
end
reload 'gem_test' #  强制加载
GemTest::Hi.hi
```

### 在 rails 中结合 bundle 测试 gem 和实现热加载

在 rails 项目中

```sh
vim Gemfile
```

添加内容

```
gem 'gem_test', path: File.expand_path('../../gem_test', __FILE__)
```

运行 `bundle` 安装

进入控制台测试

```sh
rails c
GemTest::Hi.hi
```

也可以在网页中显示出来

```sh
vim config/development.rb
```

添加内容

```rb
RELOAD_GEM = lambda do |gem|
  $".grep(Regexp.new(gem).each { |file| load file })
end

# 在下面的 configure 里
config.reload_classes_only_on_change = false
config.to_prepare do
  p 'RELOAD_GEM...'
RELOAD_GEM.call('gem_test')
end
```

### 发布

<https://rubygems.org> 注册了帐号

进入自己的 gem 目录，并且 Git 提交到了公共仓库，每次修改了需要更改版本 `bin/gem_test/VERSION.rb`

```sh
rake build # 打包
rake release
```
