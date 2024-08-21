---
title: 如何规范你的 Git commit？
date: 2023-01-14T09:44:00+08:00
updated: 2024-08-21T10:32:06+08:00
dg-publish: false
aliases:
  - 如何规范你的Git commit？
author: Luwang
category: web
comments: true
cover: //cdn.wallleap.cn/img/post/1.jpg
description: 文章描述
image-auto-upload: true
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
rating: 1
source: #
tags:
  - Git
url: //myblog.wallleap.cn/post/1
---

> **简介：**commit message 应该如何写才更清晰明了？团队开发中有没有遇到过让人头疼的 git commit？本文分享在 git commit 规范建设上的实践，规定了 commit message 的格式，并通过 webhook 在提交时进行监控，避免不规范的代码提交。

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202301142145851.png)

## 背景

Git 每次提交代码都需要写 commit message，否则就不允许提交。一般来说，commit message 应该清晰明了，说明本次提交的目的，具体做了什么操作……但是在日常开发中，大家的 commit message 千奇百怪，中英文混合使用、fix bug 等各种笼统的 message 司空见怪，这就导致后续代码维护成本特别大，有时自己都不知道自己的 fix bug 修改的是什么问题。基于以上这些问题，我们希望通过某种方式来监控用户的 git commit message，让规范更好的服务于质量，提高大家的研发效率。

## 规范建设

### 规范梳理

初期我们在互联网上搜索了大量有关 git commit 规范的资料，但只有 Angular 规范是目前使用最广的写法，比较合理和系统化，并且有配套的工具（IDEA 就有插件支持这种写法）。最后综合阿里巴巴高德地图相关部门已有的规范总结出了一套 git commit 规范。

**commit message 格式**

```
<type>(<scope>): <subject>
```

**type(必须)**

用于说明 git commit 的类别，只允许使用下面的标识。

feat：新功能（feature）。

fix/to：修复 bug，可以是 QA 发现的 BUG，也可以是研发自己发现的 BUG。

- fix：产生 diff 并自动修复此问题。适合于一次提交直接修复问题
- to：只产生 diff 不自动修复此问题。适合于多次提交。最终修复问题提交时使用 fix

docs：文档（documentation）。

style：格式（不影响代码运行的变动）。

refactor：重构（即不是新增功能，也不是修改 bug 的代码变动）。

perf：优化相关，比如提升性能、体验。

test：增加测试。

chore：构建过程或辅助工具的变动。

revert：回滚到上一个版本。

merge：代码合并。

sync：同步主线或分支的 Bug。

**scope(可选)**

scope 用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。

例如在 Angular，可以是 location，browser，compile，compile，rootScope， ngHref，ngClick，ngView 等。如果你的修改影响了不止一个 scope，你可以使用\* 代替。

**subject(必须)**

subject 是 commit 目的的简短描述，不超过 50 个字符。

建议使用中文（感觉中国人用中文描述问题能更清楚一些）。

- 结尾不加句号或其他标点符号。
- 根据以上规范 git commit message 将是如下的格式：

```
fix(DAO):用户查询缺少username属性 
feat(Controller):用户查询接口开发
```

以上就是我们梳理的 git commit 规范，那么我们这样规范 git commit 到底有哪些好处呢？

- 便于程序员对提交历史进行追溯，了解发生了什么情况。
- 一旦约束了 commit message，意味着我们将慎重的进行每一次提交，不能再一股脑的把各种各样的改动都放在一个 git commit 里面，这样一来整个代码改动的历史也将更加清晰。
- 格式化的 commit message 才可以用于自动化输出 Change log。

### 监控服务

通常提出一个规范之后，为了大家更好的执行规范，就需要进行一系列的拉通，比如分享给大家这种规范的优点、能带来什么收益等，在大家都认同的情况下最好有一些强制性的措施。当然 git commit 规范也一样，前期我们分享完规范之后考虑从源头进行强制拦截，只要大家提交代码的 commit message 不符合规范，直接不能提交。但由于代码仓库操作权限的问题，我们最终选择了使用 webhook 通过发送警告的形式进行监控，督促大家按照规范执行代码提交。除了监控 git commit message 的规范外，我们还加入了大代码量提交监控和删除文件监控，减少研发的代码误操作。

**整体流程**

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202301142145853.png)

- 服务注册：服务注册主要完成代码库相关信息的添加。
- 重复校验：防止 merge request 再走一遍验证流程。
- 消息告警：对不符合规范以及大代码量提交、删除文件等操作发送告警消息。
- DB：存项目信息和 git commit 信息便于后续统计 commit message 规范率。

