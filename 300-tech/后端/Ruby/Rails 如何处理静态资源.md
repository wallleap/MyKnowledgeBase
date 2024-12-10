---
title: Rails 如何处理静态资源
date: 2024-11-14T13:25:51+08:00
updated: 2024-11-14T13:26:42+08:00
dg-publish: false
---

要理解 Rails 如何处理静态资源，首先需要明确什么是静态资源，从广义上讲，**任何不需要动态处理或不需要由 Ruby 代码生成的东西都是静态的**。Rails 应用中通常属于静态资源的一些例子如下 (这些文件不需要由 Ruby 处理，也不是由 Ruby 生成)：

- • 媒体文件 (`jpg`, `png`, `mp4`, `ico` 等)
- • 非交互式文档 (PDF/Excel)
- • 站点地图和机器人 (`sitemap.xml`, `robots.txt`)
- • 字体文件
- • JavaScript、CSS 文件
- - • 这两类看起来仿佛不是静态资源，但是不妨思考一下，当 Rails 程序处理请求时，Ruby 是否实时重新生成任何 JavaScript 或 CSS 文件？是否动态地向这些文件注入任何内容？答案是否定的。因此，从 Rails 的角度来看，所有的 JavaScript 和 CSS 都是静态资源
	- • (但即便如此，它们却不该存在于 `public` 目录内)

了解了什么是静态资源，下面就分步看一下 Rails 生态内如何处理静态资源。

## 1. 浏览器如何请求静态资源

假设我们的 `show.html.erb` 代码包含一个 `<img>` 标签：

```
<h1><%= @post.title %></h1>

<img src="/some-pic.jpg" />
```

浏览器发出请求，收到这个页面作为响应，解析并渲染这个页面。当它遇到 `<img>` 标签时，会向 `/some-pic.jpg` 路径再发出一个请求。

但图片是一个静态资源，我们不需要也不必要为这个图片资源创建一个 route 和 controller，所以我们该如何响应这个请求？

## 2. Rails 如何处理静态资源

根据 Rails 的约定，静态资源可以放在两个目录之一： `public` 或 `app/assets/` (最终，`assets` 目录内的文件都会被处理并复制到 `public` 中)。

### 2.1 `public` 目录

`public` 是静态资源的**根目录**，它所包含的任何资源都可以直接作为 URL 访问。如果 `public` 目录内有以下两个文件：

```
public
├── another
│   └── a.pdf
└── some-pic.jpg
```

那么这两个文件分别可以通过 `/some-pic.jpg` 和 `/another/a.pdf` 的绝对路径被访问。

这里有两个 Rails 遵守网页 URL 模式规范的小细节：

1. \1. 访问 HTML 文件时可以不带 `.html` 扩展名
2. \2. 任何目录内的 `index.html` 文件都作为其所在目录路径的内容提供

即，如果 `public` 目录内的文件如下：

```
public
├── some.html
├── a-folder
│   └── index.html
```

那么：

- • 访问 `/some` 和 `/some.html` 效果相同
- • 访问 `/a-folder` 和 `/a-folder/index.html` 效果相同

> 🤔 所以...
>
> 如果把 `index.html` 直接放入 `public` 目录，那么 `routes.rb` 中设置的 `root` 路由将无效。

### 2.2 缓存

那么既然 `public` 似乎简约且合理，为什么我们还需要 `app/assets/` 目录？当放置静态资源时，该如何从这两个目录中做选择呢？

答案是根据资源在**时间维度上的静态性**，即 “资源能保持多久的静态性”，一天、一月、一年、十年？类似 `favicon.ico` 这样的资源可能数年都不会改变，而 JavaScript/CSS 文件也许每次部署都有所更改，这就是静态资源之间的区别，也是 Rails 提供两个目录的意义所在。

这里所有的区别和意义都无不指向同一个目的：**缓存**。

一般来说，服务器、网络硬件、用户的浏览器等都喜欢尽可能多地缓存静态文件。这些缓存是**基于文件名的**，如果浏览器已经缓存了 `application.js` 文件，那么它就不会再去尝试获取我们的新版 `application.js`。

除非我们每次修改 `application.js` 文件时把文件名也一起修改。

由于**相同的内容永远产生相同的哈希值，而哪怕修改一个字符也会产生迥异的哈希值**，那么把用文件内容计算出的哈希值附加到文件名上，就是那个古老且完美的解决方案。

所以如果浏览器之前缓存的是 `application-3f97d6.js` ，当我们修改了文件内容后，文件名就可能变成 `application-bc12a4.js`，浏览器意识到这是不同的文件，就不会使用缓存而是请求我们的新文件。

> Rails 中称呼这些哈希值为 “指纹”，即 *fingerprint*.

那么我们如何通过一番操作来产生这种带有哈希值的文件呢？又在哪里进行这样一番操作呢？(是的，`assets`!)

### 2.3 `assets` 目录

由于 `public` 目录内的内容都可以被直接访问，显然我们不能直接把原文件直接放到 `public` 目录内进行操作。

Rails 约定把这些“易变” (比如每年一次以上) 的资源放到 `app/assets/` 目录内，并通过 Propshaft **自动产生带有哈希值文件名的文件，并将其复制到 `public` 目录**。

