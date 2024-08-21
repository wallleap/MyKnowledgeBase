---
title: 解决 GraphQL 与 ActiveRecord 嵌套 N + 1 SQL 问题
date: 2024-08-16T11:24:00+08:00
updated: 2024-08-21T10:32:33+08:00
dg-publish: false
source: //ruby-china.org/topics/38395
---

## 背景

GraphQL 多级嵌套查询，如果不手动处理，会导致很严重的 N+1 问题，甚至 (N+1)^n

技术背景是 GraphQL + ActiveRecord + 关系型数据库

举个例子

您原来的 Resolver 代码可能如下所示：

```
class Resolvers::ProfileResolver < GraphQL::Function
  description 'User profile'

  type Types::ProfileType

  def _call(_, args, ctx)
    ctx[:current_user]
  end
end
```

当查询如下的时候：

```
query{
  profile{
    id
    works{
      comments{
        replyUser{
          name
        }
        content
      }
      name
      id
      likes{
        owner{
          name
          works{
            name
            likes{
              owner{
                name
              }
            }
          }
        }
      }
    }
    name
  }
}
```

它将导致多级嵌套 N + 1 问题。

如果你要解决它，你可以像这样写：

```
class Resolvers::ProfileResolver < OptimizedFunction
  description 'User profile'

  type Types::ProfileType

  def _call(_, args, ctx)
    User.includes([works: [comments: :reply_user, likes: [owner: [works: [likes: :owner]]]]]).find(ctx[:current_user].id)
  end
end
```

您将在每个 Resolver 中手动解决 N + 1 问题。更糟糕的是，即使只请求了一个字段，也不得不向数据库查询 includes 的所有表。

## 解决办法

- graphql-batch gem

		- 感觉并不特别好用
		- 不容易处理多层嵌套的问题
- [自己动手写了个 gem](https://github.com/tinyfeng/rgraphql_preload_ar)

## 用法

- `Resolver` 从继承 `GraphQL::Function` 改为继承于 `OptimizedFunction` 。
- 定义 `_call` 方法而不是 `call`
- 使用 `includes_klass(your_model_name)` 替换 includes 语句。

```
class Resolvers::ProfileResolver < OptimizedFunction
  description 'User profile'

  type Types::ProfileType

  def _call(_, args, ctx)
    includes_kclass(User).find(ctx[:current_user].id)
  end
end
```

## 原理

原理是解析最开始请求的 json 多叉树，与 ActiveRecord 的模型关联关系对比，从请求 json 多叉树中过滤出嵌套的关联关系树。

希望能帮助到和我一样喜欢 rails 和 graphql 的人，欢迎提出改进意见。

---

同样是 GraphQL + Rails 用户。考虑到有的人用的不是 ActiveRecord, 比如 Sequel，并且 N+1 的场景并不只是关系型数据库，任何 IO 操作都有 N+1 场景。所以这个 `graphql-batch` 的定位目前已经做到通用了（和 Node 中的 data-loader）一样。

---

GQL 的一些实践可以参考 Gitlab 的，比如 [https://gitlab.com/gitlab-org/gitlab-ce/blob/master/lib/gitlab/graphql/loaders/batch_model_loader.rb](https://gitlab.com/gitlab-org/gitlab-ce/blob/master/lib/gitlab/graphql/loaders/batch_model_loader.rb)

不过 GraphQL-Ruby 最近每一个版本都在做内部重构，所以要注意版本

---

是我的标题写的比较大，已经修改了。

`graphql-batch` 需要在 type 里每个关联的地方手动加入以下类似代码来使用，但是我感觉如果根据请求，就能自动完成需要的 association 的预加载会更方便。

```
field :product, Types::Product, null: true do
  argument :id, ID, required: true
end

def product(id:)
  RecordLoader.for(Product).load(id)
end
```

---

谢谢指导，感觉代码写的是很漂亮
