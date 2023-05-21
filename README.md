---
title: README
date: 2022-11-10 11:26
updated: 2023-02-27 11:00
cover: //cdn.wallleap.cn/img/pic.jpg
author: Luwang
comments: true
aliases:
  - 仓库整合
rating: 0
tags:
  - 整理
category: home
keywords:
  - Obsidian
  - 整理
description: 显示本知识库所有内容
source: null
url: null
---

# Inbox —— All in this . MOC

本知识库约定：

- 目录最多使用 **3 层**（第一层分类目录不计入）
- 标题只使用 `##`、`###`、`####` （一个页面只能出现一个 `#`，一般按文件名自动生成）
- 需要思维导图查看的文章，标题需要设置更严谨，需要在下层显示的使用列表
- 每个文件都添加 `temp` 目录下的 **front-matter**，尤其是 `tag` 字段（已设定成模板可以使用快捷键插入到文章头部）
- `tag` 字段一定保持统一（英文大小写符合常规）
	- 需要在个人博客中展示的文章加标签：`blog`
	- 中英文不需要空格隔开，否则会被认定为是两个标签（`B端`）
- `category` 字段一般就使用大文件夹的名称
- 为了方便迁移，除 `inbox` 文件夹和文件外，尽量使用 Obsidian 自带的语法
- 可以尽量拆分文件，并使用双链链接它们，使 MOC 联系越来越紧密，避免产生孤儿节点
- 只进行 Markdown 层面的美化，且不过度美化
- 图片全部上传至 COS，且链接以 `https://cdn.wallleap.cn/img/pic/illustration/` 开头，图片没有特殊描述就置空
- 其他资源，例如视频均放至 `asserts` 目录，且大小不应超过 50MB
- 重要文件在此文档加上链接，其他使用 DataView 获取

## 文章内图标使用（emoji）

- 举个例子、例子、举例 `🌰`
- 亮点 `✨`
- Tips、提示 `📌`
- 重点 `❗`
- 描述 `💬`
- 看一看、了解 `👀`
- 资源 `📚`
- 目标 `🎯`
- `🤡💀☠️👻🫣👽👾🤖💩🙈🐞💪🤏👂👈👉☝️🫵👆👇✌️🤞🫰🖖🫱🫲🫳🫴🤘🤙🖐️✋👌🤌👍👎✊👊🤛🤜🤚👋🤟✍️👏👐🙌🫶🤲🙏🤝🎈🎆🎇🧨🎉🎊🎁🎗️🎞️🖼️🎨👑💄💎🥇🥈🥉🏅🎖️🏆🎮🔮🪄♥️📣🔔🔑🗝️🔒🔓🧰🪜💻🖥️⌨️🖱️🧮🪔💡🔍🔎📖🔖🏷️💰📪📭📬📝📁📂🗂️📆📈📉🍂✈️🚀🛞🚨🚩🪐♨️🛎️⭐🌟🌠🌈⚡❄️🔥💧❤️💔💕💓💖💘💝💌💢💥💤💦💫⛔📛❌⭕🚫🔇🔕❗❕❓❔💯🔅🔆🔱⚜️〽️❎✅▶️⏸️⏯️⏹️⏺️⏭️⏮️⏩⏪🔀🔁◀️🔼🔽➡️⬅️⬆️⬇️↗️↘️↙️↖️↕️↔️🔄️↪️↩️⤴️⤵️ℹ️🔃`

## 使用到的插件

核心插件：标签列表、大纲、工作区、关系图谱（<kbd>Ctrl</kbd> + <kbd>G</kbd> ）、快速切换、录音机、命令面板、模板、日记、搜索、~~文件恢复~~（会和同步冲突）、文件列表、斜杠命令、星标、字数统计、Markdown 格式转换器

### 编辑增强

- 设定快捷键
	- 标题快捷键：<kbd>Ctrl</kbd> + <kbd>1</kbd>  （数字 1~6）
	- 删除段落：<kbd>Ctrl</kbd> + <kbd>D</kbd>
	- 下划线：<kbd>Ctrl</kbd> + <kbd>U</kbd>  <u>下划线</u>
