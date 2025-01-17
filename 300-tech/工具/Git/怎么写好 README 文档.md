---
title: 怎么写好 README 文档
date: 2024-08-22T11:24:30+08:00
updated: 2024-08-23T10:44:24+08:00
dg-publish: false
---

`README` 常常是工程的第一个入口，经常不知道写些什么，如何介绍工程的内容。在网上找了资料，发现这是很多人遇到的问题，也有很多热心人士发表了自己的意见

主要参考 [RichardLitt/standard-readme](https://github.com/RichardLitt/standard-readme)，里面提出了一整套 `REAMDE` 规范，包括编写规范、`linter`、生成器、徽章以及示例

另外 [kylelobo/The-Documentation-Compendium](https://github.com/kylelobo/The-Documentation-Compendium#templates) 和 [如何写好Github中的readme？](https://www.zhihu.com/question/29100816) 也给出了很多建议

`README` 必须满足下面列出的所有要求

> 注意：标准自述文件是为开源库设计的。尽管它在历史上是为 `node` 和 `npm` 项目创建的，但它也适用于其他语言的库和包管理器

要求：

- 称为 `README.md`（大写）
- 如果项目支持 [i18n](https://baike.baidu.com/item/I18N/6771940)，则必须相应地命名该文件：`README.de.md`，其中 `de` 是 [BCP 47](https://tools.ietf.org/html/bcp47) 语言标记。对于命名，请优先考虑语言的非区域子标记。如果只有一个 `README` 且语言不是英语，则允许文本中使用不同的语言而无需指定 `BCP` 标记：例如 README.md 可以是德语。在包含多种语言版本的情况下，`README.md` 固定为英语
- 是一个有效的 `Markdown` 文件
- 章节必须按下面给出的顺序出现。可省略可选部分
- 除非另有说明，否则每个章节必须具有下面列出的标题。如果 `README` 是另一种语言，则必须将标题翻译成该语言
- 不能包含无法访问链接
- 如果有代码示例，则应该像在项目的其余部分中对代码进行 `linted` 一样对它们进行 `linted`

## 目录

> 注意：这只是规范的导航指南，并不是任何符合规范的文档定义或强制使用术语

- 章节
	- 标题（`Title`）
	- 横幅（`Banner`）
	- 徽章（`Badges`）
	- 简短说明（`Short Description`）
	- 详细描述（`Long Description`）
	- 内容列表（`Table of Contents`）
	- 安全（`Security`）
	- 背景（`Background`）
	- 安装（`Install`）
	- 用法（`Usage`）
	- 附加内容（`Extra Sections`）
	- 应用编程接口（`API`）
	- 主要维护人员（`Maintainers`）
	- 致谢（`Thanks`）
	- 参与贡献方式（`Contributing`）
	- 许可证（`License`）
- 定义

## 章节规范

### 标题

- 状态（`status`）：必须
- 必要条件（`requirements`）：
	- 标题必须与 `repository、folder` 和 `package manager` 名称匹配，或者它可以有另一个相关的标题，旁边的 `repository、folder` 和 `package manager` 标题用斜体字和括号括起来。例如：

	```
	# Standard Readme Style *(standard-readme)*
    ```

		如果任何文件夹、存储库或包管理器名称不匹配，则在详细描述中必须有一条说明原因
- 建议（`suggestions`）：符合内容要求，做到显而易见

### 横幅

- 状态：可选
- 必要条件：
	- 不能有自己的标题
	- 必须链接到当前存储库中的本地图像
	- 必须直接出现在标题后面

### 徽章

- 状态：可选
- 必要条件：
	- 不能有自己的标题
	- 必须用换行符分隔
- 建议：使用 [http://shields.io](http://shields.io/) 或类似服务创建和托管图像

输入标签名和徽章信息，选择颜色即可生成静态 `SVG` 图像

```md
https://img.shields.io/badge/ZHUJIAN-BADGE-brightgreen
```

![https://img.shields.io/badge/ZHUJIAN-BADGE-brightgreen](https://img.shields.io/badge/ZHUJIAN-BADGE-brightgreen)

添加链接

在徽章图像上添加链接，点击图像跳转到仓库

```md
[![](https://img.shields.io/badge/ZHUJIAN-BADGE-brightgreen)](https://shields.io)
```

[![https://img.shields.io/badge/ZHUJIAN-BADGE-brightgreen](https://img.shields.io/badge/ZHUJIAN-BADGE-brightgreen)](https://shields.io/)

### 简短说明

- 状态：必须
- 必要条件：
	- 不能有自己的标题
	- 小于 `120` 个字符
	- 以 `>` 开始
	- 一定是单独一行
	- 必须与包装管理器 `说明` 字段中的描述匹配
	- 必须匹配 `github` 的描述（如果在 `github` 上）
- 建议（*`js` 相关*）：
	- 使用 `gh description` 设置并获取 `github` 描述
	- 使用 `npm show . description` 显示本地 `npm` 包中的描述

### 详细描述

- 状态：可选
- 必要条件：
	- 不能有自己的标题
	- 如果任何文件夹、存储库或包管理器名称不匹配，必须在此处说明原因。参考 [标题](https://github.com/RichardLitt/standard-readme/blob/master/spec.md#title)
- 建议：
	- 如果太长，考虑移动到背景章节
	- 介绍构建存储库的主要原因
	- `应该用宽泛的术语来描述模块，一般只在几段中；模块的例程或方法、冗长的代码示例或其他深入的材料的更多细节应该在后面的章节中给出。`
		`理想情况下，稍微熟悉模块的人应该能够刷新他们的内存，而不必点击“向下翻页”。当读者继续阅读文档时，他们应该会逐渐获得更多的知识。`
		~[Kirrily “Skud” Robert, perlmodstyle](http://perldoc.perl.org/perlmodstyle.html)

### 内容列表

- 状态：必须；对于小于 `100` 行的 `README` 是可选的
- 必要条件：
	- 必须链接到文件中的所有 markdown 章节
	- 必须从第二节开始；不要包含标题或目录标题
	- 必须至少有一个深度：必须捕获所有二级标题（`##`）
- 建议：
	- 可以捕获三级（`###`）和四级（`####`）深度的标题。如果是长目录，这些是可选的

### 安全

- 状态：可选
- 必要条件：
	- 如果有必要强调安全问题，则说明。否则，它应该在 `Extra Sections`

### 背景

- 状态：可选
- 必要条件：
	- 涵盖动机
	- 覆盖抽象依赖项
	- 涵盖知识来源：`see also` 也很合适

### 安装

- 状态：默认是必须的，对于文档仓库而言是可选的
- 必要条件：
	- 使用代码块说明如何安装
- 子章节（`subsections`）：
	- 依赖（`dependencies`）：如果有不寻常的依赖项或依赖项必须手动安装，必须提出
- 建议：
	- 链接到编程语言的必备站点：[npmjs](https://npmjs.com/)、[godocs](https://godoc.org/) 等
	- 包括安装所需的任何系统特定信息
	- 增加一个 `updating` 章节是有用的

### 用法

- 状态：默认是必须的，对于文档仓库而言是可选的
- 必要条件：
	- 使用代码块说明常见用法
	- 如果兼容于 `cli`，用代码块说明其用法
	- 如果可导入，用代码块表示导入功能和用法
- 子章节：
	- `CLI`：如果命令行功能存在则是必要的
- 建议：
	- 覆盖可能影响使用的基本选项：例如，如果是 `javascript`，则覆盖 `promises/callbacks`（对 `ES6` 而言）
	- 可以指向示例代码的可执行文件

### 附加内容

- 状态：可选
- 必要条件：
	- 无
- 建议：
	- 这不应称为附加内容。可容纳 `0` 个或多个章节，每个章节必须有其标题
	- 这应该包含任何其他相关的内容，放在用法之后，应用程序接口之前
	- 具体来说，如果安全章节不够重要，不需要放在上面的话，可以放在这里

### 应用编程接口

- 状态：可选
- 必要条件：
	- 描述开放的函数和对象
- 建议：
	- 描述签名、返回类型、回调和事件
	- 覆盖类型不明显的地方
	- 描述注意事项
	- 如果使用外部 `api` 生成器（如 `go-doc、js-doc` 等），那么指向外部 `api.md` 文件即可

### 主要维护人员

- 状态：可选
- 必要条件：
	- 列出存储库的维护人员，以及联系他们的一种方式（例如 `github` 链接或电子邮件）
- 建议：
	- 这应该是一个负责这个仓库人员的小名单。这不应该是所有拥有访问权限的人，例如整个组织，而是应该 `ping` 并负责管理和维护存储库的人
	- 列出过去的维护者是友好的

### 致谢

- 状态：可选
- 必要条件：
	- 必须称为 `Thanks, Credits` 和 `Acknowledgements` 的其中一种
- 建议：
	- 陈述任何对项目开发有重大帮助的人或事
	- 列出公共联系人的超链接（如果可以的话）

### 参与贡献方式

- 状态：必须
- 必要条件：
	- 说明用户可在哪里提问
	- 说明是否接受 `PR`
	- 列出参与的任何要求；例如，对提交进行审核
- 建议：
	- 如果有的话，链接到 `CONTRIBUTING` 文件
	- 尽可能友好
	- 链接到 `github` 问题库
	- 链接到行为准则（`Code of Conduct`）。`Coc` 通常位于贡献部分或文档中，或者设置在整个组织的其他位置，因此可能不需要在每个存储库中包含整个文件。但是，强烈建议始终链接到代码，无论它位于何处
	- 也可以在这里增加一个小节，列出贡献者

### 许可证

- 状态：必须
- 必要条件：
	- 列出许可证全名或标识符，参考 [`SPDX`许可证列表](https://spdx.org/licenses/)。对于未授权的存储库，添加标识 `UNLICENSED`。对于许可证的详细信息，添加 `see license in <filename>` 并链接到许可文件。（参考 [npm](https://docs.npmjs.com/files/package.json#license) 设置）
	- 列出许可证拥有者
	- 必须是最后一节
- 建议：
	- 链接到本地存储库中较长的许可证文件

## 定义

*提供这些定义是为了理清上述使用的任何术语*

- 文档仓库（`Documentation repositories`）：仓库中没有任何功能代码，比如 [RichardLitt/knowledge](https://github.com/RichardLitt/knowledge)

## README 生成器

[RichardLitt/generator-standard-readme](https://github.com/RichardLitt/generator-standard-readme) 提供了一个 `npm` 命令行工具，用于生成 `README` 文件

### 安装

```sh
$ npm install --global yo generator-standard-readme
```

### 使用

命令行输入

```sh
$ yo standard-readme
```

接下来会提出很多问题，然后生成一个 README 文件。所有问题如下：

```sh
What do you want to name your module?
What is the description of this module?
Do have a banner image?
    Where is the banner image? Ex: 'img/banner.png'
Do you want a TODO dropped where your badges should be?
Do you want a TODO dropped where your long description should be?
Do you need a prioritized security section?
Do you need a background section?
Do you need an API section?
What is the GitHub handle of the main maintainer?
Do you have a CONTRIBUTING.md file?
Are PRs accepted?
Is an MIT license OK?
    What is your license?
Who is the License holder (probably your name)?
Use the current year?
    What years would you like to specify?
```

### 示例

```sh
$ yo standard-readme
? ==========================================================================
We're constantly looking for ways to make yo better! 
May we anonymously report usage statistics to improve the tool over time? 
More info: https://github.com/yeoman/insight & http://yeoman.io
========================================================================== Yes
? What is the name of your module? zj
? What is the description of this module? yo使用示例
? Do have a banner image? No
? Do you want a standard-readme compliant badge? Yes
? Do you want a TODO dropped where more badges should be? Yes
? Do you want a TODO dropped where your long description should be? Yes
? Do you need a prioritized security section? No
? Do you need a background section? Yes
? Do you need an API section? No
? What is the GitHub handle of the main maintainer? zjZSTU
? Do you have a CONTRIBUTING.md file? No
? Are PRs accepted? Yes
? Is an MIT license OK? Yes
? Who is the License holder (probably your name)? zjZSTU
? Use the current year? Yes
   create README.md
```

生成文件如下：

````markdown
# zj                                       // 标题

[![](https://cdn.wallleap.cn/img/pic/illustration/202408221228930.svg?imageSlim)](https://github.com/RichardLitt/standard-readme)    // 徽章
TODO: Put more badges here.                // 可以添加更多徽章

> yo使用示例                                // 简短说明

TODO: Fill out this long description.      // 详细说明

## Table of Contents                       // 下面的章节列表，每个章节都可以点击跳转

- [Background](#background)
- [Install](#install)
- [Usage](#usage)
- [Maintainers](#maintainers)
- [Contributing](#contributing)
- [License](#license)

## Background                               // 背景

## Install                                  // 安装

```
```

## Usage                                    // 用法

```
```

## Maintainers                              // 主要维护人员

[@zjZSTU](https://github.com/zjZSTU)

## Contributing                             // 参与贡献方式

PRs accepted.

Small note: If editing the README, please conform to the [standard-readme](https://github.com/RichardLitt/standard-readme) specification.

## License                                  // 许可证

MIT © 2019 zjZSTU
````
