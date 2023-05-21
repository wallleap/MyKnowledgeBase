---
title: VSCode 自用设置
date: 2022-11-15 14:54
updated: 2022-11-15 15:04
cover: //cdn.wallleap.cn/img/post/1.jpg
author: Luwang
comments: true
aliases:
  - 文章别名
rating: 10
tags:
  - blog
category: web
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: null
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

2. 然后搜索`Detect Indentation`，将这个属性的**勾去掉**或者设为**false**

![禁用](https://cdn.wallleap.cn/img/pic/illustrtion/202208121601320.png)

### 2、字体

大家推荐的是`JetBrainsMono`，这里提供下载地址

[JetBrainsMono-1.0.3.zip - 蓝奏云 (lanzoub.com)](https://wallleap.lanzoub.com/ipzYE09frpli)

在设置中修改

![字体](https://cdn.wallleap.cn/img/pic/illustrtion/202208121611763.png)

```json
"editor.fontFamily": "JetBrains Mono, Fira Code, Consolas, 'Courier New', monospace",
```

字体大小一般默认14就行，需要调整可以直接在窗口中使用快捷键<kbd>Ctrl</kbd>+<kbd>+</kbd>和<kbd>Ctrl</kbd>+<kbd>-</kbd>调节整体的缩放

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

> Monokai pro 提示需要license解决办法

1. 找到Monokai pro插件安装的路径：
		`C:\Users\用户名\.vscode\extensions\monokai.theme-monokai-pro-vscode-1.1.18\js`

2. 打开`app.js`，搜索`"isValidLicense"`，找到如下地方，注释掉原来的代码，保存即可。

		`````
		 ````
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
		 ````
		`````

之后重启 VSCode 即可

还有第二种方法

1. Windows快捷键 <kbd>ctrl</kbd>+<kbd>shift</kbd>+<kbd>p</kbd>，MAC快捷键 <kbd>Command⌘</kbd>+<kbd>shift</kbd>+<kbd>p</kbd>
2. 输入 `Monokai Pro: enter license`，回车，输入：`id@chinapyg.com`
3. 输入`lincese key`，回车，输入：`d055c-36b72-151ce-350f4-a8f69`（*激活码来自 **飘云阁***）

#### (2) Flatland Monokai Theme

主要是觉得他的红色搭配背景色有点突出

![Flatland Monokai Theme](https://cdn.wallleap.cn/img/pic/illustrtion/202208121640889.png)

#### (3) Material Icon Theme

文件图标使用这个就 OK 了

![Material Icon Theme](https://cdn.wallleap.cn/img/pic/illustrtion/202208121648789.png)

Format Document ——> <kbd>Alt</kbd> + <kbd>F</kbd>

### Vim

## Code Spell Checker

用于检查代码单词是否写错，写错会有波浪线