- 下面这些用自带的
	- 加粗：<kbd>Ctrl</kbd> + <kbd>B</kbd>  **加粗**
	- 倾斜：<kbd>Ctrl</kbd> + <kbd>I</kbd>  *倾斜*
	- 删除线：<kbd>Alt</kbd> + <kbd>Shift</kbd> + <kbd>D</kbd>  ~~删除线~~（表格用的 <kbd>Ctrl</kbd>）
	- 插入链接：<kbd>Ctrl</kbd> + <kbd>K</kbd>  [链接](//wallleap.cn)

### Advanced URI

文件列表中右击文章标题，Copy Advanced URI
`obsidian://advanced-uri?vault=obsidian&filepath=web%252F%25E5%25B7%25A5%25E5%2585%25B7%252FVSCode%2520%25E8%2587%25AA%25E7%2594%25A8%25E8%25AE%25BE%25E7%25BD%25AE.md`

### Advanced Tables

操作 Markdown 表格更方便，不常用，一般都是直接复制表格到 Typora 再复制过来，但是它可以让单元格在源码模式对齐来，看起来更舒适一点

![](https://cdn.wallleap.cn/img/pic/illustrtion/202211290948868.gif)

| 快捷键                                               | 操作        |
| ------------------------------------------------- | --------- |
| <kbd>Tab</kbd>                                    | 下一单元格     |
| <kbd>Shift</kbd> + <kbd>Tab</kbd>                 | 上一单元格     |
| <kbd>Enter</kbd>                                  | 下一行       |
| <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>D</kbd> | 打开表格控制侧边栏 |

### Better Word Count

- 选中文字后在右下角会显示选中文字数量

### CodeMirror Option

- 除了 Copy 使用下面的代替，其他都勾选

### Copy button for code blocks

- 在 Read Mode 下代码块会有 Copy 按钮

### Dataview

- 整个知识库是一个数据库，可以使用命令进行查询，默认设置即可

### Emoji Toolbar

- 自己设置快捷键或者 <kbd>Ctrl</kbd> + <kbd>P</kbd> 搜索 emoji 👍，但是好像网不好不显示☹️
- 搭配 Obsidian Icon Folder 插件可以修改文件（夹）图标 😘

### Image auto upload Plugin

- 搭配 Picgo 使用，直接粘贴就能上传图片
- Picgo 建议使用 COS 作为图床，以后方便迁移（自己设定 CDN 地址）

### Markdown prettifier

- 个人主要需要使用的是 YAML 中 `updated` 字段更新时间，可以设定快捷键

### Mind Map

- 标题、列表、代码等会显示，可以设定快捷键

### Minimal Theme Settings

- 个人已经将 Accent color 写入到了主题 CSS 文件中，可以直接调整主题色

### Obsidian Git

- 懒得自己 Commit，~~直接隔段时间就 Commit 一次~~（也不太好用）改用下面手动的方式

在根目录新建文件 `WinGitPush.bat`，内容如下：

```bat
@echo off
git add .
git commit -m update
git push -f
echo.
echo.
echo.
echo PUSH FINISHED
timeout /t 2
exit
```

Win 下可以直接双击这个文件 push 到仓库（提前配置好了才行）

macOS 和 Linux

```sh
#!/usr/bin/env bash
git add .
git commit -m update
git push -f
```

### Obsius Publish

- 文件列表中选中文件右击选择 Publish to Obsius
- 自动复制好链接，可以分享给其他人查看 <https://obsius.site/1p3t376x6w5c530k4v45>

### Regex Find/Replace

- 文字/字符替换，个人用于批量修改图片链接

### Style Settings

- 搭配 CSS snippet 修改样式

## 快捷键记录

- <kbd>Ctrl</kbd> + <kbd>Shift</kbd> +<kbd>e</kbd>  Emoji ToolBar

- <kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>i</kbd>  插入模板

- <kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>p</kbd>  Run Markdown Prettify 格式化

- <kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>m</kbd>  思维导图

- <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>i</kbd>  打开调试功能

## 三天内修改的文档

```dataview
table
file.tags as "标签",
file.ctime as "创建时间"
where date(today) - file.mtime <= dur(3 days) 
```

## design 文件夹下 `#规范` 标签文件

```dataview
table 
file.folder as "文件夹",
rating as "星级"
from "400-design" and #规范
sort rating
```

## design

包含所有的设计相关的内容

常规设计尺寸和设计方法记录：[[设计规范]]

### B 端设计

#### [[0-B 端汇总]]

```dataview
table
file.tags as "标签",
file.ctime as "更新时间",
date(now)-(file.ctime) as 迄今時數
from "400-design/B 端设计"
```

### UI 设计

```dataview
table
file.tags as "标签"
from "400-design/UI 设计"
```

### 平面设计

```dataview
table
file.tags as "标签"
from "400-design/平面设计"
```

### 动效设计

```dataview
table
file.tags as "标签"
from "400-design/动效设计"
```

[[21天动效打卡教程]]

## web

```dataview
table
file.tags as "标签"
from "300-web"
```

## work

```dataview
table
file.tags as "标签"
from "200-work"
```

## life

```dataview
table
file.tags as "标签"
from "100-life"
```

### 工具

[[常用工具和软件]]
