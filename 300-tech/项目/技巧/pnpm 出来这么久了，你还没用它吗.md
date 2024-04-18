---
title: pnpm 出来这么久了，你还没用它吗
date: 2023-02-17 14:53
updated: 2023-02-17 14:58
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - pnpm 出来这么久了，你还没用它吗
rating: 1
tags:
  - tool
category: web
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: null
url: //myblog.wallleap.cn/post/1
---

关于软件包的管理工具，大家比较熟知的是 `npm` 和 `Yarn`，今天给大家介绍一个新的包管理工具[pnpm](https://link.juejin.cn/?target=https%3A%2F%2Fpnpm.io%2F "https://pnpm.io/")。

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202302171453768.webp)

通过此文，您将学习到`pnpm`的以下知识：

1. 基础使用、常用命令等
2. 依赖包是如何管理和存储的
3. workspace协议，支持`monorepo`
4. 对比`npm`、`Yarn`，有什么优势
5. 如何将`npm`、`Yarn`转为`pnpm`

阅读此文之前，需要了解的知识：

1. [内容可寻址存储(CAS)](https://link.juejin.cn/?target=https%3A%2F%2Fvibaike.com%2F106979%2F "https://vibaike.com/106979/")
2. [软链接（符号链接）](https://link.juejin.cn/?target=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E7%25AC%25A6%25E5%258F%25B7%25E9%2593%25BE%25E6%258E%25A5%2F7177630%3Ffr%3Daladdin "https://baike.baidu.com/item/%E7%AC%A6%E5%8F%B7%E9%93%BE%E6%8E%A5/7177630?fr=aladdin")
3. [硬链接](https://link.juejin.cn/?target=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E7%25A1%25AC%25E9%2593%25BE%25E6%258E%25A5%2F2088758%3Ffr%3Daladdin "https://baike.baidu.com/item/%E7%A1%AC%E9%93%BE%E6%8E%A5/2088758?fr=aladdin")

## 简介

> 节约磁盘空间并提升安装速度

`pnpm`代表`performant npm`（高性能的npm），同`npm`和`Yarn`，都属于`Javascript`包管理安装工具，它较`npm`和`Yarn`在性能上得到很大提升，被称为**快速的，节省磁盘空间的包管理工具**。

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202302171453769.webp)

当使用 `npm` 或 `Yarn` 时，如果你有 100 个项目使用了某个依赖（dependency），就会有 100 份该依赖的副本保存在硬盘上，而在使用 `pnpm` 时，依赖会被存储在内容可寻址的存储中，所以：

1. 如果你用到了某依赖项的不同版本，只会将不同版本间有差异的文件添加到仓库。 例如，如果某个包有100个文件，而它的新版本只改变了其中1个文件。那么 `pnpm update` 时只会向存储中心额外添加1个新文件，而不会因为仅仅一个文件的改变复制整新版本包的内容。
2. 所有文件都会存储在硬盘上的某一位置。 当软件包被安装时，包里的文件会硬链接到这一位置上对应的文件，而不会占用额外的磁盘空间。 这允许你跨项目地共享同一版本的依赖。

因此，您在磁盘上节省了大量空间，这与项目和依赖项的数量成正比，并且安装速度要快得多！

摘自官网：[pnpm.io/zh/motivati…](https://link.juejin.cn/?target=https%3A%2F%2Fpnpm.io%2Fzh%2Fmotivation "https://pnpm.io/zh/motivation")

## 基本使用

## 安装与使用

### 安装

通过`npm`安装，也可以在官网查看[其他安装方式](https://link.juejin.cn/?target=https%3A%2F%2Fpnpm.io%2Fzh%2Finstallation "https://pnpm.io/zh/installation")

```sh
npm install -g pnpm
```

通过下述命令查看已安装的`pnpm`的版本

```sh
pnpm -v
```

### 使用

1. 初始化，生成`package.json`文件

```sh
pnpm init
```

2. 安装依赖

```sh
pnpm install xxx
```

3. 运行`package.json`中定义的`scripts`脚本，启动服务即可

```sh
pnpm run xxx
```

**示例：创建一个`vue3`项目**

通过`pnpm create`使用`vite`套件新建一个`vue3`的项目，[直达vue官方链接](https://link.juejin.cn/?target=https%3A%2F%2Fv3.cn.vuejs.org%2Fguide%2Finstallation.html%23vite "https://v3.cn.vuejs.org/guide/installation.html#vite")

```
# 使用pnpm create 启动套件（vite，只有存在的套件才可以）创建模板项目
pnpm create vite <project-name> -- --template vue
cd <project-name>
pnpm install
pnpm dev
复制代码
```

通过上述操作，我们学到了`pnpm`项目的初始化、安装依赖、启动服务等，可以运行起来，感受一下它和`npm`运行速度的差异。

## 常用命令

[官网查看更多命令](https://link.juejin.cn/?target=https%3A%2F%2Fpnpm.io%2Fzh%2Fcli%2Fadd "https://pnpm.io/zh/cli/add")

### 设置源

- 查看源：**`pnpm config get registry`**

- 切换源：**`pnpm config set registry <淘宝源或其他源地址>`**

### 初始化

- 初始化package.json：**`pnpm init`**

注意：`pnpm init`只能一键快速生成`package.json`文件，如果要一步一步填写每个属性的值生成`package.json`文件，则需要通过`npm init`生成，如果要一键快速生成，需要增加`-y`参数`npm init -y`来生成。

### 管理依赖

- 安装依赖包到 `dependencies` ：**`pnpm add <pkg>`**

- 安装依赖包到`devDependencies`：**`pnpm add -D <pkg>`**

- 安装依赖包到`optionalDependencies`：**`pnpm add -O <pkg>`**

- 全局安装依赖包：**`pnpm add -g xxx`**

- 安装项目全部依赖：**`pnpm install`，别名`pnpm i`**

- 更新依赖包：**`pnpm update`，别名`pnpm up`**

- 删除依赖包：**`pnpm remove`，别名`pnpm rm/uninstall/un`**

### 查看依赖

- 查看本地安装的依赖：**`pnpm list`，别名`pnpm ls`**

- 查看全局安装的依赖：**`pnpm list --global`，别名`pnpm ls --g`**

- 检查过期的依赖：**`pnpm outdated`**

### 运行脚本

- 运行自定义脚本：**`pnpm run xxx`，别名`pnpm xxx`**

- 运行`test`测试脚本：**`pnpm test`**

- 启动套件创建项目： **`pnpm create`**

- 运行`start`启动命令：**`pnpm start`**

### 发布依赖包

- 发布依赖包：**`pnpm publish`**

### 管理node环境

可实现`nvm`、`n`等`node`版本管理工具，安装并切换`node.js`版本的功能。

- 本地安装并使用：**`pnpm env use <node版本号>`**

- 全局安装并使用：**`pnpm env use --global <node版本号>`**

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202302171453770.webp)

## 依赖管理

### node_modules

> pnpm基于符号链接来创建非扁平化的`node_modules`

对比`npm`和`pnpm`安装的`node_modules`：

| npm                                                                             | pnpm                                                                            |
| ------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| ![image.png](https://cdn.wallleap.cn/img/pic/illustration/202302171453771.webp) | ![image.png](https://cdn.wallleap.cn/img/pic/illustration/202302171453772.webp) |
| 所有依赖包平铺在`node_modules`目录，包括直接依赖包以及其他次级依赖包                                       | `node_modules`目录下只有`.pnpm`和直接依赖包（`vue、vite、...`），没有其他次级依赖包                      |
| 没有符号链接                                                                          | 直接依赖包的后面有符号链接的标识                                                                |

那`pnpm`怎么管理这些依赖包的呢？带着这一问题，我们继续探究。

详细看一下`pnpm`生成的`node_modules`目录如下：

```
▾ node_modules
    ▸ .bin
    ▸ .pnpm
    ▸ @vitejs    ->符号链接
    ▸ vite       ->符号链接
    ▸ vue        ->符号链接
    .modules.yaml
复制代码
```

`node_modules` 中只有一个 `.pnpm` 的文件夹以及三个符号链接`@vitejs/plugin-vue`、 `vite` 和 `vue`。 这是因为我们的项目只安装了`@vitejs/plugin-vue`、 `vite` 和 `vue`三个依赖，`pnpm`使用符号链接的方式将项目的直接依赖添加到`node_modules`的根目录下，**也就是说`node_modules`目录下只有我们项目中依赖的`dependencies`、`devDependencies`和一个`.pnpm`目录**。

以`vite`依赖包举例，看一下`vite`依赖包和`.pnpm`目录里都有些什么：

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202302171453773.webp)

展开`vite`依赖包，我们会有两个疑问：

1. `vite`是一个符号链接，那它的实际位置在哪里？
2. 依赖的其他次级依赖在哪里？

**`vite`的实际位置**

`.pnpm`称为虚拟存储目录，以平铺的形式储存着所有的项目依赖包，每个依赖包都可以通过`.pnpm/<name>@<version>/node_modules/<name>`路径找到实际位置。

即直接依赖的`vite`包 符号链接到路径：`.pnpm/vite@2.9.12/node_modules/vite`，`vite`包中的每个文件都是来自内容可寻址存储的硬链接。

`.pnpm/vite@2.9.12/node_modules/vite`

```
▾ node_modules
    ▾ .pnpm
        ...
        ▾ vite@2.9.12
            ▾ node_modules
               ▾ vite                 -> 符号链接到这个位置
                 ...
                 ▾ src
                     client
                 package.json         -> 包中的每个文件都硬链接到pnpm store中的对应文件，即<pnpm store path>/vite
    ▸ vite                            -> 符号链接到.pnpm/vite@2.9.12/node_modules/vite
    .modules.yaml
复制代码
```

**`vite`的次级依赖**

观察上面的目录结构，发现`/node_modules/.pnpm/vite@2.9.12/node_modules/vite`目录下还是没有次级依赖的`node_modules`。

**pnpm 的 `node_modules`设计 ，包的依赖项与依赖包的实际位置位于同一目录级别。**

所以 `vite` 的次级依赖包不在 `.pnpm/vite@2.9.12/node_modules/vite/node_modules/`， 而是在 `.pnpm/vite@2.9.12/node_modules/`，与`vite`实际位置位于同一目录级别。

```
▾ node_modules
    ▾ .pnpm
        ...
        ▾ esbuild@0.14.43           -> esbuild依赖包的实际位置
            ▾ node_modules
               ▸ esbuild
        ▾ vite@2.9.12
            ▾ node_modules
               ▸ esbuild             -> （依赖包）符号链接到.pnpm/esbuild@0.14.43/node_modules/esbuild，次级依赖包与依赖包实际位置同级
               ▸ postcss             ...
               ▸ reslove             ...
               ▸ rollup              ...
               ▾ vite                 -> vite依赖包的实际位置
                  ...
                  ▾ src
                     client
                  package.json        # 依赖esbuild、postcss、resolve、rollup
    ▸ vite                            -> 符号链接到.pnpm/vite@2.9.12/node_modules/vite
    .modules.yaml
复制代码
```

这里的`esbuild`等次级依赖包又是一个符号链接，仍符合刚才的逻辑，实际位置在`.pnpm/esbuild@0.14.43/node_modules/esbuild`，包内的每个文件再硬链接到`pnpm store`中的对应文件。

我们再通过官网提供的依赖图，再辅助理解一下`node_modules`依赖包之间的关系。

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202302171453774.webp)

项目依赖了`bar@1.0.0`版本，`bar`依赖了`foo@1.0.0`版本，`node_modules`下只有直接依赖包`bar@1.0.0`符号链接和`.pnpm`目录。

`Node.js`解析时，`bar@1.0.0`就会符号链接到实际位置`.pnpm/bar@1.0.0/node_modules/bar`，包中的文件（并非包文件夹）都硬链接到`.pnpm store`中的对应文件，`foo@1.0.0`做为`bar`的依赖，与`bar`的实际位置处于同一层级，符号链接指向实际位置`.pnpm/foo@1.0.0/node_modules/foo`，包中的文件再硬链接至`.pnpm store`。

[关于peerDependencies是怎么处理依赖的，可以看官网这篇文章](https://link.juejin.cn/?target=https%3A%2F%2Fpnpm.io%2Fzh%2Fhow-peers-are-resolved "https://pnpm.io/zh/how-peers-are-resolved")

**总结：pnpm使用[符号链接Symbolic link（软链接）](https://link.juejin.cn/?target=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E7%25AC%25A6%25E5%258F%25B7%25E9%2593%25BE%25E6%258E%25A5%2F7177630%3Ffr%3Daladdin "https://baike.baidu.com/item/%E7%AC%A6%E5%8F%B7%E9%93%BE%E6%8E%A5/7177630?fr=aladdin")来创建依赖项的嵌套结构，将项目的直接依赖符号链接到`node_modules`的根目录，直接依赖的实际位置在`.pnpm/<name>@<version>/node_modules/<name>`，依赖包中的每个文件再[硬链接（Hard link）](https://link.juejin.cn/?target=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E7%25A1%25AC%25E9%2593%25BE%25E6%258E%25A5%2F2088758%3Ffr%3Daladdin "https://baike.baidu.com/item/%E7%A1%AC%E9%93%BE%E6%8E%A5/2088758?fr=aladdin")到`.pnpm store`。**

### 包存储store

> `pnpm store`：`pnpm`资源在磁盘上的存储位置

一般`store`在Mac/Linux系统中，默认会设置到`{home dir}>/.pnpm-store/v3`；windows下会设置到当前盘的根目录下，比如C（`C:\.pnpm-store\v3`）、D盘（`D:\.pnpm-store\v3`）。

可以通过执行`pnpm store path`命令查看store存储目录的路径

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202302171453775.webp)

进入`store`存储路径，查看存储的内容如下：

`files/xx/xxx`以文件夹进行分类，每个文件夹内包含重新编码命名后的文件，依赖包硬链接到此处对应的文件。

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202302171453776.webp)

在项目中执行`pnpm install`的时候，依赖包存在于`store`中，直接创建依赖包硬链接到`store`中，如果不存在，则从远程下载后存储在`store`中，并从项目的`node_modules`依赖包中创建硬链接到`store`中。

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202302171453778.webp)

上图中提示包从`Content-addressable store`硬链接到`Virtual store`，以及`Content-addressable store`和`Virtual store`的作用位置。

- [内容寻址存储 — Content-addressable store(CAS)](https://link.juejin.cn/?target=https%3A%2F%2Fvibaike.com%2F106979%2F "https://vibaike.com/106979/")

它是一种存储信息的方式，根据内容而不是位置进行检索信息的存储方式，被用于高速存储和检索的固定内容，如存储。这里的CAS作用于`/Users/<username>/.pnpm-store/v3`目录。

- 虚拟存储 — Virtual store

指向存储的链接的目录，所有直接和间接依赖项都链接到此目录中，项目当中的.pnpm目录`node_modules/.pnpm`。

**因为这样的处理机制，每次安装依赖的时候，如果是相同的依赖，有好多项目都用到这个依赖，那么这个依赖实际上最优情况(即版本相同)只用安装一次。如果依赖包存在于`pnpm store`中，则从`store`目录里面去`hard-link`，避免了二次安装带来的时间消耗，如果不存在的话，就会去下载并存储在`store`中。**

**如果是 `npm` 或 `Yarn`，那么这个依赖在多个项目中使用，在每次安装的时候都会被重新下载一次。**

对比发现`pnpm install`安装速度相当之快！必须给个大大的赞！

![tempImage1655275072185.gif](https://cdn.wallleap.cn/img/pic/illustration/202302171453779.webp)

**❓紧接着会有人问，那一直往`store`里存储依赖包，`store`会不会越来越大？**

官方提供了一个命令：[`pnpm store prune`](https://link.juejin.cn/?target=https%3A%2F%2Fpnpm.io%2Fzh%2Fcli%2Fstore%23prune "https://pnpm.io/zh/cli/store#prune")，从存储中删除未引用的包。

未引用的包是系统上的任何项目中都未使用的包。 在大多数安装操作之后，包有可能会变为未引用状态。

官方举例：在 `pnpm install` 期间，包 `foo@1.0.0` 被更新为 `foo@1.0.1`。 pnpm 将在存储中保留 `foo@1.0.0` ，因为它不会自动除去包。 如果包 `foo@1.0.0` 没有被其他任何项目使用，它将变为未引用。 运行 `pnpm store prune` 将会把 `foo@1.0.0` 从存储中删除 。

运行 `pnpm store prune` 是不会影响项目的。 如果以后需要安装已经被删除的包，pnpm 将重新下载他们。建议清理不要太频繁，以防在切换分支等时pnpm需要重新下载所有删除的包，减慢安装过程。

**pnpm store的其他命令**

`pnpm store status`：查看store中已修改的包，如果包的内容与拆包时时相同的话，返回退出代码0。

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202302171453780.webp)

`pnpm store add`：只把包加入存储中，且没有修改存储外的任何项目或文件

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202302171453781.webp)

`pnpm store prune`：删除存储中未被引用的包

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202302171453782.webp)

## monorepo支持

`pnpm`跟`npm`和`Yarn`一样，内置了对[单一存储库monorepo](https://link.juejin.cn/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2Fc10d0b8c5581 "https://www.jianshu.com/p/c10d0b8c5581")的支持，只需要在项目根目录下创建 [`pnpm-workspace.yaml`](https://link.juejin.cn/?target=https%3A%2F%2Fpnpm.io%2Fzh%2Fpnpm-workspace_yaml "https://pnpm.io/zh/pnpm-workspace_yaml") 文件，定义`workspace`的根目录。

例如：

`pnpm-workspace.yaml`

```
packages:
  # all packages in subdirs of packages/ and components/
  - 'packages/**'
  - 'components/**'
  # exclude packages that are inside test directories
  - '!**/test/**'
复制代码
```

**Workspace协议（workspace:）**

> workspace：工作空间

默认情况下，如果可用的 `packages` 与已声明的可用范围相匹配，pnpm 将从工作空间链接这些 `packages`。

例如，如果 `bar` 中有 `"foo"："^1.0.0"` 的这个依赖项，则 `foo@1.0.0` 链接到 `bar`。 但是，如果 `bar` 的依赖项中有 `"foo": "2.0.0"`，而 `foo@2.0.0` 在工作空间中并不存在，则将从 npm registry 安装 `foo@2.0.0` ，这种行为带来了一些不确定性。

`pnpm` 支持 `workspace` 协议（写法：`workspace:<版本号>` ）。 当使用此协议时，pnpm 将拒绝解析除本地 `workspace` 包含的 `package` 之外的任何内容。 因此，如果您设置为 `"foo": "workspace:2.0.0"` 时，安装将会失败，因为 `"foo@2.0.0"` 不存在于此 `workspace` 中。

**接下来，我们使用`vue`代码库来理解一下Workspace协议：**

代码库地址：[github.com/vuejs/core](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fcore "https://github.com/vuejs/core")

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202302171453783.webp)

根目录可以看到有这两个文件`pnpm-lock.yaml`、`pnpm-workspace.yaml`， 其中lock文件为`pnpm install`时生成的lock文件，space文件则为`monorepo`仓库中**必须需要的**定义工作空间目录的文件。

1. 点开`pnpm-workspace.yaml`文件：[github.com/vuejs/core/…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fcore%2Fblob%2Fmain%2Fpnpm-workspace.yaml "https://github.com/vuejs/core/blob/main/pnpm-workspace.yaml")

我们看到文件内容为：

```
packages: 
    - 'packages/*'
复制代码
```

也就表示`core/packages/*`这个目录下面所有的文件为`workspace`的内容。

2. 点开`package.json`文件：[github.com/vuejs/core/…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fcore%2Fblob%2Fmain%2Fpackage.json "https://github.com/vuejs/core/blob/main/package.json")

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202302171453784.webp)

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202302171453785.webp)

我们看到用到本地`workspace`包的都标注了`workspace:*`协议，这样依赖的就一直是本地的包，而不是从`npm registry`安装的包。

3. 我们验证一下到底依赖的是不是本地包

clone代码库到本地，`pnpm install`安装依赖

查看`core/node_modules/`文件夹，发现`package.json`文件中依赖的`@vue/xxx`、`vue`包都已符号链接的形式存在，如下图：

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202302171453786.webp)

