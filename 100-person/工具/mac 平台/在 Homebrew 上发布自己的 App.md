---
title: 在 Homebrew 上发布自己的 App
date: 2023-06-09T09:21:00+08:00
updated: 2024-08-21T10:34:29+08:00
dg-publish: false
aliases:
  - 在 Homebrew 上发布自己的 App
author: Luwang
category: person
comments: true
cover: //cdn.wallleap.cn/img/post/1.jpg
description: 文章描述
image-auto-upload: true
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
rating: 1
source: #
tags:
  - tool
url: //myblog.wallleap.cn/post/1
---

[Homebrew](http://brew.sh/) 是最初由 Max Howell 用 Ruby 写的 OS X 软件管理系统，其代码开源在 [GitHub](https://github.com/Homebrew/brew/) 上。

Homebrew 给自己贴了个一句话简介：The missing package manager for OS X。翻译过来成中文就是：寻它千百度的软件包管理器。名副其实地是，Homebrew 真的很好用。安装、升级、卸载等操作，在 Homebrew 的生态下，都只需要一条命令就可以了。并且 Homebrew 会自动为你解决软件包的依赖问题。

近日，[Belleve](http://typeof.net/) 邀请我协助他维护 [`otfcc`](https://github.com/caryll/otfcc) 在 OS X 上的预编译版本，并请我为之发布 Homebrew 的软件包。以下是对发布 Homebrew 软件包的总结。

## [](https://liam.page/2016/07/30/release-your-own-app-in-Homebrew/#%E6%98%BE%E8%80%8C%E6%98%93%E8%A7%81%E7%9A%84%E5%AF%B9%E6%AF%94 "显而易见的对比") 显而易见的对比

`otfcc` 现在已经有了预编译好的 Mac 版本。对于用户来说，只需要执行

|   |   |
|---|---|
|1  <br>2  <br>3|wget <https://github.com/caryll/otfcc/releases/download/v0.2.3/otfcc-mac64-0.2.3.tar.xz>  <br>tar Jvxf otfcc-mac64-0.2.3.tar.xz  <br>sudo mv otfcc* /usr/local/bin|

好吧，那个「只」字，我应该加上引号表示夸张的。现在看看有了 Homebrew 包我们可以怎样做

|   |   |
|---|---|
|1|brew tap caryll/tap && brew install otfcc-mac64|

对比确实是显而易见的，不是吗？使用 `brew` 安装，只需要执行命令。剩下的东西，只需要交给 Homebrew 去处理就好了。

## [](https://liam.page/2016/07/30/release-your-own-app-in-Homebrew/#%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5 "基本概念") 基本概念

在 Homebrew 的架构下，至少有 4 层概念

- Keg（酒桶）：安装好的脚本、软件等；
- Cellar（酒窖）：所有用 Homebrew 安装在本地的脚本、软件组成的集合；
- Formula（配方）：定义如何下载、编译和安装脚本或软件的 Ruby 脚本；
- Tap：一个包含若干 Formula 的 GitHub 专案。

我们平时使用 `brew install foobar` 安装软件时，实际上是从 `Homebrew/homebrew-core` 这个 Tap 中获取相应的 Formula，然后将 Keg 安装在 Cellar 中。现在的问题是，`Homebrew/homebrew-core` [不允许普通用户提交自己写的小众脚本、软件](https://github.com/Homebrew/brew/blob/master/share/doc/homebrew/Acceptable-Formulae.md)。所以，我们需要建立一个新的 Tap（GitHub 专案），包含对应我们软件的 Formula，然后将 Keg 放入本地的 Cellar 中。

于是我们知道，我们要做的事情是：

1. 理顺安装程序的步骤；
2. 创建一个 Formula，将上述步骤用 Ruby 表述出来；
3. 创建一个 GitHub 专案，将上述 Formula 纳于该专案的版本控制之下。

## [](https://liam.page/2016/07/30/release-your-own-app-in-Homebrew/#%E5%88%9B%E5%BB%BA-Homebrew-Package-%E5%B9%B6%E5%AE%89%E8%A3%85 "创建 Homebrew Package 并安装") 创建 Homebrew Package 并安装

Homebrew 提供了一个 [官方指南](https://github.com/Homebrew/brew/blob/master/share/doc/homebrew/Formula-Cookbook.md)。不过，这个指南写得比较晦涩难懂。所以，我另起炉灶，按照更容易理解的方式介绍一下创建 Homebrew

### [](https://liam.page/2016/07/30/release-your-own-app-in-Homebrew/#%E7%90%86%E9%A1%BA%E6%AD%A5%E9%AA%A4 "理顺步骤") 理顺步骤

第一节中，我们已经看到手工安装 `otfcc` 的操作步骤。其中第一步是下载，这一步在执行 `brew install` 时，Homebrew 会根据 Formula 的指示完成。第二步是解包，这一步 Homebrew 也会为我们自动完成。因此，实际我们要做的，就是将二进制文件放到 `/usr/local/bin` 目录下。

### [](https://liam.page/2016/07/30/release-your-own-app-in-Homebrew/#%E5%88%9B%E5%BB%BA-Formula "创建 Formula") 创建 Formula

假设我们的软件已经有了一个发布版本，并且可供下载，比如 [https://github.com/caryll/otfcc/releases/download/v0.2.3/otfcc-mac64-0.2.3.tar.xz](https://github.com/caryll/otfcc/releases/download/v0.2.3/otfcc-mac64-0.2.3.tar.xz)。接下来，我们在终端里执行

|   |   |
|---|---|
|1|brew create <https://github.com/caryll/otfcc/releases/download/v0.2.3/otfcc-mac64-0.2.3.tar.xz>|

这条命令会在 `/usr/local/Library/Formula/` 下创建一个 `.rb` 文件，其文件名取决于传给 `brew create` 的 URL。比如，我们这里会创建 `/usr/local/Library/Formula/otfcc-mac64.rb`。

otfcc-mac64.rb

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8  <br>9  <br>10  <br>11  <br>12  <br>13  <br>14  <br>15  <br>16  <br>17  <br>18  <br>19  <br>20  <br>21  <br>22  <br>23  <br>24  <br>25  <br>26  <br>27  <br>28  <br>29  <br>30  <br>31  <br>32  <br>33  <br>34  <br>35  <br>36  <br>37  <br>38  <br>39|# Documentation: <https://github.com/Homebrew/brew/blob/master/share/doc/homebrew/Formula-Cookbook.md>  <br>#                <http://www.rubydoc.info/github/Homebrew/brew/master/Formula>  <br># PLEASE REMOVE ALL GENERATED COMMENTS BEFORE SUBMITTING YOUR PULL REQUEST!  <br>  <br>class OtfccMac64 < Formula  <br>  desc ""  <br>  homepage ""  <br>  url "https://github.com/caryll/otfcc/releases/download/v0.2.3/otfcc-mac64-0.2.3.tar.xz"  <br>  version "0.2.3"  <br>  sha256 "f393b14f9c42c890b8a60949b13a8f9b5c3f814daa8b18901656ccc3b866f646"  <br>  <br>  # depends_on "cmake" => :build  <br>  depends_on :x11 # if your formula requires any X11/XQuartz components  <br>  <br>  def install  <br>    # ENV.deparallelize  # if your formula fails when building in parallel  <br>  <br>    # Remove unrecognized options if warned by configure  <br>    system "./configure", "--disable-debug",  <br>                          "--disable-dependency-tracking",  <br>                          "--disable-silent-rules",  <br>                          "--prefix=#{prefix}"  <br>    # system "cmake", ".", *std_cmake_args  <br>    system "make", "install" # if this fails, try separate make/make install steps  <br>  end  <br>  <br>  test do  <br>    # `test do` will create, run in and delete a temporary directory.  <br>    #  <br>    # This test will fail and we won't accept that! It's enough to just replace  <br>    # "false" with the main program this formula installs, but it'd be nice if you  <br>    # were more thorough. Run the test with `brew test otfcc-win32`. Options passed  <br>    # to `brew install` such as `--HEAD` also need to be provided to `brew test`.  <br>    #  <br>    # The installed folder is not in the path, so use the entire path to any  <br>    # executables being tested: `system "#{bin}/program", "do", "something"`.  <br>    system "false"  <br>  end  <br>end|

我们先来看类的头部

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5|desc ""  <br>homepage ""  <br>url "https://github.com/caryll/otfcc/releases/download/v0.2.3/otfcc-mac64-0.2.3.tar.xz"  <br>version "0.2.3"  <br>sha256 "f393b14f9c42c890b8a60949b13a8f9b5c3f814daa8b18901656ccc3b866f646"|

`desc` 是 description 的缩写，我们应该在这里填上 `otfcc` 的简介。`homepage` 相应地填上项目主页。后面的 `url`/`version`/`sha256` 都由 Homebrew 默认给我们填好了，我们只需要检查一下是否正确即可。

接下来是依赖项部分

|   |   |
|---|---|
|1  <br>2|# depends_on "cmake" => :build  <br>depends_on :x11 # if your formula requires any X11/XQuartz components|

由于 `otfcc` 没有任何其他的依赖项，所以不需要在此填入任何内容。

接下来是安装部分

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8  <br>9  <br>10  <br>11|def install  <br>  # ENV.deparallelize  # if your formula fails when building in parallel  <br>  <br>  # Remove unrecognized options if warned by configure  <br>  system "./configure", "--disable-debug",  <br>                        "--disable-dependency-tracking",  <br>                        "--disable-silent-rules",  <br>                        "--prefix=#{prefix}"  <br>  # system "cmake", ".", *std_cmake_args  <br>  system "make", "install" # if this fails, try separate make/make install steps  <br>end|

默认情况下，Homebrew 会并行地编译源代码。不过，有些程序并不适用并行编译。此时可以将 `# ENV.deparallelize` 的注释解开。后续的内容也比较简单，先后调用 `./configure` 和 `make install`——这是标准的 `*nix` 编译安装流程。

由于我们现在讨论的是安装 `otfcc` 的预编译版本，所以不需要考虑编译的过程。只需要调用 Homebrew 提供的 `bin.install` 就可以了。

最后的测试部分，我们暂时不探讨。

于是我们就得到了最后的 `otfcc-mac64.rb`

otfcc-mac64.rb

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8  <br>9  <br>10  <br>11  <br>12  <br>13  <br>14  <br>15  <br>16  <br>17  <br>18  <br>19|# Documentation: <https://github.com/Homebrew/brew/blob/master/share/doc/homebrew/Formula-Cookbook.md>  <br>#                <http://www.rubydoc.info/github/Homebrew/brew/master/Formula>  <br># PLEASE REMOVE ALL GENERATED COMMENTS BEFORE SUBMITTING YOUR PULL REQUEST!  <br>  <br>class OtfccMac64 < Formula  <br>  desc "Parses & writes SFNT structures."  <br>  homepage "https://github.com/caryll/otfcc/releases/"  <br>  url "https://github.com/caryll/otfcc/releases/download/v0.2.3/otfcc-mac64-0.2.3.tar.xz"  <br>  version "0.2.3"  <br>  sha256 "f393b14f9c42c890b8a60949b13a8f9b5c3f814daa8b18901656ccc3b866f646"  <br>  <br>  def install  <br>    # ENV.deparallelize  # if your formula fails when building in parallel  <br>  <br>    # system "tar -Jvxf otfcc-mac64-0.2.3.tar.zx"  <br>    bin.install "otfccbuild"  <br>    bin.install "otfccdump"  <br>  end  <br>end|

保存后，你就可以执行 `brew install otfcc-mac64` 了。

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8|$ brew install otfcc-mac64  <br>==> Downloading <https://github.com/caryll/otfcc/releases/download/v0.2.3/otfcc-mac64-0.2.3.tar.xz>  <br>Already downloaded: /Library/Caches/Homebrew/otfcc-mac64-0.2.3.tar.xz  <br>🍺  /usr/local/Cellar/otfcc-mac64/0.2.3: 3 files, 882.2K, built in 0 seconds  <br>$ which otfccdump  <br>/usr/local/bin/otfccdump  <br>$ which otfccbuild  <br>/usr/local/bin/otfccbuild|

由于之前在 `brew create` 的过程中，已经下载了 `otfcc-mac64-0.2.3.tar.zx` 这一文件，所以 Homebrew 不会再次下载。执行完毕后，我们发现，`otfccdump` 和 `otfccbuild` 两个二进制程序已经被安装在了 `/usr/local/bin/` 目录下——这是 Homebrew 安装二进制的默认目录。

我们的 Formula 比较简单，不涉及 Homebrew 提供的更深的功能。如果有必要，你需要参考 [官方指南](https://github.com/Homebrew/brew/blob/master/share/doc/homebrew/Formula-Cookbook.md)。

### [](https://liam.page/2016/07/30/release-your-own-app-in-Homebrew/#%E5%88%9B%E5%BB%BA-Tap "创建 Tap") 创建 Tap

至此我们已经创建了 Formula。根据这个 Formula，我们本地的 Homebrew 已经可以将 `otfcc` 安装到 `/usr/local/bin` 目录下了。然而，其他人暂时还不能获取、安装我们的软件。为此，我们需要创建自己的 Tap。

前面已经说过，Tap 实际上是 GitHub 网站上的一个专案。这个专案有些特殊，它的名字必须是 `homebrew-foobar` 的形式。也就是说，它必须有 `homebrew-` 这一前缀。`otfcc` 放在 GitHub 上的 `caryll` 组织中开发。于是我们可以在 `caryll` 中创建一个名为 `homebrew-tap` 的专案，然后将上述 `otfcc-mac64.rb` 纳入 `homebrew-tap` 这一专案的版本控制之下。

创建专案及纳入版本控制的过程是基本的 Git 操作，这里就不做展开了。创建好的专案在此：[https://github.com/caryll/homebrew-tap](https://github.com/caryll/homebrew-tap)，可供一观。

### [](https://liam.page/2016/07/30/release-your-own-app-in-Homebrew/#%E5%AE%9E%E9%99%85%E5%AE%89%E8%A3%85%E7%9C%8B%E7%9C%8B "实际安装看看") 实际安装看看

现在，我们创建了自己的 Formula 和 Tap，并在 GitHub 上为 Tap 创建了一个公开的专案。那么，世界上所有的 Homebrew 用户就可以使用下列指令很方便地安装 `otfcc` 了。

|   |   |
|---|---|
|1|brew tap caryll/tap && brew install otfcc-mac64|

注意，这里我们首先调用了 `brew tap` 命令，将我们自建的 Tap 加入到本地 Homebrew 的搜索列表中。加入搜索列表的内容是 `caryll/tap`——相比 GitHub 上的专案名称，Homebrew 会自动在传入 `brew tap` 的地址里加上 `homebrew-` 这一前缀。而后，我们照常调用 `brew install` 命令，就能顺利地安装 `otfcc` 在 OS X 上了。

## 创建索引文件

以’VNote’为例

```
brew cask create vnote
```

然后会在 `/usr/local/Homebrew/Library/Taps/caskroom/homebrew-cask/Casks` 路径下面新建一个 `vnote.rb` 文件,使用代码编辑器打开这个文件.

## 添加元信息

在没做任何修改之前,文件长下面这个样子

```
cask 'vnote' do
  version ''
  sha256 ''

  url 'https://'
  name ''
  homepage ''

  app ''
end
```

然后写入各种信息.

```
cask 'vnote' do
  version '1.11.1'
  sha256 'ceb5636c4a11d30c75bb8b0e87cd659894b5421385f2932d7881d2ff657fc941'

  url "https://github.com/tamlok/vnote/releases/download/v#{version}/VNote_X64_#{version}.dmg"
  appcast 'https://github.com/tamlok/vnote/releases.atom',
          checkpoint: '76d7ddfb8dba8ada24d93e338adee694c780643ae15a446c3db4d2b00909e887'
  name 'VNote'
  homepage 'https://github.com/tamlok/vnote'

  app 'VNote.app'
end
```

`version` 代表版本,`sha256` 用作文件验证.`url` 是下载路径,`appcast` 至今没搞清楚有啥用.`app` 写 dmg 里面的应用名,不管是 dmg 还是 zip 的格式,cask 都是能自动识别的.保存完毕然后运行测试.

```
brew cask audit vnote
```

如果出现 `passed`,说明元信息编写正确.然后就可以提交给 [homebrew-cask](https://github.com/caskroom/homebrew-cask) 这个仓库了.提交完毕等待合并就可以了.
