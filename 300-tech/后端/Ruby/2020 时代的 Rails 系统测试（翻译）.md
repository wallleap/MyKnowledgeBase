---
title: 2020 时代的 Rails 系统测试（翻译）
date: 2024-08-16T09:44:00+08:00
updated: 2024-08-21T10:32:31+08:00
dg-publish: false
---
*本文已获得原作者（Vladimir Dementyev）和 Evil Martians 授权许可进行翻译。原文介绍了在新的 2020 时代，摒弃了基于 Java 的笨重 Selenium 之后，如何在 Rails 下构建基于浏览器的高效系统测试。作者对于系统测试概念进行了详细阐述，演示了具体配置的范例和运行效果，对 Docker 开发环境也有专业级别的涵盖。非常推荐。我的翻译 Blog 链接在这里：[https://xfyuan.github.io/2020/07/proper-browser-testing-in-rails/](https://xfyuan.github.io/2020/07/proper-browser-testing-in-rails/)*

- 原文链接：[System of a test: Proper browser testing in Ruby on Rails — Martian Chronicles, Evil Martians’ team blog](https://evilmartians.com/chronicles/system-of-a-test-setting-up-end-to-end-rails-testing)
- 作者：[Vladimir Dementyev](https://twitter.com/palkan_tula)
- 站点：Evil Martians ——位于纽约和俄罗斯的 Ruby on Rails 开发人员博客。它发布了许多优秀的文章，并且是不少 gem 的赞助商。

*【下面是正文】*

**发现 Ruby on Rails 应用中端到端浏览器测试的最佳实践集合，并在你的项目上采用它们。了解如何摒弃基于 Java 的 Selenium，转而使用更加精简的 Ferrum-Cuprite 组合，它们能直接通过纯 Ruby 方式来使用 Chrome DevTools 的协议。如果你在使用 Docker 开发环境——本文也有涵盖。**

Ruby 社区对于测试饱含激情。我们有数不胜数的 [测试库](https://www.ruby-toolbox.com/search?q=testing)，有成千上万篇关于测试主题的博客文章，甚至为此有一个专门的 [播客](https://player.fm/series/2414463)。更可怕的是，[下载量排在前三名的 Gem](https://rubygems.org/stats) 也都是有关 [RSpec](http://rspec.info/) 测试框架的！

我相信，Rails，是 Ruby 测试兴盛背后的原因之一。这个框架让测试的编写尽可能地成为一种享受了。多数情况下，跟随着 [Rails 测试指南](https://guides.rubyonrails.org/testing.html) 的教导就已足够（起码在刚开始的时候）。但事情总有例外，而在我们这里，就是 _ 系统测试 _。

对 Rail 应用而言，编写和维护系统测试很难被称为“惬意的”。我在处理这个问题上，自 2013 年的第一次 [Cucumber](https://cucumber.io/) 驱动测试算起，已经逐步改进了太多。而今天，到了 2020 年，我终于可以来跟大家分享一下自己当前（关于测试）的设置了。本文中，我将会讨论以下几点：

- 系统测试概述
- 使用 Cuprite 进行现代系统测试
- 配置范例
- Docker 化的系统测试

## 系统测试概述

“系统测试”是 Rails 世界中对于自动化端到端测试的一种通常称谓。在 Rails 采用这个名称之前，我们使用诸如 _ 功能测试 _、*浏览器测试*、甚至 _ 验收测试 _ 等各种叫法（尽管后者在意思上 [有所不同](https://en.wikipedia.org/wiki/Acceptance_testing)）。

如果我们回忆下 [测试金字塔](https://martinfowler.com/bliki/TestPyramid.html)（或者就 [金字塔](http://yakshave.fm/7)），系统测试是处于非常靠上的位置：它们把整个程序视为一个黑盒，通常模拟终端用户的行为和预期。而这就是在 Web 应用程序情况下，为什么我们需要浏览器来运行这些测试的原因（或者至少是类似 [Rack Test](https://github.com/rack/rack-test) 的模拟器）。

我们来看下典型的系统测试架构：

[](https://cdn.evilmartians.com/front/posts/system-of-a-test-setting-up-end-to-end-rails-testing/diagram-25c8249.png)

[![](https://cdn.evilmartians.com/front/posts/system-of-a-test-setting-up-end-to-end-rails-testing/diagram-25c8249.png)

](<https://cdn.evilmartians.com/front/posts/system-of-a-test-setting-up-end-to-end-rails-testing/diagram-25c8249.png>)

我们需要管理至少三个“进程”（它们有些可以是 Ruby 线程）：一个 运行我们应用的 Web 服务端，一个浏览器，和一个测试运行器。这是最低要求。实际情况下，我们通常还需要另一个工具提供 API 来控制浏览器（比如，ChromeDriver）。一些工具尝试通过构建特定浏览器（例如 capybara-webkit 和 PhantomJS）来简化此设置，并提供开箱即用的此类 API，但它们在对真正浏览器的兼容性“竞争”中都败下阵来，没能幸免于难。

当然，我们需要添加少量 Ruby Gem 测试依赖库——来把所有碎片都粘合起来。更多的依赖库会带来更多的问题。例如，[Database Cleaner](https://github.com/DatabaseCleaner/database_cleaner) 在很长时间内都是一个必备的库：我们不能使用事务来自动回滚数据库状态，因为每个线程都是用自己独立的连接。我们不得不针对每张表使用 `TRUNCATE ...` 或 `DELETE FROM ...` 来代替，这会相当慢。我们通过在所有线程中使用一个共享连接解决了这个问题（使用 [TestProf extension](https://test-prof.evilmartians.io/#/active_record_shared_connection)）。[Rails 5.1](https://github.com/rails/rails/pull/28083) 也发布了一个现成的类似功能。

这样，通过添加系统测试，我们增加了开发环境和 CI 环境的维护成本，以及引入了潜在的故障或不稳定因素：由于复杂的设置，[_flakiness_](https://github.com/evilmartians/terraforming-rails/blob/master/guides/flaky.md) 是端到端测试中最常见的问题。而大多数这些 flakiness 来自于跟浏览器的通信。

尽管通过在 Rails 5.1 中引入系统测试简化了浏览器测试，它们仍然需要一些配置才能平滑运行：

- 你需要处理 Web drivers（Rails 假设你使用 Selenium）。
- 你可以自己在容器化环境中配置系统测试（例如，象 [这篇文章中的做法](https://evilmartians.com/chronicles/ruby-on-whales-docker-for-ruby-rails-development)）。
- 配置不够灵活（例如，屏幕快照的路径）

让我们来转到代码层面吧，看看在 2020 时代如何让你的系统测试更有乐趣！

## 使用 Cuprite 进行现代系统测试

默认情况下，Rails 假设你会使用 [Selenium](https://www.selenium.dev/) 来跑系统测试。Selenium 是一个实战验证过的 Web 浏览器自动化软件。它旨在为所有浏览器提供一个通用 API 以及最真实的体验。在模拟用户浏览器交互方面，唯有有血有肉的真人才比它做得更好些。

不过，这种能力不是没有代价的：你需要安装特定浏览器的驱动，现实交互的开销在规模上是显而易见的（例如，Selenium 的测试通常都相当慢）。

Selenium 已经是很久以前发布的了，那时浏览器还不提供任何内置自动化测试兼容性。这么多年过去了，现在的情况已经大不相同，Chrome 引入了 [CDP 协议](https://chromedevtools.github.io/devtools-protocol/)。使用 CDP，你可以直接操作浏览器的 Session，不再需要中间的抽象层和工具。

从那以后，涌现了好多利用 CDP 的项目，包括最著名的——[Puppeteer](https://github.com/puppeteer/puppeteer)，一个 Node.js 的浏览器自动化库。那么 Ruby 世界呢？是 [Ferrum](https://github.com/rubycdp/ferrum)，一个 Ruby 的 CDP 库，尽管还相当年轻，也提供了不逊色于 Puppeteer 的体验。而对我们更重要的是，它带来了一个称为 [Cuprite](https://github.com/rubycdp/cuprite) 的伙伴项目——使用 CDP 的纯 Ruby Capybara 驱动。

我从 2020 年初才开始积极使用 Cuprite 的（一年前我尝试过，但在 Docker 环境下有些问题），从未让我后悔过。设置系统测试变得异常简单（全部所需仅 Chrome 而已），而且执行是如此之快，以至于在从 Selenium 迁移过来之后我的一些测试都失败了：它们缺乏合适的异步期望断言，在 Selenium 中能通过仅仅是因为 Selenium 太慢。

让我们来看下我最近工作上所用到 Cuprite 的系统测试配置。

## 注释过的配置范例

这个范例来自于我最近的开源 Ruby on Rails 项目——[AnyCable Rails Demo](https://github.com/anycable/anycable_rails_demo)。它旨在演示如何跟 Rails 应用一起使用 [刚刚发布](https://evilmartians.com/chronicles/anycable-1-0-four-years-of-real-time-web-with-ruby-and-go) 的 AnyCable 1.0，但我们也可用于本文——它有很好的系统测试覆盖着。

这个项目使用 RSpec 及其系统测试封装器。其大部分也是可以用在 Minitest 上的。

让我们从一个足以在本地机器上运行的最小示例开始。其代码放在 AnyCable Rails Demo 的 [demo/dockerless](https://github.com/anycable/anycable_rails_demo/tree/demo/dockerless) 分支上。

首先来快速看一眼 Gemfile：

```
group :test do
  gem 'capybara'
  gem 'selenium-webdriver'
  gem 'cuprite'
end
```

什么？为什么需要 `selenium-webdriver`，如果我们根本不用 Selenium 的话？事实证明，Rails 要求这个 gem 独立于你所使用的驱动而存在。好消息是，这 [已经被修复了](https://github.com/rails/rails/pull/39179)，我们有望在 Rails 6.1 中移除这个 gem。

我把系统测试的配置放在多个文件内，位于 `spec/system/support` 目录，使用专门的 `system_helper.rb` 来加载它们：

```
spec/
  system/
    support/
      better_rails_system_tests.rb
      capybara_setup.rb
      cuprite_setup.rb
      precompile_assets.rb
      ...
  system_helper.rb
```

我们来看下上面这个列表中的每个文件都是干嘛的。

### system_helper.rb

`system_helper.rb` 文件包含一些针对系统测试的通用 RSpec 配置，不过，通常而言，都如下面这样简单：

```
# Load general RSpec Rails configuration
require "rails_helper.rb"

# Load configuration files and helpers
Dir[File.join(__dir__, "system/support/**/*.rb")].sort.each { |file| require file }
```

然后，在你的系统测试中，使用 `require "system_helper"` 来激活该配置。

我们为系统测试使用了一个单独的 helper 文件和一个 support 文件夹，以避免在我们只需运行一个单元测试时加载所有多余的配置。

### capybara_setup.rb

这个文件包含针对 [Capybara](https://github.com/teamcapybara/capybara) 框架的配置：

```
# spec/system/support/capybara_setup.rb

# Usually, especially when using Selenium, developers tend to increase the max wait time.
# With Cuprite, there is no need for that.
# We use a Capybara default value here explicitly.
Capybara.default_max_wait_time = 2

# Normalize whitespaces when using `has_text?` and similar matchers,
# i.e., ignore newlines, trailing spaces, etc.
# That makes tests less dependent on slightly UI changes.
Capybara.default_normalize_ws = true

# Where to store system tests artifacts (e.g. screenshots, downloaded files, etc.).
# It could be useful to be able to configure this path from the outside (e.g., on CI).
Capybara.save_path = ENV.fetch("CAPYBARA_ARTIFACTS", "./tmp/capybara")
```

该文件也包含一个对于 Capybara 很有用的补丁，其目的我们稍后揭示：

```
# spec/system/support/capybara_setup.rb

Capybara.singleton_class.prepend(Module.new do
  attr_accessor :last_used_session

  def using_session(name, &block)
    self.last_used_session = name
    super
  ensure
    self.last_used_session = nil
  end
end)
```

`Capybara.using_session` 让你能够操作不同的浏览器 session，从而在单个测试场景内操作多个独立 session。这在测试实时功能时尤其有用，例如使用 WebSocket 的功能。

该补丁跟踪上次使用的 session 名称。我们会用这个信息来支持在多 session 测试中获取失败情况下的屏幕快照。

### cuprite_setup.rb

这个文件负责配置 Cuprite：

```
# spec/system/support/cuprite_setup.rb

# First, load Cuprite Capybara integration
require "capybara/cuprite"

# Then, we need to register our driver to be able to use it later
# with #driven_by method.
Capybara.register_driver(:cuprite) do |app|
  Capybara::Cuprite::Driver.new(
    app,
    **{
      window_size: [1200, 800],
      # See additional options for Dockerized environment in the respective section of this article
      browser_options: {},
      # Increase Chrome startup wait time (required for stable CI builds)
      process_timeout: 10,
      # Enable debugging capabilities
      inspector: true,
      # Allow running Chrome in a headful mode by setting HEADLESS env
      # var to a falsey value
      headless: !ENV["HEADLESS"].in?(%w[n 0 no false])
    }
  )
end

# Configure Capybara to use :cuprite driver by default
Capybara.default_driver = Capybara.javascript_driver = :cuprite
```

我们也为常用 Cuprite API 方法定义了一些快捷方式：

```
module CupriteHelpers
  # Drop #pause anywhere in a test to stop the execution.
  # Useful when you want to checkout the contents of a web page in the middle of a test
  # running in a headful mode.
  def pause
    page.driver.pause
  end

  # Drop #debug anywhere in a test to open a Chrome inspector and pause the execution
  def debug(*args)
    page.driver.debug(*args)
  end
end

RSpec.configure do |config|
  config.include CupriteHelpers, type: :system
end
```

下面你可以看到一个 `#debug` 帮助方法如何工作的演示：

[](https://cdn.jsdelivr.net/gh/xfyuan/ossimgs@master/20200716debug_local.av1-4989ddb.gif)

[![Debugging system tests](https://cdn.jsdelivr.net/gh/xfyuan/ossimgs@master/20200716debug_local.av1-4989ddb.gif)

](<https://cdn.jsdelivr.net/gh/xfyuan/ossimgs@master/20200716debug_local.av1-4989ddb.gif>)

### better_rails_system_tests.rb

这个文件包含一些有关 Rails 系统测试内部的补丁以及通用配置（代码注释有详细解释）：

```
# spec/system/support/better_rails_system_tests.rb

module BetterRailsSystemTests
  # Use our `Capybara.save_path` to store screenshots with other capybara artifacts
  # (Rails screenshots path is not configurable https://github.com/rails/rails/blob/49baf092439fc74fc3377b12e3334c3dd9d0752f/actionpack/lib/action_dispatch/system_testing/test_helpers/screenshot_helper.rb#L79)
  def absolute_image_path
    Rails.root.join("#{Capybara.save_path}/screenshots/#{image_name}.png")
  end

  # Make failure screenshots compatible with multi-session setup.
  # That's where we use Capybara.last_used_session introduced before.
  def take_screenshot
    return super unless Capybara.last_used_session

    Capybara.using_session(Capybara.last_used_session) { super }
  end
end

RSpec.configure do |config|
  config.include BetterRailsSystemTests, type: :system

  # Make urls in mailers contain the correct server host.
  # It's required for testing links in emails (e.g., via capybara-email).
  config.around(:each, type: :system) do |ex|
    was_host = Rails.application.default_url_options[:host]
    Rails.application.default_url_options[:host] = Capybara.server_host
    ex.run
    Rails.application.default_url_options[:host] = was_host
  end

  # Make sure this hook runs before others
  config.prepend_before(:each, type: :system) do
    # Use JS driver always
    driven_by Capybara.javascript_driver
  end
end
```

### precompile_assets.rb

[这个文件](https://github.com/anycable/anycable_rails_demo/blob/master/spec/system/support/precompile_assets.rb) 负责在运行系统测试之前预编译 assets（我不在这儿粘贴它的完整代码，只给出最有趣的部分）：

```
RSpec.configure do |config|
  # Skip assets precompilcation if we exclude system specs.
  # For example, you can run all non-system tests via the following command:
  #
  #    rspec --tag ~type:system
  #
  # In this case, we don't need to precompile assets.
  next if config.filter.opposite.rules[:type] == "system" || config.exclude_pattern.match?(%r{spec/system})

  config.before(:suite) do
    # We can use webpack-dev-server for tests, too!
    # Useful if you working on a frontend code fixes and want to verify them via system tests.
    if Webpacker.dev_server.running?
      $stdout.puts "\n⚙️  Webpack dev server is running! Skip assets compilation.\n"
      next
    else
      $stdout.puts "\n🐢  Precompiling assets.\n"

      # The code to run webpacker:compile Rake task
      # ...
    end
  end
end
```

为什么要手动预编译 assets 呢，如果 Rails 能够自动为你做这事的话？问题在于 Rails 预编译 assets 是惰性的（比如，在你首次请求一个 asset 的时候），这会使你的第一个测试用例非常非常慢，甚至碰到随机超时的异常。

另一个我想提请注意的是使用 Webpack dev server 进行系统测试的能力。这在当你进行艰苦的前端代码重构时相当有用：你可以暂停一个测试，打开浏览器，编辑前端代码并看到它被热加载了！

## Docker 化的系统测试

让我们把自己的配置提升到更高的层次，使其兼容我们的 [Docker 开发环境](https://evilmartians.com/chronicles/ruby-on-whales-docker-for-ruby-rails-development)。Docker 化版本的测试设置在 AnyCable Rails Demo 代码库的 [默认分支](https://github.com/anycable/anycable_rails_demo) 上，可随意查看，不过下面我们打算涵盖那些有意思的内容。

Docker 下设置的主要区别是我们在一个单独的容器中运行浏览器实例。可以把 Chrome 添加到你的基础 Rails 镜像上，或者可能的话，甚至从容器内使用主机的浏览器（这可以用 [Selenium and ChromeDriver](https://avdi.codes/run-rails-6-system-tests-in-docker-using-a-host-browser/) 做到）。但是，在我看来，为 `docker-compose.yml` 定义一个专用的浏览器 service 是一种更正确的 _Docker 式方式 _ 来干这个。

目前，我用的是来自 [browserless.io](https://github.com/browserless/chrome) 的 Chrome Docker 镜像。它带有一个好用的 Debug 查看器，让你能够调试 headless 的浏览器 session（本文最后有一个简短的视频演示）：

```
services:
  # ...
  chrome:
    image: browserless/chrome:1.31-chrome-stable
    ports:
      - "3333:3333"
    environment:
      # By default, it uses 3000, which is typically used by Rails.
      PORT: 3333
      # Set connection timeout to avoid timeout exception during debugging
      # https://docs.browserless.io/docs/docker.html#connection-timeout
      CONNECTION_TIMEOUT: 600000
```

把 `CHROME_URL: http://chrome:3333` 添加到你的 Rails service 环境变量，以后台方式运行 Chrome：

```
docker-compose up -d chrome
```

现在，如果提供了 URL，我们就需要配置 Cuprite 以和远程浏览器一起工作：

```
# cuprite_setup.rb

# Parse URL
# NOTE: REMOTE_CHROME_HOST should be added to Webmock/VCR allowlist if you use any of those.
REMOTE_CHROME_URL = ENV["CHROME_URL"]
REMOTE_CHROME_HOST, REMOTE_CHROME_PORT =
  if REMOTE_CHROME_URL
    URI.parse(REMOTE_CHROME_URL).yield_self do |uri|
      [uri.host, uri.port]
    end
  end

# Check whether the remote chrome is running.
remote_chrome =
  begin
    if REMOTE_CHROME_URL.nil?
      false
    else
      Socket.tcp(REMOTE_CHROME_HOST, REMOTE_CHROME_PORT, connect_timeout: 1).close
      true
    end
  rescue Errno::ECONNREFUSED, Errno::EHOSTUNREACH, SocketError
    false
  end

remote_options = remote_chrome ? { url: REMOTE_CHROME_URL } : {}
```

上面的配置假设当 `CHROME_URL` 未被设置或者浏览器未响应时，使用者想使用本地安装的 Chrome。

我们这样做以便让配置向下兼容本地的配置（我们一般不强迫每个人都使用 Docker 作为开发环境；让拒绝使用 Docker 者为其独特的本地设置而受苦吧😈）。

我们的驱动注册现在看起来是这样的：

```
# spec/system/support/cuprite_setup.rb

Capybara.register_driver(:cuprite) do |app|
  Capybara::Cuprite::Driver.new(
    app,
    **{
      window_size: [1200, 800],
      browser_options: remote_chrome ? { "no-sandbox" => nil } : {},
      inspector: true
    }.merge(remote_options)
  )
end
```

我们也需要更新自己的 `#debug` 帮助方法以打印 Debug 查看器的 URL，而不是尝试去打开浏览器：

```
module CupriteHelpers
  # ...

  def debug(binding = nil)
    $stdout.puts "🔎 Open Chrome inspector at http://localhost:3333"
    return binding.pry if binding

    page.driver.pause
  end
end
```

由于浏览器是运行在一个不同的“机器”上，因此它应该知道如何到达测试服务端（其不再是 `localhost`）。

为此，我们需要配置 Capybara 服务端的 host：

```
# spec/system/support/capybara_setup.rb

# Make server accessible from the outside world
Capybara.server_host = "0.0.0.0"
# Use a hostname that could be resolved in the internal Docker network
# NOTE: Rails overrides Capybara.app_host in Rails <6.1, so we have
# to store it differently
CAPYBARA_APP_HOST = `hostname`.strip&.downcase || "0.0.0.0"
# In Rails 6.1+ the following line should be enough
# Capybara.app_host = "http://#{`hostname`.strip&.downcase || "0.0.0.0"}"
```

最后，让我们对 `better_rails_system_tests.rb` 做一些调整。

首先，我们来让 VS Code 中的屏幕快照通知变得可点击🙂（Docker 绝对路径与主机系统是不同的）：

```
# spec/system/support/better_rails_system_tests.rb

module BetterRailsSystemTests
  # ...

  # Use relative path in screenshot message
  def image_path
    absolute_image_path.relative_path_from(Rails.root).to_s
  end
end
```

其次，确保测试都使用了正确的服务端 host（这 [在 Rails 6.1 中已经被修复了](https://github.com/rails/rails/commit/d415eb4f6d6bb24b78b968ae28c22bb7e1721285#diff-9de6fe0bff4847b77cba72441ee855c2)）：

```
# spec/system/support/better_rails_system_tests.rb

config.prepend_before(:each, type: :system) do
  # Rails sets host to `127.0.0.1` for every test by default.
  # That won't work with a remote browser.
  host! CAPYBARA_APP_HOST
  # Use JS driver always
  driven_by Capybara.javascript_driver
end
```

### In too Dip

如果你使用 [Dip](https://github.com/bibendi/dip) 来管理 Docker 开发环境（我强烈建议你这么做，它使你获得容器的强大能力，而无需记忆所有 Docker CLI 命令的成本付出），那么你可以通过在 `dip.yml` 中添加自定义命令和在 `docker-compose.yml` 中添加一个额外 service 定义，来避免手动加载 `chrome` service：

```
# docker-compose.yml

# Separate definition for system tests to add Chrome as a dependency
rspec_system:
  <<: *backend
  depends_on:
    <<: *backend_depends_on
    chrome:
      condition: service_started

# dip.yml
rspec:
  description: Run Rails unit tests
  service: rails
  environment:
    RAILS_ENV: test
  command: bundle exec rspec --exclude-pattern spec/system/**/*_spec.rb
  subcommands:
    system:
      description: Run Rails system tests
      service: rspec_system
      command: bundle exec rspec --pattern spec/system/**/*_spec.rb
```

现在，我使用如下命令来运行系统测试：

```
dip rspec system
```

就是这样了！

最后，让我向你展示一下如何使用 Browserless.io 的 Docker 镜像的 Debug 查看器进行调试：

[](https://cdn.jsdelivr.net/gh/xfyuan/ossimgs@master/20200716debug_docker.av1-c027cb7.gif)

[![Debugging system tests running in Docker](https://cdn.jsdelivr.net/gh/xfyuan/ossimgs@master/20200716debug_docker.av1-c027cb7.gif)  
](https://cdn.jsdelivr.net/gh/xfyuan/ossimgs@master/20200716debug_docker.av1-c027cb7.gif)
