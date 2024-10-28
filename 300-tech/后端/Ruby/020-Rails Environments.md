---
title: 020-Rails Environments
date: 2024-08-09T02:59:00+08:00
updated: 2024-08-21T10:32:31+08:00
dg-publish: false
---

[Rails的加载](https://eggman.tv/players/how-rails-loading-and-executing)

## Rails 的加载和启动

rails 命令 `bin/rails` 文件

Ruby 脚本

```rb
#!/usr/bin/env ruby
APP_PATH = File.expand_path("../config/application", __dir__)
require_relative "../config/boot"
require "rails/commands"
```

`../config/application`

```rb
require_relative "boot"

require "rails/all"

# Require the gems listed in Gemfile, including any gems
# you've limited to :test, :development, or :production.
Bundler.require(*Rails.groups)

module Circles
  class Application < Rails::Application
    # Initialize configuration defaults for originally generated Rails version.
    config.load_defaults 7.0

    # Configuration for the application, engines, and railties goes here.
    #
    # These settings can be overridden in specific environments using the files
    # in config/environments, which are processed later.
    #
    # config.time_zone = "Central Time (US & Canada)"
    # config.eager_load_paths << Rails.root.join("extras")
    # config.assets.compile = true
  end
end

```

`boot`

```rb
ENV["BUNDLE_GEMFILE"] ||= File.expand_path("../Gemfile", __dir__)

require "bundler/setup" # Set up gems listed in the Gemfile.
require "bootsnap/setup" # Speed up boot time by caching expensive operations.
```

`../Gemfile`

```
source "https://rubygems.org"
git_source(:github) { |repo| "https://github.com/#{repo}.git" }

ruby "3.0.0"

# Bundle edge Rails instead: gem "rails", github: "rails/rails", branch: "main"
gem "rails", "~> 7.0.2", ">= 7.0.2.3"

# ...

group :development, :test do
  # See https://guides.rubyonrails.org/debugging_rails_applications.html#debugging-with-the-debug-gem
  gem "debug", platforms: %i[ mri mingw x64_mingw ]
end

group :development do
  # Use console on exceptions pages [https://github.com/rails/web-console]
  gem "web-console"

  # Add speed badges [https://github.com/MiniProfiler/rack-mini-profiler]
  # gem "rack-mini-profiler"

  # Speed up commands on slow machines / big apps [https://github.com/rails/spring]
  # gem "spring"
end

group :test do
  # Use system testing [https://guides.rubyonrails.org/testing.html#system-testing]
  gem "capybara"
  gem "selenium-webdriver"
  gem "webdrivers"
end
```

`config.ru`

```ru
# This file is used by Rack-based servers to start the application.

require_relative "config/environment"

run Rails.application
Rails.application.load_server
```

`config/environment.rb`

```rb
# Load the Rails application.
require_relative "application"

# Initialize the Rails application.
Rails.application.initialize!
```

两种启动方式

- rails
- Rack

## Rails 的加载

代码层的加载上面

## Rails 环境

`config/environment.rb` → `config/application.rb`

- development
- production
- test

`config/environments` 目录下的配置会覆盖 `config/application.rb` 中的配置

不同环境的区分，例如 Gemfile、database.yml 等

配置选项

```rb
config.time_zone = "Central Time (US & Canada)"
```

可以通过 `rake time:zones:all` 查看所有支持的时区

```rb
config.active_record.migration_error = :page_load
```

配置 Gem 的

重要配置选项

```rb
config.after_initialize do
  ActionView::Base.sanitized_allowed_tags.delete 'div'
end

config.autoload_once_paths += ["/new/path"]
config.autoload_paths += ["/new/path"]
config.cache_classes = true
config.cache_store = :memory_store
config.consider_all_requests_local = true
config.filter_parameters += [:password]
```

中间件 middleware

在 rails 接收请求之前的前置处理逻辑

```rb
config.middleware.use Magical::Unicorns
config.middleware.insert_before Rack::Head, Magical::Unicorns
config.middleware.insert_after Rack::Head, Magical::Unicorns
```

active_record

```rb
config.active_record.pluralize_table_names = true
config.active_record.table_name_prefix = ""
config.active_record.record_timestamps = true
```

action_controller

```rb
config.action_controller.asset_host = ""
config.action_controller.perform_caching = true
config.action_controller.allow_forgery_protection = true
config.action_controller.permit_all_parameters = false
```

其他

```rb
config.action_dispatch.*
config.action_view.*
config.action_mailer.*
config.active_support.*
```

自定义配置 `config.x.*`

```rb
config.x.payment_processing.schedule = :daily
config.x.payment_processing.retries = 3
config.x.super_debugger = true
```

使用

```rb
Rails.configuration.x.payment_processing.schedule
Rails.configuration.x.payment_processing.retries
Rails.configuration.x.super_debugger

Rails.configuration.x.super_debugger.not_set  # nil
```

## Staging 运行环境

线上测试环境，不给用户使用，非常接近生产环境

新建 `config/environments/staging.rb`

复制 production.rb 的内容

```rb
require "active_support/core_ext/integer/time"

Rails.application.configure do
  # Settings specified here will take precedence over those in config/application.rb.

  # Code is not reloaded between requests.
  config.cache_classes = true

  # Eager load code on boot. This eager loads most of Rails and
  # your application in memory, allowing both threaded web servers
  # and those relying on copy on write to perform better.
  # Rake tasks automatically ignore this option for performance.
  config.eager_load = true

  # Full error reports are disabled and caching is turned on.
  config.consider_all_requests_local       = false
  config.action_controller.perform_caching = true

  # Ensures that a master key has been made available in either ENV["RAILS_MASTER_KEY"]
  # or in config/master.key. This key is used to decrypt credentials (and other encrypted files).
  # config.require_master_key = true

  # Disable serving static files from the `/public` folder by default since
  # Apache or NGINX already handles this.
  config.public_file_server.enabled = ENV["RAILS_SERVE_STATIC_FILES"].present?

  # Compress CSS using a preprocessor.
  # config.assets.css_compressor = :sass

  # Do not fallback to assets pipeline if a precompiled asset is missed.
  config.assets.compile = false

  # Enable serving of images, stylesheets, and JavaScripts from an asset server.
  # config.asset_host = "http://assets.example.com"

  # Specifies the header that your server uses for sending files.
  # config.action_dispatch.x_sendfile_header = "X-Sendfile" # for Apache
  # config.action_dispatch.x_sendfile_header = "X-Accel-Redirect" # for NGINX

  # Store uploaded files on the local file system (see config/storage.yml for options).
  config.active_storage.service = :local

  # Mount Action Cable outside main process or domain.
  # config.action_cable.mount_path = nil
  # config.action_cable.url = "wss://example.com/cable"
  # config.action_cable.allowed_request_origins = [ "http://example.com", /http:\/\/example.*/ ]

  # Force all access to the app over SSL, use Strict-Transport-Security, and use secure cookies.
  # config.force_ssl = true

  # Include generic and useful information about system operation, but avoid logging too much
  # information to avoid inadvertent exposure of personally identifiable information (PII).
  config.log_level = :info

  # Prepend all log lines with the following tags.
  config.log_tags = [ :request_id ]

  # Use a different cache store in production.
  # config.cache_store = :mem_cache_store

  # Use a real queuing backend for Active Job (and separate queues per environment).
  # config.active_job.queue_adapter     = :resque
  # config.active_job.queue_name_prefix = "circles_production"

  config.action_mailer.perform_caching = false

  # Ignore bad email addresses and do not raise email delivery errors.
  # Set this to true and configure the email server for immediate delivery to raise delivery errors.
  # config.action_mailer.raise_delivery_errors = false

  # Enable locale fallbacks for I18n (makes lookups for any locale fall back to
  # the I18n.default_locale when a translation cannot be found).
  config.i18n.fallbacks = true

  # Don't log any deprecations.
  config.active_support.report_deprecations = false

  # Use default logging formatter so that PID and timestamp are not suppressed.
  config.log_formatter = ::Logger::Formatter.new

  # Use a different logger for distributed setups.
  # require "syslog/logger"
  # config.logger = ActiveSupport::TaggedLogging.new(Syslog::Logger.new "app-name")

  if ENV["RAILS_LOG_TO_STDOUT"].present?
    logger           = ActiveSupport::Logger.new(STDOUT)
    logger.formatter = config.log_formatter
    config.logger    = ActiveSupport::TaggedLogging.new(logger)
  end

  # Do not dump schema after migrations.
  config.active_record.dump_schema_after_migration = false
end
```

修改 `config/database.yml` 添加

```rb
staging:
  <<: *default
  database: circles_staging
```

运行

```sh
bin/rails s -p 4001 -e staging
```

添加自定义配置文件 `config/application.yml`

```yml
development: &default
  domain: localhost

production:
  domain: circles.io

test:
  <<: *default

staging:
  <<: *default
  domain: staging.circles.io
```

`config/initializers/app_config.rb`

```rb
require 'psych'

# Enable aliases in YAML parsing
Psych::Parser.new(yaml: { aliases: true })

yaml_data = File.read("#{Rails.root}/config/application.yml")
AppConfig = Psych.safe_load(yaml_data, aliases: true)[Rails.env]
```

在终端测试

```sh
$ bin/rails c
Loading development environment (Rails 7.0.8.4)
3.0.0 :001 > AppConfig
 => {"domain"=>"localhost"}
3.0.0 :002 > AppConfig["domain"]
 => "localhost"
```

也可以使用第三方 Gem 让 hash 能够变成可访问链形式调用（`AppConfig.domain`）
