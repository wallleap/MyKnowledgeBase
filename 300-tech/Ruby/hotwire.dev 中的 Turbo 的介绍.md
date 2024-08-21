---
title: hotwire.dev 中的 Turbo 的介绍
date: 2024-08-16T09:28:00+08:00
updated: 2024-08-21T10:32:32+08:00
dg-publish: false
source: //ruby-china.org/topics/40818
---

hotwire.dev 出来也有一段时间了，里面的 turbo 平时好像看到只有在群里讨论，rubychina 的帖子讨论的比较少， 趁着周末，将 turbo 部分照着官方的思路仔细的过了一遍

turbo 大概就是 turbolinks + frame + stimulusReflex 的节奏，有兴趣的朋友也可以稍微看一下

## 简介

Turbo 一个 SPA 工具，不用写任何 js 代码既可以开发出高速的 SPA 程序。 即便不修改服务端生成的 html, Turbo 也可以加速连接和表单提交 (Turbo Drive)。 它将页面切割成独立的切片，切片可以懒加载，操作，就像处理独立组件一样 (TurboFrame)。 Turbo 也可以进利用 HTML 和 一些 CURD tags 来更新局部页面显示 (TurboStream)。

Turbo 目前分三个模块：

- TurboDrive
- TurboFrame
- TurboStream

### Turbo Drive

