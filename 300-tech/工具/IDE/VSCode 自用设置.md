---
title: VSCode 自用设置
date: 2022-11-15T02:54:00+08:00
updated: 2024-08-21T11:19:55+08:00
dg-publish: false
aliases:
  - 文章别名
author: Luwang
category: web
comments: true
cover: //cdn.wallleap.cn/img/post/1.jpg
description: 文章描述
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
rating: 10
source: null
tags:
  - blog
url: null
---

## 一、首选项设置

可以直接通过快捷键 <kbd>Ctrl</kbd>+<kbd>,</kbd> 或 <kbd>Command⌘</kbd>+<kbd>,</kbd> 打开

Mac 入口：

![首选项](https://cdn.wallleap.cn/img/pic/illustrtion/202208121556216.png)

Win 入口：

![设置](https://cdn.wallleap.cn/img/pic/illustrtion/202208121557647.png)

### 1、Tab 缩进字符数

个人喜欢设置为**2**，设置方式为：

1. 搜索 `Tab Size`，将其设置为**2**

![缩进](https://cdn.wallleap.cn/img/pic/illustrtion/202208121559184.png)

1. 然后搜索 `Detect Indentation`，将这个属性的**勾去掉**或者设为**false**

![禁用](https://cdn.wallleap.cn/img/pic/illustrtion/202208121601320.png)

### 2、字体

大家推荐的是 `JetBrainsMono`，这里提供下载地址

[JetBrainsMono-1.0.3.zip - 蓝奏云 (lanzoub.com)](https://wallleap.lanzoub.com/ipzYE09frpli)

在设置中修改

![字体](https://cdn.wallleap.cn/img/pic/illustrtion/202208121611763.png)

```json
"editor.fontFamily": "JetBrains Mono, Fira Code, Consolas, 'Courier New', monospace",
```

字体大小一般默认 14 就行，需要调整可以直接在窗口中使用快捷键<kbd>Ctrl</kbd>+<kbd>+</kbd>和<kbd>Ctrl</kbd>+<kbd>-</kbd>调节整体的缩放

开启连字

```json
"editor.fontLigatures": true,
```

### 3、空文件夹折叠取消

Compact Folders 取消复选框

### Auto  Save

### 高亮显示当前缩进线

```json
"editor.guides.bracketPairs": "active",
```

### 菜单栏放到右侧

```json
"workbench.sideBar.location": "right",
```

### 菜单栏图标放到底部

安装插件 Activitus Bar 插件

隐藏大图标，并且把图标弄到右侧

```json
"workbench.activityBar.visible": false,
"activitusbar.alignment" :"Right",
"activitusbar.priority": 0,
```

## 二、插件

### 1、主题

#### (1) Monokai Pro

个人用的 `  Classic `，它包含图标主题，可以自行选择

![Monokai Pro](https://cdn.wallleap.cn/img/pic/illustrtion/202208121638611.png)

> Monokai pro 提示需要 license 解决办法

1. 找到 Monokai pro 插件安装的路径：

`C:\Users\用户名\.vscode\extensions\monokai.theme-monokai-pro-vscode-1.1.18\js`

1. 打开 `app.js`，搜索 `"isValidLicense"`，找到如下地方，注释掉原来的代码，保存即可。

```js
  key: "isValidLicense",
  value: function()
  {
      var e = arguments.length > 0 && void 0 !== arguments[0] ? arguments[0] : "",
          t = arguments.length > 1 && void 0 !== arguments[1] ? arguments[1] : "";
      //if (!e || !t) return !1; 注释掉原来的代码
      if (!e || !t) return 1;
      var o = s()("".concat(i.APP.UUID).concat(e)),
          r = o.match(/.{1,5}/g),
          n = r.slice(0, 5).join("-");
      //return t === n  注释掉原来的代码
      return 1
  }
```

之后重启 VSCode 即可

还有第二种方法

1. Windows 快捷键 <kbd>ctrl</kbd> + <kbd>shift</kbd> + <kbd>p</kbd>，MAC 快捷键 <kbd>Command⌘</kbd> + <kbd>shift</kbd> + <kbd>p</kbd>
2. 输入 `Monokai Pro: enter license`，回车，输入：`id@chinapyg.com`
3. 输入 `lincese key`，回车，输入：`d055c-36b72-151ce-350f4-a8f69`（*激活码来自 **飘云阁***）

#### (2) Flatland Monokai Theme

主要是觉得他的红色搭配背景色有点突出

![Flatland Monokai Theme](https://cdn.wallleap.cn/img/pic/illustrtion/202208121640889.png)

#### (3) Material Icon Theme

文件图标使用这个就 OK 了

![Material Icon Theme](https://cdn.wallleap.cn/img/pic/illustrtion/202208121648789.png)

### Vim

允许连续按键

```sh
defaults write com.microsoft.VSCode ApplePressAndHoldEnabled -bool false
```

一些配置（settings.json）

```json
{
    "editor.tabSize": 2,
    "editor.cursorSmoothCaretAnimation": "on",
    "editor.cursorBlinking": "expand",
    "editor.lineNumbers":"relative",
    "vim.leader": "<space>",
    "vim.easymotion": true,
    "vim.useSystemClipboard": true,
    "vim.useCtrlKeys": true,
    "vim.hlsearch": true,
    "vim.normalModeKeyBindingsNonRecursive": [
        {
            "before": [
                "z",
                "z",
            ],
            "commands": [
                "editor.fold"
            ]
        },
        {
            "before": ["z", "o"],
            "commands": ["editor.unfold"]
        },
    ],
    "vim.insertModeKeyBindings": [
        {
            "before": [
                "j",
                "j"
            ],
            "after": [
                "<Esc>"
            ]
        },
        {
            "before": ["<mouse-drag>", "<mouse-up>"],
            "after": ["<mouse-drag>", "<mouse-up>"],
            "commands": []
        },
    ],
    "vim.commandLineModeKeyBindingsNonRecursive": [
        {
            "before": [
                "z",
                "z",
            ],
            "commands": [
                "editor.toggleFold"
            ]
        },
    ],
    "vim.operatorPendingModeKeyBindings": [],
    "vim.handleKeys": {
        "<C-a>": false,
        "<C-f>": true,
        "<D-c>": false
    },
}
```

- <kbd>⌘</kbd> + <kbd>c</kbd> 在插入模式的时候并不会回到正常模式
- <kbd>j</kbd> + <kbd>j</kbd> 插入模式下退出
- <kbd>z</kbd> + <kbd>z</kbd> 折叠代码块
- <kbd>z</kbd> + <kbd>o</kbd> 展开代码块
- 用鼠标选择的时候，很多则不会退回到正常模式

## Code Spell Checker

用于检查代码单词是否写错，写错会有波浪线

你可以安装 [es6-string-html](https://marketplace.visualstudio.com/items?itemName=Tobermory.es6-string-html) 扩展，然后在字符串前加上一个前缀注释 `/*html*/` 以高亮语法。

## 快捷键

<kbd>⌘</kbd> + <kbd>j</kbd> 显示隐藏控制台（终端）

<kbd>⌘</kbd> + <kbd>b</kbd> 显示隐藏侧边栏

<kbd>⌘</kbd> + <kbd>\</kbd> 拆分多列

<kbd>⌘</kbd> + <kbd>`</kbd> 在工作区中切换

<kbd>⌘</kbd> + <kbd>p</kbd>  检索文件 <kbd>⌘</kbd> + <kbd>f</kbd>  搜索当前文件中关键词  <kbd>⌘</kbd> + <kbd>shift</kbd> + <kbd>f</kbd>  搜索全部文件 <kbd>⌘</kbd> + <kbd>t</kbd> 搜索打开文件

<kbd>⌃</kbd> + <kbd>1</kbd> 聚焦到第一个 Tab（支持 1-5）

<kbd>⌘</kbd> + <kbd>shift</kbd> + <kbd>t</kbd> 重新打开一个关闭的页面

<kbd>F2</kbd> 重命名符号（变量）

Format Document ——> <kbd>Alt</kbd> + <kbd>F</kbd>

<kbd>⌘</kbd> + <kbd>⌥</kbd> + <kbd>[</kbd> 或 <kbd>]</kbd> 折叠或展开

## `.vscode` 文件夹的作用

为了统一团队的 vscode 配置，我们可以在项目的根目录下建立 `.vscode` 目录，在里面放置一些配置内容，比如：

- `settings.json`：工作空间设置、代码格式化配置、插件配置。
- `sftp.json`：ftp 文件传输的配置。

`.vscode` 目录里的配置只针对当前项目范围内生效。将 `.vscode` 提交到代码仓库，大家统一配置时，会非常方便。