按`workspace:*`协议，打开`packages/reactivity`文件夹，做一个测试，在`index.js`文件中加入`console.log('test')`，如下图：

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202302171453787.webp)

这时候再打开`node_modules/@vue/reactivity/index.js`文件，可以发现刚才在`packages`里面改的内容，显示在了`node_modules`目录下的包里。

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202302171453788.webp)

打开磁盘上存储（`pnpm store path`）的依赖包，并没有上面新增的`console.log`，上面的改动只影响了本地依赖包，而不是远程`install`下载后存储在磁盘上的包，也就是说**符合`workspace:`协议引入的依赖包就是本地的`workspace`目录（即`core/packages`）下的包**。

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202302171453789.webp)

**别名引用**

假如工作区有一个名为 `foo` 的包，可以通过这样引用 `"foo": "workspace:"`，如果是其它别名，可以这么引用：`"bar": "workspace:foo@*"`。

**相对引用**

假如`packages`下有同层级的`foo`、`bar`，其中`bar`依赖于`foo`，则可以写作`"bar": "workspace:../foo"`。

**发布workspace包**

当`workspace`包打包发布时，将会动态替换这些`workspace:`依赖。

假设我们的 `workspace` 中有 `a`、 `b`、 `c`、 `d` 并且它们的版本都是 `1.5.0`，如下：

