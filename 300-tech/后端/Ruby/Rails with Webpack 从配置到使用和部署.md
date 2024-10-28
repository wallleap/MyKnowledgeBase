---
title: Rails with Webpack 从配置到使用和部署
date: 2024-08-10T09:12:00+08:00
updated: 2024-08-19T03:35:44+08:00
dg-publish: true
---

这里的内容是由像和你一样的工程师们一行一行创造出来的，为了保护你的权益，同时也为了激励我们继续创造更好的内容，蛋人网所有内容禁止一切形式的转载和复制

JavaScript 发展太快，浏览器端虽然对 ES6 语法的支持还不是很完善，但是现在前端的各种框架和代码都在用 ES6 写了，有的甚至在用 ES7 了，因此就需要把 ES6 或者 ES7 的语法转换成传统的 ES5，然后再交给浏览器来运行。

这个专题我们就来说说怎样在 Rails 中使用 [Webpack](https://webpack.js.org/) 来管理 JS 和 CSS。

> 虽然 Rails 在 **5.1** 之后已经默认添加了 Webpack 的支持，但是需要我们指定开启，而且需要一定的定制，我们这个专题就细细来给大家说下怎样安装，配置和使用 Webpack，以及怎样组织你的 JS 代码。

## Webpack 介绍

如果你已经知道 webpack 是什么东西，用来干什么的，你可以放心的 [点击这里](https://eggman.tv/lessons/rails-with-webpack-up-and-running#p1) 直接跳到下一部分阅读。

Webpack 是现在非常流行的基于 nodejs 的静态资源 (assets) 管理框架，比如 JS 和 CSS 的压缩合并。

Webpack 中可以配置各种插件来完成资源的中间编译处理，比如 ES6/7 转成 ES5([babel](https://babeljs.io/))，把 sass 转换成 css，图片的无损压缩 ([image-webpack-loader](https://github.com/tcoopman/image-webpack-loader)) 等等。

当然我们使用 Webpack 来管理 JS 的另一个好处是不需要再手工下载第三方的 JS 库了 (比如 jQuery，React 等等)，直接在 nodejs 中安装就行了。

## Rails 的 Asset Pipeline

在 Rails 中有自己的静态资源管理组件 Asset Pipeline，负责静态资源的压缩合并等，和 Webpack 作用是类似的，但是默认情况下 Asset Pipeline 是不支持把 ES6 转化成 ES5 的，虽然有对应的 gem 可以做到，但是还不是很完善，而且这些 gem 也都是基于 nodejs 的 babel 来实现的，所以如果你想要在项目中使用 ES6，那么还是建议采用 Webpack。

> 如果在 Rails 的项目中采用 Webpack 的话，从 Webpack 的角度来说，我们就不再需要 Rails 的 Asset Pipeline 了，所有静态资源的管理都可以交给 Webpack 来处理；但是 Asset Pipeline 毕竟是 Rails 中的重要组件，相信有一定 Rails 使用经验的同学也难以割舍，而且现在很多已有项目都是采用 Asset Pipeline 方式来组织的，因此我们可以保留 Asset Pipeline，根据实际情况，结合两个一块使用。

## 安装

Rails 在 **5.1** 之后已经默认添加了 Webpack 的支持，但是需要指定开启，而且需要一定的定制。

Webpack 在 Rails 中的安装有以下要求：

- Ruby 2.2+
- Rails 4.2+
- Node.js 6.4.0+
- Yarn 0.20.1+

> Yarn 是另一个 nodejs 的 package 管理工具，和 npm 的作用类似，你可以使用 npm 来安装 yarn。

如果你是在老的 Rails 项目中使用 Webpack 并且 Rails 版本低于上面要求的话就需要先升级下 Rails 和 Ruby。

Ruby 和 Rails 的安装这里就不再重复了，nodejs 的安装请直接使用 `brew` 或者 `yum` 或者 `apt-get` 进行安装，不过还是建议直接从 [nodejs](https://nodejs.org/en/) 官方网站直接下载安装，这样才够新:)，然后安装 `yarn`:

```
$ npm install -g yarn
```

下面从已有项目和新项目两个角度分别说下怎样安装 Webpack。

> 如果你是新建项目的话同样建议你先把下面的**已有项目**部分看一遍，流程是相似的。

### 已有项目

首先打开你 Rails 项目的 `Gemfile`，添加下面的 gem:

```
gem 'webpacker', '~> 2.0'
```

然后安装 gem:

```
$ bundle
```

然后你就可以在你项目根目录下查看所有的 Webpack 命令了：

```
$ rails webpacker
```

初始化 webpack 配置：

```
# 如果网络失败请科学上网
$ rails webpacker:install
```

Rails 的 webpacker gem 已经添加了各个主流 JS 框架的支持: **React, Angular, Vue 和 Elm**，这里以**React**为例，如果你想要在你的项目中使用**React**的话，可以再运行下面的命令：

```
$ rails webpacker:install:react
```

现在项目中已经自动添加了很多配置文件，这里的大部分文件都是不用动的，个别文件会稍后来说明。

下面需要添加 ES6 中对类似 `module.exports` 语法的支持，修改你项目根目录中的**.babelrc**文件，这个是刚刚自动创建的，在 `presets` 配置项中添加 `es2015`:

```
...
"react",
"es2015"
...
```

然后安装 nodejs 的 package`babel-preset-es2015`:

```
$ yarn add babel-preset-es2015
```

至此我们的配置就都结束了。

### 新项目

首先保证你本地的 Rails 版本 `>= 5.1`，然后创建项目：

```
# 如果网络失败请科学上网
$ rails new project-name --webpack=react
```

当然如果你使用其他的 JS 框架的话上面的 `react` 可以替换为**vue | angular | elm**。

然后再参考上面的**已有项目**的 [这个部分](https://eggman.tv/lessons/rails-with-webpack-up-and-running#p2) 就可以了。

## 使用

我们写的 ES6 的代码，需要 webpack 能够实时的编译成 ES5 的语法，Webpack 提供了独立的开发服务器，让我们可以在开发环境下实时的看到编译后的代码，并且提供热加载的机制 (通过 WebSocket)，首先启动 Webpack 的开发服务器：

```
$ bin/webpack-dev-server
```

上面的命令会启动**0.0.0.0:8080**端口，Webpack 就是通过这个 server 来提供实时编译后的资源，本地的开发环境的资源都会从 0.0.0.0:8080 加载，当然这里就需要使用 Rails 已经提供的**helper**方法来完成。

所有 JS 和 CSS 文件此时就可以放在 `app/javascript` 这个目录中，要注意这里是 `app` 目录下的 `javascript` 目录，而不是 `app/assets` 下的。

在 `app/javascript` 目录中有一个 `packs` 目录，所有前端直接引用的文件都需要放在这个目录中，也就是说 Webpack 会自动编译 `app/javascript/packs` 目录下的文件，而子模块可以直接放在 `app/javascript` 下，这个原理和 Rails 的 Asset Pipeline 类似。

> 如果你想要修改默认的 8080 号端口，请参考 `config/webpacker.yml` 文件中的设置。

Rails 已经帮我们在 `app/javascript/packs` 目录中建立一个测试文件 `hello_react.jsx`:

```
import React from 'react'
import ReactDOM from 'react-dom'
import PropTypes from 'prop-types'

const Hello = props => (
  <div>Hello {props.name}!</div>
)

Hello.defaultProps = {
  name: 'David'
}

Hello.propTypes = {
  name: PropTypes.string
}

document.addEventListener('DOMContentLoaded', () => {
  ReactDOM.render(
    <Hello name="React" />,
    document.body.appendChild(document.createElement('div')),
  )
})
```

我们直接使用这个文件来测试，下面大家需要建立一个测试页面，建立对应的 controller 和路由，这个就不再赘述了，在你的测试页面中添加以下内容：

```
<%= javascript_pack_tag 'hello_react' %>
```

注意上面使用的是 `javascript_pack_tag`，不是 `javascript_include_tag`，这个就是专门用来引用 webpack 资源的 helper 方法，在本地开发环境下，它会输出绝对地址:

`http://0.0.0.0:8080/packs/hello_react.js`

请求到 webpack 的开发服务器，`javascript_pack_tag` 方法会查找 `app/javascript/packs目录下的文件`，对应的是 `stylesheet_pack_tag`，用来引入 `packs` 目录下的 css 文件，css 的使用我们在下一章节中会讲述。

最后启动你的 rails server:

```
$ rails s
```

打开你的浏览器访问**localhost:3000**，访问你的页面，应该就能看到 `hello David` 的内容了。

接下来我们会深入的讲述怎样使用 Webpack 以及怎样组织前端代码，点击这里观看第二部分：[Rails with Webpack: 深入使用和前端代码组织](https://eggman.tv/lessons/rails-with-webpack-keypoints)

这里的内容是由像和你一样的工程师们一行一行创造出来的，为了保护你的权益，同时也为了激励我们继续创造更好的内容，蛋人网所有内容禁止一切形式的转载和复制

> 这是专题文章 [Rails with Webpack: 从创建，配置到使用和部署](https://eggman.tv/c/s-rails-webpack-react-template) 的第二部分，观看第一部分请点击这里：[Rails with Webpack: 安装，配置和使用](https://eggman.tv/lessons/rails-with-webpack-up-and-running)

上一部分中我们已经配置好了项目，并且把 demo 的代码跑了起来，在这一部分中我们来讲述一下：

- 怎样组织代码 - 模块化
- 怎样从后端传递变量
- CSS 的使用
- 引入第三方模块
- 部署

## 代码组织 - 模块化

模块化相信大家都明白，简单说就是把独立的功能放到独立的文件中，同时做到代码重用，可以把前端的不同页面看做不同的大模块，页面的元素看做小模块 (可以重用)。

不同的框架就会引入不同的设计思路，模块化也是采用 Webpack 后更方便的组织代码的一种方式。

**全局变量**

比如像**jQuery**，以及已经在项目中引入了**React**，基本上在所有页面都需要用到的，因此可以把它们放入全局引用中。

打开 `app/javascript/packs/application.js` 文件，修改为以下内容：

```
import React from 'react'
import ReactDOM from 'react-dom'

window.React = React;
window.ReactDOM = ReactDOM;
```

我们把 `React` 和 `ReactDOM` 这两个模块放入了 `window` 对象中，也就是全局对象中，然后打开目模板文件 `app/views/layouts/application.html.erb`(或者你自定义的模板文件)，修改为以下内容：

```
<!DOCTYPE html>
<html>
  <head>
    <title>RailsWebpackReactTemplate</title>
    <%= csrf_meta_tags %>

    <!-- 1 -->
    <%= yield :css %>
  </head>

  <body>
    <%= yield %>
    <!-- 2 -->
    <%= javascript_pack_tag 'application' %>
    <!-- 3 -->
    <%= yield :js %>
  </body>
</html>
```

解释一下上面的代码：

- #1: 自定义代码块 `css`，方便之后动态的插入具体页面的样式文件
- #2: 采用 `javascript_pack_tag` 方法引入上面的 `app/javascript/packs/application.js` 文件
- #3: 自定义代码块 `js`，方便插入页面的 JS 模块

注意上面把 JS 的加载放到了页面的最底部，css 放到了上部，这个也是前端页面优化的一个技巧，大家要注意。

然后需要大家建立一个对应的测试页面，controller 和路由，这里不再赘述，假如我们建立的 controller 为 `WelcomeController`，对应的 view 为 `app/views/welcome/index.html.erb`，首先在 `app/javascript` 目录下建立一个新目录 `welcome`，然后在其中建立文件 `index.js`:

```
class WelcomePage extends React.Component {
  render() {
    return (
      <h1>hello <span>{this.props.name}</span></h1>
    )
  }
}

module.exports = WelcomePage
```

然后在 `app/javascript/packs` 目录中建立文件 `welcome.js`:

```
import WelcomePage from '../welcome/index'

document.addEventListener('DOMContentLoaded', () => {
  ReactDOM.render(
    <WelcomePage name="eggman.tv" />,
    document.body.appendChild(document.createElement('div')),
  )
})
```

然后在对应的 view `app/views/welcome/index.html.erb` 中：

```
<%= content_for :js do %>
  <%= javascript_pack_tag "welcome" %>
<% end %>
```

重启你的**webpack-dev-server**进程。

> 每次新建，删除，重命名 `app/javascript` 目录下文件的时候都需要重启**webpack-dev-server**进程

打开浏览器访问测试页面，你应该就能看到**Hello eggman.tv**的内容了。

大家可以参考这种思路来设计你的其他页面。

## 怎样从后端传递变量

上面的 `app/javascript/packs/welcome.js` 文件在被前端页面加载时就会运行了，如果需要传递后端的变量到这个 JS 中应该怎么办呢？

简单来说可以在页面加载完毕后手工初始化 JS 组件，把后端的变量以 JSON 的方式传递到 JS 组件中。

但是默认情况下所有的模块，比如上面的 `WelcomePage` 模块在前端的页面中都是不可访问的，也就是说被 Webpack 编译到了局部变量中，下面就需要修改 Webpack 的配置来导出模块为全部变量。

打开 `config/webpacker/shared.js`，修改 `output` 的配置项：

```
module.exports = {
  entry: packPaths.reduce(
    // ...
    }, {}
  ),

  output: {
    library: 'App',
    filename: '[name].js',
    path: output.path,
    publicPath: output.publicPath
  },
  // ...
}
```

添加了 `library: 'App'` 的配置，这样通过 `javascript_pack_tag` 方法引入的模块都会被放入 `App` 这个全局变量中，当然对应引入的 JS 文件也需要 `export` 出对应的变量。

修改 `app/javascript/packs/welcome.js` 为：

```
import WelcomePage from '../welcome/index'
import assign from 'object-assign'

const WelcomeApp = {
  init(options) {
    assign(this, {}, options);

    ReactDOM.render(
      <WelcomePage {...this} />,
      document.body.appendChild(document.createElement('div'))
    )
  }
}

module.exports = WelcomeApp
```

上面定义了一个 `WelcomeApp` 的对象，内部是一个 `init` 的方法，把接收变量 `options` 最终又传递进了 `WelcomeApp` 组件中。

最后再修改一下视图 `app/views/welcome/index.html.erb`:

```
<%= content_for :js do %>
  <%= javascript_pack_tag "welcome" %>
  <script>
    App.init(<%= {name: 'eggman.tv'}.to_json.html_safe %>)
  </script>
<% end %>
```

直接调用了 `App.init` 方法，把后端变量传递了进去，代码还是比较直观的，我们就不解释了。

这时大家可以重启一下**webpack-dev-server**，打开页面应该可以看到同样的结果。

> 这种办法会比较灵活，但是负面效果就是如果你的页面有多个 JS 文件引入，就会有 `App` 变量互相覆盖的影响，解决办法就是在 `config/webpacker/shared.js` 中配置的 `App` 参数，实际上是可以用 `[name]` 来替换为当前的 js 文件名称，当然需要处理成比较友好的形式，尤其在有子目录的情况下，比如 `app/javascript/packs/hello/world.js`，可以格式化为 `HelloWorld`，这里就不再细述了，大家可以研究一下。但是从实际情况来说，最简单的做法就是可以在每个 js 文件引用后直接调用 `App.init` 的方法就可以了

在 Webpacker gem 的官方文档中有提供 [怎样传递变量](https://github.com/rails/webpacker#pass-data-from-view) 到 JS，简单来说就是把后端变量放入 DOM 中，然后再读取 DOM 属性，同样可以作为参考。

## CSS 的使用

CSS 在 Webpack 中的使用有两种方式。

### 方法 1

直接建立 css 文件，比如建立文件 `app/javascript/packs/test.scss`:

```
h1 {
  color: blue;

  span {
    color: green;
  }
}
```

然后在 `app/views/welcome/index.html.erb` 最上面添加：

```
<%= content_for :css do %>
  <%= stylesheet_pack_tag 'test' %>
<% end %>

<%= content_for :js do %>
  # ...
```

### 方法 2

采用模块化的方式导入 JS 中。

首先建立文件 `app/javascript/welcome/index.scss`:

```
h1 {
  color: blue;

  span {
    color: red;
  }
}
```

然后在 `app/javascript/packs/welcome.js` 最上面添加：

```
import '../welcome/index.scss'
```

最后把 `app/views/welcome/index.html.erb` 中的:

```
<%= stylesheet_pack_tag 'test' %>
```

修改为:

```
<%= stylesheet_pack_tag 'welcome' %>
```

这里的关键是在哪个 JS 文件中 `import` 的 css，最终会被编译成和 JS 文件同名的.css 文件，因此这里直接采用 `stylesheet_pack_tag` 方法引入 `welcome` 的 CSS 文件。

> 当然这里不要忘了，你依旧可以采用传统的 Asset Pipeline 的方式来组织和使用 CSS

## 引入第三方模块

比如想要使用**jQuery**，现在就不需要手工下载 jquery 的源文件，或者采用 gem 的方式来安装了，可以直接通过 `yarn` 安装 jquery:

```
$ yarn add jquery
```

因为 jQuery 是作为全局使用的，因此你依旧可以修改 `app/javascript/packs/application.js` 文件，修改为以下内容：

```
import jQuery from 'jquery';
import React from 'react';
import ReactDOM from 'react-dom';

window.$ = window.jQuery = jQuery;
window.React = React;
window.ReactDOM = ReactDOM;
```

或者你也可以在你需要使用的模块中单独 `import`，不过你要避免在同一个页面的不同 JS 模块中 `import` 同一个模块，否则会造成重复加载。

## 部署

webpacker gem 中已经有个现成的 rake 任务 `rails webpacker:compile` 来处理 `app/javascript/packs` 下面资源的编译，而且已经和已有的 `rails assets:precompile` 任务挂在了一起，因此对于部署我们只需要保证：

- 服务器中已经安装 nodejs, npm 和 yarn
- 部署时运行了 `rails assets:precompile` 任务

## 其他

- 如果你本地有多个项目需要修改**webpack-dev-server**监听端口号可以修改配置文件: `config/webpacker.yml`，包括默认的 `app/javascript` 目录路径等等
- 如果你不想每次开发都启动**webpack-dev-server**和**rails**两个 server，你可以使用 [foreman](https://ddollar.github.io/foreman/)
- 不同后缀资源的文件由不同的 webpack 插件来处理，可以参看 `config/webpack/loaders` 目录中各种配置文件
