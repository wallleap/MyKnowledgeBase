---
title: 搭建 Github Pages & Issues 动态博客
date: 2024-03-25T12:28:00+08:00
updated: 2024-08-21T10:32:45+08:00
dg-publish: false
---

本文简要介绍了使用 Github Pages + Github API + 前端代码，无需自有服务器，搭建动态博客的一种解决方案。

## [#](https://fecat.win/article/4#%E4%BD%BF%E7%94%A8github-pages) 使用 Github Pages

[Github Pages 介绍](https://pages.github.com/)

介绍只有 5 步，很简单。

实际使用也很简单：

Github 将 `<username>.github.io/<repository>` 作为当前仓库的根域名，提供 web 资源访问服务。

其中 `<username>.github.io` 作为 `<username>/<username>.github.io` 这一特定仓库的根域名，提供同样的服务。

简之，只需要 push 你的项目，Github 就会自动部署到其服务器上。

每个文件都可以通过 `https://<username>.github.io/<repository>/path/to/file` 访问到。

Github 真是一个大型 web 服务仓库！

## [#](https://fecat.win/article/4#%E5%87%86%E5%A4%87github-api) 准备 Github API

Github Web 应用常用的 API 有两种：

- [REST API v3](https://developer.github.com/v3/)
- [GraphQL API v4](https://developer.github.com/v4/)

API 使用方式如其名。REST 基本就是看文档调用 API，本文要试试 GraphQL。

登陆 [GitHub's GraphQL Explorer](https://developer.github.com/v4/explorer/)，边看文档边组装请求。

例如我想获取 `luoway/blog` 下的 issues 列表

```js
{
  repository(owner: "luoway", name: "blog") { #指定仓库
    issues(
      orderBy: {field: CREATED_AT, direction: DESC} #创建时间倒序
      first: 100 #限制最多查询记录数
    ) {
      edges {
        cursor #指针，与after配合用于翻页
        node { 
          number #issue唯一标识
          title
          createdAt
        }
      }
    }
  }
}
```

## [#](https://fecat.win/article/4#%E5%88%9B%E5%BB%BAgithub-token) 创建 Github Token

使用 Github API，无论流行的 REST，新兴的 GraphQL，都离不开认证（Authentication）。

GitHub's GraphQL Explorer 中使用的是登陆认证，大概是最安全的认证方式。

但很遗憾，GraphQL API 只支持一种方式，也是 REST 支持的 [OAuth2 token](https://developer.github.com/v3/#oauth2-token-sent-in-a-header)。

接下来以 OAth2 token 的方式使用 Github API

- 首先 [创建Token](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/#creating-a-token)
- 勾选使用范围，只需要基本的读取权限

![勾选read:user](https://wx4.sinaimg.cn/mw690/b4aaf83bly1g0aytf7ejsj20kw02wq34.jpg)

- 使用 Token

## [#](https://fecat.win/article/4#%E4%BD%BF%E7%94%A8github-token) 使用 Github Token

简之两种方法：

- 后端接口转发携带 token（github 推荐方式）
- 前端异步请求携带 token

本文介绍前端异步请求携带 token 的方式。

### [#](https://fecat.win/article/4#%E7%BB%95%E8%BF%87github-token%E5%AE%89%E5%85%A8%E6%9C%BA%E5%88%B6) 绕过 Github Token 安全机制

由于 github 安全机制，`git commit` 记录中的任何有效 token 都会被 github 失效化。

而在无后端存储转发 token 的前提下，前端代码必须提交 token 用于前端请求。

创建的 token 使用范围是有限的，token 泄漏最糟的情况就是限流数被盗用完，一小时后恢复。

> The GraphQL API v4 rate limit is **5,000 points per hour**.

因此可以接受 token 泄漏的后果。那么要解决“如何绕过 github 检测 token”的问题，即不直接存储 token，间接地存储。

解决方案可以是：在获取到 token 后，将 token 以任何可还原的方式处理一下，转换成不同的字符串，然后在 js 运行时还原为真实有效的 token。

例如字符串反转

```javascript
// 反转真实token得到reversed_token
const token = 'reversed_token'.split('').reverse().join('') //js运行时还原token
```

### [#](https://fecat.win/article/4#%E5%89%8D%E7%AB%AF%E5%BC%82%E6%AD%A5%E8%AF%B7%E6%B1%82%E6%90%BA%E5%B8%A6token) 前端异步请求携带 token

[OAuth2 token](https://developer.github.com/v3/#oauth2-token-sent-in-a-header) 允许两种传递方式

- url 参数传递

```js
const API = `https://api.github.com/?access_token=${token}`
```

- 请求头传递

```js
const XHR = new XMLHttpRequest()
XHR.setRequestHeader('Authorization', 'token ' + token)
```

至此理清了：

- 如何部署站点（直接 push）
- 依赖的 API 服务（GraphQL 调用）

后续就是正式开发自己的博客了。

## [#](https://fecat.win/article/4#github-pages-%E5%89%8D%E7%AB%AF%E5%BC%80%E5%8F%91) Github Pages 前端开发

主要需要两个文件

- `index.html` 即访问 `https://<username>.github.io/` 时 Web 服务器返回的文件
- `404.html` 当访问 `https://<username>.github.io/xxx` 时，Web 服务器匹配规则大致是
	- 若名为 `<username>/xxx` 的仓库存在，则返回资源优先级为： `<username>/xxx/index.html` > `<username>/xxx/404.html` > Github 404 页面
	- 仓库不存在，则返回资源优先级为： `<username>.github.io/index.html` > `<username>.github.io/404.html` > Github 404 页面

## [#](https://fecat.win/article/4#%E5%B0%8F%E7%BB%93) 小结

在 Github 既提供自动部署的 Web 服务器，又提供数据接口服务的现状下，可以仅使用 Github 提供的服务完成搭建 Github issues 动态博客。

相比于 REST API，GraphQL API 由于只提供一种 token 认证方式，导致前端使用 GraphQL API 就需要承担 token 泄漏风险。