webhook 是作用于代码库上的，用户提交 git commit，push 到仓库的时候就会触发 webhook，webhook 从用户的 commit 信息里面获取到 commit message，校验其是否满足 git commit 规范，如果不满足就发送告警消息；如果满足规范，调用 gitlab API 获取提交的 diff 信息，验证提交代码量，验证是否有重命名文件和删除文件操作，如果存在以上操作还会发送告警消息，最后把所有记录都入库保存。

以上就是我们整个监控服务的相关内容，告警信息通过如下形式发送到对应的钉钉群里：

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202301142145854.png)

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202301142145855.png)

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202301142145856.png)

我们也有整体 git commit 的统计，统计个人的提交次数、不规范次数、不规范率等如下图：

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202301142145857.png)

## 未来思考

git hooks 分为客户端 hook 和服务端 hook。客户端 hook 又分为 pre-commit、prepare-commit-msg、commit-msg、post-commit 等，主要用于控制客户端 git 的提交工作流。用户可以在项目根目录的.git 目录下面配置使用，也可以配置全局 git template 用于个人 pc 上的所有 git 项目使用。服务端 hook 又分为 pre-receive、post-receive、update，主要在服务端接受提交对象时进行调用。

以上这种采用 webhook 的形式对 git commit 进行监控就是一种 server 端的 hook，相当于 post-receive。这种方式并不能阻止代码的提交，它只是通过告警的形式来约束用户的行为，但最终不规范的 commit message 还是被提交到了服务器，不利于后面 change log 的生成。由于公司代码库权限问题，我们目前只能添加这种 post-receive 类型的 webhook。如大家有更高的代码库权限，可以采用 server 端 pre-receive 类型的 webhook，直接拒绝不规范的 git commit message。只要 git commit 规范了，我们甚至可以考虑把代码和 bug、需求关联等等。

当然这块我们也可以考虑客户端的 pre-commit，pre-commit 在 git add 提交之后，然后执行 git commit 时执行，脚本执行没错就继续提交，反之就会驳回。客户端 git hooks 位于每个 git 项目下的隐藏文件.git 中的 hooks 文件夹里。我们可以通过修改这块的配置文件添加我们的规则校验，直接阻止不规范 message 的提交，也可以通过客户端 commit-msg 类型的 hook 进行拦截，把不规范扼杀在萌芽之中。修改每个 git 项目下面.git 目录中的 hooks 文件大家肯定觉得浪费时间，其实这里可以采用配置全局 git template 来完成。但是这又会涉及到 hooks 配置文件同步的问题。hooks 配置文件在本地，如何让 hooks 配置文件的修改能同步到所有使用的项目又成为一个问题。所以使用服务端 hook 还是客户端 hook 需要根据具体需求做适当的权衡。

git hook 不光可以用来做规范限制，它还可以做更多有意义的事情。一次 git commit 提交的信息量很大，有作者信息、代码库信息、commit 等信息。我们的监控服务就根据作者信息做了 git commit 的统计，这样不仅可以用来监控 commit message 的规范性，也可以用来监控大家的工作情况。我们也可以把 git commit 和相关的 bug 关联起来，我们查看 bug 时就可以查看解决这个 bug 的代码修改，很有利于相关问题的追溯。当然我们用同样的方法也可以把 git commit 和相关的需求关联起来，比如我们定义一种格式 feat \*786990（需求的 ID），然后在 git commit 的时候按照这种格式提交，webhook 就可以根据这种格式把需求和 git commit 进行关联，也可以用来追溯某个需求的代码量，当然这个例子不一定合适，但足以证明 git hook 功能之强大，可以给我们的流程规范带来很大的便利。

## 总结

编码规范、流程规范在软件开发过程中是至关重要的，它可以使我们在开发过程中少走很多弯路。Git commit 规范也是如此，确实也是很有必要的，几乎不花费额外精力和时间，但在之后查找问题的效率却很高。作为一名程序员，我们更应注重代码和流程的规范性，永远不要在质量上将就。

> **版权声明：**本文内容由阿里云实名注册用户自发贡献，版权归原作者所有，阿里云开发者社区不拥有其著作权，亦不承担相应法律责任。具体规则请查看《阿里云开发者社区用户服务协议》和《阿里云开发者社区知识产权保护指引》。如果您发现本社区中有涉嫌抄袭的内容，填写侵权投诉表单进行举报，一经查实，本社区将立刻删除涉嫌侵权内容。
