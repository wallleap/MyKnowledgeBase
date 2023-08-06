---
title: 003-ESLint 代码检查
date: '2023-06-19 11:31'
updated: '2023-06-28 16:53'
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - 003-ESLint 代码检查
rating: 1
tags:
  - 项目
category: web
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: null
url: //myblog.wallleap.cn/post/1
gists:
  - id: 34086cbddd6e3a2072aadfd109193a02
    url: 'https://gist.github.com/wallleap/34086cbddd6e3a2072aadfd109193a02'
    createdAt: '2023-08-01T14:23:54Z'
    updatedAt: '2023-08-01T14:23:54Z'
    filename: 003-ESLint 代码检查.md
    isPublic: true
---

## ESLint 是什么

[ESLint](https://gitee.com/link?target=http%3A%2F%2Feslint.cn%2F) 是一个代码检查工具，用来检查你的代码是否符合指定的规范

- 例如: = 的前后必须有一个空格
- 例如: 函数名后面必须有空格
- 例如: await 必须用用在 async 修饰的函数内
- 例如: ==必须转换成 3 个等
- ........

## ESLint 的好处

- 在写代码过程中，检查你代码是否错误，给你 `小黑屋` 提示
- ESLint 可以约束团队内代码的风格统一

> SLint 是法官，Standard 是法

## ESLint 的规范

规范文档: [http://www.verydoc.net/eslint/00003312.html](https://gitee.com/link?target=http%3A%2F%2Fwww.verydoc.net%2Feslint%2F00003312.html)

规范文档 2: [https://standardjs.com/rules-zhcn.html](https://gitee.com/link?target=https%3A%2F%2Fstandardjs.com%2Frules-zhcn.html)

规范文档 3: [http://eslint.cn/docs/rules/](https://gitee.com/link?target=http%3A%2F%2Feslint.cn%2Fdocs%2Frules%2F)

[Find and fix problems in your JavaScript code - ESLint - Pluggable JavaScript Linter](https://eslint.org/)

## ESLint 使用

在代码里随便多写几个回车，ESLint 会跳出来 `刀子嘴`，`豆腐心` 的提示你.

在 webpack 开发服务器终端 / 浏览器 => 小黑屋

> eslint 是来帮助你的。心态要好，有错，就改。

把第 4 步规则名字，复制到上面规范里查找违反了什么规则 / 根据第三步提示修改

### ESLint 工作过程

1. 我们 vue 初始化项目的时候，选择需要 eslint 功能，以及 standard 检查规则
2. 确保 ==vscode 根目录== 有 eslintrc 配置文件，vscode 工作区 (目录环境)，会根据配置来检查你的代码
3. 当前项目下，编写的代码，违反 eslint 规则，会在终端报错

### ESLint 工作环境

1. webpack+eslint 检查代码，在终端输出检查报错结果（还可以手动在终端使用命令检查）
2. Vscode+ESLint 插件，在编写代码区域报错结果 + 自动修复

### 在项目中使用

由于脚手架已经集成了 ESLint 库，我们就不需要手动安装了（手动安装命令：`npm install eslint`）

1、VSCode 先安装下面的插件：[VSCode-ESLint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)

2、项目目录作为 vscode **根目录**，因为 vscode+eslint 插件要使用配置文件 `.eslintrc`，按照这个的规则检查你的代码

可以配置自己团队的项目规则

![](https://cdn.wallleap.cn/img/pic/illustration/202306191347352.png)

rules 是一个对象，以键值对的格式来约定规则：

- 键名是规则名
- 值是这条规则的具体说明，最常见的有 off、warn、error

3、修改 ESLint 插件配置（Ctrl+Shift+P 搜索 setting json），添加内容

```json
"eslint.run": "onType",
"editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
},
```

4、检查是否生效

- 随便在 src 下的任意文件里多敲击几个回车，vscode 报错提示，证明 ESLint 插件开始工作
- ctrl + s 报错下是否能自动修复部分问题，可以则明 ESLint 插件开始工作
