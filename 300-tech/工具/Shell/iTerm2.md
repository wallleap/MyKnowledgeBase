---
title: iTerm2
date: 2022-11-11 14:05
updated: 2022-11-11 14:05
cover: //cdn.wallleap.cn/img/pic.jpg
author: Luwang
comments: true
aliases:
  - 文章别名
rating: 10
tags:
  - 标签
category: 分类
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: null
url: null
---

## 1. 安装 iTerm2

下载地址：<https://www.iterm2.com/downloads.html>

- 下载的是压缩文件，解压后是执行程序文件，可以直接将它拖到 Applications 目录下。

- 或者你可以直接使用 Homebrew 进行安装：

```zsh
$ brew cask install iterm2
```

## 2. 配置 iTerm2 主题

iTerm2 最常用的主题是 Solarized Dark theme，下载地址：<http://ethanschoonover.com/solarized>

下载的是压缩文件，你先解压一下，然后打开 iTerm2，按`Command + ,`键，打开 Preferences 配置界面，然后`Profiles -> Colors -> Color Presets -> Import`，选择刚才解压的`solarized->iterm2-colors-solarized->Solarized Dark.itermcolors`文件，导入成功，最后选择 Solarized Dark 主题，就可以了。
![img](https://gitee.com/wallleap/cdn/raw/master/img/202202161102122.png)

## 3. 配置 Oh My Zsh

Oh My Zsh 是对主题的进一步扩展，地址：<https://github.com/robbyrussell/oh-my-zsh>

一键安装：

```shell
$ sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

安装好之后，需要把 Zsh 设置为当前用户的默认 Shell（这样新建标签的时候才会使用 Zsh）：

```shell
$ chsh -s /bin/zsh
```

然后，我们编辑`vim ~/.zshrc`文件，将主题配置修改为`ZSH_THEME="agnoster"`。

![img](https://gitee.com/wallleap/cdn/raw/master/img/202202161102464.png)

agnoster是比较常用的 zsh 主题之一，你可以挑选你喜欢的主题，zsh 主题列表：<https://github.com/robbyrussell/oh-my-zsh/wiki/themes>

效果如下（配置了声明高亮）：
![img](https://gitee.com/wallleap/cdn/raw/master/img/202202161102760.png)

## 4. 配置 Meslo 字体

使用上面的主题，需要 Meslo 字体支持，要不然会出现乱码的情况，字体下载地址：[<https://github.com/powerline/fonts/blob/master/Meslo> Slashed/Meslo LG M Regular for Powerline.ttf]

下载好之后，直接在 Mac OS 中安装即可。

然后打开 iTerm2，按`Command + ,`键，打开 Preferences 配置界面，然后`Profiles -> Text -> Font -> Chanage Font`，选择 `Meslo LG M Regular for Powerline`字体。
![img](https://gitee.com/wallleap/cdn/raw/master/img/202202161103227.png)

当然，如果你觉得默认的`12px`字体大小不合适，可以自己进行修改。

## 5. 声明高亮

效果就是上面截图的那样，特殊命令和错误命令，会有高亮显示。
在这个路径下 `cd ~/.oh-my-zsh/custom/plugins` 安装下载

```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git
```

然后激活这个插件，通过在

```bash
vi ~/.zshrc 
```

中找到`plugins`加入插件的名字

```ini
plugins=(git zsh-autosuggestions zsh-syntax-highlighting)
```

最后当然是source一下，让改变生效

```bash
source ~/.zshrc
```

## 6. 自动建议填充

这个功能是非常实用的，可以方便我们快速的敲命令。

配置步骤，先克隆zsh-autosuggestions项目，到指定目录：

```ruby
$ git clone https://github.com/zsh-users/zsh-autosuggestions ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions
```

然后编辑`vim ~/.zshrc`文件，找到`plugins`配置，增加`zsh-autosuggestions`插件。
![img](https://gitee.com/wallleap/cdn/raw/master/img/202202161103610.png)

最后当然是source一下，让改变生效

```bash
source ~/.zshrc
```

注：上面声明高亮，如果配置不生效的话，在plugins配置，再增加zsh-syntax-highlighting插件试试。

有时候因为自动填充的颜色和背景颜色很相似，以至于自动填充没有效果，我们可以手动更改下自动填充的颜色配置，我修改的颜色值为：586e75，示例：
![img](https://gitee.com/wallleap/cdn/raw/master/img/202202161103625.png)

效果：

## 7. iTerm2 快速隐藏和显示

这个功能也非常使用，就是通过快捷键，可以快速的隐藏和打开 iTerm2，比如：<kbd>Control</kbd>+<kbd>Commond</kbd>

## 8. iTerm2 隐藏用户名和主机名

有时候我们的用户名和主机名太长，比如我的xishuai@xishuaideMacBook-Pro，终端显示的时候会很不好看（上面图片中可以看到），我们可以手动去除。

看起来你用的是 item2 + oh-my-zsh 组合，假如你用的主题是 agnoster，修改方法是进入 oh-my-zsh/themes/然后 vi agnoster.zsh-theme，编辑主题配置文件，找到如下代码：

```dart
# Context: user@hostname (who am I and where am I)
prompt_context() {
  if [[ "$USER" != "$DEFAULT_USER" || -n "$SSH_CLIENT" ]]; then
   # 修改如下代码，@Mac可以你自己定义
   # prompt_segment black default "%(!.%{%F{yellow}%}.)$USER@%m"
   prompt_segment black default "$"
  fi
}
```

![img](https://gitee.com/wallleap/cdn/raw/master/img/202202161103310.png)

小苹果： ``

## 9. Archey:终端显示logo和信息

项目地址：<https://github.com/athlonreg/archey-osx>

安装：

```zsh
$ cd && git clone https://github.com/athlonreg/archey-osx 
$ sudo mv archey-osx/ /usr/local/ 
$ sudo ln -s /usr/local/archey-osx/bin/archey /usr/local/bin/archey #中文版
$ sudo ln -s /usr/local/archey-osx/bin/archey-en /usr/local/bin/archey-en #英文版
```

设置终端自启动：

```
$ echo archey >> ~/.zshrc #中文版
$ echo archey-en >> ~/.zshrc #英文版
$ source ~/.zshrc 
```

## 10. 在Dock栏隐藏

让我们的终端变得更 Cool，让它来无影去无踪。这一步我要 iTerm 启动后不再出现在 Dock 上，打开终端输入下面的命令，然后重启 iTerm。

```
/usr/libexec/PlistBuddy -c "Add :LSUIElement bool true" /Applications/iTerm.app/Contents/Info.plist
```

这个方法是通用的，`LSUIElement`可控制 app 以无Dock，无菜单栏的方式运行，另外`LSBackgroundOnly`可让 app 以无窗口的方式在后台运行。详细说明可查看 [LaunchServicesKeys](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Articles/LaunchServicesKeys.html)

如果要恢复 Dock 图标：

```zsh
/usr/libexec/PlistBuddy -c "Delete :LSUIElement" /Applications/iTerm.app/Contents/Info.plist
```

## 11. thefuck

一个好用的工具 可以到 GitHub 搜索

[nvbn/thefuck: Magnificent app which corrects your previous console command. (github.com)](https://github.com/nvbn/thefuck)

输错命令，然后有提示，但是不想手动复制粘贴，可以直接

```sh
xxxxxx # 纠正信息
fuck
```

![](https://cdn.wallleap.cn/img/pic/illustrtion/202211111405186.gif)
