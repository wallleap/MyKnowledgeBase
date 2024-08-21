---
title: 分享一个新做的 gem：smart_collection
date: 2024-08-16T11:19:00+08:00
updated: 2024-08-21T10:32:32+08:00
dg-publish: false
source: //ruby-china.org/topics/34865
---

[Github Repo](https://github.com/CicholGricenchos/smart_collection)

在电商环境中，我们常常需要根据一些条件去选择一组商品，例如从类目 A 和类目 B 里选出价格少于 10 元的产品。而且我们希望当某个产品涨价大于 10 元的时候，会自动退出这个 collection。

这是 shopify 的一个卖点：[智能类目](https://help.shopify.com/api/reference/smartcollection)，我把它做成了一个 AR 的插件，让其他 Rails 项目也能实现类似的功能。

我们可以这样去定义一个 Collection Model：

```
class CreateCollections < ActiveRecord::Migration[5.0]
  def change
    create_table :collections do |t|
      t.text :rule
      t.datetime :cache_expires_at
      t.timestamps
    end
  end
end

class Collection < ActiveRecord::Base
  serialize :rule, JSON

  include SmartCollection::Mixin.new(
    items: :products
  )
end
```

创建一个 collection：

```
collection = Collection.create(
  rule: {
    and: [
      {
        or: [
          {
            association: {
              class_name: 'Catalog',
              id: @pen_catalog.id,
              source: 'products'
            }
          },
          {
            association: {
              class_name: 'Catalog',
              id: @pencil_catalog.id,
              source: 'products'
            }
          }
        ]
      },
      {
        condition: {
          joins: 'properties',
          where: {
            properties: {
              value: 'Red'
            }
          }
        }
      }
    ]
  }
)
```

选出商品：

```
collection.products #=> 选出水笔和铅笔中有“红色”属性的产品
collection.products.where(in_stock: true).order(id: :desc) #=> 关联返回的仍旧是scope，可以继续进行where order等操作
```

实现原理是自定义一个 association，用条件拼出来的 scope 重载掉 association_scope，因为 rails5 里 scope 可以 or，用 merge 和 or 拼接很方便。condition 子句原样支持 AR 的 joins 和 where 参数，我发现这样的表达力是最好的，就不自己造其他 dsl 了。

这时直接去用可能会比较慢，因为每个集合的 scope 都需要一条查询，如果批量会有 n+1 问题，我们需要使用缓存。

smart_collection 提供了 table 缓存和 cache_store 缓存，启用之后就可以使用 preload 了，如果缓存过期，单个读取和 preload 都会更新缓存。

启用缓存：

```
class CreateSmartCollectionCachedItems < ActiveRecord::Migration[5.0]
  def change
    create_table :smart_collection_cached_items do |t|
      t.integer :collection_id
      t.integer :item_id
    end
  end
end

class Collection < ActiveRecord::Base
  serialize :rule, JSON

  include SmartCollection::Mixin.new(
    items: :products,
    cached_by: {
      table: :default,
      expires_in: 1.hour
    }
    #cached_by: {
    #  cache_store: Rails.cache,
    #  expires_in: 1.hour
    #}
  )
end

Collection.where(id: [1, 2]).preload(products: :properties) #=> 不会有n+1问题
```

隐藏特性：对于 table 缓存，eager_load 也是可行的，但是 eager_load 的时候没法判断 collection 的缓存是否过期，所以只适合定期更新全部缓存的场景。

缺陷：在 rule 里存储的数据，如果源数据被删掉了，就会出错，未来考虑把 collection 依赖的数据都保存起来，实现 dependent: :destroy 之类的特性。

---

用 table 缓存相当于把 items 静态化到一张表里，但更新需要自己维护？ 不用 table 缓存是完全动态根据 rule 查？

是的，用缓存的话等过期就好，读取和 preload 会更新缓存，不用手动
