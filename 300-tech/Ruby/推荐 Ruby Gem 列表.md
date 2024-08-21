---
title: 推荐 Ruby Gem 列表
date: 2024-08-16T12:50:00+08:00
updated: 2024-08-21T10:32:33+08:00
dg-publish: false
source: //ruby-china.org/wiki/gems
---

本页用于介绍 Ruby 社区里面那些特别热门的 Gem，以下 Gem 可以在 [rubygems.org](http://rubygems.org/) 找到。

实用的 RubyGems 排名站点 [www.ruby-toolbox.com](https://www.ruby-toolbox.com/)

## [bootstrap](https://github.com/twbs/bootstrap-rubygem)

来自 Twitter 的 Bootstrap，是一套完成的前台 CSS 框架。 以简洁，优雅著称于世。被无数攻城狮所青睐，又让无数程序猿审美疲劳。

## [Devise](https://github.com/plataformatec/devise)

用于快速构建用户功能，如：注册，登陆，个人设置，找回密码... 同时 Ruby 社区有各类和账号体系的库可以很容易和 Devise 打通。

## [OmniAuth](https://github.com/intridea/omniauth)

如果你需要在项目中实现三方平台（如：Twitter, Facebook, 新浪微博，腾讯 QQ）账号登陆的支持，那你需要用上它。

*RailsCast: [Part1](http://asciicasts.com/episodes/235-omniauth-part-1) [Part2](http://railscasts.com/episodes/236-omniauth-part-2)*

## [will\_paginate](https://github.com/mislav/will_paginate) 和 [Kaminari](https://github.com/amatsuda/kaminari)

分页，几乎所有 Rails App 都在用，其中 will\_paginate 比较老，应用案例较多，kaminari 更新，性能和兼容性更好

## [Carrierwave](https://github.com/jnicklas/carrierwave) 和 [Paperclip](https://github.com/thoughtbot/paperclip)

这两个都是上传组件，Paperclip 是老牌产品了，也是几乎绝大多数项目都有在用它，它可以帮你处理上传图片，裁减，定义不同的图片尺寸，几乎很完美。而 Carrierwave 是后起之秀，功能和 Paperclip 差不多，但它还可以管理除图片之外的东西，而且灵活性更高（ruby-china 就是用它）。国内的各大云存储服务都已经有了 Carrierwave 的支持，例如 [carrierwave-aliyun](https://ruby-china.org/wiki/github.com/huacnlee/carrierwave-aliyun), [carrierwave-upyun](https://github.com/nowa/carrierwave-upyun)。

## [WiceGrid](https://github.com/leikind/wice_grid)

表格控件，针对 ActiveRecord，超级强大，支持任意字段排序，过滤，具体看它的 [Demo](http://wicegrid.herokuapp.com/).

## [elasticsearch-rails](https://github.com/elastic/elasticsearch-rails)

实现全文搜索或搜索有关的功能，目前要数 Elasticsearch 最火，它也有 Ruby 的实现。

## [RailsSettingsCached](https://github.com/huacnlee/rails-settings-cached)

项目经常会有一些配置信息，你需要一个库帮你管理。

## [CanCanCan](https://github.com/CanCanCommunity/cancancan)

一些应用中会用到为不同用户设定不同功能的权限，你可以试试 CanCanCan 这个 Gem 他可以帮你制定一套完善的方案，[Railscasts](http://railscasts.com/episodes/192-authorization-with-cancan) 上有使用介绍。

如果 Devise 一样，CanCanCan 在 Ruby 社区也是非常流行，所以和它有关的实现非常多，你能很容易找到资料。

## [Doorkeeper](https://github.com/doorkeeper-gem/doorkeeper)

前面说了 OmniAuth 介入 OAuth 2 的服务，有时候你可能需要自己建立一个 OAuth 2 服务，作为提供商，这个时候 Doorkeeper 可以帮到你。

### [Nokogiri](http://nokogiri.org/)

采集数据是我们需要解析复杂的 HTML 结构，从中获得需要的数据，Nokogiri 可以帮助我们完美的处理不同网页上面不同的 HTML 结构，并且有很好的编码处理能力，用它你不用担心页面是 GB2312 还是 GBK 还是 UTF-8，它都很很好的处理，解析结构可以用类似 jQuery 的 CSS Selector 的方式查找，很是方便。曾经用过 Ruby 的好几个类似插件，最终发现 Nokogiri 才是最好的。

### [Formtastic](https://github.com/justinfrench/formtastic) 和 [simple\_form](https://github.com/plataformatec/simple_form)

Rails 为我们带来和一改传统的表单构件方式，但是经过实际的使用，我们渐渐发觉这样依然还是不够“敏捷”，我们需要更加简便并具有更细致规范的表单，所以有了 Formtastic，它用起来比 Rails 默认的 form 更加简洁，但是却具有更多的功能，你可以为每个字段设定 help-text 放到文本框下面，并可以走 I18n 的方式设置语言，具体参见 [Railscasts](http://railscasts.com/episodes/184-formtastic-part-1) 上面对于 Formtastic 的介绍。而 simple\_form 和 Formtastic 功能类似，但它的写法还要简单一些。

### [Whenever](https://github.com/javan/whenever)

Linux 里面有 Cron 可以帮助我们定期执行一些任务，但是 Cron 手动写起来很是麻烦，尤其是前面时间周期的定义，Whenever 可以帮助我们用更人性化的方式编写 Cron 任务，具体参见 [Railscasts](http://railscasts.com/episodes/164-cron-in-ruby) 上面关于 Whenever 的介绍。

### [Sidekiq](https://github.com/mperham/sidekiq), [Resque](https://github.com/resque/resque) 和 [Delayed\_job](https://github.com/collectiveidea/delayed_job)

有时候一些任务的执行会很慢，而这些任务我们并不要求需要马上返回结果 (比如：发送邮件，生成图片缩略图)，那我们可以选择将这些任务放到后台执行，以便于页面不会长时间等待执行。

### [daemon-spawn](https://github.com/alexvollmer/daemon-spawn)

将一些事情作为 daemon 来启动，可以类似 Debian 的 service foo start 比如 用来管理 Resque 的启动和重启，会变得很简单。

### [Grape](https://github.com/intridea/grape)

随着 Mobile App 的增多，很多时候我们在做用 Rails 做 API Base 项目时，rails 自带的 C 和 V 层显得过于繁杂，grape 可以帮助我们快速的构建和 Rails 完美融合的 API 接口。

### [ClientSideValidations](https://github.com/bcardarella/client_side_validations)

现在越来越多网站为了改善用户体验，使用 JavaScript 来进行客户端验证。对于程序员来说，也因此增加多一份工作。而往往客户端的验证逻辑跟服务端的验证逻辑几乎一样，如果要另外再写一次验证代码，实在不够 DRY，client\_side\_validations 正是为解决此问题要出现。client\_side\_validations 会读取服务端的验证逻辑并生成对应的客户端验证逻辑（依赖 jQuery），让你几乎不用增加任何前端代码就可实现客户验证。

### [by\_star](https://github.com/radar/by_star)

这是一个辅助 ActiveRecord 的组件，让你可以简单的实现按某年，某月，某日，或者星期几，来查询数据，用起来非常简单，省下麻烦的条件组合，此外，它还可以查询上一篇，下一篇类似的功能。

### [rolify](https://github.com/EppO/rolify)

Very simple Roles library without any authorization enforcement supporting scope on resource object.

### [Faraday](https://github.com/lostisland/faraday)

HTTP Client 支持多种方式

### [pry](https://pry.github.com/)

简单强大的调试工具，轻量级的工具。直接在终端调试方便又直接

### [Seed Fu](https://github.com/mbleigh/seed-fu)

强大的 seed

### [rails\_best\_practices](https://github.com/railsbp/rails_best_practices)

编写代码总有方圆，费心费力写文档，还不如用这个工具来控制代码质量。

### [lazy\_high\_charts](https://github.com/michelson/lazy_high_charts)

当前绘图 JS 库中 Highcharts 非常优秀，rubyist 使用这个 gem 来管理和编写需要的图。

### [Better Errors](https://github.com/charliesome/better_errors)

它用一个更好的，更有用的错误页替换标准的 Rails 错误页面，对 Rack middleware 也同样有效。·[Railscasts](http://railscasts.com/episodes/402-better-errors-railspanel) 也有相应的介绍。

### [god](http://godrb.com/)

Ruby 进程监控工具

### [RuCaptcha](https://github.com/huacnlee/rucaptcha)

图片验证码，安全、简单、易用，无依赖。

## [second\_level\_cache](https://rubygems.org/gems/second_level_cache)

ActiveRecord 二级缓存插件，装上他你可以无缝的实现对 ActiveRecord 数据的二级缓存。