```
{
    "dependencies": {
        "a": "workspace:*",
        "b": "workspace:~",
        "c": "workspace:^",
        "d": "workspace:^1.5.0"
    }
}
复制代码
```

将会被转化为：

```
{
    "dependencies": {
        "a": "1.5.0",
        "b": "~1.5.0",
        "c": "^1.5.0",
        "d": "^1.5.0"
    }
}
复制代码
```

现在很多很受欢迎的开源项目都适用了pnpm的工作空间功能，感兴趣的可以前往[官网](https://link.juejin.cn/?target=https%3A%2F%2Fpnpm.io%2Fzh%2Fworkspaces%23%25E4%25BD%25BF%25E7%2594%25A8%25E7%25A4%25BA%25E4%25BE%258B "https://pnpm.io/zh/workspaces#%E4%BD%BF%E7%94%A8%E7%A4%BA%E4%BE%8B")查看哦！

## 对比npm、Yarn

**性能对比**

在`pnpm`官网上，提供了一个`benchmarks`[基准测试图表](https://link.juejin.cn/?target=https%3A%2F%2Fpnpm.io%2Fbenchmarks "https://pnpm.io/benchmarks")，它展示了`npm`、`pnpm`、`Yarn`、`Yarn pnp`在`install`、`update`等场景下的耗时：

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202302171453790.webp)

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202302171453791.webp)

