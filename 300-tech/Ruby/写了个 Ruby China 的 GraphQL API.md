---
title: 写了个 Ruby China 的 GraphQL API
date: 2024-08-16T09:37:00+08:00
updated: 2024-08-21T10:32:32+08:00
dg-publish: false
source: //ruby-china.org/topics/40354
---

上个周末想试用下 Typescript 和 Fastify（一个 node.js 的框架），想找个什么项目练练手，于是就动起了给 Ruby China 写 GraphQL API 的念头。

Live demo:[https://ruby-china-graphql-api.vercel.app/api/graphql](https://ruby-china-graphql-api.vercel.app/api/graphql)

项目开源在 Github：[https://github.com/renyuanz/ruby-china-graphql-api](https://github.com/renyuanz/ruby-china-graphql-api)

完成度大概是 30%，只接了一部分的 GET 接口：

## Query

- topics: [Topic!]
- topic(id: ID!): Topic
- me: UserMe!
- nodes: [Node!]
- node(id: ID!): Node
- users(limit: Int): [User]

## 有什么特别的地方

1. 基于 Fastify，性能快
2. 用 Typescript 和 graphql-codegen 自动从 Schema.graphql 生成 resolvers 类型
3. Oauth2 集成，修改配置后可以直接适用于任何基于 homeland 搭建的项目
4. 魔改了 GraphiQL，增加了一个登录入口

如果你觉得这个项目对你有启发或者有帮助，欢迎给个 star～

---

顺便一提：Ruby 原生的 GraphQL gem 也很好使，无缝融入 Rails：[https://github.com/rmosolgo/graphql-ruby](https://github.com/rmosolgo/graphql-ruby)

---

在用这项目，唯一感觉就是目录分层不好，还有对 i18n 的支持也是差强人意，写了个扩展修改了下 [https://github.com/308820773/rails-api-graphql](https://github.com/308820773/rails-api-graphql)
