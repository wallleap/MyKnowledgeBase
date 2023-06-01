---
title: README
date: 2023-05-21 19:06
updated: 2023-06-01 14:11
---

## Inbox —— All in this . MOC

> ✨ Luwang 的个人知识库

[![](https://img.shields.io/badge/GitHub-MyKownlegeBase-blue?style=for-the-badge&logo=github)](https://github.com/wallleap/MyKnowledgeBase)  [![GITBOOK: GARDEN (shields.io)](https://img.shields.io/badge/GitBook-garden-orange?style=for-the-badge&logo=gitbook)](https://wallleap.gitbook.io/garden/)

使用 GitHub 仓库进行备份，Obsidian 进行本地管理，GitHub 网上阅读，图片使用 PicGo + COS 图床

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
- 其他 GitBook 的说明
	- 链接是 `https://用户名或团队名/gitbook.io/空间名`，需要公开的就放到团队中，并且在设置中开启 Publish to Web
	- 仓库必须有 README.md 文件（可以当成是仓库首页）和 SUMMARY.md 文件（目录），目录的写法为二级标题 + 列表 + 链接（全都是 markdown 语法），空格需要用 `%20`

	```md
	# 目录

	## 目录1

	- 有子目录的%20目录
		- [可以跳转的](链接)
		- 不可以跳转的
	- [没有子目录可跳转的](链接)
	```

	- 不需要显示的文件或目录放到 `.bookignore` 文件中，语法和 `.gitigore` 一样

> 📌 重要提示：
> 
> - 所有看过、学过的东西最好记录（视频、文字、语音）下来，不然下次就找不到了！
> - 收集的内容也别忘记去看、去梳理，不然记了也白记！

### 目录

为了所有地方都可以看到所有文件，在这里再重新以 markdown 语法写一遍

#### 个人

- 记录
	- 笔记
		- [康奈尔笔记法](100-person/笔记/康奈尔笔记法.md)
		- [GitHub emoji](100-person/笔记/GitHub%20emoji.md)
	- 工具
		- [常用工具和软件](100-person/工具/常用工具和软件.md)
		- [打造个性化 GitHub 主页](100-person/工具/打造个性化%20GitHub%20主页.md)
		- [翻翻教程](100-person/工具/翻翻教程.md)
		- [个人输入法调教之 Rime](100-person/工具/个人输入法调教之%20Rime.md)
		- [自由输入法 RIME 简明配置指南](100-person/工具/自由输入法%20RIME%20简明配置指南.md)
		- [RIME 小鹤双拼快速设置](100-person/工具/RIME%20小鹤双拼快速设置.md)
		- [mac 打包 DMG](100-person/工具/mac%20打包%20DMG.md)
		- [macOS 个性化定制](100-person/工具/macOS%20个性化定制)
	- [备忘录](100-person/备忘录.md)
	- [常用推荐](100-person/常用推荐.md)
	- [Vim 简单使用教程](100-person/Vim%20简单使用教程.md)
- 生活

#### 工作

- 前端
	- [常用网站](200-work/前端/常用网站.md)
	- [配置记录](200-work/前端/配置记录.md)
	- [互联网公司角色](200-work/前端/互联网公司角色.md)
	- [项目发布流程](200-work/前端/项目发布流程.md)
	- [前端工作内容](200-work/前端/前端工作内容.md)
- 广告
	- [创意课程合集](200-work/广告/创意课程合集.md)
	- [在搭建信息流广告时的一些小技巧](200-work/广告/在搭建信息流广告时的一些小技巧.md)
- Office
	- Word
	- Excel
	- [PPT](200-work/Office/0-PPT学习.md)
		- [PPT页面排版](200-work/Office/PPT页面排版.md)

#### 技术

- 前端
	- 三件套
		- HTML
			- [常用 meta](300-tech/三件套/常用%20meta.md)
			- [HTML](300-tech/三件套/HTML.md)
		- CSS
			- [CSS 世界笔记](300-tech/三件套/CSS%20世界笔记.md)
			- [用 CSS 实现固定宽高比](300-tech/三件套/用%20CSS%20实现固定宽高比.md)
			- [使用 CSS 计数器更改有序列表序号样式](300-tech/三件套/使用%20CSS%20计数器更改有序列表序号样式.md)
			- [只要一行代码，实现五种 CSS 经典布局](300-tech/三件套/只要一行代码，实现五种%20CSS%20经典布局.md)
			- [用 HTML + CSS 制作前端简历](300-tech/三件套/用%20HTML%20+%20CSS%20制作前端简历.md)
		- JavaScript
			- [规范 JavaScript 命名](300-tech/三件套/规范%20JavaScript%20命名.md)
			- [手写 JS 常用 API](300-tech/三件套/手写%20JS%20常用%20API.md)
			- [ES 6 入门教程大纲](300-tech/三件套/ES%206%20入门教程大纲.md)
			- [ES 6 入门教程-阮一峰](300-tech/三件套/ES%206%20入门教程-阮一峰.md)
			- [老姚浅谈：怎么学 JavaScript？](300-tech/三件套/老姚浅谈：怎么学%20JavaScript？.md)
			- [重新认识 JavaScript 中的 Date](300-tech/%E4%B8%89%E4%BB%B6%E5%A5%97/%E9%87%8D%E6%96%B0%E8%AE%A4%E8%AF%86%20JavaScript%20%E4%B8%AD%E7%9A%84%20Date.md)
	- 框架
		- Vue
		- React
		- UI
		- [小程序笔记](300-tech/框架/小程序笔记.md)
	- [web 前端知识体系](300-tech/web%20前端知识体系.md)
	- [鱼皮-前端学习路线](300-tech/鱼皮-前端学习路线.md)
	- [前端面试押题-方方版](300-tech/前端面试押题-方方版.md)
	- [前端编码规范](300-tech/前端编码规范.md)
- 课程
	- [000-如何提问](300-tech/课程/000-如何提问.md)
	- [000-学习路线](300-tech/课程/000-学习路线.md)
	- [001-初学 Web](300-tech/课程/001-初学%20Web.md)
	- [002-HTML](300-tech/课程/002-HTML.md)
	- [003-Git 学习](300-tech/课程/003-Git%20学习.md)
	- [004-接触 CSS](300-tech/课程/004-接触%20CSS.md)
	- [005-深入 CSS](300-tech/课程/005-深入%20CSS.md)
	- [006-布局相关](300-tech/课程/006-布局相关.md)
	- [007-CSS 补充](300-tech/课程/007-CSS%20补充.md)
	- [008-JS 基本概念](300-tech/课程/008-JS%20基本概念.md)
	- [009-数字和字符串](300-tech/课程/009-数字和字符串.md)
	- [010-数组和对象基本使用](300-tech/课程/010-数组和对象基本使用.md)
	- [011-流程控制语句](300-tech/课程/011-流程控制语句.md)
	- [012-函数相关](300-tech/课程/012-函数相关.md)
	- [013-数组](300-tech/课程/013-数组.md)
	- [014-Symbol、Set、Map、WeakMap](300-tech/课程/014-Symbol、Set、Map、WeakMap.md)
	- [015-正则表达式](300-tech/课程/015-正则表达式.md)
	- [016-浏览器渲染机制](300-tech/课程/016-浏览器渲染机制.md)
	- [017-DOM 操作](300-tech/课程/017-DOM%20操作.md)
	- [018-JS 事件](300-tech/课程/018-JS%20事件.md)
	- [019-BOM](300-tech/课程/019-BOM.md)
	- [020-内存图1](300-tech/课程/020-内存图1.md)
	- [021-内存图2-调用栈](300-tech/课程/021-内存图2-调用栈.md)
	- [WebPack 的使用](300-tech/课程/WebPack%20的使用.md)
	- [Webpack 性能优化](300-tech/课程/Webpack%20性能优化.md)
	- [Vue 起手式](300-tech/课程/Vue%20起手式.md)
	- [Vue 的构造选项](300-tech/课程/Vue%20的构造选项.md)
- 工具
	- [Git 操作](300-tech/工具/Git%20操作.md)
	- [规范化 Commit 提交信息](300-tech/工具/规范化%20Commit%20提交信息.md)
	- [如何规范你的 Git commit？](300-tech/工具/如何规范你的%20Git%20commit？.md)
	- [面向 Coding 的 Git](300-tech/工具/面向%20Coding%20的%20Git.md)
	- [mac 本地 Git 服务器](300-tech/工具/mac%20本地%20Git%20服务器.md)
	- [pnpm 出来这么久了，你还没用它吗](300-tech/工具/pnpm%20出来这么久了，你还没用它吗.md)
	- [Vim-写代码更高效](300-tech/工具/Vim-写代码更高效.md)
	- [VSCode 自用设置](300-tech/工具/VSCode%20自用设置.md)
	- [iTerm2](300-tech/工具/iTerm2.md)
	- [nohup和&后台运行，进程查看及终止](300-tech/工具/nohup和&后台运行，进程查看及终止.md)
	- [Emmet 加快开发速度](300-tech/工具/Emmet%20加快开发速度.md)
	- [macOS 安装 NVM](300-tech/工具/macOS%20安装%20NVM.md)
	- [Frp + 服务器远程登录 windows](300-tech/工具/Frp%20+%20服务器远程登录%20windows.md)
	- [win11 安装 arch 子系统](300-tech/工具/win11%20安装%20arch%20子系统.md)
	- [手把手教你快速搭建一个代码在线编辑预览工具](300-tech/工具/手把手教你快速搭建一个代码在线编辑预览工具.md)
	- [除了 Markdown 编辑器，你还需要会用程序来处理它](300-tech/工具/除了%20Markdown%20编辑器，你还需要会用程序来处理它.md)
- 源码
	- [vant4 源码](300-tech/源码/vant4%20源码.md)
- 资源
	- [前端优质开源项目](300-tech/资源/前端优质开源项目.md)
	- [前端优质学习资源](300-tech/资源/前端优质学习资源.md)
- 项目
	- 山竹记账
	- [一篇文章搞定 GitHub API 调用](300-tech/项目/一篇文章搞定%20GitHub%20API%20调用.md)
	- [基于 GitHub API 的个人博客](300-tech/项目/基于%20GitHub%20API%20的个人博客.md)
	- [Vue2 组件库和文档](300-tech/项目/Vue2%20组件库和文档.md)
	- [VuePress 搭建组件库文档](300-tech/项目/VuePress%20搭建组件库文档.md)
	- [使用 Vuepress 搭建基于 Vue 3.2 的组件库](300-tech/项目/使用%20Vuepress%20搭建基于%20Vue%203.2%20的组件库.md)
	- [几行代码教你解决微信生成海报及二维码](300-tech/%E9%A1%B9%E7%9B%AE/%E5%87%A0%E8%A1%8C%E4%BB%A3%E7%A0%81%E6%95%99%E4%BD%A0%E8%A7%A3%E5%86%B3%E5%BE%AE%E4%BF%A1%E7%94%9F%E6%88%90%E6%B5%B7%E6%8A%A5%E5%8F%8A%E4%BA%8C%E7%BB%B4%E7%A0%81.md)
	- [前端 QRCode.js 生成二维码插件](300-tech/%E9%A1%B9%E7%9B%AE/%E5%89%8D%E7%AB%AF%20QRCode.js%20%E7%94%9F%E6%88%90%E4%BA%8C%E7%BB%B4%E7%A0%81%E6%8F%92%E4%BB%B6.md)
- [编程学习之道](300-tech/编程学习之道.md)

### 文章内图标使用（emoji）

全部使用 GitHub 支持的 emoji

- 举个例子、例子、举例 `🌰`
- 亮点 `✨`
- Tips、提示 `📌`
- 重点 `❗`
- 描述 `💬`
- 看一看、了解 `👀`
- 资源 `📚`
- 目标 `🎯`
- `🤡💀☠️👻🫣👽👾🤖💩🙈🐞💪🤏👂👈👉☝️🫵👆👇✌️🤞🫰🖖🫱🫲🫳🫴🤘🤙🖐️✋👌🤌👍👎✊👊🤛🤜🤚👋🤟✍️👏👐🙌🫶🤲🙏🤝🎈🎆🎇🧨🎉🎊🎁🎗️🎞️🖼️🎨👑💄💎🥇🥈🥉🏅🎖️🏆🎮🔮🪄♥️📣🔔🔑🗝️🔒🔓🧰🪜💻🖥️⌨️🖱️🧮🪔💡🔍🔎📖🔖🏷️💰📪📭📬📝📁📂🗂️📆📈📉🍂✈️🚀🛞🚨🚩🪐♨️🛎️⭐🌟🌠🌈⚡❄️🔥💧❤️💔💕💓💖💘💝💌💢💥💤💦💫⛔📛❌⭕🚫🔇🔕❗❕❓❔💯🔅🔆🔱⚜️〽️❎✅▶️⏸️⏯️⏹️⏺️⏭️⏮️⏩⏪🔀🔁◀️🔼🔽➡️⬅️⬆️⬇️↗️↘️↙️↖️↕️↔️🔄️↪️↩️⤴️⤵️ℹ️🔃`

### 使用到的插件

核心插件：标签列表、大纲、工作区、关系图谱（<kbd>Ctrl</kbd> + <kbd>G</kbd> ）、快速切换、录音机、命令面板、模板、日记、搜索、~~文件恢复~~（会和同步冲突）、文件列表、斜杠命令、星标、字数统计、Markdown 格式转换器

#### 编辑增强

- 设定快捷键
	- 标题快捷键：<kbd>Ctrl</kbd> + <kbd>1</kbd>  （数字 1~6）
	- 删除段落：<kbd>Ctrl</kbd> + <kbd>D</kbd>
	- 下划线：<kbd>Ctrl</kbd> + <kbd>U</kbd>  <u>下划线</u>
- 下面这些用自带的
	- 加粗：<kbd>Ctrl</kbd> + <kbd>B</kbd>  **加粗**
	- 倾斜：<kbd>Ctrl</kbd> + <kbd>I</kbd>  *倾斜*
	- 删除线：<kbd>Alt</kbd> + <kbd>Shift</kbd> + <kbd>D</kbd>  ~~删除线~~（表格用的 <kbd>Ctrl</kbd>）
	- 插入链接：<kbd>Ctrl</kbd> + <kbd>K</kbd>  [链接](//wallleap.cn)

#### Advanced URI

文件列表中右击文章标题，Copy Advanced URI

`obsidian://advanced-uri?vault=obsidian&filepath=web%252F%25E5%25B7%25A5%25E5%2585%25B7%252FVSCode%2520%25E8%2587%25AA%25E7%2594%25A8%25E8%25AE%25BE%25E7%25BD%25AE.md`

#### Advanced Tables

操作 Markdown 表格更方便，不常用，一般都是直接复制表格到 Typora 再复制过来，但是它可以让单元格在源码模式对齐来，看起来更舒适一点

![](https://cdn.wallleap.cn/img/pic/illustrtion/202211290948868.gif)

| 快捷键                                               | 操作        |
| ------------------------------------------------- | --------- |
| <kbd>Tab</kbd>                                    | 下一单元格     |
| <kbd>Shift</kbd> + <kbd>Tab</kbd>                 | 上一单元格     |
| <kbd>Enter</kbd>                                  | 下一行       |
| <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>D</kbd> | 打开表格控制侧边栏 |

#### Better Word Count

- 选中文字后在右下角会显示选中文字数量

#### CodeMirror Option

- 除了 Copy 使用下面的代替，其他都勾选

#### Copy button for code blocks

- 在 Read Mode 下代码块会有 Copy 按钮

#### Dataview

- 整个知识库是一个数据库，可以使用命令进行查询，默认设置即可

#### ~~Emoji Toolbar~~

- 自己设置快捷键或者 <kbd>Ctrl</kbd> + <kbd>P</kbd> 搜索 emoji 👍，但是好像网不好不显示☹️
- 搭配 Obsidian Icon Folder 插件可以修改文件（夹）图标 😘
- 删掉，用着在 Obsidian 迁移的时候会很难受

#### Image auto upload Plugin

- 搭配 Picgo 使用，直接粘贴就能上传图片
- Picgo 建议使用 COS 作为图床，以后方便迁移（自己设定 CDN 地址）

#### ~~Markdown prettifier~~ → Linter

- 个人主要需要使用的是 YAML 中 `updated` 字段更新时间，直接 <kbd>Ctrl</kbd> + <kbd>S</kbd> 就可以对文档进行格式化

#### Mind Map

- 生成思维导图
- 标题、列表、代码等会显示，可以设定快捷键

#### Minimal Theme Settings

- 个人已经将 Accent color 写入到了主题 CSS 文件中，可以直接调整主题色

#### Obsidian Git

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

#### Obsius Publish

- 临时分享文章给别人看必备
- 文件列表中选中文件右击选择 Publish to Obsius
- 自动复制好链接，可以分享给其他人查看 <https://obsius.site/1p3t376x6w5c530k4v45>

#### Regex Find/Replace

- 文字/字符替换，个人用于批量修改图片链接

#### Style Settings

- 搭配 CSS snippet 修改样式

### 快捷键记录

- <kbd>Ctrl</kbd> + <kbd>Shift</kbd> +<kbd>e</kbd>  Emoji ToolBar
- <kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>i</kbd>  插入模板
- <kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>p</kbd>  Run Markdown Prettify 格式化
- <kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>m</kbd>  思维导图
- <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>i</kbd>  打开调试功能

↓ 以下的是插件生成的，只能在 Obsidian 中查看 ↓

### 三天内修改的文档

```dataview
table
file.tags as "标签",
file.ctime as "创建时间"
where date(today) - file.mtime <= dur(3 days) 
```

### design 文件夹下 `#规范` 标签文件

```dataview
table 
file.folder as "文件夹",
rating as "星级"
from "400-design" and #规范
sort rating
```

### design

包含所有的设计相关的内容

常规设计尺寸和设计方法记录：[[设计规范]]

#### B 端设计

##### [[0-B 端汇总]]

```dataview
table
file.tags as "标签",
file.ctime as "更新时间",
date(now)-(file.ctime) as 迄今時數
from "400-design/B 端设计"
```

#### UI 设计

```dataview
table
file.tags as "标签"
from "400-design/UI 设计"
```

#### 平面设计

```dataview
table
file.tags as "标签"
from "400-design/平面设计"
```

#### 动效设计

```dataview
table
file.tags as "标签"
from "400-design/动效设计"
```

[[21天动效打卡教程]]

## tech

```dataview
table
file.tags as "标签"
from "300-tech"
```

## work

```dataview
table
file.tags as "标签"
from "200-work"
```

## person

```dataview
table
file.tags as "标签"
from "100-person"
```

### 工具

[[常用工具和软件]]