Turbo Drive 是 Turbolinks 的延续，当 Turbo 安装后（Turbo Drive 模块随 Turbo 启用）Turbo 自动接管连接到同一个域名的 [标签。当点击这些标签后，Turbo 阻止默认的浏览器行为，相对应的，它通过 History API 修改 浏览器 URL, 之后向新页面地址用 fetch 请求，获得 response，之后替换 body, 合并 head.](https://ruby-china.org/topics/40818)

与 Turbolinks 不同的是，Turbo 除了处理 links，还处理 form 的提交和响应，这意味着，整个 web app 都被 Turbo 包裹起来了。不需要再使用以前的 `data-remote=true` 了。（:: 这意味着 link_to remote: true 在 Turbo 下不好用了::）

### Turbo Frame

Turbo 重新发明了 HTML 中的旧技术 Frame，却避免了之前 frame 的缺陷。在 Turbo Frame 的帮助下，你可以对待局部页面就想是自己的组件一样，这个组件的链接和变淡提交只替换自己的那部分。这避免了局部页面在以前需要一些自定义 js 来处理局部页面交互的各种麻烦。

同时 Turbo Frame 将整体页面打成碎片后，这些碎片都可以有自己的 cache 时间线。通过 Turbo::Frame, 你可以将类似 toolbar 设计为一个 frame, 可以使用懒加载，在根目录上做缓存。

### Turbo Streams

通过异步 websocket 通信来更新局部页面是现代交互 web 应用的显著特点。在 TurboStream 的帮助下，通过一个你已经在首屏渲染完成的 html，你就可以实现它。通过设置一些简单的 CURD Container tags, 你就可以通过 web socket 发送 html 片段，在页面上更新新的内容。这些，都不需要独立的 API，不需要搞 JSON, 不需要用 js 实现 html 结构，只使用你已经编辑好的 html，带上一个 update tag，就能搞掂一切。

## 安装 Turbo

1. `gem 'turbo-rails'` 扔到 gemfile
2. `bundle install`
3. `rails turbo:install`

rails turbo:install 会修改你的 application.js 并引入，所以不用管细节

但是如果你需要使用 Turbo 或者 cable 对象，则需要通过下面的语句拿出来。 `import { Turbo, cable } from "@hotwired/turbo-rails"`

## 特别注意

### 有关于 form

Rails UJS 有些 helper，用 XMLHttpRequest 发送链接和表单来处理 ajax，由于 Turbo 取代了这些方法，所以:: 需要确保以下两者其一满足

- 默认条件下跑 Rails 6.1, 这样会关闭 Rails Ujs form 的操作
- 在 `config/application.rb` 中设置 `config.action_view.form_with_generates_remote_forms = false`

### 有关于 link_to remote

在 Turbo 下，link_to remote 不会正常工作！！！ Links that have been made remote will not stick within frames nor will they allow you to respond with turbo stream actions.

Remote Links 既不会被 Frames 正确识别，也不会再 turbo stream 里响应。一个简单的建议就是将 link_to 换为 button_to，button_to 会通过 form 来进行工作。

## Turbo Drive

Turbo drive 是 Turbo 用来提高页面级别导航的模块。它监听了 link 的点击，form 的提交，将这些时间放在后台处理，不用 reload 整个页面，就可以更新 page，跟 Turbolinks 一样。

- page navigation basic
- application visits
- restoration visits
- canceling visits before they start
- disabling turbo on specific page
- displaying progress
- reloading when assets change
- ensuring specific pages trigger a full reload
- setting a root location
- redirect after form submission
- stream after a form submission

### Page Navigation Basic 页面访问基础

(熟悉 turbolinks 的朋友可以略过了，几乎一样)

Turbo drive 将页面访问模式化为 对一个地址，带个 action 的访问。

visit 相当于 从点击到渲染的整个访问声明周期，包含了：

- 改变 browser history
- 请求网络
- 从缓存中拷贝页面拿来显示
- 渲染最终网络返回
- 更新滚动条的位置

Visit 包括两种类型

- application visit：action 为 advance 或者 replace
- restoration visit: action 为 restore

#### Application visits

点击一个启用了 turbo-drive 的 link 或者 js 的 `Turbo.visit(location)`

Application visit 总是发起一个网络请求，当返回值到来是，turbo drive 渲染 html，完成访问。

如果可能，Turbo Drive 会在访问开始的一刹那从缓存里拿到一个页面的预览进行渲染。这会提高经常访问的页面的访问速度的幻觉（比如经常点 news, 之后在你在请求 news 的时候，会从缓存里找到之前 news 的缓存，直接先渲染出来这个缓存，如果有这个缓存的话）

如果 visit 的 location 有滚动条位置，就跳到位置，否则就到页面的最顶上。

application visits 最后会改变 browser history

默认的 visit action 是 advance. 在这中访问中，Turbo Drive 将 browser history 通过 history.pushState 压栈（意味着可以浏览器中用 back 回退）

在 Turbo 的 Ios 和 Android 的 adapter 中，都是新开栈（iOS 是 controller，Android 是 activity）

当你不需要浏览器返回生效的时候，可以使用 `replace visit`, 他使用 `history.replaceState` 在 a 标签中加入 `data-turbo-action="replace"` 即可使用这种方式。js 中的话，使用类似 `Turbo.visit(“/edit”, { action: “replace” })` 即可。

### restoration visits（恢复页面访问）

当浏览器使用 返回或前进 时，turbo drive 会 使用 restoration visit 来处理。如果有缓存的话，turbolinks 会从缓存里拿页面直接渲染，否则会重新请求一下。

restoration visit 的 action 是 restore

#### 小插曲，理解缓存

Turbo drive 维护了一个最近访问的页面缓存。这个缓存主要有俩目的：

- 在恢复访问的时候不用访问网络即可显示页面
- 在进行 application visit 的时候，用一个显示临时的预览来提高访问速度的感知

当进行一个标准访问，即 application visits 的时候，Turbo Drive 立即从缓存里找缓存了的页面来临时预览，同时在网络里也请求这个页面地址。这样会造成一个页面会瞬间加载完毕的假象。

Turbo drive 在渲染页面之前会使用 `cloneNode(true)` 保存页面的副本到缓存中，这意味着任何 attached event listeners 和 associated data 都会被丢弃

`turbo:before-cache` 事件可以被监听，这块可以用来 reset form，收起展开的 ui, 或者销毁第三方 widgets, 以便用来准备新的展示。

```
document.addEventListener(“turbo:before-cache”, **function**() {
  *// …*
})
```

预览页面回在 html 上加 `data-turbo-priview`

我们也可以在 head meta 的 `turbo-cache-control` 中来控制是否开启缓存

- no-preview 只会在 restoration visit 用，在 advance 中不会先提供预览页面
- no-cache 不会缓存

### Canceling Visits Before They Start（在访问生效前取消访问）

在事件 `turbo:before-visit` 中可以使用 `event.detail.url` 来检查访问 location，可以使用 event.preventDefault 来进行取消访问

Restoration visits 是不能被取消的，且不会撞到这个事件。

### disabling turbo drive on specific links 在特殊的链接上关闭 turbo drive

`data-turbo=false`

### Displaying progress 显示进度

在 turbo drive 访问的时候，浏览器不会显示自带的进度条，turbo drive 安装了一个基于 css 的 进度条，用来显示请求的状态

默认状态栏是打开的，超过 500ms 的时候就会显示。可以通过 `Turbo。setProgressBarDelay` 来修改

进度条是 div.turbo-progress-bar，默认加载 document 上，可以通过 css 来修改样式

```
.turbo-progress-bar {
  height: 5px;
  background-color: green;
}

//关闭
.turbo-progress-bar {
  visibility: hidden;
}

```

### Reloading when assets change 当资源改变的时候 reload

Turbo drive 能够检测 head 的变化，确保用户拥有最新的 script 和 styles.

声明 `data-turbo-track=‘reload’` 的 asset，同时这个节点包含版本标识在 URL 中（就是 不同的 js 版本，url 不一样）

### Ensuring Specific Pages Trigger a Full Reload 特殊页面 full reload

```
<meta name="turbo-visit-control" content="reload">

```

### Setting a Root Location 设置 root location

默认情况下 Turbo Drive 只处理同源 url，其他 url，将退回到 full page load.

如果想更进一步只处理某个 scope，可以在 meta 指出

```
<meta name="turbo-root" content="/app">

```

### Redirecting after a form submission 在 form 提交后的跳转

Turbo drive 处理 form 提交跟 link 类似，但是 form 提交与 links 的核心不同点在于：form 提交可以发起带状态的 POST method 的请求，link 只会发起一个 没状态的 http get 请求。

在一个带状态的请求被 form 发起后，turbo drive 期待服务器返回一个 303 响应（303 其实是 302 的一个细化，明确了 method，即再用 GET 请求一下 location 的资源，注意 307 则是向 location 重新发送一次相同 method 的请求）这个 303 响应继续呗请求 且用来导航且在不刷新的情况下更新页面

当然当 response 返回 4xx 或 5xx 的时候，这个要求就被打破了。 当服务器返回 422 或者 服务器返回 500 的时候，显示错误信息或报错 500 页面是允许显示渲染的

turbo 不允许常规的返回 200 的原因是因为浏览器针对 POST 请求的 reload 有一个默认的行为： “Are you sure you want to submit this form again ? (你确定要重复提交此表单么？)”对话框，这个对话框不在 Turbo 的控制范围。Turbo 会等待在 form 提交的当前 url 上渲染，而不是改变 form 的 action，当使用 get reload 这个 action url 的时候，很可能这个路由都不存在。

### Streaming After a Form Submission 提交后的 streaming

在 header 含有 content_type: text/html; turbo-stream 的 form 提交后，服务器也可能响应一个 turbo stream message，会跟随一个 或多个 turbo-stream 元素在 response body 中。这可以让你在不进行 navigating 的情况下更新页面的局部

## 用 turbo frame 解耦

Turbo frame 可以将一个页面划分为几个预定义好的局部。在一个 frame 里的任何 links 或者 form submit 都会被捕获到，在接受到返回后，frame 的内容会被自动更新，无论 server 端提供的是整体文档，或者是只提供了 frame 更新完毕的版本的代码片段，只有对应的 frame 会在 response 中替换原始的内容

使用 `<turbo-frame> 在页面中包围一段frame` 这个 turbo-frame 需要有独立的 id，id 将会被用作匹配替换内容。例如：

```
<body>
  <div id="navigation">Links targeting the entire page</div>

  <turbo-frame id="message_1">
    <h1>My message title</h1>
    <p>My message content</p>
    <a href="/messages/1/edit">Edit this message</a>
  </turbo-frame>

  <turbo-frame id="comments">
    <div id="comment_1">One comment</div>
    <div id="comment_2">Two comments</div>

    <form action="/messages/comments">...</form>
  </turbo-frame>
</body>

```

当编辑连接点击后，返回的内容，包含了 的代码片段会被提出来，更换掉原来的 frame

返回的内容里不在 turbo-frame 下的，会被忽略更新

### lazily loading frames

Frame 不需要在页面加载的时候立即加载。如果 turbo-frame 有 src 的话，这个 src 的引用会继续异步加载

```
<body>
  <h1>Imbox</h1>

  <div id="emails">
    ...
  </div>

  <turbo-frame id="set_aside_tray" src="/emails/set_aside">
  </turbo-frame>

  <turbo-frame id="reply_later_tray" src="/emails/reply_later">
  </turbo-frame>
</body>

```

可以在 tubor-frame 里放东西，就是 src 没加载完成的时候的显示

```
<turbo-frame id="set_aside_tray" src="/emails/set_aside">
  <img src="/icons/spinner.gif">
</turbo-frame>

```

### 懒加载 frame 会提高缓存的应用

将页面做为 frame 加载可以使得页面更加简洁更好实现，同事还有一个重要的原因，是提高缓存使用率。复杂的包含了几个模块的页面很难有效缓存，特别是他们混合了针对独立用户的信息，越多的模块，就会产生越多的独立的依赖 key，缓存越复杂。

Frame 将片段（segment）改变了这种情况，他们将 frame 变为了不同的用户的不同的时间段。有的 frame 依然依赖用户，但是还有的 frame 是不依赖单个用于的，可以全用户共享。

虽然懒加载 frame 成本通常很低，但是你在使用的时候还是要保持慎重，特别是在加载的时候 frame 带来页面抖动 但是如果说你的 frame 本身并不是立即显示在加载页面上，则随便用，比如这是隐藏在 model 下的，或者在首屏以下。

### 在 frame 里跳出 frame 更新 以及在 frame 外面更新 frame

默认情况下，frame 导航只对 frame 内部起作用。当然 frame 内部的 link 和 from 也可以操作整个页面，只需要将 frame 设置一个 `target="_top"` 就好。

```
<body>
  <h1>Imbox</h1>
  ...
  <turbo-frame id="set_aside_tray" src="/emails/set_aside" target="_top">
  </turbo-frame>
</body>

<body>
  <h1>Set Aside Emails</h1>
  ...
  <turbo-frame id="set_aside_tray" target="_top">
    ...
  </turbo-frame>
</body>

```

有事你希望在外面操作 frame，就用 data-turbo-frame 属性即可：

```
<body>
  <turbo-frame id="message_1">
    ...
    <a href="/messages/1/edit">
      Edit this message (within the current frame)
    </a>

    <a href="/messages/1/permission" data-turbo-frame="_top">
      Change permissions (replace the whole page)
    </a>
  </turbo-frame>

  <form action="/messages/1/delete" data-turbo-frame="message_1">
    <input type="submit" value="Delete this message">
    (with a confirmation shown in a specific frame)
  </form>
</body>

```

## Turbo Stream 使用

Turbo stream 通过推送页面变化片段到 turbo-stream 节点。每个 stream 节点都指定了一个 action 和一个 target 去确定在这个节点里的操作。这些节点被服务器的 websocket 或者其他的传输手段进行推送内容

action 有几种情况

- append 在后面添加
- prepend 在最前面添加
- replace 替换
- update 更新
- remove 删除

每个 turbo-stream 节点必须包含一个 template 节点

### http response streaming

Turbo 能够自动的处理 stream element 当他们收到 mime 为 `fext/html; turbo-stream` 的 form 的提交。turbo 会自动的将这个 type 加到 accept 头当中，这样的话，服务器就知道可以响应 stream。

```
def destroy
  @message = Message.find(params[:id])
  @message.destroy

  respond_to do |format|
    format.turbo_stream { render turbo_stream: turbo_stream.remove(@message) }
    format.html         { redirect_to messages_url }
  end
end

```

服务端模板的重用

Turbo streams 的核心就是拥有重用已存在的服务端模板去提供直接更新局部模板。HTML 模板在首次页面加载中渲染的例如 list 中的 message 模板也可以用作进入了新 message 时，动态添加的新的 message。这就是所谓的 html over the wire 的本质 不需要把 message 转为为 json 让 js 处理，之后在客户端渲染出 html 的模板，就是直接 server 渲染好就完了。

```
<!-- app/views/messages/_message.html.erb -->
<div id="<%= dom_id message %>">
  <%= message.content %>
</div>

<!-- app/views/messages/index.html.erb -->
<h1>All the messages</h1>
<%= render partial: "messages/message", collection: @messages %>

# app/controllers/messages_controller.rb
class MessagesController < ApplicationController
  def index
    @messages = Message.all
  end

  def create
    message = Message.create!(params.require(:message).permit(:content))

    respond_to do |format|
      format.turbo_stream do
        render turbo_stream: turbo_stream.append(:messages, partial: "messages/message",
          locals: { message: message })
      end

      format.html { redirect_to messages_url }
    end
  end
end

```

当 form 提交了新的 message，controller 的 action 处理后，局部模板 会呗渲染出来，给 turbo_stream, 生成类似这样的响应

```
Content-Type: text/html; turbo-stream; charset=utf-8

<turbo-stream action="append" target="messages">
  <template>
    <div id="message_1">
      The content of the message.
    </div>
  </template>
</turbo-stream>

```

这个 `messages/message` 也可以用在 edit update，create 操作上，渲染完毕后通过 websocket 发过去进行更新

### 当必要时有序的使用 turbo stream

建议不要一开始就直接完全依赖 turbo stream 做交互设计，确保整个网站或应用 能再 turbo stream 不工作的情况下仍然能正常的使用起来，之后再慢慢来。

Websocket 在若罔状态下有可能连接不上或断开连接，这样如果你完全依赖它，网站就很尴尬了。

### 如何在 stream 使用中跑 js？

stream 限制了 action 的种类，就五种：append, prepend, replace, update 和 remove. 如果你需要更复杂的行为，应该使用 stimulus 来处理了。

### 与 Rails 的交互

靠 gem

[GitHub - hotwired/turbo-rails: Turbo gives you the speed of a single-page web application without having to write any JavaScript.](https://github.com/hotwired/turbo-rails)

在 active record 中使用 Broadcastable concern, 直接 从模型就可以发 WebSoket 更新。使用 `Turbo::Stream::TagBuilder`, 可以渲染出 节点

---

在 action 里面 render turbo_stream 是普通的 http 响应，不会受 websocket 稳定性影响；在 model 里面用 broadcasts 才走 websocket（actioncable）。

---

入行就一直在写 react 最近在自己搭建传统服务端渲染项目，尝试 turbolinks

我所了解的 turbolinks 优点

1. 检索 head, 减少重复的 head 其中的 css 与 javascript 标签的解析与加载时间
2. 预览

但是如果加上

```
turbo-cache-control: no-cache
```

剩下的优点 似乎只剩下 上述 **1** 了？

因为后退经常发生叠加元素或其他非幂等的情况，一般会不 cache,

看了 rubychina 的做法是在 body 上加上了

```
<body id='<%= "#{controller_name}-#{action_name}"%>'>
  <%= yield %>
</body>
```

hotwire + react-rails a 标签跳转到有 react_component 组件页 会出现 组件未被加载的情况，目前只能在 a 标签加上 data-turbo="false"， 请问是不是还有其他更合适方案。
