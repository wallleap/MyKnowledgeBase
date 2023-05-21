---
title: mac打包DMG
date: 2022-12-14 14:06
updated: 2022-12-14 14:06
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - mac打包DMG
rating: 10
tags:
  - 工具
  - macOS
category: tools
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: #
url: //myblog.wallleap.cn/post/
---

最近在开发一个名为 timeGO 的 macOS 应用程序，一款倒计时应用。看到网上别人家 app 的 dmg 安装包，美得很，本文就以这个 app 为例说一下如何将自己的应用发布并打包成带图标和安装提示背景的 dmg 安装包。

![](https://cdn.wallleap.cn/img/pic/illustrtion/202212141406758.png)

十里使用 xcode 开发 timeGO 这款应用，因为要精美所以本身应用也得精美对吧！所以十里为应用开发专门设计了精美的 appIcon (图标)，这样导出的 app 会有精美的图标。发布 app 的方式有很多种，这里说一个比较常用的发布到本地的方式。

1.  xcode 打开应用工程
    
2.  菜单栏选择 **product** -> **Archive**，此时 xcode 就会打包应用，完成后会跳到类似如下的包管理窗口:
    
    ![](https://cdn.wallleap.cn/img/pic/illustrtion/202212141406759.png)
    
3.  点击 `Distribute App` 按钮，选择 `Copy App`：
    
    ![](https://cdn.wallleap.cn/img/pic/illustrtion/202212141406760.png)
    
4.  点击 `Next`，选择导出名称和目录，这里名称使用默认，目录选择 `Desktop`:
    
    ![](https://cdn.wallleap.cn/img/pic/illustrtion/202212141406761.png)
    
5.  点击 `Export` 之后，就会在 `Desktop` 中看到一个步骤4中的名称相同的目录，里面就是导出的 app:
    
    ![](https://cdn.wallleap.cn/img/pic/illustrtion/202212141406762.png)
    

以上就完成了 app 的发布。

## 准备素材

制作 dmg 前需要准备 app 的图标文件 icns 和背景图。

## app 的图标文件

这里需要的图标文件为 icns 格式，十里在开发 timeGO 的时候只准备了 png 格式的文件，难道还得进行转换不成，其实不需要，macOS 的应用图标都是 icns 格式，所以可以从刚刚导出的 app 中获取这个图标文件。

1.  在刚才导出的 app 上右键，选择 `显示包内容`
    
2.  此时会打开显示 app 内的文件，依次进入 `Contents` - `Resources`
    
3.  不出意外就会看到一个格式为 icns 的图片文件，其样子与 app 的图标一样，这就是我们想要的 icns，将其拷贝到桌面：
    
    ![](https://cdn.wallleap.cn/img/pic/illustrtion/202212141406763.png)
    

## 背景图片

相信读者在安装一下 dmg 文件的应用时，都会知道，打开 dmg 文件后，目录中有个背景图片提示将 app 拖放到 applications 文件夹中，像 qq for mac的也是如此：

![](https://cdn.wallleap.cn/img/pic/illustrtion/202212141406764.png)

所以我们也需要一个这样的背景图，自己设计就好了，说明意思就 OK，十里使用 sketch 设计了一个简陋的背景图( png 格式)，也放在了桌面，名为 dmg.png:

![](https://cdn.wallleap.cn/img/pic/illustrtion/202212141406765.png)

## 制作 dmg

查看 app 、准备的 icns 图标和背景图共占空间的大小，十里这里是：

| 文件 | 空间大小 |
| --- | --- |
| timeGO.app | 约13.1MB |
| AppIcon.icns | 65KB |
| dmg.png | 33KB |
| 共计 | 约13.2MB(肯定小于13.3MB) |

这里统计三个文件的大小是因为一会儿要使用。

## 新建空的 dmg 文件

1.  打开 `磁盘工具.app`
    
2.  菜单栏中：`文件` - `新建映像` - `空白映像`
    
3.  在出来的对话框中按您的需求修改橙色框圈起来的地方，其中空间大小设置略大于上面提到的 13.2MB 即可，这里设置为 14.5MB：
    
    ![](https://cdn.wallleap.cn/img/pic/illustrtion/202212141406766.png)
    
4.  创建完成后就可以在桌面看到空白的映像
    

## 放置 dmg 中的文件

1.  将第一节中导出的 app 文件、桌面上的 icns 文件和背景图全部拖进新建的映像文件中:
    
2.  在映像中创建 `/Applications` 的软链接(在终端下执行命令，cd 的目录改为您对应映像挂载的目录)
    
    ```
    cd /Volumes/timeGO
    ln -s /Applications Applications
    ```
    

## 配置映像文件的显示选项

1.  打开映像挂载目录，目录中会看到四个文件：timeGO.app, dmg.png, AppIcon.icns 和 Applications
    
2.  目录中右键点击 `查看显示选项`，按照下图更改选项：
    
    ![](https://cdn.wallleap.cn/img/pic/illustrtion/202212141406767.png)
    
3.  把 dmg.png 拖入显示选项下面的框中，为目录设置背景，调整 `timeGO.app` 和 `Applications` 到合适位置
    
4.  调整窗口大小刚好覆盖背景图
    
5.  在桌面映像挂载目录上右击选择 `显示简介`，会跳出目录的简介对话框，拖动目录中的图标文件到简介窗口左上角的 logo 位置，就把挂载目录的图标修改为应用图标了
    
6.  最后隐藏图标文件和背景图：
    
    ```
    cd /Volumes/timeGO
    chflags hidden AppIcon.icns
    chflags hidden dmg.png
    ```
    

最终打开映像的挂载目录会是类似下面的样子：

![](https://cdn.wallleap.cn/img/pic/illustrtion/202212141406758.png)

## 转换 dmg

最后一步转换 dmg 文件，这一步主要起到的作用是压缩文件，减小 dmg 文件的占用空间。

1.  弹出挂载的映像目录
    
2.  打开 `磁盘工具.app`
    
3.  菜单栏 - `映像` - `转换`，在弹出的对话框中选择刚刚创建的 dmg 文件：
    
    ![](https://cdn.wallleap.cn/img/pic/illustrtion/202212141406769.png)
    
4.  点击 `选取` 后，在弹窗中配置名称和导出目录：
    
    ![](https://cdn.wallleap.cn/img/pic/illustrtion/202212141406770.png)
    
5.  点击 `转换`，完成后就看到了最终的 dmg 文件了
    

最后可以打开这个 dmg 验证一下，是不是骚气了！