---
title: 1. 前言
date: 2024-03-06T01:58:18+08:00
updated: 2024-08-21T10:34:29+08:00
dg-publish: false
---

## 1. 前言

抓包调试是移动应用程序开发和调试中非常重要的一部分，无论是数据模拟还是恶意软件分析。我们最常见的方法是在手机上设置 Wi-Fi 代理，并将流量代理到桌面应用程序（例如 Charles 和 Fiddler ）的 MITM 服务器。

但是这种工作方式不但操作繁琐，还存在一些技术障碍问题。

- 在 Wifi 中配置代理非常麻烦，调试完后还需要再改回来。
- 某些网络框架不会读取系统代理，无法捕获到流量。
- 安装根证书的时候，将证书同步到手机不方便。
- Wifi 代理是全局的，无法选择指定应用生效。

因此，给大家介绍一种新的移动端抓包调试方案。

## 2. 准备

本篇文章所讲的主要工具是 [Reqable](https://reqable.com/) , 一款桌面端（ Windows 、Mac 、Linux ） + 移动端（ Android 、iOS ）的全平台 API 调试工具。

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202403061357184.png)

下载地址： <https://reqable.com/download>

### 2.1 桌面端版本

安装完成后，启动 Reqable 桌面端应用。打开二维码页面，如下：

![未命名.png](https://cdn.wallleap.cn/img/pic/illustration/202403061357425.png)

### 2.2 移动端版本

安装后，启动 Reqable 移动端应用。选择 `协作模式`，并扫描上一步桌面端的二维码。

![Screenshot_20240118_155629.png](https://cdn.wallleap.cn/img/pic/illustration/202403061357764.png)

连接成功后，Reqable 会自动将桌面端的 `根证书` 同步到手机端。Reqable 移动端会记住远程设备（电脑）的 IP 地址和端口，下一次启动会自动进行连接。如果远程设备（电脑）的 IP 地址和端口发生变化，在侧边栏点击扫码图标重新扫描即可。

下一步，我们开始安装 `根证书`，这可能是整个过程最复杂的一步。

由于 Reqable 桌面端的根证书已经被同步到了移动端，因此不需要打开浏览器输入 xxx 地址下载根证书了，直接在移动端保存根证书即可。打开侧边栏，点击 `证书管理` 页面，进行证书相关的操作。

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202403061357037.png)

Reqable 移动的会自动检查证书的安装状态，如果未安装成功，页面上出现红色提示：证书未安装。上图是 Android 的根证书安装指引，iOS 则简单很多，按照应用内的操作提示处理即可。

*注意：如果 Android 的证书安装到用户证书目录，记得在项目中配置网络安全配置 xml 文件。*

最后，点击右下角的调试按钮，允许通知和 VPN 服务，进入调试模式。

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202403061357204.png)

同时，Reqable 桌面端也会自动进入调试模式。

![未命名 2.png](https://cdn.wallleap.cn/img/pic/illustration/202403061357492.png)

可以看到，手机端的流量已经显示在 Reqable 桌面端了，后续我们可以在桌面端进行断点、重写、脚本等功能的处理。

### 2.3 安装辅助服务（ Android ）

从上图可以看到，应用程序一栏显示的是手机的 IP 地址，而不是真实的应用程序信息。除此之外，可能部分应用的流量也无法被捕获。这种情况下，我们需要安装并启用 `Reqable 辅助服务`。

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202403061357503.png)

重新启动调试功能，可以看到流量应用信息已经显示出来了。

![未命名 3.png](https://cdn.wallleap.cn/img/pic/illustration/202403061357849.png)

## 3. 调试

准备工作完成之后，我们开始正式的调试。为了更好的展示功能，写了一个简单的 Android 应用 `My Application`，嵌套一个 WebView ，加载 Reqable 的首页。

```kotlin
val web = findViewById<WebView>(R.id.webview)
web.webViewClient = WebViewClient()
val settings = web.settings
settings.javaScriptEnabled = true
web.loadUrl("https://reqable.com")
```

由于在测试的 Android 设备上根证书安装到了用户目录，我们在 `My Application` 里面配置好 xml 文件信任用户证书。另一种方式是将 `My Application` 的 targetSDK 调低至 23 。

为了去除其他应用流量的干扰，我们点击右上角菜单，并添加当前的测试应用 `My Application`（注意，选择应用的功能只在 Android 平台上可用，iOS 平台不可用）。

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202403061357730.png)

接下来启动调试，可以看到 Reqable 桌面端已经捕获了手机端的流量信息。

![未命名 4.png](https://cdn.wallleap.cn/img/pic/illustration/202403061357750.png)

现在我们来假定一个测试场景，需要将 `My Application` 中所有网络请求数据中的 `Reqable` 文字改成 `Awesome`，那该怎么做呢？

只需要写一行 Python 脚本即可：

```python
response.body.replace('Reqable', 'Awesome')
```

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202403061357834.png)

然后，我们重新加载 WebView 测试下。

![image.png](https://cdn.wallleap.cn/img/pic/illustration/202403061357868.png)

OK ，没什么问题。

## 4. 结束

感谢大家阅读，以上演示的功能只是 Reqable 这个项目的冰山一角，有更多的功能已经实现或者正在实现，也欢迎大家下载体验并提供建议： <https://reqable.com/>
