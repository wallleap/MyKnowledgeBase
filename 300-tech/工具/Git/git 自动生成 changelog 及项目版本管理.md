---
title: git 自动生成 changelog 及项目版本管理
date: 2024-08-22T03:53:45+08:00
updated: 2024-08-22T03:54:34+08:00
dg-publish: false
---

## 版本管理

**1.version 类别介绍**

每个 npm 包中都有一个 package.json 文件，如果要发包的话，package.json 中的 version 就是版本号了。

version 字段结构为：'0.0.0-0'

分别代表：大号.中号.小号 - 预发布号，对应 majon.minor.patch-prerelease

下面来看看 npm 中 version 的类别及描述。

- major
	- 如果没有预发布号，则直接升级一位大号，其他位都置为 0
	- 如果有预发布号：
		- 1.  中号和小号都为 0，则不升级大号，而将预发布号删掉。即 2.0.0-1 变成 2.0.0，这就是预发布的作用
		- 2.  如果中号和小号有任意一个不是 0，那边会升级一位大号，其他位都置为 0，清空预发布号。即 2.0.1-0 变成 3.0.0
- minor
	- 如果没有预发布号，则升级一位中号，大号不动，小号置为空
	- 如果有预发布号：
		- 1.  如果小号为 0，则不升级中号，将预发布号去掉
		- 2.  如果小号不为 0，同理没有预发布号
- patch
	- 如果没有预发布号：直接升级小号，去掉预发布号
	- 如果有预发布号：去掉预发布号，其他不动
- premajor
	- 直接升级大号，中号和小号置为 0，增加预发布号为 0
- preminor
	- 直接升级中号，小号置为 0，增加预发布号为 0
- prepatch
	- 直接升级小号，增加预发布号为 0
- prerelease
	- 如果没有预发布号：增加小号，增加预发布号为 0
	- 如果有预发布号，则升级预发布号

**2.version 运用**

执行命令 `npm version` 可以自动更改 `package.json` 中的版本号。

比如，`package.json` 包中的 version 是 `1.1.1`，然后执行如下命令：

```
<span data-line-num="1">npm version prerelease -m "这里你可以添加此次更新版本号的描述"</span>
```

执行完之后，`package.json` 的版本号则会变成 `1.1.1-0`，同时，在 git 中会多一个 commit log。

值得注意的是，执行 `npm version` 必须保证工作目录是干净的，没有任何未提交的文档，否则会报错。

一个项目都是由很多人一起合作，然而每个人的开发习惯，提交格式并不统一，这对于统计 bug 和 需求有一定困难。

所以，commit message 规范就格外重要。

**格式化 commit 的好处**

1.提供更多的历史信息，方便快速浏览。

2.可以过滤某些文档，方便快速查找

3.可以直接从 commit 生成 change log。

**commit message 的格式**

```
<span data-line-num="1"></span>
<span data-line-num="2">类别: 做了什么 简短描述做了什么</span>
<span data-line-num="3"></span>
<span data-line-num="4">详细描述做了什么（可省略）</span>
<span data-line-num="5"></span>
```

- 类别 type：
	- feat: 新功能
	- fix：修补
	- docs: 文档
	- style: 格式
	- refactor: 重构（既不是新增，也不是代码变动）
	- test：增加测试
	- chore：构建过程中或辅助工具的变动。

```
<span data-line-num="1">例如：</span>
<span data-line-num="2"></span>
<span data-line-num="3">fix: bug(<span>123</span>) 修补了…………</span>
<span data-line-num="4"></span>
<span data-line-num="5">修改了....</span>
<span data-line-num="6"></span>
<span data-line-num="7">feat: 需求(<span>123</span>) 新增了某某功能</span>
<span data-line-num="8"></span>
<span data-line-num="9">详细描述某某功能（可省略）</span>
<span data-line-num="10"></span>
```

## **自动生成 change log**

如果所有的 commit 都符合格式，那么发布新版本时，change log 就可以用脚本自动生成。

conventional-changelog 就是生成 Change log 的工具。

先命令全局安装：

```
<span data-line-num="1">$ npm install -g conventional-changelog-cli</span>
```

然后进入项目，执行命令：

```
<span data-line-num="1">$ <span>cd</span> my-project</span>
<span data-line-num="2">$ conventional-changelog -p angular -i CHANGELOG.md <span>-s</span></span>
```

> conventional-changelog -p angular -i CHANGELOG.md -w 则会在命令行中生成 log，并不会生成文件

上面命令不会覆盖 CHANGELOG.md 文档，只会在文档前面新增，如果你想生成所有发布的 Change log，要改为运行下面的命令：

```
<span data-line-num="1">$ conventional-changelog -p angular -i CHANGELOG.md <span>-s</span> -r 0</span>
```

如若你不需要手动修改 CHANGELOG.md 文档，可以自动提交 CHANGELOG.md 文档：

```
<span data-line-num="1">conventional-changelog -p angular -i CHANGELOG.md <span>-s</span> &amp;&amp; git commit -m <span>"docs: 更改CHANGELOG.md"</span></span>
```

每次执行这么长的命令，很难，我们可以配置在 npm 中

```
<span data-line-num="1">{</span>
<span data-line-num="2">  <span>"scripts"</span>: {</span>
<span data-line-num="3">    <span>"changelog"</span>: <span>"conventional-changelog -p angular -i CHANGELOG.md -s"</span> </span>
<span data-line-num="4">  } </span>
<span data-line-num="5">}</span>
```

下次运行直接命令:`npm run changelog` 即可。

这时候，可以看见，项目中就会多了一个 CHANGELOG.md 文档，如果你的提交不符合规范，那么在最开始使用的时候，这个文档里，可能什么都没有。

而且，要注意的是，在我们每次 changelog 之前，都必须要使用 `npm version` 升级版本，否则，commit 一直都会有之前的记录。