通过上图，可以看出`pnpm`的运行速度基本上是`npm`的两倍，运行速度排名`pnpm > Yarn > npm`。

**功能对比**

[官网链接](https://link.juejin.cn/?target=https%3A%2F%2Fpnpm.io%2Fzh%2Ffeature-comparison "https://pnpm.io/zh/feature-comparison")

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202302171453792.webp)

通过上图可以看出`pnpm`独有的两个功能：

- 管理Node.js版本(`pnpm env use --global xxx`)
- 内容可寻址存储（`CAS`）

**竞争**

- `Yarn`

`Yarn` 在 [v3.1](https://link.juejin.cn/?target=https%3A%2F%2Fdev.to%2Farcanis%2Fyarn-31-corepack-esm-pnpm-optional-packages--3hak%23new-install-mode-raw-pnpm-endraw- "https://dev.to/arcanis/yarn-31-corepack-esm-pnpm-optional-packages--3hak#new-install-mode-raw-pnpm-endraw-") 添加了 `pnpm` 链接器。 因此 `Yarn` 可以创建一个类似于 `pnpm` 创建的 `node_modules` 目录结构。

此外，`Yarn` 团队计划实现内容可寻址存储，以提高磁盘空间效率。

- `npm`

`npm` 团队决定也采用 `pnpm` 使用的符号链接的 `node_modules` 目录结构（相关 [RFC](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fnpm%2Frfcs%2Fblob%2Fmain%2Faccepted%2F0042-isolated-mode.md "https://github.com/npm/rfcs/blob/main/accepted/0042-isolated-mode.md")）。

## npm或Yarn 转 pnpm

可参考`vue`代码库的这一次升级[commit log](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fcore%2Fcommit%2F61c5fbd3e35152f5f32e95bf04d3ee083414cecb%23diff-18ae0a0fab29a7db7aded913fd05f30a2c8f6c104fadae86c9d217091709794c "https://github.com/vuejs/core/commit/61c5fbd3e35152f5f32e95bf04d3ee083414cecb#diff-18ae0a0fab29a7db7aded913fd05f30a2c8f6c104fadae86c9d217091709794c")

操作步骤：

1. 全局安装`pnpm`

```
npm install -g pnpm
复制代码
```

2. 删除`npm`或`yarn`生成的`node_modules`

```
# 项目目录下运行或手动物理删除
rm -rf node_modules
复制代码
```

3. `pnpm import`从其他软件包管理器的`lock` 文件生成 `pnpm-lock.yaml`，再执行`pnpm install --frozen-lockfile`（相当于`npm ci`）生成依赖，防止没有lock文件意外升级依赖包，导致项目出错

```
# 生成`pnpm-lock.yaml`
pnpm import

# 安装依赖
pnpm install --frozen-lockfile
复制代码
```

4. 删除`npm`或`yarn`生成的`lock`文件

```
# 删除package-lock.json
rm -rf package-lock.json
# 删除yarn.lock
rm -rf yarn.lock
复制代码
```

5. 项目中的`npm`命令等修改为`pnpm`，包括`README`文档、运行命令等

[可参考代码库，本人亲测，已成功升级](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Feyunhua%2Fcode-commit-check%2Fcommit%2F52ca9c0701a13190b4a8ef8a2c690afffe67bf0f "https://github.com/eyunhua/code-commit-check/commit/52ca9c0701a13190b4a8ef8a2c690afffe67bf0f")

## 卸载pnpm

**卸载全局安装的包**

通过`pnpm ls --g`查看全局安装的包，只有通过`pnpm install/add xxx --global`安装的包才为全局包哦！

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202302171453793.webp)

