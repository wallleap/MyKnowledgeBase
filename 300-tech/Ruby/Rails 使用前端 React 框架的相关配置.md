---
title: Rails 使用前端 React 框架的相关配置
date: 2024-08-16T11:17:00+08:00
updated: 2024-08-21T10:32:31+08:00
dg-publish: false
source: //ruby-china.org/topics/38631
---

在 rails 中使用 React.js 框架相关的配置记录

## 目标

1. 创建基础项目
2. 配置项目热加载

## 1. 创建基础项目

首先使用 `rails new` 命令创建一个 rails 的基础项目，完整的命令如下：

`rails new railsUseReact --skip-turbolinks --skip-coffee --webpack=react`

执行上述命令时稍微要等待一会，安装完毕之后即可继续。

## 2. 配置项目热加载

现在有了一个模板项目，我们需要创建一个路由用于测试。执行如下命令创建一个新的路由：

`rails g controller Welcome index`

接着我们需要开两个控制台，分别在项目根目录执行如下命令

```
# 控制台1
rails s
```

```
# 控制台2
bin/webpack-dev-server
```

浏览器访问：[http://localhost:3000/welcome/index](http://localhost:3000/welcome/index) 可以看到我们创建的页面

接下来我们先来看看模板项目为我们创建的一个 javascript 目录，它用来存放 React.js 组件相关的代码：

```
➜  railsUseReact git:(master) ✗ tree app/javascript
app/javascript
└── packs
    ├── application.js
    └── hello_react.jsx # 这里是一个React组件我们会使用它
```

然后修改 `app/views/welcome/index.html.erb` 使用 `javascript_pack_tag` 加载我们的 hello_react 组件，代码如下：

文件：app/views/welcome/index.html.erb

```
<h1>Welcome#index</h1>
<%= javascript_pack_tag 'hello_react' %>
```

刷新浏览器我们可以看到如下的输出：

```
Welcome#index
Hello React!
```

以上的修改可以查看 [提交记录](https://github.com/monsterooo/railsUseReact/commit/779d5163159fb4b7113b2bd47f8a0f06c7e4ffe3)

接着我们需要来对 `app/javascript/packs/hello_react.jsx` 组件进行一些改动，`hello_react.jsx` 只保留入口组件。我们把真正的 Hello 组件单独存放。

创建如下的目录和文件：

```
➜  railsUseReact git:(master) ✗ tree app/javascript
app/javascript
├── components
│   └── Hello
│       ├── index.js
│       └── style.css
└── 其他文件
```

修改 Hello 组件为如下的内容：

文件：app/javascript/components/Hello/index.js

```
import React from 'react';
import PropTypes from 'prop-types';
import './style.css';

class Hello extends React.Component {
  render() {
    return (
      <div className="hello">Hello {this.props.name}!</div>
    )
  }
}

Hello.defaultProps = {
  name: 'David'
}

Hello.propTypes = {
  name: PropTypes.string
}

export default Hello;
```

文件：app/javascript/packs/hello_react.jsx

```
import React from 'react'
import ReactDOM from 'react-dom'
import Hello from '../components/Hello';

document.addEventListener('DOMContentLoaded', () => {
  ReactDOM.render(
    <Hello name="React" />,
    document.body.appendChild(document.createElement('div')),
  )
})
```

再次刷新浏览器可以看到如下内容：

```
Welcome#index
Hello React!
```

一切工作正常，接下来就配置 [热加载](https://github.com/rails/webpacker/blob/master/docs/webpack-dev-server.md)

启动 webpack 热加载

文件：config/webpacker.yml

```
dev_server:
  hmr: true # 这里修改为true即可启动热加载
```

然后我们需要更新 `hello_react.jsx` 组件里面的内容使热加载是可以重新渲染组件

文件：app/javascript/packs/hello_react.jsx

```
import React from 'react'
import ReactDOM from 'react-dom'
import Hello from '../components/Hello';

const renderApp = (Component) => {
  ReactDOM.render(
    <Component name="React" />,
    document.body.appendChild(document.createElement('div')),
  )
}
document.addEventListener('DOMContentLoaded', () => {
  renderApp(Hello);
})

if (process.env.NODE_ENV !== 'production' && module.hot) {
  module.hot.accept('../components/Hello', () => {
    const NextHello = require('../components/Hello').default;
    renderApp(NextHello);
  });
}
```

接着安装 `react-hot-loader` 包来自动进行热替换，命令如下：

```
./bin/yarn add react-hot-loader
```

然后修改我们的 `Hello` 组件使用 react-hot-loader 包装

文件：app/javascript/components/Hello/index.js

```
import React from 'react';
import PropTypes from 'prop-types';
import { hot } from 'react-hot-loader';
import './style.css';

class Hello extends React.Component {
  render() {
    return (
      <div className="hello">Hello {this.props.name}!123</div>
    )
  }
}

Hello.defaultProps = {
  name: 'David'
}

Hello.propTypes = {
  name: PropTypes.string
}

export default hot(module)(Hello);
```

重启服务，然后再次访问一些正常。你可以尝试修改一下 `app/javascript/components/Hello/index.js` 组件的内容比如如下代码，可以通过 Chrome 调试中的 Network 查看页面并没有刷新然后组件的数据已经重新加载过了

以上的修改可以查看 [提交记录](https://github.com/monsterooo/railsUseReact/commit/80941a399584c2a3a1d86ed4cc4ce32730cfe0d6)

整个配置完毕，感谢收看

---

很多前端包，比如 `select2` 可以直接打包 `yarn add select2` 引入了。

---

如果是在 rails 中处理的话 建议如非必要，不要上 react 的 router 和 redux 还是仍由 rails 管理路由，就加载个 react.js 在 head 中 之后去找 dom 渲染

---
