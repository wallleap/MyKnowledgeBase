---
title: 使用 associationist 玩转 Rails 虚拟关联
date: 2024-08-16T11:18:00+08:00
updated: 2024-08-21T10:32:32+08:00
dg-publish: false
source: //ruby-china.org/topics/38571
---

[Github Repo](https://github.com/CicholGricenchos/associationist) [中文文档](https://github.com/CicholGricenchos/associationist/blob/master/README_zh.md)

一般来说 Rails 的关联是要在数据表里通过外键实现的，但是有时候会有一些形式上是关联的数据，却没法通过数据表实现。

例如我们想在一个保存为 text 字段的帖子中，查询所有提到的人，并且使用 includes 一并加载出来，也就是我们想：

```
Post.first.mentioned_people # => 所有被提到的Person
Post.includes(:mentioned_people).all
```

往常我们是做不到的，因为 mentioned_people 和 post 之间并没有真正存在的关联。这时候我们可以用 associationist 虚拟出一个自定义的关联，具体代码是这样的：

```
class Post < ApplicationRecord
  include Associationist::Mixin.new(
    name: :mentioned_people,
    type: :collection,
    scope: -> post {
      Person.where(id: post.extract_mentioned_ids)
    }
  )

  def extract_mentioned_ids # 返回一个id数组
    content.scan(/提取id的模式/).map(&:first).map(&:to_i)
  end
end
```

现在直接在 post 对象上调用已经可以正确取出关联了，并且可以像往常一样在之后叠加 limit, count 等方法。

```
Post.first.mentioned_people
Post.first.mentioned_people.limit(1)
Post.first.mentioned_people.count
```

不过只定义了 scope 的情况，在 preload 的时候还是会有 n+1 问题，因为 active record 的 preloader 还不知道怎么批量加载这些关联，需要我们自己定义：

```
class Post < ApplicationRecord
  include Associationist::Mixin.new(
    name: :mentioned_people,
    type: :collection,
    scope: -> post {
      Person.where(id: post.extract_mentioned_ids)
    },
    # preloader需要返回一个key为对象，value为关联数据的hash
    preloader: -> posts {
      people_ids = posts.map(&:extract_mentioned_ids).inject(:+)
      people_hash = Person.where(id: people_ids).map{|person| [person.id, person]}.to_h
      posts.map do |post|
        [post, post.extract_mentioned_ids.map{|id| people_hash[id]}]
      end.to_h
    }
  )
end
```

这个 preloader 做的事就是将一组 post 关联的 people id 先取出来，然后使用一次 sql 查询查出，再安装回 post 上。

现在 includes 方法也可以正常使用了，并且可以像往常一样添加多级的 includes：

```
Post.includes(:mentioned_people).all
Post.includes(mentioned_people: :address).all
```

事实上可以定义成虚拟关联的不只 scope，可以是任何对象：

```
class Product < ApplicationRecord
  include Associationist::Mixin.new(
    name: :stock,
    preloader: -> products {
      products.map{|product| [product, 1]}.to_h
    }
  )
end

Product.first.stock #=> 1
```

这样我们可以把一些复杂 sql 甚至不是 sql 的东西抽象成关联。事实上我设计 associationist 的初衷，就是想让一个很复杂的库存查询的取用变简单。

目前 associationist 只提供了最基础的虚拟关联，其实还可以更进一步去提供一些操作 collection proxy 的方法等等，这么做可以让 active record 真正变成多数据源的，不仅仅能用 sql。（不过没需求就算了）

顺便一提，我之前做的用来实现 shopify 的智能类目的 gem：[https://ruby-china.org/topics/34865](https://ruby-china.org/topics/34865)，已经改为基于 associationist 实现。 smart_collection 里面的 scope 是不定的，对于每个条目都可能生成不同的 scope，所以 preloader 没有这个 Post 的这么好写，要额外用到一个 cache store 或者 cache table，有需要的可以参考参考。