1. `pnpm rm -g xxx`列出每个全局包进行删除

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202302171453794.webp)

2. `pnpm root -g`找到全局目录的位置并手动删除它

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202302171453795.webp)

**移除`pnpm cli`**

1. 通过独立脚本安装的，可以通过`rm -rf $PNPM_HOME`进行移除（谨慎：删除前确定好删除的内容）
2. 使用`npm`安装的`pnpm`，可以通过`npm rm -g pnpm`进行移除

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202302171453796.webp)

**删除全局内容可寻址存储**

`rm -rf $(pnpm store path)`

如果您不在主磁盘中使用 `pnpm` ，您必须在使用 `pnpm` 的每一个磁盘中运行上述命令。 因为 `pnpm` 会为每一个磁盘创建一个专用的存储空间。

## 一些疑问

还有一些问题，需要进一步的验证和考究：

1. `webpack`打包的时候，`pnpm`依赖包之间引用是怎么处理的？

2. 手动修改`pnpm store`中的包的内容，其他引用地方是不是影响了？

3. 删除项目文件夹，`pnpm prune`的机制是什么，能否正确处理？

## 结语

`pnpm`是高性能的`npm`，通过`内容可寻址存储(CAS)`、`符号链接(Symbolic Link)`、`硬链接(Hard Link)`等管理依赖包，达到多项目之间依赖共享，减少安装时间，也非常的好上手，通过`npm install -g pnpm`安装，`pnpm install`安装依赖即可。

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202302171453797.webp)

**又get了一个新的知识，有不对的和想要探讨的，欢迎评论留言哦！**

**参考文档**

1. [pnpm.io/zh/motivati…](https://link.juejin.cn/?target=https%3A%2F%2Fpnpm.io%2Fzh%2Fmotivation "https://pnpm.io/zh/motivation")
2. [juejin.cn/post/705334…](https://juejin.cn/post/7053340250210795557 "https://juejin.cn/post/7053340250210795557")
3. [juejin.cn/post/700179…](https://juejin.cn/post/7001794162970361892 "https://juejin.cn/post/7001794162970361892")

> 本文原作者 [俄小发](https://juejin.cn/user/993614678466078/posts "https://juejin.cn/user/993614678466078/posts"), 欢迎留言交流 🥳
