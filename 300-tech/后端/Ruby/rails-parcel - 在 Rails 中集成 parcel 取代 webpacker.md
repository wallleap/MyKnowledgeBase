---
title: rails-parcel - 在 Rails 中集成 parcel 取代 webpacker
date: 2024-08-16T10:06:00+08:00
updated: 2024-08-21T10:32:32+08:00
dg-publish: false
source: //ruby-china.org/topics/39542
---

parcel 是一个极速零配置 Web 应用的打包工具，可以取代 webpacker。

写了一个简单的 gem，用 parceljs 来代替 webpacker 集成在 rails。

够用但不完善，暂时能满足我现在的项目需求。

`README` 摘要如下：

## rails-parcel

Gem integrates parcel JS module bundler into your Rails application. It is inspired by gems webpacker.

## Installation

Add this line to your application's Gemfile:

```
gem 'rails-parcel'
```

And then execute:

```
$ bundle install
```

Or install it yourself as:

```
$ gem install rails-parcel
```

Then run

```
$ bin/rails g parcel:install
```

Usage

```
$ bin/rake parcel:clobber
$ bin/rake parcel:watch
$ bin/rake parcel:serve
```

详情见：

- [https://rubygems.org/gems/rails-parcel](https://rubygems.org/gems/rails-parcel)
- [http://github.com/bkbabydp/rails-parcel](https://github.com/bkbabydp/rails-parcel)

集成了 `standard-version` 工具链作为 `git commit` 规范要求。

欢迎挑刺和建议。
