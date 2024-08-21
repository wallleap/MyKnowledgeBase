---
title: Mac 微信双开
date: 2024-05-28T02:25:00+08:00
updated: 2024-08-21T10:34:27+08:00
dg-publish: false
---

![Uploading file...now9p]()

## 0 Abstract

[](https://github.com/CLOUDUH/dual-wechat#0-abstract)

本文主要介绍了如何在 Mac 上通过脚本实现微信双开，并通过自动操作创建应用程序，修改图标后直接当作第二个微信使用，无需打开终端输入代码，且双开后无终端出现。

## 1 基本原理

[](https://github.com/CLOUDUH/dual-wechat#1-%E5%9F%BA%E6%9C%AC%E5%8E%9F%E7%90%86)

> 如果您不想理解原理，或者不懂 Shell，可以直接跳转到最后章节。

使用 Linux nohup 命令实现，将微信主程序输出到一个“黑洞”里，即使此时关掉终端也不会影响双开微信的运行。

将以下代码复制到终端（Teriminal）运行，即可开启第二个微信，开启后，关闭终端也可以正常运行。

```shell
nohup /Applications/WeChat.app/Contents/MacOS/WeChat > /dev/null 2>&1 
```

弊端是每次都需要复制此代码到终端，非常不优雅，接下来的章节介绍如何使用自动操作打包。

## 2 使用自动操作打包脚本

[](https://github.com/CLOUDUH/dual-wechat#2-%E4%BD%BF%E7%94%A8%E8%87%AA%E5%8A%A8%E6%93%8D%E4%BD%9C%E6%89%93%E5%8C%85%E8%84%9A%E6%9C%AC)

自动操作（Automator）是 Mac 电脑上自带的一款软件，通过设置，可以实现电脑上的大部分操作自动进行，可以简单理解为一个更加硬核的快捷指令（Shortcuts）。

1. 在启动台找到**自动操作（Automator）**，点开打开。

[![](https://github.com/CLOUDUH/dual-wechat/raw/master/img/Pasted%20image%2020221117125646.png)](https://github.com/CLOUDUH/dual-wechat/blob/master/img/Pasted%20image%2020221117125646.png)

1. 选择**新建文稿**

[![](https://github.com/CLOUDUH/dual-wechat/raw/master/img/Pasted%20image%2020221117125724.png)](https://github.com/CLOUDUH/dual-wechat/blob/master/img/Pasted%20image%2020221117125724.png)

1. 选择文稿类型为**应用程序**。

[![](https://github.com/CLOUDUH/dual-wechat/raw/master/img/Pasted%20image%2020221117125818.png)](https://github.com/CLOUDUH/dual-wechat/blob/master/img/Pasted%20image%2020221117125818.png)

1. 找到**实用工具——运行 Shell 脚本**，双击或者拖拽至右侧空白处。

[![](https://github.com/CLOUDUH/dual-wechat/raw/master/img/Pasted%20image%2020221117130113.png)](https://github.com/CLOUDUH/dual-wechat/blob/master/img/Pasted%20image%2020221117130113.png)

1. **复制代码**至文本框，Shell 类型**默认/bin/zsh 或/bin/bash**即可。

```shell
nohup /Applications/WeChat.app/Contents/MacOS/WeChat > /dev/null 2>&1 
```

[![](https://github.com/CLOUDUH/dual-wechat/raw/master/img/Pasted%20image%2020221117130307.png)](https://github.com/CLOUDUH/dual-wechat/blob/master/img/Pasted%20image%2020221117130307.png)

1. 保存文件（Cmd+S），名称随意，位置选择**应用程序（Applications）**，文件格式选择**应用程序**，存储即可。

[![](https://github.com/CLOUDUH/dual-wechat/raw/master/img/Pasted%20image%2020221117130623.png)](https://github.com/CLOUDUH/dual-wechat/blob/master/img/Pasted%20image%2020221117130623.png)

1. 打开**启动台**，图标出现，点击即可启动第二个微信。

[![](https://github.com/CLOUDUH/dual-wechat/raw/master/img/Pasted%20image%2020221117131301.png)](https://github.com/CLOUDUH/dual-wechat/blob/master/img/Pasted%20image%2020221117131301.png)

## 3 嫌丑换图标

[](https://github.com/CLOUDUH/dual-wechat#3-%E5%AB%8C%E4%B8%91%E6%8D%A2%E5%9B%BE%E6%A0%87)

这个图标只是脚本启动 APP 的图标，脚本执行之后出现的第二个微信还是原版本的，如果还是嫌丑就改图标，自己设计或者去网上找一个心仪的图标。我喜欢这个黑白的，因为有植物大战僵尸里面模仿者（Imitater）的感觉。

[![](https://github.com/CLOUDUH/dual-wechat/raw/master/img/Pasted%20image%2020221117132053.png)](https://github.com/CLOUDUH/dual-wechat/blob/master/img/Pasted%20image%2020221117132053.png)

1. 复制（Cmd+C）下载好的图标文件。
2. 在应用程序（Applications）文件夹里找到刚保存的脚本程序，**右击图标，点击显示简介**。

[![](https://github.com/CLOUDUH/dual-wechat/raw/master/img/Pasted%20image%2020221117132548.png)](https://github.com/CLOUDUH/dual-wechat/blob/master/img/Pasted%20image%2020221117132548.png)

1. **点击右上方的小图标**，周围变蓝即可，然后粘贴（Cmd+V），即可完成。

[![](https://github.com/CLOUDUH/dual-wechat/raw/master/img/Pasted%20image%2020221117132844.png)](https://github.com/CLOUDUH/dual-wechat/blob/master/img/Pasted%20image%2020221117132844.png)

## 4 效果展示及懒人包

[](https://github.com/CLOUDUH/dual-wechat#4-%E6%95%88%E6%9E%9C%E5%B1%95%E7%A4%BA%E5%8F%8A%E6%87%92%E4%BA%BA%E5%8C%85)

把双开启动软件放在 dock 栏，点击之后就可以实现微信双开，而且全程无终端打开，也不会出现其他方法中关闭终端，第二个微信就关闭的情况。

[![](https://github.com/CLOUDUH/dual-wechat/raw/master/img/Pasted%20image%2020221117133051.png)](https://github.com/CLOUDUH/dual-wechat/blob/master/img/Pasted%20image%2020221117133051.png)

同样的，如果更改代码，也可以实现其他软件的双开，可以自行探索。

如果你实在太懒了，不想操作，那么点击下面链接，下载我修改好的可以直接使用。

## 5 已知问题

[](https://github.com/CLOUDUH/dual-wechat#5-%E5%B7%B2%E7%9F%A5%E9%97%AE%E9%A2%98)

这种骚操作存在一定问题，这是 WeChat 程序决定的，暂无解决方案：

- 点击微信 A 通知横幅，会跳转至微信 B 窗口。
- 电脑休眠之后，再次开启可能会提示“登录遇到问题，请重新登录”
- 重新登录时，可能必须使用扫码才能登录。
