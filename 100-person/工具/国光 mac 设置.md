---
title: 国光 mac 设置
date: 2024-07-17 17:43
updated: 2024-07-17 18:45
---

[国光的 macOS Monterey 12.X 配置记录 | 国光 (sqlsec.com)](https://www.sqlsec.com/2022/01/monterey.html#%E5%86%99%E5%9C%A8%E5%89%8D%E9%9D%A2)

[国光的 macOS 配置优化记录 | 国光 (sqlsec.com)](https://www.sqlsec.com/2019/12/macos.html)

[国光的 macOS Ventura 13 优化配置（基于 ARM 平台） | 国光 (sqlsec.com)](https://www.sqlsec.com/2023/07/ventura.html)

## 系统设置

### [](https://www.sqlsec.com/2022/01/monterey.html#%E5%AE%9E%E7%94%A8%E5%91%BD%E4%BB%A4 "实用命令") 实用命令

1. **取消 4 位数密码限制**

macOS 10.14 后不允许设置 4 位数以下的密码，对于一个弱口令的坚定维护者，每次敏感操作都要输入一长串的密码，国光我是接受不了的，所以可以使用下面的命令来取消关闭这个密码负责的策略的限制：

BASH

```bash
# 取消4位数密码限制 
➜ pwpolicy -clearaccountpolicies

# 更改密码
➜ passwd
```

1. **允许安装任意来源的 App**

有时候我们需要安装一些非官方 AppStore 的软件，这个时候就得到「安全与隐私」-「通用」里面来勾选开启任何来源，但是默认是没有这个选项的，在 macOS 早期是可以顺利开启的，但是到了近几年的版本我们得如需下面命令来手动开启：

BASH

```bash
# APP安装开启任何来源
➜ sudo spctl --master-disable
```

然后下面的这个任何来源的隐藏选项就出现啦

1. **修改主机名和共享名称**

在 shell 环境下会看到如下效果：

BASH

```bash
guang@guangdeMacBook-Pro ~ % 
```

这个主机名看上去实在比较臃肿，可以使用如下命令来修改：

BASH

```bash
# 修改主机名
➜ sudo scutil --set HostName {自定义主机名}
```

同样也可以修改共享电脑名称，使用如下命令：

BASH

```bash
# 修改共享名称
➜ sudo scutil --set ComputerName {自定义电脑名}
```

这样其他用户使用隔空投送的时候就会看到此时设置的自定义的共享名称了。 实际上这条命令的效果和在「设置」-「共享」里面的效果一模一样

1. **安装 Xcode Command Line Tools**

Command Line Tools 是在 Xcode 中的一款工具，macOS 下不少开发工具都会依赖这个，所以我们手动安装一下，后面安装其他工具可以省下不少麻烦：

BASH

```bash
# 安装 xcode 命令行工具
➜ xcode-select --install
```

1. **减少程序坞的响应时间**

如果只有一个屏幕的时候，国光我经常在「程序坞与菜单栏」勾选「自动隐藏和显示程序坞」，这样增加屏幕的利用率，但是 macOS 默认情况下唤出这个程序坞有点慢一拍的感觉，我们可以通过命令来减少响应时间：

BASH

```bash
# 设置启动坞动画时间设置为 0.5 秒
➜ defaults write com.apple.dock autohide-time-modifier -float 0.5 && killall Dock

# 设置启动坞响应时间最短
➜ defaults write com.apple.dock autohide-delay -int 0 && killall Dock
```

不要担心改不回来，可以使用下面的命令恢复系统初始化的设置：

BASH

```bash
# 恢复启动坞默认动画时间
➜ defaults delete com.apple.dock autohide-time-modifier && killall Dock

# 恢复默认启动坞响应时间
➜ defaults delete com.apple.Dock autohide-delay && killall Dock
```

1. **修改启动台行和列数**

启动台里面也可以设置应用的列和宽，使用如下命令即可：

BASH

```bash
# 设置列数为 9
➜ defaults write com.apple.dock springboard-columns -int 9

# 设置行数为 6
➜ defaults write com.apple.dock springboard-rows -int 6

# 重启 Dock 生效
➜ killall Dock
```

国光设置的 9 列 6 行的效果如下：

这个效果因人而异 ，大家可以自己去摸索，如果你想恢复默认的话可以使用下面的命令：

BASH

```bash
# 恢复默认的列数和行数
➜ defaults write com.apple.dock springboard-rows Default
➜ defaults write com.apple.dock springboard-columns Default

# 重启 Dock 生效
➜ killall Dock
```

## [](https://www.sqlsec.com/2022/01/monterey.html#%E8%A7%A6%E6%8E%A7%E4%BC%98%E5%8C%96 "触控优化") 触控优化

MBP 上使用触控板或者台式机外接妙控板进行鼠标指针拖移的话，默认得按压下去才可以拖动窗口，长时候按压难免让手指更加酸痛而且效率不高，这个时候在「辅助功能」-「指针控制」-「鼠标与触控板」开启传说中的「三指拖移」：

这才是 macOS 的触控板真正的精髓呀！效率提升杠杠的。

## [](https://www.sqlsec.com/2022/01/monterey.html#%E5%85%89%E6%A0%87%E5%93%8D%E5%BA%94 "光标响应") 光标响应

macOS 下的默认光标使用方向键左右移动是非常慢的，在终端下敲命令的时候尤为明显。这个时候得手动调快一些，才可以达到 Linux 下那种顺滑的移动效果，只需要到将「按键重复」调快一些，然后将「重复前延迟」调短一些即可：

[![](https://image.3001.net/images/20220123/1642948419864.png)](https://image.3001.net/images/20220123/1642948419864.png)

## [](https://www.sqlsec.com/2022/01/monterey.html#%E5%85%B3%E9%97%AD-SIP "关闭 SIP") 关闭 SIP

有些软件需要关闭 SIP 才可以实用，比如 `proxychains-ng` 这种系统级代理软件

### [](https://www.sqlsec.com/2022/01/monterey.html#%E7%99%BD%E8%8B%B9%E6%9E%9C "白苹果") 白苹果

白苹果得进恢复模式才可以修改，重启 Mac，按住 Option 键进入启动盘选择模式，按 `⌘` + `R` 进入 Recovery 模式。

「菜单栏」 ->「 实用工具（Utilities）」-> 「终端（Terminal）」：

BASH

```bash
# 关闭SIP
➜ csrutil disable

# 查看SIP状态
➜ csrutil status

System Integrity Protection status: disabled.(表明关闭成功)
```

### [](https://www.sqlsec.com/2022/01/monterey.html#%E9%BB%91%E8%8B%B9%E6%9E%9C "黑苹果") 黑苹果

黑苹果的话也可以进恢复模式，但是不如直接修改配置文件更方便高效。以 OC 为例，只要勾选「AllowToggleSip」：

[![](https://image.3001.net/images/20220123/16429486348168.png)](https://image.3001.net/images/20220123/16429486348168.png)

然后就可以在开机选择系统的时候手动切换 SIP 开关状态了：

[![](https://image.3001.net/images/20220123/16429487194499.jpeg)](https://image.3001.net/images/20220123/16429487194499.jpeg)

## [](https://www.sqlsec.com/2022/01/monterey.html#%E5%BC%80%E6%9C%BA%E8%87%AA%E5%90%AF "开机自启") 开机自启

有些类似截图等软件是每次开机必备的，貌似新版本系统软件里面的「开启启动」类似的选项都不太好用，不用担心，我们可以手动给他们设置开机自启：

[![](https://image.3001.net/images/20220123/16429491555746.png)](https://image.3001.net/images/20220123/16429491555746.png)

不过有些不良心的软件会偷偷自启，但是设置里面却看不到，不要头疼，我们可以使用腾讯柠檬清理来管理这类偷偷自启的软件：

[![](https://image.3001.net/images/20220124/16430351431948.png)](https://image.3001.net/images/20220124/16430351431948.png)

## [](https://www.sqlsec.com/2022/01/monterey.html#%E7%BC%96%E7%A8%8B%E5%BC%80%E5%8F%91 "编程开发") 编程开发

### [](https://www.sqlsec.com/2022/01/monterey.html#iTerm2 "iTerm2")iTerm2

iTerm2 的颜值真的挺高的（配合 Zsh 主题的话），但是默认情况下有的丑，需要我们稍微设置一下颜值才会高一点。

- **Preferences - Apperance**

主要设置一下主题以及标签页的显示位置，下面是国光的设置习惯：

[![](https://image.3001.net/images/20220124/16430353388480.png)](https://image.3001.net/images/20220124/16430353388480.png)

另外记得到这个 Windows 标签页勾选「Hide scrollbars」隐藏滚动条，让终端看上去更简约一点。

- **新建配置 - 字体设置**

新建一个自己的配置，并设置为默认「Set as Default」，在新的配置里面可以进行字体相关的设置，字体大小国光设置的是 16 号，使用的是 macOS 自带的 Monaco 高颜值字体：

[![](https://image.3001.net/images/20220124/16430355004637.png)](https://image.3001.net/images/20220124/16430355004637.png)

- **新建配置 - 窗口设置**

[![](https://image.3001.net/images/20220124/16430356065859.png)](https://image.3001.net/images/20220124/16430356065859.png)

### [](https://www.sqlsec.com/2022/01/monterey.html#Homebrew "Homebrew")Homebrew

Homebrew 是一款自由及开放源代码的软件包管理系统，用以简化 macOS 系统上的软件安装过程，程序猿开发必备神器。

BASH

```bash
➜ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

> 第一次安装可能有点慢，给终端挂个代理可以提高下速度：
>
> [![](https://image.3001.net/images/20220123/16429478396298.png)](https://image.3001.net/images/20220123/16429478396298.png)
>
> BASH
>
> ```bash
> # ClashX 代理示例
> export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7890
> ```
>
> 没有代理的话可与去刷个抖音冷静一下。

国光的 brew 常用命令与软件：

BASH

```bash
# 更新 Homebrew
➜ brew update

#  搜索相关的包
➜ brew search [关键词] 

# 查看包的信息
➜ rew info [软件名]
 
# 查看已安装的包
➜ brew list

# 更新某个软件
➜ brew upgrade [软件名]

# 清理所有软件的旧版
➜ brew cleanup

# 卸载某个软件
➜ brew uninstall [软件名]

# ip 命令 查看ip地址很方便
➜ brew install iproute2mac
```

诸如 Nginx、MySQL 等软件，都是有一些服务端软件在后台运行，如果你希望对这些软件进行管理，可以使用 `brew services` 命令来进行管理：

BASH

```bash
# 查看所有服务 
➜ brew services list：

# 单次运行某个服务
➜ brew services run [服务名]

# 运行某个服务，并设置开机自动运行
➜ brew services start [服务名]

# 停止某个服务
➜  brew services stop [服务名]

# 重启某个服务
➜ brew services restart
```

国光的 brew 常用配置：

BASH

```bash
# ip 命令 查看ip地址很方便
brew install iproute2mac

# 安装 brew-cask
brew install cask

# 空格预览 markdown
brew install qlmarkdown

# 空格预览代码文件
brew install syntax-highlight
```

### [](https://www.sqlsec.com/2022/01/monterey.html#Oh-My-Zsh "Oh My Zsh")Oh My Zsh

macOS Monterey 使用 zsh 作为默认 Shell，不过这个自带的 zsh 还不够好看，需要使用 Oh My Zsh 这个 zsh 美化增强脚本，来让自己的终端 Shell 颜值更逼格。

#### [](https://www.sqlsec.com/2022/01/monterey.html#%E5%AE%89%E8%A3%85 "安装") 安装

BASH

```bash
# 安装卡的话就挂代理
➜ sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

#### [](https://www.sqlsec.com/2022/01/monterey.html#%E4%B8%BB%E9%A2%98 "主题") 主题

用户目录下的 `.zshrc` 文件为 Oh My Zsh 的配置文件

BASH

```bash
#修改配置文件
➜ vim ~/.zshrc
```

修改此行（等于号 后面填写自己的主题）：

BASH

```bash
ZSH_THEME=robbyrussell
```

Oh My Zsh 已经内置了很多主题了：

BASH

```bash
# 查看自带的主题
➜ ls -l ~/.oh-my-zsh/themes
```

国光以前也比较喜欢花里胡哨的折腾这些，后来发现画船听雨大佬的 zsh 主题就是默认的 =，= 后面我就也没有再折腾这些花里胡哨的主题了。

#### [](https://www.sqlsec.com/2022/01/monterey.html#%E6%8F%92%E4%BB%B6 "插件") 插件

##### [](https://www.sqlsec.com/2022/01/monterey.html#autojump "autojump")autojump

目录切换神器，大大提高工作效率。

BASH

```bash
➜ brew install autojump
```

在 `~/.zshrc` 中配置

CODE

```none
plugins=(其他的插件 autojump)
```

输入 `zsh` 命令生效配置后即可正常使用 `j` 命令，下面是简单的演示效果：

BASH

```bash
# 第一次 cd 进入某个目录
➜  ~ cd Documents/Hexo
➜  Hexo cd ~

# 后面就可以直接通过 j 命令跳转到那个目录
➜  ~ j hexo
/Users/sec/Documents/Hexo
```

##### [](https://www.sqlsec.com/2022/01/monterey.html#autosuggestions "autosuggestions")autosuggestions

终端下自动提示接下来可能要输入的命令，这个实际使用效率还是比较高的

BASH

```bash
# 拷贝到 plugins 目录下
➜ git clone https://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
```

在 `~/.zshrc` 中配置：

BASH

```bash
plugins=(其他的插件 zsh-autosuggestions)
```

输入 `zsh` 命令生效配置

##### [](https://www.sqlsec.com/2022/01/monterey.html#zsh-syntax-highlighting "zsh-syntax-highlighting")zsh-syntax-highlighting

命令输入正确会绿色高亮显示，可以有效地检测命令语法是否正确

BASH

```bash
# 拷贝到 plugins 目录下
➜ git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

在 `~/.zshrc` 中配置：

BASH

```bash
plugins=(其他的插件 zsh-syntax-highlighting)
```

输入 `zsh` 命令生效配置。

#### [](https://www.sqlsec.com/2022/01/monterey.html#%E5%85%B6%E4%BB%96 "其他") 其他

##### [](https://www.sqlsec.com/2022/01/monterey.html#%E5%85%B3%E6%8E%89-URL-%E5%8F%8D%E6%96%9C%E6%9D%A0%E8%BD%AC%E4%B9%89 "关掉 URL 反斜杠转义") 关掉 URL 反斜杠转义

经常在 zsh 里面粘贴 URL 会出现下面的情况：

CODE

```none
http://xxx/?id=1
```

粘贴到 iTerm2 的 zsh 中会变成这样：

CODE

```none
http://xxx/\?id\=1
```

为了避免每次粘贴后都要修改的麻烦，我们需要手工关掉 URL 特殊符号转义，编辑 `.zshrc` 配置文件，取消下面内容的注释：

INI

```ini
DISABLE_MAGIC_FUNCTIONS=true
```

编辑完成后，保存输入 `zsh` 命令重新加载配置文件，再次粘贴就不会出现这种情况了。

##### [](https://www.sqlsec.com/2022/01/monterey.html#%E7%A6%81%E7%94%A8-zsh-%E8%87%AA%E5%8A%A8%E6%9B%B4%E6%96%B0 "禁用 zsh 自动更新") 禁用 zsh 自动更新

新版本的 zsh 禁用自动更新的方式稍微有点变化，编辑 `.zshrc` 配置文件，取消下面内容的注释即可：

BASH

```bash
zstyle ':omz:update' mode disabled
```

### [](https://www.sqlsec.com/2022/01/monterey.html#proxychains-ng "proxychains-ng")proxychains-ng

终端命令行下代理神器，可以让指定的命令走设置好的代理，内网渗透、科学上网必备工具：

BASH

```bash
# 安装
➜ brew install proxychains-ng

# 配置文件
➜ vim /usr/local/etc/proxychains.conf
```

将结尾的 `socks4 127.0.0.1 9095` 改为

CODE

```none
socks5 127.0.0.1 7890
```

> 国光的 macOS 下 ClashX 的默认代理为 7890 然后我就习惯了 7890 端口 这里根据自己的情况来设置。

下面是常用的几个命令：

BASH

```bash
# 代理终端基本示例
➜ proxychains4 curl https://www.google.com.hk

# 全局代理 bash shell
➜ proxychains4 -q /bin/bash

# 全局代理 zsh shell
➜ proxychains4 -q /bin/zsh
```

如果发现你的 proxychains 并不能代理的话，尝试根据前面的部分关掉 SIP 后再试试看。

比较巧的是写本文的 1 小时前 proxychains-ng 刚刚支持 macOS 12 Monterey 系统的版本：

[![](https://image.3001.net/images/20220123/16429523314084.png)](https://image.3001.net/images/20220123/16429523314084.png)

### [](https://www.sqlsec.com/2022/01/monterey.html#Vim "Vim")Vim

macOS 自带的 vim 是没有任何配色的，可以下面是国光常用的配色方案，先在用户目录下新建一个 vim 的配置文件：

BASH

```bash
➜ vim ~/.vimrc
```

内容如下：

BASH

```bash
set nu                " 显示行号
colorscheme desert    " 颜色显示方案
syntax on             " 打开语法高亮
```

配置修改为完，输入 `zsh` 命令生效配置，国光我这里使用的 `desert` 配色方案，其他自带的配色可以参考这个目录下：

BASH

```bash
➜  ls /usr/share/vim/vim*/colors 

README.txt	delek.vim	industry.vim	pablo.vim	slate.vim
blue.vim	desert.vim	koehler.vim	peachpuff.vim	tools
darkblue.vim	elflord.vim	morning.vim	ron.vim		torte.vim
default.vim	evening.vim	murphy.vim	shine.vim	zellner.vim
```

感兴趣的朋友可以自己去一个个尝试一下。

### [](https://www.sqlsec.com/2022/01/monterey.html#Git "Git")Git

Github 会根据这个邮箱显示对应的 commit 记录的头像

BASH

```bash
# 配置邮箱 
➜ git config --global user.email "xxxxx@xxx.com"

# 配置用户名
➜ git config --global user.name "国光"
```

下图中间两个 commit 记录没有头像的原因就是没有配置上述邮箱的问题造成的：

[![](https://dn-coding-net-tweet.codehub.cn/photo/2019/a7d2fd31-e811-4c1a-8a96-5c4058c47a53.jpg)](https://dn-coding-net-tweet.codehub.cn/photo/2019/a7d2fd31-e811-4c1a-8a96-5c4058c47a53.jpg)

#### [](https://www.sqlsec.com/2022/01/monterey.html#%E9%85%8D%E7%BD%AEsocks5%E4%BB%A3%E7%90%86 "配置socks5代理") 配置 socks5 代理

在天朝 使用 git clone 的正确姿势，配置这个代理的前提是自己得有一个可以访问国外的 Socks5 代理（实际上不嫌麻烦的话每次手动开启下终端代理也可以）：

BASH

```bash
# 我这里习惯用 7890 端口 具体根据自己的配置来灵活设置
➜ git config --global http.proxy 'socks5://127.0.0.1:7890'
➜ git config --global https.proxy 'socks5://127.0.0.1:7890'

# 取消 git 代理
➜ git config --global --unset http.proxy
➜ git config --global --unset https.proxy

# 查看 git 配置
➜ cat ~/.gitconfig

# 或者这样也可以查看 git 的配置
➜ git config -l
```

### [](https://www.sqlsec.com/2022/01/monterey.html#Python "Python")Python

#### [](https://www.sqlsec.com/2022/01/monterey.html#Python2 "Python2")Python2

macOS Monterey 自带 Python 2.7.18 的版本，不过没有安装 pip，得自己再折腾完善一下。

BASH

```bash
# 安装 pip （貌似 Python2 的 PIP 太新会出问题 可以指定最后一个稳定的版本）
➜ sudo easy_install pip==20.3.4

# 查看pip的版本
➜ pip -V
pip 20.3.4 from /Library/Python/2.7/site-packages/pip-20.3.4-py2.7.egg/pip (python 2.7)
```

> 貌似 macOS 12.3 Beta 版本开始 macOS 开始不自带 Python2 工具了，不过还好有 pyenv 这个神器。

#### [](https://www.sqlsec.com/2022/01/monterey.html#Python3 "Python3")Python3

macOS Monterey 自带 Python 3.8.9 的版本，手动敲 Python3 会自动调用安装对应依赖，等待安装好即可。

同时也会自动安装好 pip3：

BASH

```bash
➜ pip3 -V
pip 20.2.3 from /Library/Developer/CommandLineTools/Library/Frameworks/Python3.framework/Versions/3.8/lib/python3.8/site-packages/pip (python 3.8)
```

但是版本不够新，我们可以手动升级最新版的 pip3：

BASH

```bash
/Library/Developer/CommandLineTools/usr/bin/python3 -m pip install --upgrade pip
```

#### [](https://www.sqlsec.com/2022/01/monterey.html#pyenv "pyenv")pyenv

pyenv 是一个强大 Python 包管理工具，可以灵活地切换各种 Python 版本。

BASH

```bash
# 安装 pyenv
➜ brew install pyenv
```

将以下内容写入到 `~/.zshrc` 配置文件：

BASH

```bash
alias brew='env PATH="${PATH//$(pyenv root)\/shims:/}" brew'
```

接着为 pyenv 配置 shell 环境：

BASH

```bash
echo 'eval "$(pyenv init --path)"' >> ~/.zprofile
echo 'eval "$(pyenv init -)"' >> ~/.zshrc
```

常用的一些命令下面一一列出来。

- **版本安装**

BASH

```bash
# 查看已经安装的Python版本
➜ pyenv versions

# 查看当前的 Python 版本
➜ pyenv version

# 查看可安装的版本
➜ pyenv install -l

# 安装与卸载 pypy3.8-7.3.7
➜ pyenv install pypy3.8-7.3.7
➜ pyenv uninstall pypy3.8-7.3.7
```

所安装的版本都在 `~/.pyenv/versions` 目录下。

- **版本切换**

BASH

```bash
# global 全局设置 一般不建议改变全局设置
➜ pyenv global <python版本>

# shell 会话设置 只影响当前的shell会话
➜ pyenv shell <python版本>
# 取消 shell 会话的设置
➜ pyenv shell --unset

# local 本地设置 只影响所在文件夹
➜ pyenv local <python版本>
```

pyenv 的 global、local、shell 的优先级关系是：shell > local > global

### [](https://www.sqlsec.com/2022/01/monterey.html#Node-js "Node.js")Node.js

国光我 Nodejs 用的不多，主要就用来跑跑 Hexo 博客和使用 Gitbook 来写点知识点，又因为 Node.js 版本也比较凌乱，所以这里主要是通过 nvm 来进行配置管理。

BASH

```bash
# 安装 nvm
➜ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash

# 查看版本信息
➜ zsh
➜ nvm --version
0.39.1

# 查看当前 node 的版本
➜ nvm version 

# 安装最新稳定版 node
➜ nvm install stable

# 列出所有远程服务器的版本
➜ nvm ls-remote

# 安装指定版本
➜ nvm install v12.22.9
➜ nvm install <version>

# 列出所有已安装的版本
➜ nvm ls

# 卸载指定的版本
➜ nvm uninstall <version>

# 切换使用指定的版本node
➜ nvm use <version>

# 显示当前的版本
➜ nvm current
```

### [](https://www.sqlsec.com/2022/01/monterey.html#Hexo "Hexo")Hexo

Hexo 是一个由 Node.js 编写的博客系统，国光我的博客用了 Hexo 已经很多年了，大家如果要写博客的话，也建议大家尝试 Hexo 看看。

安装配置好 Node.js 之后，只需要如下命令即可安装好 Hexo：

BASH

```bash
➜ npm install hexo-cli -g
```

下面是国光常用的一个 Hexo 命令：

BASH

```bash
# 清除缓存重新生成并启动 hexo 本地服务
$ hexo clean && hexo g && hexo s
```

因为国光我是把 Hexo 博客直接部署到阿里云服务器的，所以我不用 `hexo d` 这个命令好多年了。

### [](https://www.sqlsec.com/2022/01/monterey.html#MySQL "MySQL")MySQL

#### [](https://www.sqlsec.com/2022/01/monterey.html#%E5%AE%89%E8%A3%85-MySQL "安装 MySQL") 安装 MySQL

BASH

```bash
# 搜索可以安装的版本
➜ brew search mysql

# 安装对应的版本
➜ brew install mysql@5.7
```

#### [](https://www.sqlsec.com/2022/01/monterey.html#%E9%85%8D%E7%BD%AE%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F "配置环境变量") 配置环境变量

BASH

```bash
# 查看 MySQL 可执行文件的位置
➜ cd /usr/local/Cellar/mysql@5.7/*/bin && pwd
/usr/local/Cellar/mysql@5.7/5.7.37/bin
```

然后下面内容写入 `~/.zshrc` 配置文件中：

BASH

```bash
export PATH="${PATH}:/usr/local/Cellar/mysql@5.7/5.7.37/bin"
```

写入完成后可以使用 `zsh` 刷新一下配置或者手动 `source ~/.zshrc` 一下。

#### [](https://www.sqlsec.com/2022/01/monterey.html#%E5%B8%B8%E7%94%A8%E6%93%8D%E4%BD%9C "常用操作") 常用操作

BASH

```bash
# 查看 M有SQL 服务状态
➜ brew services info mysql@5.7
➜ mysql.server status

# 启动 MySQL 服务
➜ brew services start mysql@5.7
➜ mysql.server start

# 重启 MySQL 服务
➜ brew services restart mysql@5.7
➜ mysql.server restart

# 停止 MySQL 服务
➜ brew services stop mysql@5.7
➜ mysql.server stop
```

#### [](https://www.sqlsec.com/2022/01/monterey.html#%E8%AE%BE%E7%BD%AE-MySQL-%E5%AF%86%E7%A0%81 "设置 MySQL 密码") 设置 MySQL 密码

BASH

```bash
# 默认是 root 用户是空密码 可以直接登录
➜ mysql -uroot

# 修改 root 密码的 SQL语句
mysql > use mysql;
mysql > set password for 'root'@'localhost' = password('你设置的密码');

# 刷新权限 并退出
mysql > flush privileges;
mysql > quit; 
```

#### [](https://www.sqlsec.com/2022/01/monterey.html#%E6%95%B0%E6%8D%AE%E5%BA%93%E5%A4%96%E8%BF%9E "数据库外连") 数据库外连

这是个可选操作 根据自己的实际情况自行决定是否开启（有被攻击的风险）：

SQL

```sql
mysql > grant all on *.* to root@'%' identified by '你设置的密码' with grant option;
mysql > flush privileges;
```

大家根据实际情况来决定是否开启外连。

### [](https://www.sqlsec.com/2022/01/monterey.html#Redis "Redis")Redis

BASH

```bash
# 安装 redis
➜ brew install redis

# 启动 redis 服务端
➜ redis-server

# 启动 redis 客户端
➜ redis-cli

# 编辑默认配置文件
➜ sudo vim /usr/local/etc/redis.conf
```

### [](https://www.sqlsec.com/2022/01/monterey.html#Gitbook "Gitbook")Gitbook

Gitbook 确实是一个写文档的神器，对于我来说是必装的了。这个坑点就是使用高版本的 Node.js 安装会出各种玄学问题，换 10.X 的版本即可：

BASH

```bash
# 安装并切换老版本的 Node.js
➜ nvm install v10.24.1
➜ nvm use v10.24.1

# 安装 Gitbook-cli
➜ npm install -g gitbook-cli

# 验证版本并安装相关依赖
➜ gitbook -V
CLI version: 2.3.2
GitBook version: 3.2.3
```

## [](https://www.sqlsec.com/2022/01/monterey.html#%E5%AE%89%E5%85%A8%E7%9B%B8%E5%85%B3 "安全相关") 安全相关

大家都是搞安全的，配置环境这个对你们来说很简单的吧，这里就不再赘述了，之前的老文章有这部分的配置，大家也可以参考一下，然后发散一下，基本上足够了：[国光的 macOS 配置优化记录](https://www.sqlsec.com/2019/12/macos.html)

总之 macOS 用来做渗透等安全研究还是比较舒服的。

## [](https://www.sqlsec.com/2022/01/monterey.html#%E5%8F%82%E8%80%83%E8%B5%84%E6%96%99 "参考资料") 参考资料

- [国光的 macOS 配置优化记录](https://www.sqlsec.com/2019/12/macos.html)

## 系统设置

### [](https://www.sqlsec.com/2019/12/macos.html#%E5%AE%9E%E7%94%A8%E5%91%BD%E4%BB%A4 "实用命令") 实用命令

1. **取消 4 位数密码限制**

macOS 10.14 后不允许设置 4 位数一下的密码，对于一个弱口令的坚定维护者，每次敏感操作都要输入一长串的密码，国光我是接受不了的，所以可以使用下面的命令来取消关闭这个密码负责的策略的限制：

BASH

```bash
# 取消4位数密码限制 
pwpolicy -clearaccountpolicies

# 更改密码
passwd
```

1. **允许 App 的来源**

有时候我们需要安装一些非官方 AppStore 的软件，这个时候就得到「安全与隐私」-「通用」里面来勾选开启任何来源，但是默认是没有这个选项的，在 macOS 早期是可以顺利开启的，但是到了近几年的版本我们得如需下面命令来手动开启：

BASH

```bash
# APP安装开启任何来源
sudo spctl --master-disable
```

然后下面的这个任何来源的隐藏选项就出现啦：

[![](https://image.3001.net/images/20200627/15932318244185.png)](https://image.3001.net/images/20200627/15932318244185.png)

1. **修改主机名和共享名称**

在 zsh shell 环境下会看到如下效果：

BASH

```bash
guang@guangdeMacBook-Pro ~ % 
```

这个主机名看上去实在比较臃肿，可以使用如下命令来修改：

BASH

```bash
# 修改主机名
sudo scutil --set HostName xxxxx
```

同样也可以修改共享名称，使用如下命令：

BASH

```bash
# 修改共享名称
sudo scutil --set ComputerName xxxxx
```

这样其他用户使用隔空投送的时候就会看到此时设置的自定义的共享名称了。

1. **xcode 命令行工具**

Command Line Tools 是在 Xcode 中的一款工具，macOS 下不少开发工具都会依赖这个，所以我们手动安装一下，后面安装其他工具可以省下不少麻烦：

BASH

```bash
# 安装 xcode 命令行工具
xcode-select --install
```

### [](https://www.sqlsec.com/2019/12/macos.html#%E7%A8%8B%E5%BA%8F%E5%9D%9E "程序坞") 程序坞

首先移除掉入国内基本上不会使用的 APP：

[![](https://image.3001.net/images/20200625/15930478227897.png)](https://image.3001.net/images/20200625/15930478227897.png)

「自动显示和隐藏程序坞」一般使用触控板的时候呢，默认是不勾选隐藏的，如果勾选隐藏的话，我们需要定制一 Dock 的唤醒动画。

因为 macOS 系统默认的程序坞唤醒是有些许延迟的，Dock 程序坞显示隐藏缓慢的原因，是因为 OS X 隐藏和显示 Dock 的动画持续时间被设置成了 1 秒，想要改变这一时间，只需要打开终端，选择以下代码的其中一项执行就可以实现：

BASH

```bash
# 设置启动坞动画时间设置为 0.5 秒
defaults write com.apple.dock autohide-time-modifier -float 0.5 && killall Dock

# 恢复启动坞默认动画时间
defaults delete com.apple.dock autohide-time-modifier && killall Dock

# 设置启动坞响应时间最短
defaults write com.apple.dock autohide-delay -int 0 && killall Dock

# 恢复默认启动坞响应时间
defaults delete com.apple.Dock autohide-delay && killall Dock
```

时间可以自己调整，基本上调整下来感觉速度比原来快多了。

### [](https://www.sqlsec.com/2019/12/macos.html#%E5%90%AF%E5%8A%A8%E5%8F%B0 "启动台") 启动台

启动台里面也可以设置应用的列和宽，使用如下命令即可：

BASH

```bash
# 设置列数
defaults write com.apple.dock springboard-columns -int 9

# 设置行数
defaults write com.apple.dock springboard-rows -int 6

# 重启 Dock 生效
killall Dock
```

国光设置的 9 列 6 行的效果如下：

[![](https://image.3001.net/images/20200625/15930696443262.jpg)](https://image.3001.net/images/20200625/15930696443262.jpg)

显示数量明显比默认的效果要好很多 ，如果你想恢复默认的话可以使用下面的命令。

恢复默认的配置：

BASH

```bash
# 恢复默认的列数和行数
defaults write com.apple.dock springboard-rows Default
defaults write com.apple.dock springboard-columns Default

# 重启 Dock 生效
killall Dock
```

### [](https://www.sqlsec.com/2019/12/macos.html#%E8%BE%93%E5%85%A5%E6%B3%95 "输入法") 输入法

使用搜狗输入法替代自带的简体拼音输入法之后，为了切换方便，得手动临时去掉自带的简体拼音输入法：

[![](https://image.3001.net/images/20200625/15930768628180.png)](https://image.3001.net/images/20200625/15930768628180.png)

### [](https://www.sqlsec.com/2019/12/macos.html#%E4%B8%89%E6%8C%87%E6%8B%96%E7%A7%BB "三指拖移") 三指拖移

MBP 上使用触控板拖移的话，默认得按压下去才可以拖动窗口，长时候按压再加上钢板手感的蝶式键盘会让手指更加酸痛，这个时候开启

[![](https://image.3001.net/images/20200625/15930526249767.png)](https://image.3001.net/images/20200625/15930526249767.png)

### [](https://www.sqlsec.com/2019/12/macos.html#%E5%85%89%E6%A0%87%E5%93%8D%E5%BA%94 "光标响应") 光标响应

macOS 下的默认光标使用方向键左右移动是非常慢的，这个时候得手动调快一些，才可以达到 Linux 下那种顺滑的移动效果，只需要到将「按键重复」调快一些，然后将「重复前延迟」调短一些即可：

[![](https://image.3001.net/images/20200625/15930711274021.png)](https://image.3001.net/images/20200625/15930711274021.png)

### [](https://www.sqlsec.com/2019/12/macos.html#%E5%85%B3%E9%97%AD-SIP "关闭 SIP") 关闭 SIP

后面不关闭 SIP 的话类似于 `proxychains-ng` 这种代理神器就无法使用。

重启 Mac，按住 Option 键进入启动盘选择模式，按 `⌘` + `R` 进入 Recovery 模式。

「菜单栏」 ->「 实用工具（Utilities）」-> 「终端（Terminal）」：

BASH

```bash
# 关闭SIP
csrutil disable

# 查看SIP状态
csrutil status

System Integrity Protection status: disabled.(表明关闭成功)
```

## [](https://www.sqlsec.com/2019/12/macos.html#%E8%BD%AF%E4%BB%B6%E7%9B%B8%E5%85%B3 "软件相关") 软件相关

这里比较多，国光只简单列举几个典型的列子，关于国光的软件清单大家可以参考我的这篇文章：

### [](https://www.sqlsec.com/2019/12/macos.html#Google-Chrome "Google Chrome")Google Chrome

Chrome 浏览器受欢迎的一个主要原因就是有丰富的插件，下面是国光使用的 Chrome 插件清单：

| 插件名                | 说明                       | 链接地址                                                                                                                 |
| :----------------- | :----------------------- | :------------------------------------------------------------------------------------------------------------------- |
| Charset            | 修改网站的默认编码                | [Github-Chrome-Charset](https://github.com/jinliming2/Chrome-Charset)                                                |
| Fatkun 图片批量下载      | 批量爬取下载图片神器               | [chrome 网上应用店](https://chrome.google.com/webstore/detail/fatkun-batch-download-ima/nnjjahlikiabnchcpehcpkdeckfgnohf) |
| Link Grabber       | 批量提取网页链接                 | [chrome 网上应用店](https://chrome.google.com/webstore/detail/link-grabber/caodelkhipncidmoebgbbeemedohcdma)              |
| Proxy SwitchyOmega | 轻松快捷地管理和切换多个代理           | [chrome 网上应用店](https://chrome.google.com/webstore/detail/proxy-switchyomega/padekgcemlokbadohgkifijomclgjgif)        |
| Separate Window    | 局部视频选定播放支持 20 倍播放        | [chrome 网上应用店](https://chrome.google.com/webstore/detail/separate-window/cbgkkbaghihhnaeabfcmmglhnfkfnpon)           |
| Shodan             | 信息收集插件之端口探测              | [chrome 网上应用店](https://chrome.google.com/webstore/detail/shodan/jjalcfnidlmpjhdfepjhjbhnhkbgleap)                    |
| Tampermonkey       | 支持各种强大的插件                | [chrome 网上应用店](https://chrome.google.com/webstore/detail/tampermonkey/dhdgffkkebhmkfjojejmpbldmpobfkfo)              |
| uBlacklist         | 禁止特定的网站显示在 Google 的搜索结果中 | [chrome 网上应用店](https://www.sqlsec.com/2019/12/macos.html)                                                            |
| Wappalyzer         | 网站指纹基础信息探测               | [chrome 网上应用店](https://chrome.google.com/webstore/detail/wappalyzer/gppongmhjkpfnbhagpmjfkannfbllamg)                |
| 超级简单的自动刷新          | 顾名思义，自动刷新                | [chrome 网上应用店](https://chrome.google.com/webstore/detail/super-simple-auto-refresh/gljclgacfalmnebgmhknodlplmngmfpi) |

### [](https://www.sqlsec.com/2019/12/macos.html#iTerm2 "iTerm2")iTerm2

**Apperance**

窗口主题和标签页的停靠位置，这里我设置的左边，iTerm2 标签页放在顶部有时候很容易误关！！主题自带的 「Minima」和「Compact」都还是不错的，然后标签位置选择「Left」：

[![](https://dn-coding-net-tweet.codehub.cn/photo/2019/a5523b36-eb56-47fd-8f92-e9c811a8d1aa.jpg)](https://dn-coding-net-tweet.codehub.cn/photo/2019/a5523b36-eb56-47fd-8f92-e9c811a8d1aa.jpg)

**Profiles**

新建一个自己的配置，并设置为默认「Set as Default」，在新的配置里面可以进行字体相关的设置，字体大小国光设置的是 16 号，使用的是 macOS 自带的 Monaco 高颜值字体：

[![](https://dn-coding-net-tweet.codehub.cn/photo/2019/eb94e283-eb45-414a-87d7-2f9658ec8f85.jpg)](https://dn-coding-net-tweet.codehub.cn/photo/2019/eb94e283-eb45-414a-87d7-2f9658ec8f85.jpg)

接着在「Window」标签下可以设置 iTerm2 终端窗口大小、背景透明程度、背景毛玻璃虚化程度，下面是国光我设置的细节图：

[![](https://dn-coding-net-tweet.codehub.cn/photo/2019/cfe1b1c7-e523-4a34-859d-80afd469a6da.jpg)](https://dn-coding-net-tweet.codehub.cn/photo/2019/cfe1b1c7-e523-4a34-859d-80afd469a6da.jpg)

**配置效果**

[![](https://dn-coding-net-tweet.codehub.cn/photo/2019/027daa5c-a4fb-4e12-93d6-945a73eae0b9.jpg)](https://dn-coding-net-tweet.codehub.cn/photo/2019/027daa5c-a4fb-4e12-93d6-945a73eae0b9.jpg)

**一键调出 iTerm2 终端**

有时候我们只突然想用命令行进行一个即时操作，这种情况下再去打开 iTerm2 然后新建标签，这样就会显得比较凌乱不优雅，实际上 iTerm2 自带快捷键唤醒功能，我们只需要开启并制定快捷键即可，首先创建一个热键配置：

[![](https://image.3001.net/images/20200625/15930873122338.png)](https://image.3001.net/images/20200625/15930873122338.png)

国光我配置的是双击 Control 键唤醒，然后勾选浮动窗口，这样就可以随时随地唤醒了，无论当前是否处在全屏应用中都可以唤醒：

[![](https://image.3001.net/images/20200625/15930872608071.png)](https://image.3001.net/images/20200625/15930872608071.png)

实际效果很赞，这个功能很实用，强烈建议大家尝试一下，只需 3 翻钟，你就会爱介个操作。

### [](https://www.sqlsec.com/2019/12/macos.html#Typora "Typora")Typora

Typora 用来写 markdown 自带的默认体验就不差，国光我主要就是在图像这里进行了调整，默认使用相对路径，报错到当前 md 文件的 imgs 目录下，另外 Typora 后面新的版本支持自定义命令上传，所以国光我就写了图床脚本，用起来很舒服：

[![](https://image.3001.net/images/20200627/15932318369965.png)](https://image.3001.net/images/20200627/15932318369965.png)

另外根据自己的喜欢，在「偏好设置」-「外观」 勾选「侧边栏的大纲视图允许折叠和展开」，关于 Typora 的主题，国光我使用的是 [Vue 黑](https://github.com/blinkfox/typora-vue-theme) 的主题，后面发现 [Pie](https://theme.typora.io/theme/Pie/) 主题颜值也还是不错的。

### [](https://www.sqlsec.com/2019/12/macos.html#VS-Code "VS Code")VS Code

VS Code 使用的也比较多，国光这里简单列举一下我平时使用的插件列表：

**Chinese (Simplified) Language Pack for Visual Studio Code**：官方中文插件

**VSCode Great Icons**：文件夹图标主题

**filesize**：左下角显示文件大小

**ASCIIDecorator**：常用来生成一些酷炫的 logo

**Vibrancy**：VS Code 毛玻璃效果插件

**Terminal**：命令行爱好者必备插件

**Beautify**：代码格式美化插件

**Excel Viewer**：CSV 表格内容显示插件

**Diff**：比较 2 个文件的区别

**Python**：微软官方的 Python 插件

**kite**：Python 语句补全插件 自动语义提示效率很高（得配合 Kite 这个 APP 来使用）

**hexdump for VSCode**: 以十六进制显示指定文件

上一个默认代码配色的效果图：

[![](https://dn-coding-net-tweet.codehub.cn/photo/2019/3752ccf0-330a-4adf-8ddf-4c77d3c0aa45.jpg)](https://dn-coding-net-tweet.codehub.cn/photo/2019/3752ccf0-330a-4adf-8ddf-4c77d3c0aa45.jpg)

### [](https://www.sqlsec.com/2019/12/macos.html#Atom "Atom")Atom

第一次启动 Atom 可能会出现如下的报错：

CODE

```none
The package `spell-check` cannot load the system dictionary for `zh-CN`. See the settings for ways of changing the languages used, resolving missing dictionaries, or hiding this warning.
```

这个是 Atom 拼写检查插件报错，解决方法很简单 `Packages / Spell Check` 去掉勾选 **Use Locales**

#### [](https://www.sqlsec.com/2019/12/macos.html#%E5%AD%97%E4%BD%93 "字体") 字体

国光习惯编辑器使用的是 `IBMPlexMono` 字体

谷歌字体详情链接：[https://fonts.google.com/specimen/IBM+Plex+Mono](https://fonts.google.com/specimen/IBM+Plex+Mono)

`Settings`-`Editor`-`Font Family` 中 Atom 默认的字体为：

CODE

```none
Menlo, Consolas, DejaVu Sans Mono, monospace
```

国光安装第三方字体手设置了为：

CODE

```none
IBMPlexMono-MediumItalic
```

> 实际上 macOS 自带的字体 Monaco 也是很不错的

#### [](https://www.sqlsec.com/2019/12/macos.html#%E6%A0%87%E9%A2%98%E6%A0%8F "标题栏") 标题栏

`Settings`-`Core`-`Title Bar`，国光这里为了颜值将其设置为了 `hidden`

这样就把 Atom 的标题栏隐藏起来了，整体看上去更加清爽一点。

#### [](https://www.sqlsec.com/2019/12/macos.html#%E6%8F%92%E4%BB%B6 "插件") 插件

Atom 插件安装慢的话 可以手动使用 `apm` 配合 `proxychains4` 命令来安装，安装成功后重启，这样总的下来速度会快很多：

##### [](https://www.sqlsec.com/2019/12/macos.html#activate-power-mode "activate-power-mode")activate-power-mode

只勾选 `Auto Toggle`、`Particles - Enabled`、`Enable`，其他全部取消勾选。

在 `Particles Colours` 下面选择 `Colour at the cursor` 表示烟花效果的颜色是光标所在的颜色

这样在编码的时候就会直接触发烟花效果了，而且不会有烦人的屏幕震动和码字速度统计。

##### [](https://www.sqlsec.com/2019/12/macos.html#%E5%85%B6%E4%BB%96%E6%8F%92%E4%BB%B6 "其他插件") 其他插件

**kite**：Python 语句补全插件 自动语义提示效率很高 （得配合 Kite 这个 APP 来使用）

**atom-beautify**：atom 代码格式美化插件

**atom-clock**：底部状态栏时钟插件

**platformio-ide-terminal**：Atom 必备终端命令行插件 颜值也很高

**minimap**：编辑器右侧代码小地图

**minimap-find-and-replace**：代码小地图查找和替换高亮

**highlight-selected**：鼠标高亮选择

**minimap-highlight-selected**：代码小地图中鼠标选择高亮，依赖 **highlight-selected** 插件

**busy-signal**：和 linter 插件配套使用

**linter-ui-default**：和 linter 插件配套使用，依赖 **intentions**, **busy-signal**

**linter**：代码法检测

**intentions**：和 linter 插件配套使用

**ide-python**：Python 代码检测

**one-vibrancy**：Atom 毛玻璃透明效果

[![](https://dn-coding-net-tweet.codehub.cn/photo/2019/29117fa6-7733-40e0-b3d1-f70caecc5f40.jpg)](https://dn-coding-net-tweet.codehub.cn/photo/2019/29117fa6-7733-40e0-b3d1-f70caecc5f40.jpg)

### [](https://www.sqlsec.com/2019/12/macos.html#Sublime-Text3 "Sublime Text3")Sublime Text3

Sublime Text3 插件的话首先得安装 Package Control，然后才可以愉快地安装插件，安装方法很简单 `菜单栏` -`Tools`-`Install Package Control`，如果安装卡顿的话 开一个全局代理的话速度会快很多，这里不再赘述。安装成功后可以在 `菜单栏` -`Sublime Text`-`Preferences`-`Package Control`- 输入 `Install Package`，等待左下角进度加载完即可安装任意插件。

> 吐槽一下 安装插件这么常用的功能 为什么要做的这么复杂 这让小白该怎么办 =,=

Sublime Text3 国光也就经常编辑文本使用，所以插件安装的不多，下面只列出了一些国光自己用的插件，大家也可以去官网自己搜索想要的插件：[https://packagecontrol.io](https://packagecontrol.io/)

LocalizedMenu：Sublime Text 中文汉化插件

**A File Icon**：文件图标插件，提高颜值

**kite**：Python 语句补全插件 自动语义提示效率很高 （得配合 Kite 这个 APP 来使用）

## [](https://www.sqlsec.com/2019/12/macos.html#%E5%BC%80%E5%8F%91%E7%9B%B8%E5%85%B3 "开发相关") 开发相关

### [](https://www.sqlsec.com/2019/12/macos.html#Homebrew "Homebrew")Homebrew

Homebrew 是一款自由及开放源代码的软件包管理系统，用以简化 macOS 系统上的软件安装过程，程序猿开发必备神器。

#### [](https://www.sqlsec.com/2019/12/macos.html#%E5%AE%89%E8%A3%85 "安装") 安装

BASH

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

# 给 homebrew 文件夹的写入权限
sudo chown -R $(whoami) /usr/local/*
```

#### [](https://www.sqlsec.com/2019/12/macos.html#brew-%E5%8A%A0%E9%80%9F "brew 加速")brew 加速

**替换 Homebrew Bottles 源**

- **zsh** 用户

BASH

```bash
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles' >> ~/.zshrc

source ~/.zshrc

# 再更新一下试试看效果 注意网速 应该可以跑满
brew update
```

- **bash** 用户

BASH

```bash
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles' >> ~/.bash_profile

cd ~
source ~/.bash_profile

# 再更新一下试试看效果 注意网速 应该可以跑满
brew update
```

#### [](https://www.sqlsec.com/2019/12/macos.html#brew-%E9%87%8D%E7%BD%AE%E6%81%A2%E5%A4%8D%E9%BB%98%E8%AE%A4 "brew 重置恢复默认")brew 重置恢复默认

手残搞错了源也不要紧，依次执行下面命令可以将其重置回来

**重置 brew.git**

BASH

```bash
git -C "$(brew --repo)" remote set-url origin https://github.com/Homebrew/brew.git

git -C "$(brew --repo homebrew/core)" remote set-url origin https://github.com/Homebrew/homebrew-core.git

brew update
```

**手动删除 shell 配置记录**

- **zsh** 用户

编辑 `~/.zshrc` 配置文件，删除之前添加的记录：

BASH

```bash
export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles
```

- **bash** 用户

编辑 `~/.bashrc` 配置文件，删除之前添加的记录：

CODE

```none
export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles
```

#### [](https://www.sqlsec.com/2019/12/macos.html#brew-%E5%B8%B8%E7%94%A8%E8%BD%AF%E4%BB%B6 "brew 常用软件")brew 常用软件

BASH

```bash
# brew 相关权限设置
sudo chown -R $(whoami) /usr/local/share/man/man8
chmod u+w /usr/local/share/man/man8

# ip 命令 查看ip地址很方便
brew install iproute2mac

# 查看主机配置信息
brew install screenfetch

# 查看主机配置信息（这个包有点大 不如 screenfetch 好看）
brew install neofetch
```

### [](https://www.sqlsec.com/2019/12/macos.html#proxychains "proxychains")proxychains

终端命令行下代理神器，可以让指定的命令走设置好的代理，内网渗透、科学上网必备工具：

BASH

```bash
# 安装
brew install proxychains-ng

# 配置文件
vim /usr/local/etc/proxychains.conf
```

将结尾的 `socks4 127.0.0.1 9095` 改为

CODE

```none
socks5 127.0.0.1 1086
```

> 国光的 macOS 下 SSR 的默认代理为 1086 然后我就习惯了 1086 端口 这里根据自己的情况来设置

**常用方法**

BASH

```bash
# 代理终端命令
proxychains4 curl www.google.com

# 全局代理Bash shell
proxychains4 -q /bin/bash

# 全局代理zsh shell
proxychains4 -q /bin/zsh
```

实际上新版的 ShadowSocksX-NG-R8 也附带了终端代理功能

[![](https://image.3001.net/images/20200625/15930919394015.png)](https://image.3001.net/images/20200625/15930919394015.png)

实际上具体的命令如下：

BASH

```bash
export http_proxy=http://127.0.0.1:1087;export https_proxy=http://127.0.0.1:1087;
```

这会临时在当前终端的 shell 环境生成一个 http 与 https 代理，想要永久代理的话就将这两条命令写入到 .zshrc 的配置文件即可，再补充一个使用 socks5 代理的命令：

BASH

```bash
export all_proxy="socks5://127.0.0.1:1086"
```

### [](https://www.sqlsec.com/2019/12/macos.html#Vim "Vim")Vim

macOS 自带 vim 命令，这里主要就是开启 macOS vim 下自带的一个配色：

BASH

```bash
# 查看 vim 版本
vim --version

# 这样查看版本号也可以
ls /usr/share/vim/

# 查看自带的配色方案  vim80 当前vim 的版本号（macOS Mojave）
ls /usr/share/vim/vim80/colors
```

自带了如下的配色：

BASH

```bash
default.vim   elflord.vim   koehler.vim   pablo.vim     shine.vim     zellner.vim
blue.vim      delek.vim     evening.vim   morning.vim   peachpuff.vim slate.vim
darkblue.vim  desert.vim    industry.vim  murphy.vim    ron.vim       torte.vim
```

配色可以自己一个个尝试一下，下面新建一个 vim 的配置文件：

BASH

```bash
vim ~/.vimrc
```

内容如下：

BASH

```bash
set nu                " 显示行号
colorscheme desert    " 颜色显示方案
syntax on             " 打开语法高亮
```

配置修改为完，输入 `zsh` 命令生效配置

### [](https://www.sqlsec.com/2019/12/macos.html#Oh-My-Zsh "Oh My Zsh")Oh My Zsh

macOS 下默认的命令行 Shell 环境为 Bash，不过在 10.15 之后为了取代 bash，macOS Catalina 使用 zsh 作为默认 Shell，不够这个自带的 zsh 还不够完美，需要使用 Oh My Zsh 这个 zsh 美化增强脚本，来让自己的终端 Shell 颜值更逼格。

#### [](https://www.sqlsec.com/2019/12/macos.html#%E5%AE%89%E8%A3%85-1 "安装") 安装

BASH

```bash
# 可选命令：全局代理(速度更快)
proxychains4 -q /bin/bash

# 或者这样添加一个临时代理(速度更快)
export ALL_PROXY=socks5://127.0.0.1:1086

# curl安装
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

如果输入 `zsh` 命令报类似下面这样错误的话：

CODE

```none
[oh-my-zsh] Insecure completion-dependent directories detected:
...
```

那么是目录权限问题，使用如下两条命令即可解决问题：

BASH

```bash
chmod 755 /usr/local/share/zsh
chmod 755 /usr/local/share/zsh/site-functions
```

#### [](https://www.sqlsec.com/2019/12/macos.html#%E4%B8%BB%E9%A2%98 "主题") 主题

BASH

```bash
#修改配置文件
vim ~/.zshrc
```

修改此行（等于号 后面填写自己的主题）：

BASH

```bash
ZSH_THEME=robbyrussell
```

Oh My Zsh 已经内置了很多主题了：

BASH

```bash
# 查看自带的主题
ls -l ~/.oh-my-zsh/themes
```

国光以前也比较喜欢花里胡哨的折腾这些，后来发现画船听雨大佬的 zsh 主题就是默认的 =，= 后面我就也没有再折腾这些花里胡哨的主题了。

#### [](https://www.sqlsec.com/2019/12/macos.html#%E6%8F%92%E4%BB%B6-1 "插件") 插件

接下来主要就是一些相关插的配置了。

##### [](https://www.sqlsec.com/2019/12/macos.html#autojump "autojump")autojump

目录切换神器，大大提高工作效率。

BASH

```bash
# macOS 安装
brew install autojump
```

在 `~/.zshrc` 中配置

CODE

```none
plugins=(其他的插件 autojump)
```

输入 `zsh` 命令生效配置后即可正常使用 `j` 命令，下面是简单的演示效果：

BASH

```bash
# 第一次 cd 进入某个目录
➜  ~ cd Documents/Hexo
➜  Hexo cd ~

# 后面就可以直接通过 j 命令跳转到那个目录
➜  ~ j hexo
/Users/sec/Documents/Hexo
```

##### [](https://www.sqlsec.com/2019/12/macos.html#autosuggestions "autosuggestions")autosuggestions

终端下自动提示接下来可能要输入的命令，这个实际使用效率还是比较高的

BASH

```bash
# 拷贝到 plugins 目录下
git clone https://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
```

在 `~/.zshrc` 中配置：

BASH

```bash
plugins=(其他的插件 zsh-autosuggestions)
```

输入 `zsh` 命令生效配置

##### [](https://www.sqlsec.com/2019/12/macos.html#zsh-syntax-highlighting "zsh-syntax-highlighting")zsh-syntax-highlighting

命令输入正确会绿色高亮显示，可以有效地检测命令语法是否正确

BASH

```bash
# 拷贝到 plugins 目录下
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

在 `~/.zshrc` 中配置：

BASH

```bash
plugins=(其他的插件 zsh-syntax-highlighting)
```

输入 `zsh` 命令生效配置

#### [](https://www.sqlsec.com/2019/12/macos.html#%E5%85%B6%E4%BB%96%E9%85%8D%E7%BD%AE "其他配置") 其他配置

**关掉 URL 反斜杠转义**

经常在 zsh 里面粘贴 URL 会出现下面的情况：

CODE

```none
http://xxx/?id=1
```

粘贴到 iTerm2 的 zsh 中会变成这样：

CODE

```none
http://xxx/\?id\=1
```

为了避免每次粘贴后都要修改的麻烦，我们需要手工关掉 URL 特殊符号转义，编辑 .zshrc 配置文件，第一行加入如下内容：

INI

```ini
DISABLE_MAGIC_FUNCTIONS=true
```

编辑完成后，保存输入 `zsh` 命令重新加载配置文件，再次粘贴就不会出现这种情况了。

### [](https://www.sqlsec.com/2019/12/macos.html#Git "Git")Git

#### [](https://www.sqlsec.com/2019/12/macos.html#%E9%85%8D%E7%BD%AE%E9%82%AE%E7%AE%B1%E5%92%8C%E7%94%A8%E6%88%B7%E5%90%8D "配置邮箱和用户名") 配置邮箱和用户名

Github 会根据这个邮箱显示对应的 commit 记录的头像

BASH

```bash
# 配置邮箱 
git config --global user.email "xxxxx@xxx.com"

# 配置用户名
git config --global user.name "国光"
```

下图中间两个 commit 记录没有头像的原因就是没有配置上述邮箱的问题造成的：

[![img](https://dn-coding-net-tweet.codehub.cn/photo/2019/a7d2fd31-e811-4c1a-8a96-5c4058c47a53.jpg)](https://dn-coding-net-tweet.codehub.cn/photo/2019/a7d2fd31-e811-4c1a-8a96-5c4058c47a53.jpg)

#### [](https://www.sqlsec.com/2019/12/macos.html#%E9%85%8D%E7%BD%AEsocks5%E4%BB%A3%E7%90%86 "配置socks5代理") 配置 socks5 代理

在天朝 使用 git clone 的正确姿势，配置这个代理的前提是自己得有一个可以访问国外的 Socks5 代理（比较敏感的话题 这里就不多说了）

BASH

```bash
# 我这里习惯用1086端口 具体根据自己的配置来灵活设置
git config --global http.proxy 'socks5://127.0.0.1:1086'
git config --global https.proxy 'socks5://127.0.0.1:1086'

# 取消git代理
git config --global --unset http.proxy
git config --global --unset https.proxy

# 查看 git 配置
cat ~/.gitconfig

# 或者这样也可以查看git的配置
git config -l
```

### [](https://www.sqlsec.com/2019/12/macos.html#Python "Python")Python

macOS 自带 Python 2.7 的版本，而且没有安装 pip，得自己再折腾完善一下。

#### [](https://www.sqlsec.com/2019/12/macos.html#pip2-amp-pip3 "pip2&pip3")pip2&pip3

macOS 里面 Python 自带 easy_install，用来安装 pip2 和 pip3 非常方便

BASH

```bash
# 安装 pip （貌似 Python2 的 PIP 太新会出问题 可以指定版本）
sudo easy_install pip==20.3.4

# 查看pip的版本
pip --version
pip3 --version

# 更新最新版 pip
pip install --upgrade pip
pip3 install --upgrade pip
```

#### [](https://www.sqlsec.com/2019/12/macos.html#iPython3 "iPython3")iPython3

Python2 要被淘汰掉了，iPython 这种辅助工具包的话 安装一个 iPython3 的话基本上可以满足日常的调试需求了：

BASH

```bash
# 安装ipython3
pip3 install ipython -i https://pypi.tuna.tsinghua.edu.cn/simple --user
```

#### [](https://www.sqlsec.com/2019/12/macos.html#pyenv "pyenv")pyenv

pyenv 是一个强大 Python 包管理工具，可以灵活地切换各种 Python 版本，macOS 下强烈建议大家安装体验一下，具体安装细节可以参考我的这篇文章：[macOS pyenv 入门使用记录](https://www.sqlsec.com/2019/12/pyenv.html)

### [](https://www.sqlsec.com/2019/12/macos.html#Node-js "Node.js")Node.js

国光我 Nodejs 用的不多，主要就用来跑跑 Hexo 博客和使用 Gitbook 来写点知识点，所以我一般都是直接在 Nodejs 官网下载 pkg 安装包一键安装的，但是评论区网页安利 nvm ，不能辜负网友们的好意，那么国光就来研究学习记录一下 macOS 下 nvm 的配置吧。

#### [](https://www.sqlsec.com/2019/12/macos.html#%E7%AE%80%E4%BB%8B "简介") 简介

nvm 是一个使用的 nodejs 版本管理工具，可以用来管理很多 node 版本和 npm 版本。

#### [](https://www.sqlsec.com/2019/12/macos.html#%E5%8D%B8%E8%BD%BD-Node "卸载 Node") 卸载 Node

若电脑已经安装 node，需要卸载掉，检查是否安装 node：

BASH

```bash
node -v
```

如果有版本返回，说明电脑已经安装 node，此时需要把 node 卸载掉，若未安装 node 忽略以下操作，可以使用如下命令卸载：

BASH

```bash
sudo npm uninstall npm -g
sudo rm -rf /usr/local/lib/node /usr/local/lib/node_modules /var/db/receipts/org.nodejs.*
sudo rm -rf /usr/local/include/node /Users/$USER/.npm
sudo rm /usr/local/bin/node
sudo rm /usr/local/share/man/man1/node.1
sudo rm /usr/local/lib/dtrace/node.d
```

验证是否卸载完成：

BASH

```bash
node  -v
npm  -v
```

#### [](https://www.sqlsec.com/2019/12/macos.html#%E5%AE%89%E8%A3%85-nvm "安装 nvm") 安装 nvm

**项目地址**：[https://github.com/nvm-sh/nvm](https://github.com/nvm-sh/nvm)

这里自己去 release 里面去找最新的版本，然后替换到我下面 URL 里面的 `0.35.3` 即可：

BASH

```bash
# 临时代理加速（可选操作）
export ALL_PROXY=socks5://127.0.0.1:1086

# 下载并安装
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash
```

安装完成后，到 `~/.zshrc` 里面插入如下环境变量配置（自己检查一下 如果末尾已经插入了 就不用自己插入了）：

BASH

```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
```

执行完操作后，验证版本信息：

BASH

```bash
$ nvm --version
0.35.3
```

#### [](https://www.sqlsec.com/2019/12/macos.html#%E5%9F%BA%E6%9C%AC%E5%91%BD%E4%BB%A4 "基本命令") 基本命令

BASH

```bash
# 查看当前 node 的版本
nvm version 

# 安装最新稳定版 node
nvm install stable

# 列出所有远程服务器的版本
nvm ls-remote

# 安装指定版本
nvm install v12.18.1
nvm install <version>

# 列出所有已安装的版本
nvm ls

# 卸载指定的版本
nvm uninstall <version>

# 切换使用指定的版本node
nvm use <version>

# 显示当前的版本
nvm current
```

### [](https://www.sqlsec.com/2019/12/macos.html#Gitbook "Gitbook")Gitbook

首先得安装 `npm` 命令，[官网](https://nodejs.org/zh-cn/) 直接下载 pkg 安装包 安装即可，也可以参考上面一个小节使用 nvm 来配置。值得一提的是目前 Gitbook 和 Node.js 14 的版本之间会出现兼容性问题，建议大家尝试安装使用 Node.js 12 LTS 版本

BASH

```bash
# 安装Gitbook
➜  ~ sudo npm install -g gitbook-cli

# 验证是否安装成功
➜  ~ gitbook -V
CLI version: 2.3.2
GitBook version: 3.2.3
```

卸载 Gitbook 可以使用如下命令：

BASH

```bash
# 卸载 Gitbook
npm uninstall -g gitbook-cli

# 清除npm缓存
npm cache clean -f
```

#### [](https://www.sqlsec.com/2019/12/macos.html#%E5%9F%BA%E6%9C%AC%E5%91%BD%E4%BB%A4-1 "基本命令") 基本命令

下面是 Gitbook 常用的一些命令：

BASH

```bash
# Gitbook 目录结构初始化
gitbook init

# 本地预览
gitbook serve

# Gitbook指定端口启动Web服务
gitbook serve --port 8000

# 生成静态html
gitbook build

# 生成pdf文件（得安装配置好 https://calibre-ebook.com/download）
gitbook pdf
```

#### [](https://www.sqlsec.com/2019/12/macos.html#%E6%96%87%E7%AB%A0%E7%BB%93%E6%9E%84 "文章结构") 文章结构

**README.md 与 SUMMARY.md 编写**

- README.md

一本 Gitbook 的简介

CODE

```none
# Gitbook 使用入门


> GitBook 是一个基于 Node.js 的命令行工具，可使用 Github/Git 和 Markdown 来制作精美的电子书。

本书将简单介绍如何安装、编写、生成、发布一本在线图书。
```

- SUMMARY.md

一本书的目录结构

VERILOG

```verilog
# Summary

* [Introduction](README.md)
* [基本安装](howtouse/README.md)
   * [Node.js安装](howtouse/nodejsinstall.md)
   * [Gitbook安装](howtouse/gitbookinstall.md)
   * [Gitbook命令行速览](howtouse/gitbookcli.md)
* [图书项目结构](book/README.md)
   * [README.md 与 SUMMARY编写](book/file.md)
   * [目录初始化](book/prjinit.md)
* [图书输出](output/README.md)
   * [输出为静态网站](output/outfile.md)
   * [输出PDF](output/pdfandebook.md)
* [发布](publish/README.md)
   * [发布到Github Pages](publish/gitpages.md)
* [结束](end/README.md)
```

#### [](https://www.sqlsec.com/2019/12/macos.html#%E6%8F%92%E4%BB%B6%E7%9B%B8%E5%85%B3 "插件相关") 插件相关

**基本插件安装**

下面演示一下 `侧边栏宽度可调节` 插件的安装，后面就不再赘述了。

项目地址：[https://github.com/yoshidax/gitbook-plugin-splitter](https://github.com/yoshidax/gitbook-plugin-splitter)

在 Gitbook 项目的根目录下建立 `book.json` 文件，内容如下：

JSON

```json
{
    "plugins": ["splitter"]
}
```

然后执行

CODE

```none
gitbook install
```

即可完成插件的安装

[![](https://image.3001.net/images/20200115/15790595345170.jpg)](https://image.3001.net/images/20200115/15790595345170.jpg)

**gitbook-plugin-search-plus**

简介：支持中文搜索

项目地址：[https://github.com/lwdgit/gitbook-plugin-search-plus](https://github.com/lwdgit/gitbook-plugin-search-plus)

JSON

```json
{
    plugins: ["-lunr", "-search", "search-plus"]
}
```

**gitbook-plugin-anchor-navigation-ex**

简介：文章目录 TOC 导航以及返回顶部

项目地址：[https://github.com/zq99299/gitbook-plugin-anchor-navigation-ex](https://github.com/zq99299/gitbook-plugin-anchor-navigation-ex)

JSON

```json
{
  "plugins": ["anchor-navigation-ex"]
}
```

因为该插件会自动生成自己的目录层级序号，国光本人喜欢自己来分层，所以这里需要配置一下关闭这个功能：

JSON

```json
"pluginsConfig": {
  "anchor-navigation-ex": {
      "showLevel": false
  }
}
```

**toggle-chapters**

简介：Gitbook 的左侧目录折叠

项目地址：[https://github.com/poojan/gitbook-plugin-toggle-chapters](https://github.com/poojan/gitbook-plugin-toggle-chapters)

JSON

```json
{
    "plugins": ["toggle-chapters"]
}
```

#### [](https://www.sqlsec.com/2019/12/macos.html#book-json "book.json")book.json

经过上面的简单配置后，我的 book.json 具体内容如下：

JSON

```json
{
  "title": "Python学习记录",
    "author": "国光",
    "language": "zh-hans",
    "links": {
        "sidebar": {
          "个人博客": "https://www.sqlsec.com"
        }
      },
    "plugins": ["splitter","-lunr", "-search", "search-plus","anchor-navigation-ex","toggle-chapters"],
    "pluginsConfig": {
        "anchor-navigation-ex": {
            "showLevel": false
        }
    }
}
```

#### [](https://www.sqlsec.com/2019/12/macos.html#%E5%AF%BC%E5%87%BA-PDF "导出 PDF") 导出 PDF

使用 `gitbook pdf` 生成导出 PDF 文件默认是会报这样的错误：

BASH

```bash
InstallRequiredError: "ebook-convert" is not installed.
Install it from Calibre: https://calibre-ebook.com
```

提示缺少 ebook-convert , 此时去 [官网](https://calibre-ebook.com/download) 下载对应系统版本的安装包安装即可，然后配置好 path 环境变量就大功告成了：

BASH

```bash
sudo ln -s /Applications/calibre.app/Contents/MacOS/ebook-convert /usr/local/bin
```

### [](https://www.sqlsec.com/2019/12/macos.html#Hexo "Hexo")Hexo

Hexo 是一个由 Node.js 编写的博客系统，完美支持 markdown 并且部署安装也比较方便，国光我的博客用了 Hexo 已经很多年了，大家如果要写博客的话，也建议大家尝试 Hexo 看看。

安装配置好 Node.js 之后，只需要如下命令即可安装好 Hexo：

BASH

```bash
$ npm install hexo-cli -g
```

下面是国光常用的一个 Hexo 命令：

BASH

```bash
# 清除缓存重新生成并启动 hexo 本地服务
$ hexo clean && hexo g && hexo s
```

因为国光我是把 Hexo 博客直接部署到阿里云服务器的，所以我不用 `hexo d` 这个命令好多年了。

### [](https://www.sqlsec.com/2019/12/macos.html#MySQL "MySQL")MySQL

#### [](https://www.sqlsec.com/2019/12/macos.html#%E5%AE%89%E8%A3%85-MySQL "安装 MySQL") 安装 MySQL

BASH

```bash
# 搜索可以安装的版本
brew search mysql

# 安装对应的版本
brew install mysql@5.7
```

#### [](https://www.sqlsec.com/2019/12/macos.html#%E9%85%8D%E7%BD%AE%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F "配置环境变量") 配置环境变量

国光在 10.14.6 系统下安装的 MySQL 可执行文件的位置是：

BASH

```bash
/usr/local/Cellar/mysql@5.7/5.7.28/bin
```

大家根据实际情况来配置下环境变量：

BASH

```bash
vim ~/.zshrc

export PATH="${PATH}:/usr/local/Cellar/mysql@5.7/5.7.28/bin"

source ~/.zshrc
```

#### [](https://www.sqlsec.com/2019/12/macos.html#%E5%9F%BA%E6%9C%AC%E6%93%8D%E4%BD%9C%E7%9B%B8%E5%85%B3 "基本操作相关") 基本操作相关

BASH

```bash
# 查看 M有SQL 服务状态
mysql.server status

# 启动 MySQL 服务
mysql.server start

# 重启 MySQL 服务
mysql.server restart

# 停止 MySQL 服务
mysql.server stop
```

#### [](https://www.sqlsec.com/2019/12/macos.html#%E8%AE%BE%E7%BD%AE-MySQL-%E5%AF%86%E7%A0%81 "设置 MySQL 密码") 设置 MySQL 密码

默认是 root 用户是空密码 可以直接登录：

BASH

```bash
mysql -uroot
```

[![当前用户的确是 root 用户](https://image.3001.net/images/20200509/15890187043210.png)](https://image.3001.net/images/20200509/15890187043210.png) 设置密码的 SQL 语句如下：

SQL

```sql
# 修改 root 密码的 SQL语句
use mysql;
set password for 'root'@'localhost' = password('你设置的密码');

# 刷新权限 并退出
flush privileges;
quit; 
```

#### [](https://www.sqlsec.com/2019/12/macos.html#%E6%95%B0%E6%8D%AE%E5%BA%93%E5%BC%80%E5%90%AF%E5%A4%96%E8%BF%9E "数据库开启外连") 数据库开启外连

这是个可选操作 根据自己的实际情况自行决定是否开启：

SQL

```sql
grant all on *.* to root@'%' identified by '你设置的密码' with grant option;
flush privileges;
```

大家根据实际情况来决定是否开启外连。

### [](https://www.sqlsec.com/2019/12/macos.html#Docker "Docker")Docker

**Docker 官网下载**：[Docker Desktop for Mac and Windows | Docker](https://www.docker.com/products/docker-desktop)

[romeoz/docker-apache-php](https://hub.docker.com/r/romeoz/docker-apache-php/tags) ：PHP 版本很全，基本上都覆盖了，可以很方便地部署运行一个 Docker 下的靶场环境，更多的镜像可以参考我的另一篇文章：[Docker 常用镜像整理](https://www.sqlsec.com/2020/11/docker4.html)

### [](https://www.sqlsec.com/2019/12/macos.html#Redis "Redis")Redis

BASH

```bash
# 安装 redis
brew install redis

# 启动 redis 服务端
redis-server

# 启动 redis 客户端
redis-cli

# 编辑默认配置文件
sudo vim /usr/local/etc/redis.conf
```

## [](https://www.sqlsec.com/2019/12/macos.html#%E5%AE%89%E5%85%A8%E7%9B%B8%E5%85%B3 "安全相关") 安全相关

macOS 下也有很多原生的渗透测试工具，安装配置起来比 Windows 都要方便很多，安全相关的工具只做思路分享，更多安全工具等待着你自己去尝试。

### [](https://www.sqlsec.com/2019/12/macos.html#aircrack-ng "aircrack-ng")aircrack-ng

WiFi 抓包以及跑包工具，macOS 上抓包一般不用这个，不够用来跑 WiFi 握手包还是蛮方便的。

BASH

```bash
# macOS安装binwalk
➜  ~ brew install aircrack-ng

# 查看版本信息
➜  ~ aircrack-ng

  Aircrack-ng 1.6  - (C) 2006-2020 Thomas d'Otreppe
  https://www.aircrack-ng.org

  usage: aircrack-ng [options] <input file(s)>
  ...
```

### [](https://www.sqlsec.com/2019/12/macos.html#binwalk "binwalk")binwalk

文件分析工具，旨在协助研究人员对文件进行分析，提取及逆向工程，CTF 比赛中常用来解隐写相关的题目。

BASH

```bash
# macOS安装binwalk
➜  ~ brew install binwalk

# 查看版本信息
➜  ~ binwalk

Binwalk v2.2.0
Craig Heffner, ReFirmLabs
https://github.com/ReFirmLabs/binwalk

Usage: binwalk [OPTIONS] [FILE1] [FILE2] [FILE3] ...
```

### [](https://www.sqlsec.com/2019/12/macos.html#burpsuite "burpsuite")burpsuite

Web 安全抓包改包必备神器，具体配置可以参考我的这篇文章：[macOS 下如何优雅的使用 Burp Suite](https://www.sqlsec.com/2019/11/macbp.html)

[![](https://dn-coding-net-tweet.codehub.cn/photo/2019/9fba59d8-c066-4f0e-af50-43619d7e995d.jpg)](https://dn-coding-net-tweet.codehub.cn/photo/2019/9fba59d8-c066-4f0e-af50-43619d7e995d.jpg)

### [](https://www.sqlsec.com/2019/12/macos.html#Cobalt-Strike "Cobalt Strike")Cobalt Strike

红队团队协作内网渗透神器，内网穿透代理非常简单方便，具体配置了可以参考我的这篇文章：[macOS 下如何优雅的安装 Cobalt Strike](https://www.sqlsec.com/2020/07/cobaltstrike.html)

### [](https://www.sqlsec.com/2019/12/macos.html#dirsearch "dirsearch")dirsearch

Web 目录扫描神器，自带的字典很强大，逐渐替代了御剑等一些工具。

BASH

```bash
# 进入安全工具的目录下（国光习惯将安全工具放在 Documents 目录下）
➜ cd /Users/sec/Documents/Sec

# git clone 下载最新版本的 dirsearch
➜ git clone https://github.com/maurosoria/dirsearch.git
Cloning into 'dirsearch'...
remote: Enumerating objects: 1744, done.
remote: Total 1744 (delta 0), reused 0 (delta 0), pack-reused 1744
Receiving objects: 100% (1744/1744), 17.73 MiB | 1.26 MiB/s, done.
Resolving deltas: 100% (1015/1015), done.

# 创建 dirsearch 的快捷方式
➜ ln -s /Users/sec/Documents/Sec/dirsearch/dirsearch.py /usr/local/bin/dirsearch

# 刷新 zsh
➜ zsh

# 发起一个基本扫描
➜ dirsearch -u https://www.baidu.com -e *

 _|. _ _  _  _  _ _|_    v0.3.9
(_||| _) (/_(_|| (_| )

Extensions: Applications | HTTP method: get | Threads: 10 | Wordlist size: 6104

Error Log: /Users/sec/Documents/Sec/dirsearch/logs/errors-19-12-26_22-09-24.log

Target: https://www.baidu.com

[22:09:24] Starting:
[22:09:24] 302 -  231B  - /.bash_history.php  ->  http://www.baidu.com/forbiddenip/forbidden.html
[22:09:24] 302 -  231B  - /.env.php  ->  http://www.baidu.com/forbiddenip/forbidden.html
[22:09:24] 302 -  231B  - /.env.sample.php  ->  
...
```

### [](https://www.sqlsec.com/2019/12/macos.html#hashcat "hashcat")hashcat

密码破解神器，支持 GPU 破解：

BASH

```bash
# macOS安装hashcat
➜  ~ brew install hashcat

# 查看版本信息
➜  ~ hashcat --version
v5.1.0
```

### [](https://www.sqlsec.com/2019/12/macos.html#httrack "httrack")httrack

看到好看的前端界面不要羡慕，httrack 在手，统统都可以原封不动的扒下来 =，=

BASH

```bash
# macOS安装httrack
➜  ~ brew install httrack

# 查看版本信息
➜  ~ httrack --version
HTTrack version 3.49-2
```

### [](https://www.sqlsec.com/2019/12/macos.html#masscan "masscan")masscan

世界上最快的端口扫描器，一般和 nmap 配合使用，可以极大地提高效率：

BASH

```bash
# macOS安装masscan
➜  ~ brew install masscan

# 查看版本信息
➜  ~ masscan --version

Masscan version 1.0.4 ( https://github.com/robertdavidgraham/masscan )
Compiled on: Aug 21 2018 02:07:01
Compiler: gcc 4.2.1 Compatible Apple LLVM 10.0.0 (clang-1000.10.43.1)
OS: Apple
CPU: unknown (64 bits)
GIT version: unknown
```

### [](https://www.sqlsec.com/2019/12/macos.html#metasploit "metasploit")metasploit

黑客 TOP10 工具之一，漏洞攻击框架，必备神器之一，macOS 下配置可以参考我的这篇文章 [macOS 下优雅的使用 Metasploit](https://www.sqlsec.com/2019/11/macmsf.html)

BASH

```bash
# 查看版本信息
➜  ~ msfconsole -V
Framework Version: 5.0.66-dev-75dc82f764050ff800dc2bf0482c31aae693edd4

➜  ~ msfconsole
 ______________________________________________________________________________
|                                                                              |
|                          3Kom SuperHack II Logon                             |
|______________________________________________________________________________|
|                                                                              |
|                                                                              |
|                                                                              |
|                 User Name:          [   security    ]                        |
|                                                                              |
|                 Password:           [               ]                        |
|                                                                              |
|                                                                              |
|                                                                              |
|                                   [ OK ]                                     |
|______________________________________________________________________________|
|                                                                              |
|                                                       https://metasploit.com |
|______________________________________________________________________________|


       =[ metasploit v5.0.66-dev-75dc82f764050ff800dc2bf0482c31aae693edd4]
+ -- --=[ 1956 exploits - 1091 auxiliary - 336 post       ]
+ -- --=[ 558 payloads - 45 encoders - 10 nops            ]
+ -- --=[ 7 evasion                                       ]

msf5 >
```

### [](https://www.sqlsec.com/2019/12/macos.html#nmap "nmap")nmap

端口扫描必备神器：

BASH

```bash
# macOS安装nmap
➜  ~ brew install nmap

# 查看版本信息
➜  ~ nmap -v
Starting Nmap 7.80 ( https://nmap.org ) at 2019-12-26 21:07 CST
Read data files from: /usr/local/bin/../share/nmap
WARNING: No targets were specified, so 0 hosts scanned.
Nmap done: 0 IP addresses (0 hosts up) scanned in 0.03 seconds
```

### [](https://www.sqlsec.com/2019/12/macos.html#sqlmap "sqlmap")sqlmap

Web 安全必备工具，SQL 注入神器：

BASH

```bash
# 下载可能会卡 请自行备好代理
➜  ~ brew install sqlmap
```

[![](https://image.3001.net/images/20191226/15773682044024.jpg)](https://image.3001.net/images/20191226/15773682044024.jpg)

如果安装卡这里的话，通过 `proxychains4` 也没有解决的话 ，国光我这里是这样临时解决的，终端下临时添加一个 socks5 代理：

BASH

```bash
# socks5 proxy
export ALL_PROXY=socks5://127.0.0.1:1086
```

配置修改为完，输入 `zsh` 命令生效配置

### [](https://www.sqlsec.com/2019/12/macos.html#wpscan "wpscan")wpscan

WordPress 漏洞扫描器，收录了历届 WordPress 漏洞，可以辅助渗透测试一些 WordPress 相关的站点

BASH

```bash
# 部署可能会卡 请自行备好代理
➜  ~ brew install wpscan

# 查看 wpscan 版本信息
➜  ~ wpscan --version
_______________________________________________________________
        __          _______   _____
        \ \        / /  __ \ / ____|
         \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
          \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
           \  /\  /  | |     ____) | (__| (_| | | | |
            \/  \/   |_|    |_____/ \___|\__,_|_| |_|

        WordPress Security Scanner by the WPScan Team
                       Version 2.9.4
          Sponsored by Sucuri - https://sucuri.net
      @_WPScan_, @ethicalhack3r, @erwan_lr, @_FireFart_
_______________________________________________________________

Current version: 2.9.4
Last database update: 2019-12-26
```

历史上国光就有记录 macOS 配置的习惯，之前也写过两篇文章，但是都有点过时了，macOS Ventura 13 变化挺大，正巧最近又入手了 MacBook Pro 14 寸 M1 Pro 的 ARM 芯片的 Mac，所以就单独写了本篇文章记录一下，也顺便给其他网友做个配置参考，让大家少走一点配置环境的弯路。

## [](https://www.sqlsec.com/2023/07/ventura.html#%E7%B3%BB%E7%BB%9F%E8%AE%BE%E7%BD%AE "系统设置") 系统设置

### [](https://www.sqlsec.com/2023/07/ventura.html#%E5%AE%9E%E7%94%A8%E5%91%BD%E4%BB%A4 "实用命令") 实用命令

1. **取消 4 位数密码限制**

BASH

```bash
pwpolicy -clearaccountpolicies
```

1. **允许安装任意来源的 App**

BASH

```bash
sudo spctl --master-disable
```

1. **xcode 命令行工具**

BASH

```bash
xcode-select --install
```

1. **程序坞自动隐藏加速**

BASH

```bash
# 设置启动坞动画时间设置为 0.5 秒 
defaults write com.apple.dock autohide-time-modifier -float 0.5 && killall Dock

# 设置启动坞响应时间最短
defaults write com.apple.dock autohide-delay -int 0 && killall Dock

# 恢复启动坞默认动画时间
defaults delete com.apple.dock autohide-time-modifier && killall Dock

# 恢复默认启动坞响应时间
defaults delete com.apple.Dock autohide-delay && killall Dock
```

1. **启动台自定义行和列**

BASH

```bash
# 设置列数
defaults write com.apple.dock springboard-columns -int 7

# 设置行数
defaults write com.apple.dock springboard-rows -int 6

# 重启 Dock 生效
killall Dock

# 恢复默认的列数和行数
defaults write com.apple.dock springboard-rows Default
defaults write com.apple.dock springboard-columns Default

# 重启 Dock 生效
killall Dock
```

### [](https://www.sqlsec.com/2023/07/ventura.html#%E4%B8%AA%E4%BA%BA%E4%B9%A0%E6%83%AF "个人习惯") 个人习惯

#### [](https://www.sqlsec.com/2023/07/ventura.html#%E8%BE%93%E5%85%A5%E6%B3%95 "输入法") 输入法

可以考虑再安装个第三方的输入法，目前搜狗输入法在 Mac 上还算是比较老实，可以考虑试试看：

[![](https://image.3001.net/images/20230711/16890781657024.png)](https://image.3001.net/images/20230711/16890781657024.png)

#### [](https://www.sqlsec.com/2023/07/ventura.html#%E5%85%89%E6%A0%87%E5%93%8D%E5%BA%94 "光标响应") 光标响应

重复率最快，，延迟设置最低，这样可以提高工作效率：

[![](https://image.3001.net/images/20230711/16890781717456.png)](https://image.3001.net/images/20230711/16890781717456.png)

#### [](https://www.sqlsec.com/2023/07/ventura.html#%E4%B8%89%E6%8C%87%E6%8B%96%E7%A7%BB "三指拖移") 三指拖移

触控板干活效率必备，资深 macOS 用户必开的一个隐藏选项：

[![](https://image.3001.net/images/20230711/16890781785848.png)](https://image.3001.net/images/20230711/16890781785848.png)

## [](https://www.sqlsec.com/2023/07/ventura.html#%E8%BD%AF%E4%BB%B6%E8%AE%BE%E7%BD%AE "软件设置") 软件设置

### [](https://www.sqlsec.com/2023/07/ventura.html#iTerm2 "iTerm2")iTerm2

主题选择自带的 「Minima」和「Compact」都还是不错的」：

[![](https://image.3001.net/images/20230711/1689078184221.png)](https://image.3001.net/images/20230711/1689078184221.png)

新建一个自己的配置，并设置为默认「Set as Default」，在新的配置里面可以进行字体相关的设置，字体大小国光设置的是 16 号，使用的是 macOS 自带的 Monaco 高颜值字体：

[![](https://image.3001.net/images/20230711/1689078190459.png)](https://image.3001.net/images/20230711/1689078190459.png)

国光的窗口设置一些喜好：

[![](https://image.3001.net/images/20230711/16890782483161.png)](https://image.3001.net/images/20230711/16890782483161.png)

效果图：

[![](https://image.3001.net/images/20230711/16890783044959.png)](https://image.3001.net/images/20230711/16890783044959.png)

## [](https://www.sqlsec.com/2023/07/ventura.html#%E5%BC%80%E5%8F%91%E9%85%8D%E7%BD%AE "开发配置") 开发配置

### [](https://www.sqlsec.com/2023/07/ventura.html#%F0%9F%9A%80 "🚀")🚀

配环境前确保网络是 OK 的，ARM 架构的 MacBook 可以直接使用 iOS 和 iPad OS 上经典的 🚀：

[![](https://image.3001.net/images/20230711/16890783193280.jpg)](https://image.3001.net/images/20230711/16890783193280.jpg)

### [](https://www.sqlsec.com/2023/07/ventura.html#%E9%98%B2%E7%81%AB%E5%A2%99 "防火墙") 防火墙

同时一些软件是破解版，屏蔽 Hosts 大法也太不优雅了，还是搞个正经的防火墙软件比较靠谱，国光这里主推：[Little Snitch](https://www.obdev.at/products/littlesnitch/download.html)

这个破解版不确定多不多，但是正版的序列号貌似可以多人复用，所以价格理论上也不贵的，师傅们可以自己去了解一下。

[![](https://image.3001.net/images/20230711/16890783292455.png)](https://image.3001.net/images/20230711/16890783292455.png)

### [](https://www.sqlsec.com/2023/07/ventura.html#Git "Git")Git

如果要使用 Github 协作项目的话，那么 GitHub 会根据我们本地的 Git 配置去箱显示对应的 commit 记录的头像：

BASH

```bash
# 配置邮箱 
➜ git config --global user.email "xxxxx@xxx.com"

# 配置用户名
➜ git config --global user.name "国光"
```

下图中间两个 commit 记录没有头像的原因就是没有配置上述邮箱的问题造成的：

[![](https://image.3001.net/images/20230711/16890783436308.jpg)](https://image.3001.net/images/20230711/16890783436308.jpg)

### [](https://www.sqlsec.com/2023/07/ventura.html#Vim "Vim")Vim

macOS 自带的 vim 是没有任何配色的，可以下面是国光常用的配色方案，先在用户目录下新建一个 vim 的配置文件：

BASH

```bash
vim ~/.vimrc
```

内容如下：

BASH

```bash
set nu                " 显示行号
colorscheme desert    " 颜色显示方案，其他方案查看：ls /usr/share/vim/vim*/colors 
syntax on             " 打开语法高亮
```

### [](https://www.sqlsec.com/2023/07/ventura.html#Python "Python")Python

简单使用 iPython 还是建议装一下，平时调试会比较方便：

BASH

```bash
➜ brew install ipython
```

不过个人还是建议使用 pyenv 来管理 Python。

pyenv 是一个强大 Python 包管理工具，可以灵活地切换各种 Python 版本，Linux 和 macOS 强烈建议使用 pyenv 来管理我们的 Python 版本，优雅高效且不会破坏掉系统自带的 Python 环境：

BASH

```bash
# 安装 pyenv
➜ brew install pyenv
```

接着为 pyenv 配置 shell 环境，提高工作效率，可自动联想 Tab 补全我们本地安装的 Python 版本：

BASH

```bash
echo 'eval "$(pyenv init -)"' >> ~/.zshrc
```

安装 Python 也很简单：

BASH

```bash
# 查看已经安装的Python版本
➜ pyenv versions

# 查看当前的 Python 版本
➜ pyenv version

# 查看可安装的版本
➜ pyenv install -l

# 安装与卸载 pypy3.8-7.3.11
➜ pyenv install pypy3.8-7.3.11
➜ pyenv uninstall pypy3.8-7.3.11
```

版本切换确实很方便，所安装的版本都在 `~/.pyenv/versions` 目录下：

BASH

```bash
# global 全局设置 一般不建议改变全局设置
➜ pyenv global <python版本>

# shell 会话设置 只影响当前的shell会话
➜ pyenv shell <python版本>
# 取消 shell 会话的设置
➜ pyenv shell --unset

# local 本地设置 只影响所在文件夹
➜ pyenv local <python版本>
```

pyenv 的 global、local、shell 的优先级关系是：shell > local > global

### [](https://www.sqlsec.com/2023/07/ventura.html#Java "Java")Java

无论是 [Oracle JDK](https://www.oracle.com/hk/java/technologies/downloads/) 还是近期比较流行的 [Azul Zulu JDK](https://www.azul.com/downloads/)，我们都可以先自己安装一遍，默认都在安装在

**/Library/Java/JavaVirtualMachines** 目录下：

[![](https://image.3001.net/images/20230725/16902964292395.png)](https://image.3001.net/images/20230725/16902964292395.png)

所以我们切换版本时候就不是很优雅，这里推荐使用 jevn 来切换我们的 Java 版本，类似于 pyenv 一样，很优雅：

BASH

```bash
# 安装 jenv
brew instal jenv
```

brew 安装后我们得配置一下 zshrc 的环境：

BASH

```bash
echo 'export PATH="$HOME/.jenv/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(jenv init -)"' >> ~/.zshrc
```

然后新开个标签页即可正常使用 jenv 来管理我们的 Java 环境了，下面是 jenv 的基本使用：

BASH

```bash
# 查看当前的 Java 版本
jenv version

# 手动添加本地的 Java Home 路径
jenv add /Library/Java/JavaVirtualMachines/jdk-20.jdk/Contents/Home/
jenv add /Library/Java/JavaVirtualMachines/jdk1.8.0_202.jdk/Contents/Home/
jenv add /Library/Java/JavaVirtualMachines/jdk1.8.0_291.jdk/Contents/Home/
jenv add /Library/Java/JavaVirtualMachines/zulu-17.jdk/Contents/Home/

# 列出目前 jenv 所有可切换管理的版本
jenv versions

#global 全局设置 一般不建议改变全局设置
➜ jenv global <java 版本>

# shell 会话设置 只影响当前的shell会话
➜ jenv shell <java 版本>
# 取消 shell 会话的设置
➜ jenv shell --unset

# local 本地设置 只影响所在文件夹
➜ jenv local <java 版本>
```

下面是简单的 jenv 使用演示效果图，真的很优雅：

[![](https://image.3001.net/images/20230725/16902977603509.png)](https://image.3001.net/images/20230725/16902977603509.png)

### [](https://www.sqlsec.com/2023/07/ventura.html#Node-js "Node.js")Node.js

国光我 Nodejs 用的不多，主要就用来跑跑 Hexo 博客和使用 Gitbook 来写点知识点，又因为 Node.js 版本也比较凌乱，所以这里主要是通过 nvm 来进行配置管理。

BASH

```bash
# 安装 nvm
➜ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash

# 新开个终端，输入下面命令，完善 zsh 补全配置
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"

# 查看版本信息
➜ zsh
➜ nvm --version
0.39.3

# 查看当前 node 的版本
➜ nvm version 

# 安装最新稳定版 node
➜ nvm install stable

# 列出所有远程服务器的版本
➜ nvm ls-remote

# 安装指定版本
➜ nvm install v18.16.1
➜ nvm install <version>

# 列出所有已安装的版本
➜ nvm ls

# 卸载指定的版本
➜ nvm uninstall <version>

# 切换使用指定的版本node
➜ nvm use <version>

# 显示当前的版本
➜ nvm current
```

### [](https://www.sqlsec.com/2023/07/ventura.html#Homebrew "Homebrew")Homebrew

Homebrew 是 macOS（或 Linux）缺失的软件包的管理器，开发必备神器，安装也比较简单（网络 OK 的情况）：

BASH

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

安装完成后，如果没有 brew 命令的话，就继续配置 zsh 环境变量（上面安装完会提示 COPY 运行即可）：

BASH

```bash
(echo; echo 'eval "$(/opt/homebrew/bin/brew shellenv)"') >> /Users/security/.zprofile
```

brew 的基本使用：

BASH

```bash
# 更新 Homebrew
➜ brew update

#  搜索相关的包
➜ brew search [关键词] 

# 查看包的信息
➜ rew info [软件名]
 
# 查看已安装的包
➜ brew list

# 更新某个软件
➜ brew upgrade [软件名]

# 清理所有软件的旧版
➜ brew cleanup

# 卸载某个软件
➜ brew uninstall [软件名]
```

brew 常用软件推荐：

BASH

```bash
# 安装 brew-cask
brew install cask

# 空格预览 markdown
brew install qlmarkdown

# 空格高亮预览代码文件
brew install syntax-highlight
```

### [](https://www.sqlsec.com/2023/07/ventura.html#Oh-My-Zsh "Oh My Zsh")Oh My Zsh

Oh My Zsh 这个 zsh 美化增强脚本，来让自己的终端 Shell 颜值更逼格：

BASH

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

[![](https://image.3001.net/images/20230711/16890783761892.png)](https://image.3001.net/images/20230711/16890783761892.png)

Zsh 插件推荐：

BASH

```bash
# 目录切换神器
➜ brew install autojump

# 自动建议提示接下来可能要输入的命令
git clone https://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions

# 命令语法检测
git clone https://github.com/zsh-users/zsh-syntax-highlighting $ZSH_CUSTOM/plugins/zsh-syntax-highlighting
```

在 `~/.zshrc` 中配置启用这些插件：

CODE

```none
plugins=(其他的插件...... autojump zsh-autosuggestions zsh-syntax-highlighting)
```

其他功能配置：

BASH

```bash
# 关掉 URL 反斜杠转义
echo "DISABLE_MAGIC_FUNCTIONS=true" >> ~/.zshrc

# 禁用 on my zsh 自动更新
echo " zstyle ':omz:update' mode disabled" >> ~/.zshrc
```

### [](https://www.sqlsec.com/2023/07/ventura.html#MySQL "MySQL")MySQL

安装 MySQL：

BASH

```bash
# 搜索可以安装的版本
➜ brew search mysql

# 安装对应的版本
➜ brew install mysql@5.7

# 写入环境变量
echo 'export PATH="/opt/homebrew/opt/mysql@5.7/bin:$PATH"' >> ~/.zshrc

# 为了让编译器找到 mysql@5.7 还需要写入
echo 'export LDFLAGS="-L/opt/homebrew/opt/mysql@5.7/lib"' >> ~/.zshrc
echo 'export CPPFLAGS="-I/opt/homebrew/opt/mysql@5.7/include"' >> ~/.zshrc

# 为了让 pkg-config 找到 mysql@5.7 还需要写入
echo 'PKG_CONFIG_PATH="/opt/homebrew/opt/mysql@5.7/lib/pkgconfig"' >> ~/.zshrc
```

MySQL 服务相关：

BASH

```bash
# 查看 M有SQL 服务状态
➜ brew services info mysql@5.7
➜ mysql.server status

# 启动 MySQL 服务
➜ brew services start mysql@5.7
➜ mysql.server start

# 重启 MySQL 服务
➜ brew services restart mysql@5.7
➜ mysql.server restart

# 停止 MySQL 服务
➜ brew services stop mysql@5.7
➜ mysql.server stop
```

接着初始化 MySQL 设置，主要配置一下 root 密码已经是否远程登录登，根据提示来操作就行了：

BASH

```bash
mysql_secure_installation
```

[![](https://image.3001.net/images/20230711/16890783845769.png)](https://image.3001.net/images/20230711/16890783845769.png)

数据库外连，这是个可选操作 根据自己的实际情况自行决定是否开启（有被攻击的风险）：

SQL

```sql
mysql > grant all on *.* to root@'%' identified by '你设置的密码' with grant option;
mysql > flush privileges;
```

大家根据实际情况来决定是否开启外连。

### [](https://www.sqlsec.com/2023/07/ventura.html#Redis "Redis")Redis

BASH

```bash
# 安装 redis
➜ brew install redis

# 查看 redis 服务状态
➜ brew services info redis

# 启动 redis 服务端
➜ brew services start redis

# 启动 redis 客户端
➜ redis-cli

# 编辑默认配置文件
➜ sudo vim /opt/homebrew/etc/redis.conf
```

### [](https://www.sqlsec.com/2023/07/ventura.html#proxychains-ng "proxychains-ng")proxychains-ng

终端命令行下代理神器，可以让指定的命令走设置好的代理，内网渗透、XX 上网必备工具：

BASH

```bash
# 安装
➜ brew install proxychains-ng

# 配置文件
➜ vim /opt/homebrew/etc/proxychains.conf
```

将结尾的 `socks4 127.0.0.1 9095` 改为我们自己的代理端口即可。

下面是常用的几个命令：

BASH

```bash
# 代理终端基本示例
➜ proxychains4 curl https://www.google.com.hk

# 全局代理 bash shell
➜ proxychains4 -q /bin/bash

# 全局代理 zsh shell
➜ proxychains4 -q /bin/zsh
```

[![](https://image.3001.net/images/20230711/16890783936816.png)](https://image.3001.net/images/20230711/16890783936816.png)

如果发现你的 proxychains 并不能代理的话，尝试关掉 SIP 后再试试看。

### [](https://www.sqlsec.com/2023/07/ventura.html#Gitbook "Gitbook")Gitbook

Gitbook 确实是一个写文档的神器，国光之前是必装的，目前逐步使用 mkdocs 来替代 Gitbook，不过还是要记录一下 Gitbook 的使用，安装还是有坑的：就是使用高版本的 Node.js 安装会出各种玄学问题，换 10.X 的版本即可：

BASH

```bash
# 安装并切换老版本的 Node.js
➜ nvm install v10.24.1
➜ nvm use v10.24.1

# 安装苹果官方的 rosetta 兼容性工具
/usr/sbin/softwareupdate --install-rosetta --agree-to-license

# 安装 Gitbook-cli
➜ npm install -g gitbook-cli

# 验证版本并安装相关依赖
➜ gitbook -V
CLI version: 2.3.2
GitBook version: 3.2.3
```

### [](https://www.sqlsec.com/2023/07/ventura.html#MkDocs "MkDocs")MkDocs

MkDoc 是有一个优雅的写文档神器，使用 Python 安装很方便：

BASH

```bash
# pyenv 切换合适的版本
➜ pyenv local 3.9.17

# 安装 mkdocs
➜ pip install mkdocs

# 安装 material 主题
➜ pip install mkdocs-material
```

### [](https://www.sqlsec.com/2023/07/ventura.html#Hexo "Hexo")Hexo

快速简约强大的博客框架：

BASH

```bash
# 安装 hexo
➜ brew install hexo

# Hexo 基础命令
➜ hexo clean # 清除缓存
➜ hexo g     # 生成静态文件
➜ hexo s     # 启动 Hexo 服务
```

### [](https://www.sqlsec.com/2023/07/ventura.html#%E8%99%9A%E6%8B%9F%E5%8C%96 "虚拟化") 虚拟化

#### [](https://www.sqlsec.com/2023/07/ventura.html#Docker-Desktop "Docker Desktop")Docker Desktop

BASH

```bash
# 使用 brew cask 可以很快的安装好 Docker 官方客户端
➜ brew install --cask docker

# 跑个 Kali Linux 看看
➜ docker pull docker.io/kalilinux/kali-rolling
➜ docker run --name "KaliLinux" --tty --interactive kalilinux/kali-rolling
```

[![](https://image.3001.net/images/20230711/16890784023850.png)](https://image.3001.net/images/20230711/16890784023850.png)

#### [](https://www.sqlsec.com/2023/07/ventura.html#OrbStack "OrbStack")OrbStack

[OrbStack](https://orbstack.dev/) 是一种在 macOS 上运行 Docker 容器和 Linux 机器的快速、轻便且简单的方法。可以将其视为强大的 WSL 和 Docker Desktop 替代方案，全部集成在一个易于使用的应用程序中。

BASH

```bash
# Homebrew Cask 安装更优雅一点
➜ brew install orbstack

# docker 切换 OrbStack
➜ docker context use orbstack
```

- Docker

[![](https://image.3001.net/images/20230711/16890784106716.png)](https://image.3001.net/images/20230711/16890784106716.png)

- Linux 子系统

[![](https://image.3001.net/images/20230711/16890784158314.png)](https://image.3001.net/images/20230711/16890784158314.png)

Docker 的一些镜像国内拉取很慢，我们可以配置一下一些国内的加速源：

BASH

```bash
{
    "ipv6": true,
  	"registry-mirrors": [
    	"http://hub-mirror.c.163.com",
    	"https://registry.docker-cn.com",
    	"https://mirror.baidubce.com",
    	"https://kn77wnbv.mirror.aliyuncs.com",
    	"https://y0qd3iq.mirror.aliyuncs.com",
    	"https://6kx4zyno.mirror.aliyuncs.com",
    	"https://0dj0t5fb.mirror.aliyuncs.com",
    	"https://docker.nju.edu.cn",
    	"https://kuamavit.mirror.aliyuncs.com",
    	"https://y0qd3iq.mirror.aliyuncs.com",
    	"https://docker.mirrors.ustc.edu.cn"
  ]
}
```

速度果然确快了很多：

[![](https://image.3001.net/images/20230711/16890789523093.png)](https://image.3001.net/images/20230711/16890789523093.png)

#### [](https://www.sqlsec.com/2023/07/ventura.html#Parallels-Desktop "Parallels Desktop")Parallels Desktop

Parallels Desktop for mac Crack 18.1.1.53328 破解项目：[https://git.icrack.day/somebasj/ParallelsDesktopCrack.git](https://git.icrack.day/somebasj/ParallelsDesktopCrack.git)

根据教程下载指定版本的 PD：[https://download.parallels.com/desktop/v18/18.1.1-53328/ParallelsDesktop-18.1.1-53328.dmg](https://download.parallels.com/desktop/v18/18.1.1-53328/ParallelsDesktop-18.1.1-53328.dmg)

正常安装好，先不要登录 PD 账号。接着克隆项目并运行自动化破解脚本：

BASH

```bash
# 克隆项目地址
➜ git clone https://git.icrack.day/somebasj/ParallelsDesktopCrack.git

# 进项目文件夹并运行脚本
cd ParallelsDesktopCrack && chmod +x ./install.sh && sudo ./install.sh
```

破解过程还是很顺利的，Parallels Desktop 18 确实 YYDS：

[![](https://image.3001.net/images/20230711/16890784223169.png)](https://image.3001.net/images/20230711/16890784223169.png)

我们需要屏蔽以下的 Hosts：

INI

```ini
127.0.0.1 download.parallels.com
127.0.0.1 update.parallels.com
127.0.0.1 desktop.parallels.com
127.0.0.1 download.parallels.com.cdn.cloudflare.net
127.0.0.1 update.parallels.com.cdn.cloudflare.net
127.0.0.1 desktop.parallels.com.cdn.cloudflare.net
127.0.0.1 www.parallels.cn
127.0.0.1 www.parallels.com
127.0.0.1 www.parallels.de
127.0.0.1 www.parallels.es
127.0.0.1 www.parallels.fr
127.0.0.1 www.parallels.nl
127.0.0.1 www.parallels.pt
127.0.0.1 www.parallels.ru
127.0.0.1 www.parallelskorea.com
127.0.0.1 reportus.parallels.com
127.0.0.1 parallels.cn
127.0.0.1 parallels.com
127.0.0.1 parallels.de
127.0.0.1 parallels.es
127.0.0.1 parallels.fr
127.0.0.1 parallels.nl
127.0.0.1 parallels.pt
127.0.0.1 parallels.ru
127.0.0.1 parallelskorea.com
127.0.0.1 pax-manager.myparallels.com
127.0.0.1 myparallels.com
127.0.0.1 my.parallels.com
```

但是 Parallels Desktop 会很猥琐的去偷偷取消注释我们的 hosts 文件，所以我们一上来就安利的 Little Snitch 就很优雅，很有必要了。

最后发现 M1 下的 PD 虚拟机 120Hz 的 Windows 体验确实很丝滑：

[![](https://image.3001.net/images/20230711/16890820228497.jpg)](https://image.3001.net/images/20230711/16890820228497.jpg)

## [](https://www.sqlsec.com/2023/07/ventura.html#%E5%AE%9E%E7%94%A8%E5%B7%A5%E5%85%B7 "实用工具") 实用工具

- **命令增强**

BASH

```bash
# tree 命令列文件也很方便
➜ brew install tree

# 查看系统状态信息
➜ brew install neofetch
➜ brew install fastfetch
```

- **网络相关**

BASH

```bash
# ip 命令 查看ip地址很方便
➜ brew install iproute2mac

# frps 与 frpc 穿透神器
➜ brew install frps # 配置文件 frps -c /opt/homebrew/etc/frp/frps.ini
➜ brew install frpc # 配置文件 frpc -c /opt/homebrew/etc/frp/frpc.ini
```

- **媒体处理**

BASH

```bash
# 图片压缩使用开源的 imageoptim 很方便
➜ brew install imagemagick

# you-get 数码下载神器
➜ brew install ffmpeg
➜ brew install you-get

# 复制克隆离线镜像网站
➜ brew install httrack
```

## [](https://www.sqlsec.com/2023/07/ventura.html#%E5%AE%89%E5%85%A8%E5%B7%A5%E5%85%B7 "安全工具") 安全工具

### [](https://www.sqlsec.com/2023/07/ventura.html#%E4%B8%BB%E6%9C%BA%E6%8E%A2%E6%B5%8B "主机探测") 主机探测

BASH

```bash
# 安装  nmap 与自带的 ncat 命令
brew install nmap

# 安装扫描速度更快的 masscan
brew install masscan

# 使用 ARP 协议来发现本地网络上的 IPv4 主机并指纹识别
➜ brew install arp-scan
➜ arp-scan --localnet
Interface: en0, type: EN10MB, MAC: c8:89:f3:b3:24:1d, IPv4: 10.1.1.180
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
10.1.1.1	00:0c:29:7c:19:2f	VMware, Inc.
10.1.1.10	a0:36:9f:89:ad:30	Intel Corporate
10.1.1.11	00:0c:29:b1:fa:11	VMware, Inc.
10.1.1.100	00:11:32:fa:6e:7c	Synology Incorporated
10.1.1.218	a8:a1:59:9f:57:aa	ASRock Incorporation
10.1.1.128	ec:4d:3e:86:cb:2e	Beijing Xiaomi Mobile Software Co., Ltd
10.1.1.199	78:11:dc:7d:d5:9a	XIAOMI Electronics,CO.,LTD

# go 编写的快速高效的端口扫描工具
➜ brew install naabu
```

### [](https://www.sqlsec.com/2023/07/ventura.html#%E5%AD%90%E5%9F%9F%E6%94%B6%E9%9B%86 "子域收集") 子域收集

BASH

```bash
# 快速被动子域枚举工具
➜ brew install subfinder

# 新生代子域名检测工具
➜ brew install findomain
```

### [](https://www.sqlsec.com/2023/07/ventura.html#SQL-%E6%B3%A8%E5%85%A5 "SQL 注入")SQL 注入

BASH

```bash
# 基本上 sqlmap 足够了
brew install sqlmap
```

### [](https://www.sqlsec.com/2023/07/ventura.html#%E7%9B%AE%E5%BD%95%E6%89%AB%E6%8F%8F "目录扫描") 目录扫描

BASH

```bash
# 基本上 dirsearch 足够使用了
pip3 install dirsearch
```

### [](https://www.sqlsec.com/2023/07/ventura.html#%E6%96%87%E4%BB%B6%E5%88%86%E6%9E%90 "文件分析") 文件分析

BASH

```bash
# binwalk CTF MISC 神器
brew install binwalk

# EXIF 元数据信息查看
brew install exiftool
```

### [](https://www.sqlsec.com/2023/07/ventura.html#%E7%A7%BB%E5%8A%A8%E5%AE%89%E5%85%A8 "移动安全") 移动安全

1. **macOS 安装 adb**

BASH

```bash
# brew 安装
➜ brew install --cask android-platform-tools
```

### [](https://www.sqlsec.com/2023/07/ventura.html#%E5%AF%86%E7%A0%81%E7%A0%B4%E8%A7%A3 "密码破解") 密码破解

BASH

```bash
# 密码破解神器
➜ brew install hashcat

# 经典老牌的 UNIX 密码破解工具
➜ brew install john

# WiFi 无线安全必备
➜ brew install aircrack-ng
```

### [](https://www.sqlsec.com/2023/07/ventura.html#Metasploit "Metasploit")Metasploit

下载最新版本的安装包：[https://osx.metasploit.com/metasploitframework-latest.pkg](https://osx.metasploit.com/metasploitframework-latest.pkg)

安装很简单，双击 **metasploitframework-latest.pkg** 安装包，一步步往下安装就可以了，macOS 下手动升级 Metasploit 版本国光这里建议也这样升级，比较方便省心。

macOS 下 Metasploit 的可执行文件的位置为：`/opt/metasploit-framework/bin`，所以我们需要收到配置一下环境变量：

BASH

```bash
echo 'export PATH="$PATH:/opt/metasploit-framework/bin/"' >> ~/.zshrc
```

接着新开个终端即可正常识别到 msf 系列的命令了：

BASH

```bash
➜  ~ msfconsole

 ** Welcome to Metasploit Framework Initial Setup **
    Please answer a few questions to get started.

# 输入 y 确定初始化一个新的数据库
Would you like to use and setup a new database (recommended)? y

# 认证信息保存在这里，我们了解一下就行
Writing client authentication configuration file /Users/security/.msf4/db/pg_hba.conf
```

[![](https://image.3001.net/images/20230711/16890785105921.png)](https://image.3001.net/images/20230711/16890785105921.png)

### [](https://www.sqlsec.com/2023/07/ventura.html#Burp-Suite "Burp Suite")Burp Suite

首先 [官网下载](https://portswigger.net/burp/releases) 截止目前（2023 年 07 月 09 日）最新的稳定版 2023.06.2：

[![](https://image.3001.net/images/20230711/16890785208302.png)](https://image.3001.net/images/20230711/16890785208302.png)

先正常安装打开的话是提示需要激活的，我们关掉 BP 先不管它。使用之前的 TrojanAZhen/BurpSuitePro-2.1 注册机并不能正常工作了，这是因为从 2022.9 开始提示 BP 官方换了注册机制，所以我们得使用新的注册机才可以。

新版注册机的地址为：[h3110w0r1d-y/BurpLoaderKeygen](https://github.com/h3110w0r1d-y/BurpLoaderKeygen)

下载下来，将其放入到 BP Jar 包的同级目录下：

[![](https://image.3001.net/images/20230711/1689078526252.png)](https://image.3001.net/images/20230711/1689078526252.png)

首先先带着注册机运行一下 BP：

BASH

```bash
cd /Applications/Burp\ Suite\ Professional.app/Contents/Resources/app && "/Applications/Burp Suite Professional.app/Contents/Resources/jre.bundle/Contents/Home/bin/java" "--add-opens=java.desktop/javax.swing=ALL-UNNAMED" "--add-opens=java.base/java.lang=ALL-UNNAMED" "--add-opens=java.base/jdk.internal.org.objectweb.asm=ALL-UNNAMED" "--add-opens=java.base/jdk.internal.org.objectweb.asm.tree=ALL-UNNAMED" "--add-opens=java.base/jdk.internal.org.objectweb.asm.Opcodes=ALL-UNNAMED" "-javaagent:BurpLoaderKeygen.jar"  "-jar" "/Applications/Burp Suite Professional.app/Contents/Resources/app/burpsuite_pro.jar"
```

提示需要激活，先放这儿：

[![](https://image.3001.net/images/20230711/16890785334135.png)](https://image.3001.net/images/20230711/16890785334135.png)

另开一个终端，运行注册机：

BASH

```bash
/Applications/Burp\ Suite\ Professional.app/Contents/Resources/jre.bundle/Contents/Home/bin/java -jar /Applications/Burp\ Suite\ Professional.app/Contents/Resources/app/BurpLoaderKeygen.jar
```

像之前一样，常规走一下注册流程即可：

[![](https://image.3001.net/images/20230711/16890785396644.jpg)](https://image.3001.net/images/20230711/16890785396644.jpg)

但是不能每次启动都需要借助这两串命令行吧，这也太不优雅了，所以需要我们编辑 vmoptions.txt 文件，直接将之前的参数追加到 vmoptions.txt 后面：

BASH

```bash
echo "--add-opens=java.desktop/javax.swing=ALL-UNNAMED" >> /Applications/Burp\ Suite\ Professional.app/Contents/vmoptions.txt
echo "--add-opens=java.base/java.lang=ALL-UNNAMED" >> /Applications/Burp\ Suite\ Professional.app/Contents/vmoptions.txt
echo "--add-opens=java.base/jdk.internal.org.objectweb.asm=ALL-UNNAMED" >> /Applications/Burp\ Suite\ Professional.app/Contents/vmoptions.txt
echo "--add-opens=java.base/jdk.internal.org.objectweb.asm.tree=ALL-UNNAMED" >> /Applications/Burp\ Suite\ Professional.app/Contents/vmoptions.txt
echo "--add-opens=java.base/jdk.internal.org.objectweb.asm.Opcodes=ALL-UNNAMED" >> /Applications/Burp\ Suite\ Professional.app/Contents/vmoptions.txt
echo "-javaagent:BurpLoaderKeygen.jar" >> /Applications/Burp\ Suite\ Professional.app/Contents/vmoptions.txt
echo "-Xmx2048m" >> /Applications/Burp\ Suite\ Professional.app/Contents/vmoptions.txt
```

效果如下：

[![](https://image.3001.net/images/20230711/16890785469654.png)](https://image.3001.net/images/20230711/16890785469654.png)

之后直接在应用程序里面启动 BP 就可以了，和官方正版使用毫无差别，优雅永不过时～

[![](https://image.3001.net/images/20230711/16890785596406.png)](https://image.3001.net/images/20230711/16890785596406.png)

### [](https://www.sqlsec.com/2023/07/ventura.html#%E7%BB%BC%E5%90%88%E6%89%AB%E6%8F%8F "综合扫描") 综合扫描

#### [](https://www.sqlsec.com/2023/07/ventura.html#AWVS "AWVS")AWVS

使用 Docker 搭建 Acunetix Web Vulnerability Scanner（AWVS）扫描器的很方便的， 我的 M1 Macbook 测试也可以跑 x86 的 Dokcer 容器：

BASH

```bash
# 拉取 awvs 镜像
docker pull secfa/docker-awvs

# 将 容器的 3443 端口映射到 13443 端口
docker run -d --name "AWVS" -p 13443:3443 --cap-add LINUX_IMMUTABLE secfa/docker-awvs
```

搭建完成后访问：[https://YOUR_IP:13443/](https://your_ip:13443/) 即可

默认的用户名为：[admin@admin.com](mailto:admin@admin.com) 密码为：Admin123

[![](https://image.3001.net/images/20230711/16890785645160.png)](https://image.3001.net/images/20230711/16890785645160.png)

#### [](https://www.sqlsec.com/2023/07/ventura.html#Nessus "Nessus")Nessus

Tenable Nessus® Essentials 是一款免费漏洞扫描程序，可作为漏洞评估的绝佳切入点。可获得无异于 Nessus Professional 订阅用户所享的强大扫描程序，支持扫描 16 个 IP 也足够我们个人使用了。

简而言之就是 Essentials 是 适用于教育工作者、学生以及网络安全领域的入门从业人员，功能和 Nessus Professional 没区别，只是批量扫描的 IP 变少了。

首先我们得使用企业域名注册获取激活码，注册地址：[https://zh-cn.tenable.com/products/nessus/nessus-essentials](https://zh-cn.tenable.com/products/nessus/nessus-essentials)

> 小 Tips：国内的 qq 和 163、gmail 这类大众域名多半是不行的，我这里使用的是 com 域名绑定的 QQ 邮箱注册的，这样很容易获取到激活码。

有激活码的话，Docker 搭建 Nessus 就简单很多了。

BASH

```bash
拉取镜像创建
docker pull tenableofficial/nessus

创建容器映射到宿主机的 8834 端口
docker run -d --name "Nessus"  -p 18834:8834 -e ACTIVATION_CODE=<你的激活码> -e USERNAME=<自定义用户名> -e  PASSWORD=<自定义密码>  tenableofficial/nessus
```

搭建完成后访问：[https://YOUR_IP:18834/](https://your_ip:18834/) 即可：

[![](https://image.3001.net/images/20230711/16890817445667.jpg)](https://image.3001.net/images/20230711/16890817445667.jpg)

访问出现没有出现 Nessus 界面？（移动网络可能会初始化识别，建议电信或者企业专线网络，或者挂代理或者软路由）

BASH

```bash
docker exec -it Nessus bash
cd /opt/scripts
./configure_scanner.py # 重新初始化操作
```

搭建好看了，看了可以看到我们的免费激活码支持做多 16 台设备，且插件都可以保持更新到最新版本，且激活码有效期有好几年呢，足够我们日常使用了：

[![](https://image.3001.net/images/20230711/1689081804931.jpg)](https://image.3001.net/images/20230711/1689081804931.jpg)

#### [](https://www.sqlsec.com/2023/07/ventura.html#OSV-Scanner "OSV-Scanner")OSV-Scanner

Google 新开源的漏洞扫描器，主要使用了 OSV-Scanner 来查找项目依赖漏洞。

漏洞来源全球各大漏洞库：

- [https://github.com/github/advisory-database](https://github.com/github/advisory-database) Github 安全公告
- [https://github.com/pypa/advisory-database](https://github.com/pypa/advisory-database) Python 的 pypy 安全公告
- [https://github.com/golang/vulndb](https://github.com/golang/vulndb) Go 的漏洞库
- [https://github.com/RustSec/advisory-db](https://github.com/RustSec/advisory-db) Rust 的漏洞库
- [https://github.com/cloudsecurityalliance/gsd-database](https://github.com/cloudsecurityalliance/gsd-database) 全球安全漏洞库
- [https://github.com/google/oss-fuzz-vulns](https://github.com/google/oss-fuzz-vulns) OSV 漏洞库
- …… 等

支持扫描自以下项目的漏洞：AlmaLinux、Alpine、Android、crates.io、Debian GNU/Linux、GitHub Actions、Go、Hex、Linux kernel、Maven、npm、NuGet、OSS-Fuzz、Packagist、Pub、PyPI、Rocky Linux、RubyGems……

使用细节大家可以参考官方文档：[https://google.github.io/osv-scanner/usage/](https://google.github.io/osv-scanner/usage/)

国光这里只做简单的的使用：

BASH

```bash
osv-scanner -r /path/to/your/dir
```

[![](https://image.3001.net/images/20230711/16890827373626.png)](https://image.3001.net/images/20230711/16890827373626.png)

## [](https://www.sqlsec.com/2023/07/ventura.html#%E6%94%AF%E6%8C%81%E4%B8%80%E4%B8%8B "支持一下") 支持一下

本文可能实际上也没有啥技术含量，但是写起来还是比较浪费时间的，在这个喧嚣浮躁的时代，个人博客越来越没有人看了，写博客感觉一直是用爱发电的状态。如果你恰巧财力雄厚，感觉本文对你有所帮助的话，可以考虑打赏一下本文，用以维持高昂的服务器运营费用（域名费用、服务器费用、CDN 费用等）

|   |   |
|---|---|
|微信<br><br>[![](https://image.3001.net/images/20200421/1587449920128.jpg)](https://image.3001.net/images/20200421/1587449920128.jpg)|支付宝<br><br>[![](https://image.3001.net/images/20200421/15874503376388.jpg)](https://image.3001.net/images/20200421/15874503376388.jpg)|

国光我还重写打赏页面 用以感谢 支持我的朋友，详情请看 [打赏列表 | 国光](https://www.sqlsec.com/reward/)

---

亲爱的读者们，在这个信息爆炸的时代，网络安全的重要性日益凸显，但同时，这个行业的挑战和误解也随之而来。作为一名网络安全的忠实守护者，我有幸在这个领域深耕多年，见证了无数技术的进步与变迁。

我始终坚信，知识的力量能够改变世界。因此，我用心制作了 [**网络安全系列课程**](https://course.sqlsec.com/)，不仅希望传授给大家宝贵的知识，更希望激发大家对网络安全的热爱和责任感。现在，我正准备推出第二期课程，并更新备受期待的内网安全教程，这是我对网络安全教育事业的承诺和热爱。

然而，正如大家所知，网络安全行业充满了不确定性和挑战。攻击门槛也在不断提高，即便是有 10 年经验的安全专家，有时也可能无法及时发现最新的漏洞，甚至在外人眼中，他们的努力和成就可能与实习生无异。但我相信，真正的价值和成就，是在于我们对知识的执着追求和对技术的不懈探索。

在完成这些课程后，我将暂时离开网络安全领域，转向其他行业。这不仅是一个艰难的决定，也是一个新的开始。但在此之前，我希望能够完成我的心愿，为大家带来更高质量的课程。

如果您对我的课程感兴趣，或者认同我对网络安全教育的执着和热情，请考虑购买我的课程，支持我的工作。您的支持不仅是对我努力的认可，更是对网络安全教育事业的一份贡献。

感谢每一位读者的陪伴和支持，让我们共同守护这个数字世界的安全！
