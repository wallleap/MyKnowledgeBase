---
title: Rails 5.1 System Tests
date: 2024-08-08T11:00:00+08:00
updated: 2024-08-21T10:32:31+08:00
dg-publish: false
---

这里的内容是由像和你一样的工程师们一行一行创造出来的，为了保护你的权益，同时也为了激励我们继续创造更好的内容，蛋人网所有内容禁止一切形式的转载和复制

## 目标

1. 了解如何写一个简单的 System Test
2. 如何通过读源码找到缺失的文档
3. 理解 System Tests 清理测试数据的原理
4. 理解什么时候应该使用 System Tests 以及如何使用 System Tests

## 介绍

Rails **5.1** 中引入了 **System Tests** , System Tests 的实现主要是对 [Capybara](https://github.com/teamcapybara/capybara) 的封装，同时加入了一些很方便的特性。Rails 框架非常好的一点是它会把当前 Web 开发的最佳实践集成到框架里，同时给开发者制定一套默认配置，这样开发者在使用这些最佳实践的时候避免做很多选择和配置，System Tests 的引入是 Rails 这种框架理念的体现。

## 例子

下面是一个 System Test，测试当没有商品加入购物车时购物车是不显示在页面上的，但是一旦有商品加入到购物车里购物车就会显示在页面上。

```
require "application_system_test_case"

class CartsTest < ApplicationSystemTestCase
  test "reveal and hide cart" do
    visit store_index_url

    assert_no_selector "#cart"

    first('.entry').click_on 'Add to Cart'

    assert_selector "#cart"
    assert_content "Your Cart"

    accept_confirm do
      click_on 'Empty Cart'
    end
    assert_no_selector "#cart"
    assert_no_content "Your Cart"
  end
end
```

我们可以看到 CartsTest 继承了 ApplicationSystemTestCase, 这样 CartsTest 就会获得操作页面的一些方法，例如 `visit`, `click_on`, `accept_confirm`, 同时也会获得验证页面内容的一些方法，如 `assert_selector`, `assert_content`. 上面的测试可以这样表述:

- 访问 store_index_url 这个 url, 页面上应该看不到 id 为 cart 的元素;
- 在第一个 class 为 entry 的元素里点击 ‘Add to Cart’， 会看到页面上有一个 id 为 cart 的元素，同时会看到文字 "Your Cart"
- 当点击 "Empty Cart" 时浏览器会弹出一个对话框让我们确认是不是真的要清空购物车，确认清空后，页面上不会有 id 为 cart 的元素，同时也不会看到文字 "Your Cart"

## 缺失的文档

### 页面验证

我在编写 System Tests 的过程中遇到的第一个问题是文档的缺失, 例如除了上面的 `assert_selecor`, `assert_content` 外还有哪些验证页面内容的方法？到 [Capybara的github主页](https://github.com/teamcapybara/capybara) 没有找到相关的信息，到 [Rails Guides](http://guides.rubyonrails.org/testing.html#system-testing) 也没有找到。最后的办法就是直接读源代码，Rails 使用 [Minitest](https://github.com/seattlerb/minitest) 作为基本的测试框架, 而 Capybara 针对 Minitest 有一套验证页面的方法, 这些方法定义在 Capybara 的 [lib/capybara/minitest.rb](https://github.com/teamcapybara/capybara/blob/master/lib/capybara/minitest.rb) 这个文件中, 里面定义了所有对应 Minitest 的验证页面内容的方法, 例如 `assert_title`, `assert_xpath`, `assert_css`, `assert_table` 等等。

### 操作页面的 DSL

关于操作页面的 DSL, [Capybara的官方文档](https://github.com/teamcapybara/capybara#the-dsl) 有一定的描述, 要想查看完整的 DSL, 可以查看 Capybara 的 [lib/capybara/session.rb](https://github.com/teamcapybara/capybara/blob/master/lib/capybara/session.rb) 这个文件, 如下所示:

```
class Session
   include Capybara::SessionMatchers

   NODE_METHODS = [
     :all, :first, :attach_file, :text, :check, :choose,
     :click_link_or_button, :click_button, :click_link, :field_labeled,
     :fill_in, :find, :find_all, :find_button, :find_by_id, :find_field, :find_link,
     :has_content?, :has_text?, :has_css?, :has_no_content?, :has_no_text?,
     :has_no_css?, :has_no_xpath?, :resolve, :has_xpath?, :select, :uncheck,
     :has_link?, :has_no_link?, :has_button?, :has_no_button?, :has_field?,
     :has_no_field?, :has_checked_field?, :has_unchecked_field?,
     :has_no_table?, :has_table?, :unselect, :has_select?, :has_no_select?,
     :has_selector?, :has_no_selector?, :click_on, :has_no_checked_field?,
     :has_no_unchecked_field?, :query, :assert_selector, :assert_no_selector,
     :assert_all_of_selectors, :assert_none_of_selectors,
     :refute_selector, :assert_text, :assert_no_text
   ]
   # @api private
   DOCUMENT_METHODS = [
     :title, :assert_title, :assert_no_title, :has_title?, :has_no_title?
   ]
   SESSION_METHODS = [
     :body, :html, :source, :current_url, :current_host, :current_path,
     :execute_script, :evaluate_script, :visit, :refresh, :go_back, :go_forward,
     :within, :within_element, :within_fieldset, :within_table, :within_frame, :switch_to_frame,
     :current_window, :windows, :open_new_window, :switch_to_window, :within_window, :window_opened_by,
     :save_page, :save_and_open_page, :save_screenshot,
     :save_and_open_screenshot, :reset_session!, :response_headers,
     :status_code, :current_scope,
     :assert_current_path, :assert_no_current_path, :has_current_path?, :has_no_current_path?
   ] + DOCUMENT_METHODS
   MODAL_METHODS = [
     :accept_alert, :accept_confirm, :dismiss_confirm, :accept_prompt,
     :dismiss_prompt
   ]
   DSL_METHODS = NODE_METHODS + SESSION_METHODS + MODAL_METHODS

   .....
```

## 清理测试数据

无论在编写单元测试还是集成测试的过程中都会遇到清理测试数据的问题，前一个运行的测试可能会向测试数据库写入一些测试数据，而后运行的测试可能与前一个测试写入的数据相矛盾而导致失败, 为了让每个测试互不影响，需要一种清理测试数据的策略. 目前普遍使用的有两种策略，一种是在运行测试之前或之后清空所有数据; 另外一种方式是在测试运行之前开始一个数据库事务，并在测试结束后回滚当前事务，最终测试过程中生成的数据不会持久化到数据库中。 [DatabaseCleaner](https://github.com/DatabaseCleaner/database_cleaner) 是专门负责做数据清理的 Gem, 刚提到的第一种策略在 DatabaseCleaner 里叫 `truncation`, 而第二种策略叫做 `transaction`。

一般在”单元测试 " 或者”模拟的集成测试“里都是采用的 `transaction` 这种策略，但是在端对端的真正的集成测试里一般采用 `truncation` 这种策略， 这是因为在端对端的集成测试里，测试脚本通常和应用服务器运行在不同的线程里，而在不同的线程里一般是不共享数据库连接的，所以在测试脚本里写入到数据库的改变如果不做 `Commit`, 运行在应用服务器里的应用代码是无法读到的. 下面我们通过看测试日志就会看到 Rails 5.1 的 System Tests 仍然采用的是 `transaction` 的策略，接着我们发现原来是 Rails 5.1 新提交了一个功能 [使测试线程和应用服务器线程共享数据库连接](https://github.com/rails/rails/pull/28083)，不得不说这个做法很聪明!

为了证明我们的发现，我们可以运行购物车测试的同时查看 Rails 测试日志，你会发现类似于如下截图的日志: ![img_alt](https://imgs.eggman.tv/15001690861714149transaction_begin-png.png)

![img_alt](https://imgs.eggman.tv/15001690964723904transaction_end-png.png)

第一个截图的第一行日志是在测试开始之前 `开始事务`，第二个截图的最后一行日志是在测试结束后 `回滚事务`。

Rails 在 Controller Test 和 Integration Test 中采用了 `transaction` 的方式清理测试数据，而 System Tests 在采用 `transaction` 之后，Rails 中各种测试的清理测试数据的策略就保持了一致性；并且 `transaction` 比 `truncation` 清理数据的速度要快。

## 使用不同的 Driver

Capybara 默认支持 Selenium 作为 Driver, Selenium 会真正启动浏览器运行测试，这种测试最全面但是运行速度也会比较慢，Capybara 支持一些 `Headless Driver`, 这些 `Headless Driver` 驱动不需要 UI 的浏览器，测试运行更快一些，下面演示如何修改 System Tests 的配置使用 `Poltergeist` 作为 `Headless Driver`:

- 修改 test/application_system_test_case.rb 使用 `Poltergeist`:

```
require "test_helper"
require 'capybara/poltergeist'

class ApplicationSystemTestCase < ActionDispatch::SystemTestCase
  # driven_by :selenium, using: :chrome, screen_size: [1400, 1400]
  driven_by :poltergeist
end
```

- 修改 Gemfile 并且运行 `bundle install` 安装 `Poltergeist`:

```
group :development, :test do
  # Call 'byebug' anywhere in the code to stop execution and get a debugger console
  gem 'byebug', platforms: [:mri, :mingw, :x64_mingw]
  # Adds support for Capybara system testing and selenium driver
  gem 'capybara', '~> 2.13'
  gem 'poltergeist'
  # gem 'selenium-webdriver'
end
```

- [安装PhantomJS](https://github.com/teampoltergeist/poltergeist#installing-phantomjs)

完成以上几个步骤之后运行测试，你会发现测试运行的过程中不会启动浏览器，测试运行也更快了.

## 如何使用 System Test

端对端的系统测试会启动浏览器，同时启动应用服务器，这是最接近真实运行环境的测试，同时测试会驱动系统中各个层的代码运行，死角更少，但同时带来两个问题:

- 测试运行慢
- 由于 System Tests 里要验证 UI 内容或元素，而前台的显示可能修改比较频繁，这样导致 System Tests 很脆弱

针对测试运行慢的问题，我们应该尽量用 System Tests 覆盖系统的主要场景，而不是为系统的所有使用场景编写 System Tests; 应该写粗粒度的 System Tests, 比如覆盖一个主要场景的成功情况，而不需要覆盖一个主要场景的各个失败分支; 细粒度的测试可以交给运行更快的单元测试或者模拟集成测试.

针对第二个问题，需要我们抽象测试，尽量用业务术语表达测试，这样当 UI 元素需要修改的时候，在抽象比较好的情况下只需要修改某个地方，而不是需要修改很多地方。举个例子：前面提到的购物车测试还停留在实现细节上，测试购物车应该显示在页面上是通过验证页面上有 id 为 cart 的元素以及页面上显示”Your Cart", 业务层面其实只关心“购物车显示还是不显示”，于是可以做如下抽象:

```
require "application_system_test_case"

class CartsTest < ApplicationSystemTestCase

  test "reveal and hide cart" do
    visit store_index_url

    assert_cart_hide

    first('.entry').click_on 'Add to Cart'

    assert_cart_show

    accept_confirm do
      click_on 'Empty cart'
    end
    assert_cart_hide
  end

  def assert_cart_show
    assert_selector "#cart"
    assert_content "Your Cart"
  end

  def assert_cart_hide
    assert_no_selector "#cart"
    assert_no_content "Your Cart"
  end
end
```

抽象后你会发现测试是以业务语言表达的，而不是底层实现细节，这样不但测试可读性提高了，测试维护的代价也更低了: 假设购物车元素不再采用 cart 作为 id, 那我们只需要修改 assert_cart_show 和 assert_cart_hide.

## 参考

- [Rails 5.1: Loving JavaScript, System Tests, Encrypted Secrets, and more](http://weblog.rubyonrails.org/2017/4/27/Rails-5-1-final/)
- [Full-Stack Testing with Rails System Tests](https://chriskottom.com/blog/2017/04/full-stack-testing-with-rails-system-tests/)
- [RailsGuides中关于System Test的文档](http://guides.rubyonrails.org/testing.html#system-testing)
- [Capybara](https://github.com/teamcapybara/capybara)
