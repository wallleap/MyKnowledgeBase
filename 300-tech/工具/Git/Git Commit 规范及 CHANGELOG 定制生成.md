---
title: Git Commit 规范及 CHANGELOG 定制生成
date: 2024-08-22T11:01:04+08:00
updated: 2024-08-22T11:03:10+08:00
dg-publish: false
source: https://juejin.cn/post/7033385543094239245
---

## 前言

日常项目开发维护时，有些 `CHANGELOG` 可能是手动去维护更新的。这种手动维护日志的情况，虽说可能让内容看起来更整洁、美观且通透，但需要投入的资源也相应增多。而且如果是在多人协助开发的情况下，日志可能与 `Commit` 并不会实时同步，而在发版时同步堆积日志看起来也并不是什么明智之举。而依据 `Commit` 自动生成 `CHANGELOG` 或许会是更好的选择，之前写过 [Git Commit 规范](https://juejin.cn/post/6985500205554597918 "https://juejin.cn/post/6985500205554597918") 这么一篇文章 ，其主要内容大致就是如何规范提交及自动生成 `CHANGELOG`，不过自动生成的 `CHANGELOG` 内容并不那么美观。因此，在这我希望可以更进一步去完善它，希望可以通过自动化命令去规范代码信息提交，也希望可以定制 `CHANGELOG` 内容风格，使它看起来更整洁美观，而这就是本篇文章要讲述的内容：**Git Commit 规范及 CHANGELOG 定制生成**。

> demo 项目地址：[github.com/niezicheng/…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fniezicheng%2Fchangelog-standard "https://github.com/niezicheng/changelog-standard")

## Commitizen（规范 commit 命令行工具）

### 安装

```zsh
npm install -g commitizen
```

下面使用 `git cz` 和 `VSCode 插件` 两种方式规范 `git` 提交，个人推荐 `git-cz` 方式 (可配置性更高)

### git-cz

以下两种方式：**全局安装** 和 **项目中使用** 二者选其一即可，两种方式同时使用也行

创建 `changelog.config.js` 配置文件, 详细配置内容可查看 [demo 项目地址](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fniezicheng%2Fchangelog-standard "https://github.com/niezicheng/changelog-standard") 或 [git-cz 官方文档](https://link.juejin.cn/?target=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2Fgit-cz "https://www.npmjs.com/package/git-cz")

```js
touch changelog.config.js
```

#### 全局安装

全局安装 `git-cz`

```zsh
npm install -g git-cz
```

安装后直接用 `git cz` 替换 `git commit` 提交信息

```zsh
git cz
```

![](https://cdn.wallleap.cn/img/pic/illustration/202408221101832.png?imageSlim)

#### 项目中使用

`commitizen` 安装配置 `git-cz`

```zsh
commitizen init git-cz --save-dev --save-exact
```

执行上面安装完成后 `package.json` 中会自动添加以下内容

```json
"devDependencies": { "git-cz": "^4.8.0" }, "config": { "commitizen": { "path": "./node_modules/git-cz" } }
```

安装后直接用 `cz` 替换 `git commit` ，全局安装 `git-cz` 后也可以使用 `git cz` 命令替换 `git commit`

```zsh
cz
```

![](https://cdn.wallleap.cn/img/pic/illustration/202408221101833.png?imageSlim)

### VSCode 插件方式

1. `VSCode` 商店安装 [Visual Studio Code Commitizen Support](https://link.juejin.cn/?target=https%3A%2F%2Fmarketplace.visualstudio.com%2Fitems%3FitemName%3DKnisterPeter.vscode-commitizen "https://marketplace.visualstudio.com/items?itemName=KnisterPeter.vscode-commitizen") 插件
2. `command + shift + p` 输入 `conventional commit`，就能有 `commitizen` 的效果
3. 根目录下创建自定义配置文件 `.cz-config.js` ，添加对应配置内容信息即可。具体配置见 [demo 项目地址](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fniezicheng%2Fchangelog-standard "https://github.com/niezicheng/changelog-standard")（效果如下图所示）

**注意⚠️**：修改完 `.cz-config.js` 配置文件内容后可能需要重启 `VSCode` 才会生效。貌似是插件的 [bug](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FKnisterPeter%2Fvscode-commitizen%2Fissues%2F199 "https://github.com/KnisterPeter/vscode-commitizen/issues/199")

![](https://cdn.wallleap.cn/img/pic/illustration/202408221101834.png?imageSlim)

## standard-version

### 安装

```zsh
npm i --save-dev standard-version
```

结合笔者自身日常实际中项目在 `package.json` 配置 `script` 命令。执行 `npm run release` 命令即可，参数可以查看下面常见参数说明或 [standard-version 官方文档](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fconventional-changelog%2Fstandard-version "https://github.com/conventional-changelog/standard-version")。

```json
"scripts": { "prerelease": "standard-version --release-as patch --dry-run", // 预览生成 CHANGELOG 内容 "release": "standard-version --release-as patch" // 升级补丁版本、生成/更新 CHANGELOG 文件 }
```

#### 全局安装

```zsh
npm i -g standard-version
```

全局安装完成后，我们可以直接在终端使用 `standard-version` 命令。

为防止 `standard-version` 命令的默认行为混乱代码，下面示例命令均携带 `--dry-run` 参数先进行结果预览

```zsh
standard-version --release-as patch --dry-run // 预览生成 CHANGELOG 内容并配置生成补丁版本
```

### 默认行为

```zsh
standard-version --dry-run // 预览
```

`standard-version` 命令主要做了如下事情：

1. 缓存当前文件变化
2. 将历史回退至最后一个 `git` 标签的位置（查看 `package.json` 中提供的信息来定位 `git` 标签）
3. 更新版本号（未指定时按照默认规则进行版本更新）
4. 产出 `changelog` 文件（可自定义输出内容风格）
5. 提交变动（创建一个新的提交）
6. 打上新版本的 `git tag`

### 更新规则（默认）

```zsh
standard-version --dry-run
```

- `feature` 提交会更新 `minor`
- `bug fix` 会更新 `patch`
- `BREAKING CHANGES` 会更新 `major`

更多版本相关 [知识内容](https://juejin.cn/post/6932803368226127886#heading-8 "https://juejin.cn/post/6932803368226127886#heading-8")

![](https://cdn.wallleap.cn/img/pic/illustration/202408221101835.png?imageSlim)

### standard-version 自定义

- 自定义产出的 `CHANGELOG` 内容
- 自定义 `standard-version` 提供的行为
- 自定义更新指定版本号

#### CHANGELOG 内容定制

**自定义配置方式**

1. 在 `package.json` 文件内进行配置
2. 新建 `.versionrc` 配置文件进行配置（配置文件可以为 `.versionrc`, `.versionrc.json` 或 `.versionrc.js`）

更多详细内容可查看 [standard-version 官方文档](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fconventional-changelog%2Fstandard-version "https://github.com/conventional-changelog/standard-version")

下面以构建 `.versionrc` 配置文件的方式为例进行配置

根目录下创建 `.versionrc` 配置文件， `.versionrc` 配置可查看 [demo 项目地址](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fniezicheng%2Fchangelog-standard "https://github.com/niezicheng/changelog-standard")，配置详情参数可查看 [Conventional Changelog Configuration Spec](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fconventional-changelog%2Fconventional-changelog-config-spec%2F "https://github.com/conventional-changelog/conventional-changelog-config-spec/")

```zsh
touch .versionrc // 创建 .versionrc 配置文件
```

执行 `npm run prerelease` 命令预览 `changelog` 内容（前提是在前面 **安装** 内容部分配置了相应的 `script`）。如下图所示：

```zsh
npm run prerelease
```

![](https://cdn.wallleap.cn/img/pic/illustration/202408221101836.png?imageSlim)

#### standard-version 行为定制

`standard-version` 默认行为是可以定制化配置的（是否跳过对应的行为）

- 方式一：`.versionrc` 文件中配置

```json
{ "skip": { "bump": true, // 缓存变化，并重置 git 状态至最近的 tag 节点 "changelog": true, // 自动产出 changelog 文档 "commit": true, // 提交变动 "tag": true // 在 git 中增加 tag 标识 } }
```

- 方式二：命令行配置

通过命令参数的形式配置，具体见 [常见参数说明 --skip](https://link.juejin.cn/?target=)

```zsh
standard-version --skip.changelog false
```

#### 发布版本定制

通过命令参数的形式配置，具体见 [常见参数说明 ----release-as](https://link.juejin.cn/?target=)

### 常见参数说明

#### \--dry-run（内容预览）

**预览**命令执行后生成 `CHANGELOG` 内容效果

```zsh
standard-version --release-as patch --dry-run
```

#### \--release-as（指定发布）

**手动控制** `standard-version` 命令升级发布的版本

```zsh
standard-version --release-as [option | 具体版本号]
```

主要 `option` 选项

```css
major：主版本号（大版本） [2.0.0] minor：次版本号（小更新） [1.1.0] patch：补丁号（补丁） [1.0.1]
```

更多版本 [知识内容](https://juejin.cn/post/6932803368226127886#heading-8 "https://juejin.cn/post/6932803368226127886#heading-8")

示例：

```zsh
standard-version --release-as patch // 升级补丁类型版本号 standard-version --release-as 6.6.6 // 强制静态版本号升级 standard-version --first-release // 不升级版本情况下标记一个版本
```

#### \--prerelease（预发布）

当即将发布 **大版本改动** 前，但是又不能保证这个版本的功能 `100%` 正常，这个时候可以发布先行版本（预发布）

```zsh
standard-version --prerelease [option]
```

主要 `option` 选项

```css
alpha：内部版本 [1.0.0-alpha.0] beta：公测版本 [1.0.0-beta.0] rc：候选版本 [1.0.0-rc.0]
```

基本使用

```zsh
// 首次执行 standard-version --prerelease [option] 版本会从 1.0.0 变为 1.0.1-[option.]0 // 后续在此执行 standard-version --prerelease [option] 版本会从 1.0.1-[option.]0 变为 1.0.1-[option.]1
```

**注意⚠️**：单独使用 `--prerelease` 参数其实并不符合 `SemVer` 规范，结合 `--release-as` 使用或许是个不错的选择

结合 `--release-as` 使用

```scss
// 版本会从 1.0.0 变为 2.0.0-alpha.0 npx standard-version --release-as major --prerelease alpha // 版本会从 2.0.0-alpha.0 变为 2.0.0-alpha.1 npx standard-version --prerelease alpha
```

#### \--skip（行为控制）

是否跳过 `standard-version` 命令的某个默认行为

```zsh
standard-version --skip.[option] [false/true]
```

主要 `option` 选项

```sql
bump 缓存变化，并重置 git 状态至最近的 tag 节点 changelog 自动产出 changelog 文档 commit 提交变动 tag 在 git 中增加 tag 标识
```

## Commitlint & Husky（规范 commit message 验证）

- `commitlint`: 用于检查提交信息;
- `husky` 是 `hook` 工具，作用于 `git-commit` 和 `git-push` 阶段

### 依赖安装

```zsh
npm i husky @commitlint/config-conventional @commitlint/cli -D
```

### 配置信息

#### commitlint 配置

根目录下新建 `commitlint.config.js` 配置文件，也可以在 `package.json` 文件内配置 (二者选其一)

**commitlint.config.js 文件**

**`rules` 也可以不添加使用默认的 `rules` 配置**

[更多 rules 配置信息](https://link.juejin.cn/?target=https%3A%2F%2Fcommitlint.js.org%2F%23%2Freference-rules "https://commitlint.js.org/#/reference-rules")

```js
module.exports = { extends: ['@commitlint/config-conventional'], rules: { 'body-leading-blank': [2, 'always'], // body 开始于空白行 'header-max-length': [2, 'always', 72], // header 字符最大长度为 72 'subject-full-stop': [0, 'never'], // subject 结尾不加 '.' 'type-empty': [2, 'never'], // type 不为空 'type-enum': [2, 'always', [ 'feat', // 新特性、需求 'fix', // bug 修复 'docs', // 文档内容改动 'style', // 不影响代码含义的改动，例如去掉空格、改变缩进、增删分号 'refactor', // 代码重构 'test', // 添加或修改测试 'chore', // 不修改 src 或者 test 的其余修改，例如构建过程或辅助工具的变动 'revert', // 执行 git revert 打印的 message ]], } };
```

**package.json 文件**

```json
{ "commitlint": { "extends": ["@commitlint/config-conventional"] } }
```

#### husky 配置

**旧 `husky` 配置，我试了下好像没啥效果, 可以使用下面新的配置**

**package.json 文件**

```json
{ "husky": { "hooks": { "commit-msg": "commitlint -E HUSKY_GIT_PARAMS" } } }
```

**新的 `husky` 配置**

**Active Hooks**

执行下面命令会生成一个 `.husky` 文件夹

```zsh
npx husky install
```

**Add Hooks**

向 `.husky` 文件那添加内容

```zsh
npx husky add .husky/commit-msg 'npx --no-install commitlint --edit $1'
```

代码提交未按照配置规范，如下图所示：

![](https://cdn.wallleap.cn/img/pic/illustration/202408221101837.png?imageSlim)

正确规范提交，不过使用 `git commit` 提交信息要带上 🎸，而使用前面的 `git cz` 则通过文件配置即可

![](https://cdn.wallleap.cn/img/pic/illustration/202408221101838.png?imageSlim)

最终的 `CHANGELOG` 文件内容效果如下图所示：

![](https://cdn.wallleap.cn/img/pic/illustration/202408221101839.png?imageSlim)

参考：

- [Git Commit 规范（Conventional Commit）](https://juejin.cn/post/6985500205554597918 "https://juejin.cn/post/6985500205554597918")
- [standard-version 官方文档](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fconventional-changelog%2Fstandard-version "https://github.com/conventional-changelog/standard-version")
- [git-cz 官方文档](https://link.juejin.cn/?target=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2Fgit-cz "https://www.npmjs.com/package/git-cz")
- [自动产出changelog-第一节：规范提交代码](https://link.juejin.cn/?target=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000039794568 "https://segmentfault.com/a/1190000039794568")
- [自动产出changelog-第二节：自动产出](https://link.juejin.cn/?target=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000039813329 "https://segmentfault.com/a/1190000039813329")
- [commit规范及自动生成changelog](https://juejin.cn/post/6863342912320339982 "https://juejin.cn/post/6863342912320339982")