> 在生产环境下 (即 `ENV=production`) Propshaft 确实会进行上述操作，但在开发环境下，Propshaft 直接提供静态资源，不会进行上述操作，所以我们在 `public` 目录中看不到带有哈希值的文件。
>
> 如果开发环境下我们确实想在 `public` 内看到这些文件，可以使用 `rails assets:precompile` 命令 （这个命令通常仅在生产环境使用）。

静态资源放在 `public` 还是 `assets` 目录的另一个权衡是： `assets` 中的静态资源必须通过 *helper methods* 引用，而 `public` 中的静态资源只能通过硬编码引用。

有些人把所有的静态资源都放在 `assets` 目录，几乎不在 `public` 内放任何资源，这也没关系，因为它们最终都会在 `public` 目录内。

### 2.4 如何使用静态资源

由于最终产生的静态资源是带有哈希值的，那么当我们在 HTML 中引用 `assets` 目录内的其它静态资源时，直接使用文件名是行不通的，`<script src="/application.js">` 只会得到 404。

但计算哈希值的过程不是我们处理的，所以我们也不知道某个资源最终会产生什么哈希值，那该怎么办？

Propshaft 提供了一系列查找静态资源的 *helper methods*，如：

- • `stylesheet_link_tag`
- • `font_path`
- • `image_path`
- • `url_to_asset`
- • ...

比如它通过 “指纹注入” 增强了 `javascript_include_tag` helper ，使得 `<%= javascript_include_tag 'application' %>` 会产生 `<script src="application-8ca5d7.js>` 这样带有正确哈希值的文件。

所以我们只需要把静态资源放到 `assets` 目录，并通过相应的 helper 方法引用它们即可，无需考虑 “指纹” 的存在。

### 2.5 开发和生产环境下的静态资源

在**开发模式**下，`ActionDispatch::Static` 中间件负责处理静态资源。请求通过一系列的中间件处理才会到达 `routes.rb`，但在经过 `ActionDispatch::Static` 中间件时，如果它检测到 `public` 目录内有与请求路径匹配的文件，则会直接提供该文件而不会进一步转发请求。

不过根据下面这段源码来看，`ActionDispatch::Static` 中间件只会处理 **GET** 和 **HEAD** 请求：

```
# Static asset delivery only applies to GETs and HEADs
if request.get? || request.head?
  # `find_file` checks for a matching filename in `public`
  if found = find_file(rquest.path_info, accept_encoding: request.accept_encoding)
    serve request, *found
  end
end
```

> ⚠️**Warning**:
>
> 所以要注意静态文件的命名：如果我们的 `routes.rb` 中包含 `resources :posts` 路由，而 `public` 目录内有名为 `posts` 或者 `posts.html` 的文件，那么路由将永远不会被访问。

在**生产模式**下，静态资源可以通过多种方式交付，如使用专用 Web 服务器 Nginx 直接提供 Rails `public` 目录的内容， 或使用 CDN。无论哪种方式，最终结果是相同的：**Web 服务器直接提供静态资源，对静态内容的请求永远不会到达 Rails 应用**，这使得 Rails 应用不必忙于静态内容的请求。

## 3. 番外

### 3.1 Propshat vs. Sprockets

2011 年发布 Rails 3.1 时，Sprockets 是当之无愧的明星。它提供了相当强大但也很复杂的功能，如对 SCSS、CoffeeScript 的支持。

2024 年，在现代 JavaScript 和 CSS 已经发展的如此优秀的今天，Sprockets 臃肿的功能、复杂的配置和低效的性能都已是美人迟暮、将军白头，其提供的很多功能都不再必要，甚至有更好的方案。

所以 Rails 8 开始将 Propshaft 作为新的默认资产管道。Propshaft 充分利用了 HTTP/2 的特性，提高了资源加载速度，且由于它专注于核心功能，唯一的作用几乎就是**为静态资源添加 “指纹识别” 并复制到 `public` 目录**，所以也大大降低了复杂性。

### 3.2 CDN

Web 开发中一个长期存在的原则是： **直接从 Web 服务器提供的唯一内容应该是特定于用户的**，即静态资源相关的内容甚至不应该直接通过我们的主服务器来提供，因为即便只通过 Nginx 来提供，也会浪费主服务器的资源。

更好的方式是使用 CDN。

CDN 大致有两种类型：

1. \1. "Pull" 方式：即 CDN 作为第三方，从源服务器获取资源 (通常只获取一次)，然后无限期地为此资源提供服务。这种方式需要在 Rails 中配置 CDN 子域名：`config.asset_host = "cdn.example.com"`。
2. \2. "Reverse Proxy" 方式：这种方式最初由 Cloudflare 推广，本质上是让 CDN 控制我们的主域名，即所有的请求都通过 CDN，相当于 CDN 直接作为反向代理，这种方式不需要修改 Rails 程序，但需要修改 DNS。

Rails 指南对 Pull 方式的 CDN 配置做了非常详细的介绍，非常值得一看。

## 4. 参考链接

> - • How Propshaft Works: A Rails Asset-Pipeline (Visual) Breakdown
> - • How CDNs Work (Propshaft / Static Assets Pt. 2)
> - • Rails Guides - The Asset Pipeline
