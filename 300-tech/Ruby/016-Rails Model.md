---
title: 016-Rails Model
date: 2024-08-07 14:59
updated: 2024-08-07 15:00
---

## CoC

Conventions

### table name

```rb
class User < ActiveRecord::Base
end
```

- User → users
- Person → people
- Blog → blogs
- BlogTag → blog_tags

### columns

| Column         | Column Type                                       |
| -------------- | ------------------------------------------------- |
| **id**         | int(11) NOT NULL AUTO_INCREMENT                   |
| username       | varchar(255) COLLATE utf8_unicode_ci DEFAULT NULL |
| password       | varchar(255) COLLATE utf8_unicode_ci DEFAULT NULL |
| **created_at** | datetime NOT NULL                                 |
| **updated_at** | datetime NOT NULL                                 |

对应数据类型

| Rails 数据类型 | MySQL 对应类型                    | PostgreSQL 对应类型             | SQLite 对应类型                 |
|----------------|----------------------------------|--------------------------------|--------------------------------|
| string         | varchar(255) / text               | character varying / text       | varchar / text                  |
| integer        | int                              | integer                        | integer                        |
| float          | float                            | double precision               | float                          |
| boolean        | tinyint(1)                       | boolean                        | boolean                        |
| datetime       | datetime                         | timestamp                      | datetime                       |
| text           | text                             | text                           | text                           |
| timestamp      | timestamp                        | timestamp                      | timestamp                      |

**修改表名**

```rb
class User < ActiveRecord::Base
  self.table_name = "user"  # users → user
end
```

`migrate` 下移植文件也需要修改对应的表名

## CRUD 基础操作

```rb
class User < ActiveRecord::Base
end
```

有了模型就可以对其进行操作

```rb
user = User.new username: "test", password: "123456"
user.save  # 成功返回 user 对象，失败返回 false

user2 = User.create username: "test2", password: "123456"

user3 = User.new
user3.username = "test3"
user3.password = "123456"
user3.save
```

查询 基本 SQL 语句支持的关键字都有

```rb
user = User.where(username: "test").first
user = User.find_by(username: "test")
user = User.first

users = User.where(["username like ?", "%test%"]).order("id desc").limit(2)
```

更新

```rb
user = User.find_by(username: "test")
user.username = "test1"
user.save

User.update_all "status = 1, must_change_password = 'true'"
```



