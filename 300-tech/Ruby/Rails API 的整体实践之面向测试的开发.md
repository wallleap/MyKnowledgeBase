---
title: Rails API 的整体实践之面向测试的开发
date: 2024-08-16T09:52:00+08:00
updated: 2024-08-21T10:32:31+08:00
dg-publish: 
source: //ruby-china.org/topics/39171
---

这个是我的一个 [系列文章](https://ruby-china.org/topics/39100) 的正文的第二篇了。[上一篇](https://ruby-china.org/topics/39115) 是讲接口设计和接口行为的，似乎没有取得太大的反响。这一次，说一说关于 Rails 的东西。

老实讲，Rails 我用了三年有余了，基本上 Web 建站就全靠它。因为实在也没有换用别的框架（这里有个特例，是关于 Grape 的，我会在下一篇讲）或者语言的必要。

我在这里，会讲到有关 Rails 的基本用法的一切，包括如何去构建测试，如何去构建一个完整的 API（就像上一篇文章所讲的那样），以及一些配套的 gem 等。

框架所起的作用，是它提供了一套既有的模式，简化了我们大量的骨架性的工作。我发现，很多初学者，即使不怎么精通这门语言，也能很快上手框架的开发，做一些简单的需求。Rails 就是这样，很多入门选手，可以称之为是一个 Rails 选手，却不是一个合格的 Ruby 选手，只是因为他们是先接触的 Rails，然后再去学 Ruby 的。我不知道这是 Rails 的幸运，还是 Rails 的不幸。

## 构建一套完整的 API

内容估计有点多，我想放在下次再讲。

## 面向测试的开发

Rails 指南里关于 [测试](https://ruby-china.github.io/rails-guides/testing.html) 的一节，只讲到了框架为支持测试提供了哪些方法，并没有讲到在实践中如何去构建测试。例如，它对于 Controller 层的一个测试范例这样写道：

```
# articles_controller_test.rb
class ArticlesControllerTest < ActionDispatch::IntegrationTest
  test "should get index" do
    get articles_url
    assert_response :success
  end
end
```

什么意思呢？它是说，先是调用一个接口，然后它应该成功。就没了。我不知道这样的测试能说明什么问题。这样的测试，多多少少会让人有点不放心的感觉。

那么，问题来了。Controller 的测试应该怎么去写呢？或者更进一步讲，分层次的测试应该怎么去写呢？的确，测试应该要分层，我们不可能把所有测试都放在 Controller 里面，它会变得臃肿不堪。而事实上，很多测试，如果放在 Controller 内，这样的测试代码会很难写。分层的测试是必要的。

在介绍每个层次的测试怎么写之前，先列出一下测试的层次有哪些：

1. Controler
2. Policy
3. ActiveRecord
4. ActiveJob
5. Utils or core

注意到，这里涉及到 MVC 框架里的 M 和 C 的部分，没有 V. 永远不要用代码去测有关视图的部分，它应该用肉眼去验证，或者交给前端开发者报告问题，而不是交给机器。试想一下，下面的测试代码是不是有些愚蠢：

```
test "should get index" do
  get articles_url
  assert_response :success

  hash = JSON.parse(response.body)
  articles = hash['articles']
  assert_equal 2, articles.length
  article1, article2 = articles
  assert_equal 1, article2.id
  assert_equal 'title 1', article1.title
  assert_equal 'content 1', article1.content
  assert_equal 2, article2.id
  assert_equal 'title 2', article2.title
  assert_equal 'content 2', article2.content
  # ...
end
```

### Controller 的测试

### 基本模式

Controller 的测试，一般只需要测试正确的接口行为就可以了。我们应该把一般通用的异常情况交给其他的层去测试，如权限问题、参数问题等。所以，一个 Controller 的测试，一般只用写成如下简单的形式：

```
test "should get index" do
  get articles_url
  assert_response :success
  assert_equals articles(:one, :two, :three), assigns(:articles).to_a
end
```

这里用到两个组件，fixtures 和 rails-controller-testing. 前者是 Rails 标配的，后者需要你自行在 Gemfile 中添加：

```
gem 'rails-controller-testing'
```

这要求，我们需要在 Controller 的处理方法里暴露出 `@articles` 实例变量，像这样：

```
def index
  @articles = Article.index
end
```

是的，我们去测方法的返回值，而不是 JSON 或者 HTML 视图。将测试粒度控制在我们方便处理的范围，是我们专业测试人员的义务。由于 index 方法不可能返回任何的值，我们就只能测试它的实例变量了。事实上，这些实例变量会交给视图层去渲染，所以也不算对方法造成污染。

当然，这只是一个简单的情况。对于复杂的接口行为，依然可以采用这种方式。你可以暴露出更多的实例变量，一个、两个或更多个，只要是视图层需要的都会暴露出来。然后我们去测试这些实例变量，就好了。

TODO： factory_bot

### 接口调用

在上面的例子里，接口调用采用的是如下的方式：

```
get articles_url
```

有时候需要传递参数，还有 Headers. 这个时候，我推荐在大多数情况下借助一下 factory_bot 的用法：

```
test "should put update" do
  put article_url(@article), params: { article: attributes_for(:article, title: 'updated title') }
  assert_response :success
  assert_equals 'updated title', assigns(:article).title
end
```

注意这一节代码 `attributes_for(:article, title: 'updated title')`，它是 factory_bot 的一个方法。该方法会返回 `:article` 工厂的属性值，当然 `title` 属性会被覆盖为 `updated title`。在测试 update 的接口的时候，我们是不需要测试所有的字段更新的，只用测试自己最关心的几个就好了。当然，不放心的人可以测试完整的。使用 `attributes_for` 包装的一个考虑是，它会自动控制 validation 的问题。有可能你只传递 `title` 属性接口会报错，用 `attributes_for` 包装，就不用关心 validation 的问题了。

## Policy 的测试

刚才说过，Controller 层只需要测试正确的行为，一般的错误行为交给其他层去测试。Controller 层最容易出现的错误是：

1. required 参数没有传
2. ActiveRecord 的验证没有通过
3. 调用者身份没有权限

错误 1 由捕获异常通用地处理了，可以不必测；错误 2 交给 ActiveRecord 层测试；而错误 3 就是这里讲的 Policy 的测试了。

Policy 的测试是很直白的，就是直接构建 Policy 的类实例，测试它的方法即可：

```
article = articles(:one)
assert Pundit.policy(article.user, article).update?
assert_not Pundit.policy(users(:two), article).update?
```

这里测试 update 接口的用户权限问题。我把它们都写在一起了。事实上它告诉我们，唯有 `article.user` 才可以更新这篇文章，而其他人则不可以。

## Model 的测试

在 Rails 中，Model 层采用的是 ActiveRecord 模式。这里不讨论 ActiveRecord 模式的优劣，但 ActiveRecord 确实是一种包含太多的东西设计模式。在 ActiveRecord 中，会定义到：

1. 数据验证
2. 回调
3. 业务逻辑

这三种情形的测试没有统一的方式一言以蔽之。不过，数据验证倒是可以讲讲。

在应用数据验证测试之前，我推荐使用 fixture 和 factory. Rails 自带有 fixture，而 factory 可以使用 factory_bot 这个 gem.

应用 factory_bot 的时候，注意一点，常规的工厂一定是有效的实例。亦即，往往在 Model 的测试里，下面的测试是第一个要通过的：

```
test 'normal build should be valid' do
  assert build(:user).valid?
end
```

如果以上的测试不通过，修改工厂的实现让其通过。只有在这个的基础上，才能去测试其他的验证方法，如：

```
test 'username should be present' do
  assert_not build(:user, username: nil).valid?
end

test 'username should be unique' do
  user = create(:article)
  assert_not build(:user, username: user.username).valid?
end
```

我在实践中，像 *"username should be valid"* 这样的简单测试，我往往不会去写，而是当成一个天然的自信势必会通过的。这涉及到一个测试覆盖率的问题，究竟覆盖多少测试要根据实际情况和个人风格而定。

### 其他测试

测试说到这里，也算是作结了。其他测试可能包括异步任务的测试、应用框架的测试、核心类或者工具类的测试等等。这些测试方法，作为测试驱动开发的基本涵养，并不是我能够一两句话说清的。为其发声的大师实在是太多太多了，比如在 [京东图书](https://book.jd.com/) 里搜索 *测试驱动开发* 会找大一大箩筐的图书条目。

---

最头疼就是构建数据了，还要构造各种不一样的数据来测不同情况下的条件....有时候甚至觉得就直接拿现有的数据来测方便挺多，但这肯定违背了测试独立性的原则。

可以试试 factory_bot.

---

可以配合易文档 [https://easydoc.top](https://easydoc.top/) 写一个接口文档，在线接口调试，然后还可以生成调用示例、Mock。方便测试和前后端调试
