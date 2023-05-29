
**[Clash for Windows 中文汉化教程：使用Clash订阅 - 优质盒子 (uzbox.com)](https://uzbox.com/tech/clash.html)**

### Clash 是谁？

Clash 是由 Dreamacro 开发的，是一个使用 Go 开发的、基于规则的隧道，无需服务器和[虚拟主机](https://uzbox.com/tag/xunizhuji)，在中国大陆地区可用于突破防火长城的限制。 2020 年末，Dreamacro 在 [GitHub](https://uzbox.com/tag/github) 仓库释出 Clash 的衍生版本 Premium，而不提供其原始码。

Clash for Windows 是运行在 Windows 上的一图形化 Clash 分支。通过 Clash API 来配置和控制 Clash 核心程序，便于用户可视化操作和使用。

Clash 是一个基于规则的跨平台[代理软件](https://uzbox.com/tag/dailiruanjian)核心程序。支持 SS / VMess 协议 。

### VPN 和 Clash 的区别

在很多人心目中一想到 VPN，就会想到翻墙工具，其实不是。VPN 是一种加密通讯技术，是一个统称。它有很多的具体实现，比如 PPTP 、L2TP 、IPSec 和 OpenVPN 。VPN 的出现远早于 GFW ，所以它不是为了翻墙而生的。 VPN 最主要的功能，并不是用来翻墙，只是它可以达到翻墙的目的。

它的功能是：在公用网络上建立专用网络，进行加密通讯。在企业网络和高校的网络中应用很广泛。（很多大学会提供 VPN 接口，以便随时随地可以从公网接入校园网以便下载知网论文等资源。）你接入 VPN ，其实就是接入了一个专有网络，你的网络访问都从这个出口出去，你和 VPN 之间的通信是否加密，取决于你连接 VPN 的方式或者协议。

Clash 则是一款比较出色的[代理](https://uzbox.com/tag/daili)客户端软件，支持很多种代理协议。v2ray，[Shadowsocks](https://uzbox.com/tag/shadowsocks)，[ShadowsocksR](https://uzbox.com/tag/shadowsocksr)等是协议服务端，例如v2ray的代理协议是VMess 协议，Shadowsocks的代理协议是SS协议。而VPN则是虚拟机专用网络，两者完全不是一个概念。

Clash 无需服务器，无需虚拟主机，也不需要域名，Clash是一个[网络代理](https://uzbox.com/tag/wangluodaili)协议的客户端，客户端可以在电脑，[安卓](https://uzbox.com/tag/anzhuo)，苹果等平台上使用。

### 如何下载安装中文版Clash？

#### 安装原版Clash for Windows

首先第一步是先安装原版的Clash，在Clash的github项目地址中下载最新版的Clash。

Clash最新版本镜像下载：**[点击下载 Clash.for.Windows.Setup.0.20.5](https://github.com/Fndroid/clash_for_windows_pkg/releases/download/0.20.5/Clash.for.Windows.Setup.0.20.5.exe)**

Clash最新版本下载地址：https://github.com/Fndroid/clash_for_windows_pkg/releases

![](https://cdn.wallleap.cn/img%2Fpic%2Fillustrtion%2F202210310931648.png)

截至到2022年10月12日，Clash 最新版本是[v 0.20.5](https://github.com/Fndroid/clash_for_windows_pkg/releases/tag/0.20.5)，下载安装适合你电脑的版本。.exe版本是需要安装的版本，.7z的版本是解压缩后可以直接使用的。建议下载.exe的版本。

![](https://cdn.wallleap.cn/img%2Fpic%2Fillustrtion%2F202210310934307.png)

下载完毕后进行安装，安装默认位置，点击安装！

![](https://cdn.wallleap.cn/img%2Fpic%2Fillustrtion%2F202210310937737.png)

之后将运行Clash for Windows前面的勾取消掉，点后点击完成。接下来安装[Clash for Windows中文补丁](https://uzbox.com/tag/clash-for-windowszhongwenbuding)。

#### 安装Clash for Windows中文补丁

Clash for Windows中文项目地址：https://github.com/BoyceLig/Clash_Chinese_Patch/releases

![Clash for Windows v0.19.29 汉化补丁](https://cdn.wallleap.cn/img%2Fpic%2Fillustrtion%2F202210310938281.png)

下载app.7z或app.zip文件（两个压缩包内容一样）后，压缩压缩包，请自行替换下面路径中的app.asar文件。
**Clash for Windows\resources\app.asar**
![](https://cdn.wallleap.cn/img%2Fpic%2Fillustrtion%2F202210310938809.png)

复制app.asar文件，到安装目录下的resources文件夹中，替换掉原有的文件。

![](https://cdn.wallleap.cn/img%2Fpic%2Fillustrtion%2F202210310938751.png)

替换成功后，打开Clash for Windows，已经成功汉化成中文版的Clash for Windows了。

![](https://cdn.wallleap.cn/img%2Fpic%2Fillustrtion%2F202210310939454.png)

### Clash for Windows 订阅

中文版的Clash for Windows已经安装成功了，接下来就是如何使用中文版的Clash for Windows。

![](https://cdn.wallleap.cn/img%2Fpic%2Fillustrtion%2F202210310939746.png)

示例：将链接复制到配置文件下载框内，点击下载。

![](https://cdn.wallleap.cn/img%2Fpic%2Fillustrtion%2F202210310940109.png)

Clash for Windows本地文件拖拽导入，如果无法通过 URL 下载配置文件，则可以尝试在浏览器中下载配置文件后，通过拖拽方式导入yaml文件。

如果你有自己搭建的节点，想转换成Clash订阅链接的话，可以访问下面几个订阅转换器进行Clash订阅链接转换。

**节点转换成Clash订阅链接：**

https://sub-web.qingsay.com/

https://www.con8.tk/

https://bianyuan.xyz/

https://acl4ssr-sub.github.io/

https://sub.ccsub.site/

https://api.nameless13.com/

Clash for Windows订阅文件已经成功下载到本地的Clash for Windows客户端中。下面返回到常规中，点击系统代理的按钮，是按钮变成绿色。

如果想开机自动启动Clash for Windows，也可以将开机启动的按钮点亮。

![](https://cdn.wallleap.cn/img%2Fpic%2Fillustrtion%2F202210310940564.png)

现在，基本配置已经完成了，Clash for Windows已经开始工作了。

Clash for Windows的代理工作方式，分为全局，规则，直连，脚本四种。

## 更详细的 Clash for Windows 使用教程

[Clash for Windows教程：配置TUN/TAP虚拟网卡](https://uzbox.com/tech/clash-atp.html)

[Clash for windows 如何设置 telegram 电报代理上网](https://uzbox.com/tech/clash-telegram.html)

CFW官方网站：https://docs.cfw.lbyczf.com/

如果你想使用安卓手机安装clash，请浏览：**[Clash for Android 中文使用教程：Clash订阅](https://uzbox.com/app/clash-for-android-2.html)**

**警告：clash软件仅供软件技术交流，严禁在违法违规的环境下使用！**