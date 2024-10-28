---
title: DHH：Rails 7 将在 2021+ 年为 JavaScript 提供三个很好的答案
date: 2024-08-16T10:07:00+08:00
updated: 2024-08-21T10:32:31+08:00
dg-publish: false
source: //ruby-china.org/topics/41661
---

DHH 新写了一篇文章介绍 Rails 7 里处理 JavaScript 的三个方案。

[https://world.hey.com/dhh/rails-7-will-have-three-great-answers-to-javascript-in-2021-8d68191b](https://world.hey.com/dhh/rails-7-will-have-three-great-answers-to-javascript-in-2021-8d68191b)

这三个方案是：

1. 基于 Hotwire 的轻 JavaScript 默认盏。
2. 通过 jsbundling-rails 支持 esbuild/rollupjs/webpack 构建 js，然后交给 asset pipeline 处理，适合前端构建比较复杂的项目。
3. 前后端分离的 API mode。

其实这些方案现在就可以用了，前两个通过安装对应的 gem 实现，API mode 则已经在 6 中内置。

机翻如下（原文有很多链接，这里不处理了）：

---

Rails 从一开始就毫无歉意地是全栈的。我们一直在寻求为现代 Web 开发提出的所有主要基础设施问题提供更多默认答案。从与数据库交谈，到发送和接收电子邮件，到连接 Web 套接字，到呈现 HTML，再到与 JavaScript 集成。这种全栈策略是 Rails 成功的关键，但它仍然是一个持久的争议源。什么东西太多了？什么还不够？

为了始终如一地回答这个常青问题，我们关注 Rails 学说，尤其是 The Menu Is Omakase 的第三个支柱。这就是为什么我们如此担心默认设置，也是为什么替代选项如此重要的原因。

多年来，没有人比问题的 JavaScript 部分更担心默认值，或者更仔细地检查替代品。尤其是最近，随着不断出现的流失和根本性变化，新的选择成为人们关注的焦点。但是经过大量实验，我相信我们现在对 Rails 7 有了一个可靠的答案。

## Rails 7 将默认导入映射的 Hotwire

在 Rails 7 中，我们将 Webpacker、Turbolinks、UJS 替换为导入地图以及来自 Hotwire 的 Turbo 和 Stimulus 作为默认设置。这是我们提供的最全面的答案。Turbolinks + UJS 提供了一个很好的基线，让应用程序感觉他们拥有单页应用程序的活泼，但是一旦你需要做任何动态的事情，它几乎就是带上你自己的 JS-FU。

Hotwire 的野心要大得多。Turbo 已经掌握了 Turbolinks 的基础知识，并在表单提交、框架和流方面走得更远。现在，您真的可以只用 Turbo 创建大量动态元素（无需编写任何自定义 JavaScript！）。最重要的是，还有 Stimulus，这是您已经拥有的 HTML 的适度 JavaScript 框架（通过从 Rails 服务器端视图生成它）。它是对后端使用 JSON 的重型 JavaScript 客户端应用程序的完整替代方案。

这个 Hotwire 设置随后通过 HTTP2、ES6/ESM 和 Rails 7 中的导入映射提供服务。我已经深入解释了为什么这个三重威胁会改变现代网络的游戏规则，所以我不会在这里重复这些论点。一世 ' 基于 JavaScript CDN 的包管理是真正的交易。我在这个 Alpha 预览版中介绍了所有这些如何协同工作：Rails 7 中没有 Webpack 的现代 JavaScript。

我们已经在这个堆栈上运行了几个星期的 HEY 版本，它是一个桃子（完整的报告和推出即将推出！）。构建不需要单独的监视过程，不需要与配置搏斗，即时重新加载，根本不需要节点工具，没有整体性能或能力损失。这是世界上最好的——对我们在 Basecamp 来说。

我说“为我们”是因为 HEY 的开发显然考虑到了这一愿景。它从来没有装满数千个 JavaScript 文件（我们在基于浏览器的 ESM 版本中发布了大约 130 个）。它从不使用需要转译器的代码，比如 JSX。这是 Hotwire 为发布而磨练的地方。所以我想这当然是它在这里工作得很好。

但同时，HEY 也没有失态。事实上，HEY 作为现代 Web 应用程序所寻求的保真度就在那里。这是一个与 Gmail 正面竞争的应用程序，Gmail 长期以来一直被认为是您肯定希望使用厚客户端框架、JSON 兜售和所有最新的转译器技巧构建的那种应用程序。

然而 Gmail 下载了大约 3 兆字节值得的 JavaScript 来呈现其收件箱。HEY 下载不到 60 KB。如果您可以使用此堆栈构建 Gmail 竞争对手，并获得数以万计付费客户的广泛赞誉，那么您可能可以使用它构建任何东西。

这意味着它是 Rails 中默认堆栈的绝佳选择。

## Rails 7 完全支持传统的 JS 捆绑

但另一个 Rails 教义支柱是我们正在努力搭建一个大帐篷。Hotwire 和导入地图显然不是每个人的答案。这是一个很好的答案。这是默认答案。但这不是唯一的答案。Rails 需要成为开发传统单页 JavaScript 应用程序的绝佳框架——完成客户端路由、繁重的状态管理以及该风格的所有其他复杂性。它将会是。

因此，在 Rails 7 中，我们同时提供了一种很好的默认方式来避免处理整个 node/npm/bundling 设置，并提供一个完全支持的替代路线，包括所有这些。但我们将以与以前不同的方式来做这件事。

打包机大约在五年前诞生，其使命是使 JavaScript 捆绑管道易于为那些不一定有兴趣成为 JavaScript 专家的 Rails 开发人员使用。当时 ES6 需要在浏览器中广泛使用转译，而 npm 需要访问 node 的包生态系统，确实没有办法绕过它。我们要么接受这一现实，要么将 Rails 降级为仅作为此类应用程序的 API。我们选择了拥抱。

但是今天，为 Webpacker 所做的权衡开始变得不那么有意义了。它有点卡在两条更清晰的路径之间的泥泞中间。有一条完全放弃捆绑管道的新路径，它更容易设置，依赖更少，Ruby 和 JavaScript 之间的分工也更少。然后是全包路径，我们根本不试图隐藏或包装 JavaScript 的复杂性。我们只是提供了一个桥梁，通过它生成的 JavaScript 可以在 Rails 应用程序中使用，但让 JavaScript 生态系统来提供所有答案。

我编写了 Webpacker 的初始版本，我们在 Basecamp 和 HEY 中都使用了它，效果很好，它作为当时和现在之间的过渡阶段很好地为社区服务。但我认为五年前所做的权衡并不能很好地为现在或未来服务。

取而代之的是，Rails 7 将提供一条更简洁、更传统的 JavaScript 世界替代路径。您可以通过将源代码保存在 app/javascript 中，通过 package.json 定义运行构建脚本来开发 JavaScript，然后将最终构建交给 app/assets/builds 中的资产管道，因此它们可以进行摘要标记，CDN- 序言，并在应用程序中提供。这是 jsbundling-rails gem 采用的方法。

这个 gem 提供了 esbuild、rollup.js 或 Webpack 所需的基本设置。它为所选的 bundler 安装基线依赖项，在适当的地方准备一个配置文件，并依赖于 app/javascript/application.js 中的默认入口点和放置在 app/assets/builds 中的构建的双重约定。就是这样！管理和更新依赖项的责任流向了开发人员。

如果您在 React 和 JSX 或其他需要转译步骤的 JavaScript 框架上全力以赴，那么您可能应该选择这条路径。你可以用 import maps 来做 React，但它会通过 htm，对于那些全押的人来说，这很可能是一种妥协。

如果您今天使用 Webpacker，那么切换到通过 jsbundling-rails gem 提供的捆绑程序之一是一个非常温和的跳跃。你甚至不必坚持使用 Webpack。从根本上说，它们都以相同的方式工作：获取一个切入点，生成一个构建。我们最初在 Basecamp 3 中采用了这条路径，从 Webpacker 转换为 esbuild。但是有了 HEY，我们迈出了直接导入地图的全部步骤。这两种选择都需要付出很多努力。JavaScript 仍然是 JavaScript。您主要是在调整一些导入路径。

如果你还没有强烈的偏好，但你知道你需要一个打包器，我鼓励你从 esbuild 开始。它非常快，并且为 JSX 甚至 TypeScript 进行了预配置。

## Rails 7 将为 JavaScript 提供三个明确的选择

最后，您可以选择简单地使用 Rails 作为 API。将使用它的单页 JavaScript 应用程序完全保留在不同的项目和存储库中。Rails 长期以来一直通过 --api 支持这条路径，并将继续这样做。这不是我为中小型团队推荐的路径，但如果您所在的大型组织致力于在前端和后端部门之间建立高墙，那么这可能是有意义的。

这就是 Rails 7 及更高版本中 JavaScript 的故事。带有 Hotwire 和导入映射的默认路径，使用与流行的 JavaScript 捆绑器之一进行瘦集成的备用路径，最后是带有单独的前端存储库的严格 API 路径。

2021 年现代 Web 开发现实的三个可靠答案。

Rails 7 的 alpha 版本很快就会发布，我们打算在年底之前庆祝最终版本。请通过帮助测试和改进这些版本来帮助我们实现目标！
