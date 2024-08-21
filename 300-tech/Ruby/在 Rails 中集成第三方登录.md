---
title: 在 Rails 中集成第三方登录
date: 2024-08-16T09:42:00+08:00
updated: 2024-08-21T10:32:32+08:00
dg-publish: false
---

> 这篇文章主要结合个人近期的集成经历浅述一下在 Rails 上第三方登录的集成，涉及它的基本流程以及集成过程中遇到的问题。最后花了挺大篇幅来阐述 Ruby 社区所提供的解决方案，让开发者能够更快速地完成集成工作，并且能够尽快在开发环境完成业务流程的调试。原文发布于 [https://www.lanzhiheng.com/posts/third-part-login-integration-with-ruby-on-rails](https://www.lanzhiheng.com/posts/third-part-login-integration-with-ruby-on-rails)

---

在找工作的间隙无意中接到了一个外包项目（感谢同事跟朋友的推荐），首期要求是帮客户部署一个 [homeland](https://github.com/ruby-china/homeland) 论坛，并扩充已有的功能。

> 值得一提，homeland 是一个完全基于 Ruby On Rails 搭建出来的 Web 应用，源码经过多年的锤炼有很高的参考价值，是个不错的学习范本。

其中一个稍微大一些的改动是，客户希望能够通过微信来登录论坛，N 年没集成过登录了，上一次做这类集成还是当 Python 程序员的时候，正好也借此机会来熟悉一下 Rails 的相关生态。

## 第三方登录的基本流程

现代的第三方登录基本上都遵循了 [Oauth](https://oauth.net/2/) 标准，这样一来用户自建的网站要集成这些第三方的登录就有一套比较规范的做法了。遵循了同样的标准，在流程上会有很多的相似之处。要在网页端集成第三方登录，基本流程大概如下：

1. 点击页面中的 xxx（第三方平台的名称）登录按钮。
2. 把第三方派发的应用 id，第三方的授权域，安全起见所提供的 state 码，以及本网站的回调地址作为跳转链接的参数，拼接成一段 url。然后跳转到该 url。
3. 顺利的话，成功跳转之后会调起第三方平台的授权操作页。
4. 用户授权，成功之后则重定向到第二步提供的回调地址，返回自身的页面中。并携带关键的信息。
5. 上一步返回的信息一般是授权码（code），后台应用可以利用这个码来获取平台的访问凭证（access_token），最后利用访问凭证来调用第三方特定接口即可得到到用户在平台的基本信息。
6. 用户信息都会带有该平台的唯一凭证（针对某个平台的用户，这个值是唯一的），我们可以在数据库存储平台唯一凭证与用户实体的映射关系。后期只需要判断这个映射关系，如果存在了，就直接登录。
7. 如果关联的用户不存在则让用户填写一些基本信息快速注册并登录，或者自动给客户创建一个账号，立即登录。

针对最后一点，不同的网站有着不同的做法。有些网站，比如 [Laravel China](https://learnku.com/laravel) 就是采用了第一种做法，授权并获取平台用户信息之后会让你填写一些该网站的特定信息（用户名，密码，等），然后快速注册。而 [Ruby China](https://ruby-china.org/) 则是采用了第二种做法，获取到用户的关键信息之后则自动给用户创建一个账号，然后直接登录。

两种做法各有各的好处，只是 RubyChina 的做法若果遇到用户名已经被注册的情况，就只能通过类似下面这种加后缀的方式来粗暴处理了。

```
def xxx
  if User.where(login: user.login).exists?
    user.login = "#{user.github}-github" # TODO: possibly duplicated user login here. What should we do?
  end
end
```

## 痛点

从上面的基本流程可以看到，开发者要做的事情其实并不算特别复杂。

- 利用第三方服务提供的信息拼接出一个特定的 URL，用户点击之后就能够跳转到授权页面。
- 授权成功，利用回调地址上的授权码，通过第三方接口获得访问凭证，利用凭证来获取平台用户信息。
- 判断网站中是否有关联的用户存在，存在则直接登录，否则就创建新的用户，并用新用户登录。

个人认为这里有两个比较痛的地方：

1. 遵循同样的标准，登录流程都是大同小异。然而不同的平台，与本网站交互所使用的字段难以统一。在拼接 URL，查找或创建关联用户的时候必定得根据不同的平台特征编写不同的处理方法。
2. 第三方平台通常都很难调试，并不是所有第三方平台都提供沙盒环境。比如：支付宝据说是有（文档说有，还没试验过），微信的沙盒环境似乎并不能调试网站的第三方登录，需要到正式环境上去调试。

第一个痛点似乎还勉强能够接受，毕竟一个网站的第三方登录撑死可能就 4-5 个，为他们写特殊的处理方法也花不了多少时间，无非就是截取自己需要的字段，并做相应的处理，并在代码重构的时候抽取出公共方法即可。

然而第二个痛点就比较烦人了。有些平台如果没有提供沙箱环境，通常都需要用户先开通网站第三方登录的功能，平台要进一步审核，通过之后才可以在生产环境上进行调试。有经验的朋友应该知道审核这个流程最不让人省心了，万一审核不通过，可能你的第三方登录代码要等上一周才能够真正去调试。在此之前你就只能够抱着早已完成的代码，并怀着忐忑的心情一直等待了。

## Rails 是如何解决痛点的

Ruby 社区似乎总是能够很好地解决程序员们的痛点。Rubyist 通过优雅的封装，做出了许多开箱即用（out of box）的软件。要快速地对接第三方平台，可以看看下面这个列表：

- [OmniAuth](https://github.com/omniauth/omniauth)
- [Provider Strategies](https://github.com/omniauth/omniauth/wiki/List-of-Strategies) for OmniAuth
- [Devise](https://github.com/heartcombo/devise)
- Debug in [Developer](https://github.com/omniauth/omniauth/blob/master/lib/omniauth/strategies/developer.rb)

## 1. Omniauth

[OmniAuth](https://github.com/omniauth/omniauth) 的官方文档把它形容成：

> Standardized Multi-Provider Authentication

第一次接触的时候我也是一脸的懵逼。不过其实它做的事情很简单，就是把第三方用户相关的信息写入到 `request.env` 中，然后程序员们就可以在控制器的动作中通过下面代码来获取相关信息：

```
def auth_hash
  request.env['omniauth.auth']
end
```

具体的结构可以看 [这里](https://github.com/omniauth/omniauth/wiki/Auth-Hash-Schema)。官方下面的这段描述我觉得更为全面

> OmniAuth was intentionally built not to automatically associate with a User model or make assumptions about how many authentication methods you might want to use or what you might want to do with the data once a user has authenticated. This makes OmniAuth incredibly flexible.

也就是说我们可以把 OmniAuth 想像成一个平台，它自身并不会作出各种假设，它只定义了一种类似于**平台规范**的东西（比如：[Auth-Hash-Schema](https://github.com/omniauth/omniauth/wiki/Auth-Hash-Schema)）。开发者可以结合第三方平台的文档以及 OmniAuth 的规范来开发相关的插件，第三方特定的行为都在对应的插件内完成。这些插件就是下面会说到的**策略**，而第三方平台在这里会被称为**Provider(供应商) **。

## 各种不同供应商的策略 (Strategies)

OmniAuth 官方并没有为所有第三方编写策略。因为除了你自身网站之外的其他平台都属于第三方，要官方独自维护所有常用的第三方平台登录策略并不现实。何不把它们的开发维护工作分散出去，让真正有需求的人去完成，OminAuth 只需要做好平台的本分工作即可。至于策略要怎么维护，用户们需要在自己的项目中采用哪些策略，都交给他们自己就好了。

说起来这个战略真的太棒了，官方自身只需要维护一份较为常用的 [策略列表](https://github.com/omniauth/omniauth/wiki/List-of-Strategies)，让有需要的开发者自行选择。充分利用了开源的力量。

如果你需要对接微信登录，就在列表中搜索 WeChat，即可找到 [对应的策略](https://github.com/NeverMin/omniauth-wechat-oauth2)，然后在自己的项目中引入它就好了。如果你需要的第三方平台不在官方列表中，那你可能要去别的地方寻找，最坏的情况下就自己写一个吧。不会很难，看一下微信登录的策略其实也就一两个文件，[几十行代码而已](https://github.com/NeverMin/omniauth-wechat-oauth2/blob/master/lib/omniauth/strategies/wechat.rb)。

## Devise 集成

现在很多项目都会用到 [Devise](https://github.com/heartcombo/devise) 来做授权。安装简单，配置方便，还会提供不少的帮助方法，很好地协助开发者完成授权流程。而且 Devise 已经跟 OmniAuth 得到了很好的集成，它可以根据我们所配置的第三方策略，生成对应的路由，而开发者需要做的只是根据不同的策略提供商定制相应的行为。采用了这套集成方案之后，最起码我们不用怎么去操心路由的事情了。专注于业务逻辑即可。

Homeland 其实就是采用了这套方案，不了解的同学很容易就跑偏，不知道往哪去调整代码。我自己也走了好几次弯路，最后找到了 [这份文档](https://github.com/heartcombo/devise/wiki/OmniAuth:-Overview#before-you-start)，一看之下一切脉络都清晰了。官方提供的是 facebook 的对接案例，其他平台其实也是大同小异，差异之处无非是平台返回的用户信息不一致罢了，开发者只需从中筛选，编写自己的业务即可。

个人建议，如果是一些没有历史包袱的应用需要去集成第三方登录，不妨考虑采用 Devise + OmniAuth 这套方案，比起自己去一步步实现真的要省下不少的时间（感谢社区的维护者们）。目测半个小时，十几行代码就能集成完毕。

## 调试

在开发者的生涯中花在调试上的时间，比起写真正代码的时间要多得多，对开发者而言，调试能力尤其重要。然而在集成第三方应用的时候，影响项目进度的就不仅仅是开发者自身的能力了。合作伙伴经验，第三方的审核速度都会影响整个集成的推进。

在对接第三方登录的时候，如果碰巧遇到那些没有提供沙箱环境，登录功能需要开通审核，还要限定域名的第三方（好吧，我说的是微信），那么就只能等到审核通过之后（运气好的话），到线上环境去调试了。这将极大影响项目的进度以及开发者的信心（他们会总是惦记着那还没跑通的流程），不过幸运的是在某种程度上 Ruby 社区很好地帮我们降低这种影响。

OminAuth 的策略列表中有一个名为 [Developer](https://github.com/omniauth/omniauth/blob/master/lib/omniauth/strategies/developer.rb) 的策略，看名字就知道它是面向开发者的，也建议只在开发环境中才开启这个策略。在这个策略的帮助下，你可以自定义用户信息相关的字段，并指定其中哪一个要作为 `uid`，接着就可以在你的开发环境模拟第三方登录流程了。以下是我在 Homeland 基础上添加的代码：

```
Devise.setup do |config|
# ...
  if Setting.has_module? :wechat
    config.omniauth :wechat, Setting.wechat_app_id, Setting.wechat_app_secret,
                    authorize_params: { scope: "snsapi_login" }
  end

  unless Rails.env.production?
    config.omniauth :developer, fields: %w[nickname sex province city country headimgurl unionid], uid_field: :unionid
  end
end

```

我是采用了 Devise + OmniAuth 的方式，看一下文档就能够知道以上配置的意义。我在 Developer 策略中自定义了授权成功后微信用户信息中会包含的字段（可以从微信文档或者 [微信策略](https://github.com/NeverMin/omniauth-wechat-oauth2) 中获取到这些信息）。并把 `uid` 设置为微信提供的 `unionid`。

> 用户统一标识。针对一个微信开放平台帐号下的应用，同一用户的 unionid 是唯一的。

通过访问授权地址 `http://127.0.0.1:4000/account/auth/developer`（这个地址就相当于第三方平台的用户授权地址，只是这里没有附上回调地址以及平台相关的参数罢了），便能获得一个填写信息的界面：

[](https://lanzhiheng.s3-ca-central-1.amazonaws.com/auth-address.png)

[![auth-address.png](https://lanzhiheng.s3-ca-central-1.amazonaws.com/auth-address.png)

](<https://lanzhiheng.s3-ca-central-1.amazonaws.com/auth-address.png>)

填写完之后便进入回调，在回调动作中通过 `request.env['omniauth.auth']` 获得填写的用户信息，就能假装自己在用微信登录了。

```
> request.env['omniauth.auth']
=> {"provider"=>"developer",
 "uid"=>"3333333333333333333333333333333333333333",
 "info"=>
  {"nickname"=>"lanzhiheng",
   "sex"=>"12",
   "province"=>"lanzhiheng",
   "city"=>"lan",
   "country"=>"lanzhiheng",
   "headimgurl"=>
    "https://eiin-chi-test.oss-cn-beijing.aliyuncs.com/node/avatar/11/8e4dc8.jpeg",
   "unionid"=>"3333333333333333333333333333333333333333"},
 "credentials"=>{},
 "extra"=>{}}
```

接下来就是拿这串信息去编写微信相关的业务逻辑。如果要对接支付宝，就根据支付宝的文档去定义相关的用户数据字段，用同样的方式去调试，假装自己在用支付宝登录，其他平台也大同小异。在编写完对应的业务逻辑，并在本地调试完整个流程之后，心里对这些代码也有个底了，最后只需简单改改配置或者换换方法名即可。程序员们也不用一直惦记着那还没调试过的代码了，到时候生产环境调试基本上就是一步到位，这才叫酸爽。

## 尾声

这篇文章主要结合个人近期的集成经历浅述一下在 Rails 上第三方登录的集成，涉及它的基本流程以及集成过程中遇到的问题。最后花了挺大篇幅来阐述 Ruby 社区所提供的解决方案，让开发者能够更快速地完成集成工作，并且能够尽快在开发环境完成业务流程的调试。

[在 Rails 中集成第三方登录 · Ruby China (ruby-china.org)](https://ruby-china.org/topics/40273)

---
