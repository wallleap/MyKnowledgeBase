---
title: 自由输入法 RIME 简明配置指南
date: 2023-02-20T03:22:00+08:00
updated: 2024-08-21T10:34:29+08:00
dg-publish: false
aliases:
  - 自由输入法 RIME 简明配置指南
author: Luwang
category: tool
comments: true
cover: //cdn.wallleap.cn/img/post/1.jpg
description: 文章描述
image-auto-upload: true
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
rating: 1
source: null
tags:
  - tool
url: //myblog.wallleap.cn/post/1
---

## 为什么选择 RIME

输入法是一种工具。工具千千万万，好工具唯趁手耳。Rime 恰恰是这些工具中一个特殊的存在。

之所以说「特殊」，原因在于，绝大部份工具，长什么样、能实现什么功能，出厂即定型，唯有一些自定义选项，也是出厂即以划定的框框。然而千人千面，每个人对于工具的需求都是不同的，这些不同的需求，往往都是细节上的不同。这是普通输入法工具无法满足的。

而 Rime 则不然。本质上，Rime 只是一个输入法引擎，你需要什么功能、你需要什么样的输入方案、你在输入细节上有什么需求，都可以通过自定义来实现。它能实现你在其他输入法中无法、不能实现的功能，最终把它打磨成你想要、趁手的样子。

也正是因为 Rime 的这个优点即是它的缺点——高度定制化带来的高准入门槛。定制困难劝退了一大批人。

本文希望，通过一些有逻辑顺序、 相对简单的配置方法，帮助有定制化输入法需求的人快速入门，起码先实现 80% 的需求，再慢慢学习、雕琢，使它成为自己想要的样子。

**总结**：

高自由

全平台兼容：Windows、macOS、Linux，甚至安卓都有衍生（[同文安卓输入法平台](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Fosfans%2Ftrime)）。

- 众多可选方案：包括小鹤、微软在内的双拼方案，还能实现小鹤双拼形码辅助；注音、五笔、仓颉……以及自定方案。
- 标点自定、模糊音、词语联想、个人词库，基本囊括常规输入工具的功能。

高门槛

- 无图形界面，自定全靠代码。
- 配置繁琐，容易出错。

## 如何安装 RIME

**资源：**

1. [RIME 安装包下载](https://sspai.com/link?target=https%3A%2F%2Frime.im%2Fdownload)
2. [输入方案下载](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Frime)
3. [rime_pro 增强包下载](https://sspai.com/link?target=https%3A%2F%2Fpan.baidu.com%2Fs%2F1jI1a8eE)

### 安装 *以 Windows 平台小鹤双拼方案为例*

按照上文 1 下载安装包，跟随安装向导完成安装。在**用户资料夹**页面，下方空白处选择使用默认位置。(默认位置在 windows 的个人文件夹 ~\AppData\Roaming\Rime，也可以选择**我来指定位置**，在下方空白框内填入你指定的文件夹的**绝对路径**。下文所称用户资料夹，均指这个文件夹）。

![](https://cdn.wallleap.cn/img/pic/illustration/202302201523176.png)

小狼豪安装选项

按照上文 2 下载所需的输入方案，全拼用户可略过此步。

- 在开始菜单，点击**【小狼毫】输入法设定**，点击获取更多输入方案，打开 rime package installer 命令行界面。
- 将上文 2 下载的输入方案拖入该窗口，回车完成安装。(若第一步选择了指定文件夹，需从默认文件夹将输入方案拷贝粘贴至你指定的文件夹内)。
- 重新打开**【小狼毫】输入法设定**，在左侧方案名称内勾选所需使用的方案，**小鹤双拼**。
- 点击**中**进入下一步**【小狼毫】界面风格设定**，选取喜欢的界面风格。不喜欢没关系，后面会给一个微软拼音同款自定界面。

实际上，到此步，Rime 小狼毫输入法已经基本可用了。当然，我们并不会满足于此。下面进入定制阶段。

## 如何定制 RIME

关于如何定制，实际上，网络上可供搜罗的教程很多，每个人都有自己的定制。正如上文所说，千人千面。所以，搜罗这些教程未必能满足于自己的需求。不过，前人栽树后人乘凉，我们可以利用高手大神做好的适合大部分人群的配置文件来进行个人定制化。

上文 3 **Rime_pro 增强包**就是这样一份全面的配置。

### 需要事前了解的常识

在进行这一步之前，需要了解几条常识：

Rime 的各种配置，均是由 .yaml 文件所定义。 yaml 是一种标记语言。 .yaml 文件实际上是文本文档。可使用记事本、或 Emeditor 等进行编辑。

对 Rime 进行自定义，是通过对 .custom.yaml 文件修改达成。不同的 .custom.yaml 文件，控制不同的功能实现。 .custom.yaml 实际上是相当于对 .yaml 文件打补丁，在重新部署后，会将 .custom.yaml 中的内容写入 .yaml 文件中，完成自定。

- 例一： weasel.yaml 是常规设置，主要控制托盘图标、候选词横竖排列、界面配色等等功能。那么，我们需要定制界面配色，只需在 weasel.custom.yaml 中修改，重新部署后就可实现。
- 例二： default.yaml 是默认设置，主要控制快捷键、按键上屏等等。同样，作修改就编辑 default.custom.yaml 文件即可。
- 例三：以上是全局设置，亦即不论使用何种输入方案，均起作用。 double_pinyin_flypy.custom.yaml 这种则是输入法方案设置。主要实现特殊标点符号、词库等功能。是针对特定输入方案的配置。

可见，我们绝大部分的自定，都只需修改对应的 .custom.yaml 文件即可。

所有自定修改，都必须**重新部署**。在开始菜单可以找到**【小狼毫】重新部署。**

- 在开始菜单可以找到**【小狼毫】重新部署。**
- 右键托盘图标**重新部署**。

### 现在开始配置 Rime

解压 Rime_pro 软件增强包，并把里面的文件拷贝到 ~/Rime （此处是上文所述指定用户资料夹），覆盖即可。

请注意，该增强包在最新版小狼毫不能很好地支持小鹤双拼方案，故而，需首先将 double_pinyin_flypy.custom.yaml 文件中的内容清空。

**现在，来配置小鹤双拼方案。**将如下代码，复制黏贴进入 double_pinyin_flypy.custom.yaml 文件。

```
patch:

# 載入朙月拼音擴充詞庫
"translator/dictionary": luna_pinyin.extended

# 输入双拼码的时候不转化为全拼码
translator/preedit_format: {}

#载入custom_phrase自定义短语
engine/translators:

   - punct_translator
   - reverse_lookup_translator
   - script_translator
   - table_translator@custom_phrase #表示调用 custom_phrase段编码
   - table_translator

custom_phrase:
 dictionary: ""
 user_dict: custom_phrase
 db_class: stabledb
 enable_completion: false
 enable_sentence: false
 initial_quality: 1

#  符号快速输入和部分符号的快速上屏
punctuator:
 import_preset: symbols
 half_shape:
#      "#": "#"
   '`': ["·","`"]
#      "~": "~"
#      "@": "@"
#      "=": "="
#      "!": "!"
#      "/": ["/", "÷"]
   '\': "、"
#      "'": {pair: ["「", "」"]}
#      "[": ["【", "["]
#      "]": ["】", "]"]
#      "$": ["¥", "$", "€", "£", "¢", "¤"]
#      "<": ["《", "〈", "«", "<"]
#      ">": ["》", "〉", "»", ">"]
```

这些代码的含义，已经有详细注释说明了。如不需要某项自定义，将其注释掉就可禁用了。 如果需要某些自定义，可以找到相关教程，添加相应的代码段即可。\
**注意：**

- **该文档只有最开头需要一句 patch: ，从其他教程拷贝的自定义代码段，请注意不要再次带入 patch: 。**
- **该文档有严格的缩进要求，请注意按照格式缩进。**

**现在，来配置扩展词库。**打开 luna_pinyin.extended.dict.yaml 文件。找到如下代码段。 

```
---
name: luna_pinyin.extended
version: "2014.10.28"
sort: by_weight
use_preset_vocabulary: true
#此處爲明月拼音擴充詞庫（基本）默認鏈接載入的詞庫，有朙月拼音官方詞庫、明月拼音擴充詞庫（漢語大詞典）、明月拼音擴充詞庫（詩詞）、明月拼音擴充詞庫（含西文的詞彙）。如果不需要加載某个詞庫請將其用「#」註釋掉。
#雙拼不支持 luna_pinyin.cn_en 詞庫，請用戶手動禁用。
import_tables:

- luna_pinyin

#- luna_pinyin.cn_en

- luna_pinyin.computer

#- luna_pinyin.emoji

- luna_pinyin.hanyu

#- luna_pinyin.kaomoji

- luna_pinyin.movie
- luna_pinyin.music
- luna_pinyin.name
- luna_pinyin.poetry
- luna_pinyin.sgmain
- luna_pinyin.i

#

- f_myphrases
- f_mysecretphrases

...
```

双拼用户需将 luna_pinyin.cn_en 禁用。禁用的方式很简单，在相应代码行前加上 **#** 将其注释掉即可。**当然，全拼用户请跳过这步。**这些词库，大家根据需要禁用或启用。这里我禁用了两个 emoji 词库。

**现在，来配置自定义短语**。在文件夹中，新建文本文档，更名为： Custom_phrase.txt。复制如下代码段到这个文档。 

```
# Rime table
# coding: utf-8
#@/db_name custom_phrase.txt
#@/db_type tabledb
#
# 用於【朙月拼音】系列輸入方案
# 【小狼毫】0.9.21 以上
#
# 請將該文件以UTF-8編碼保存爲
# Rime用戶文件夾/custom_phrase.txt
#
# 碼表各字段以製表符（Tab）分隔
# 順序爲：文字、編碼、權重（決定重碼的次序、可選）
#
# 雖然文本碼表編輯較爲方便，但不適合導入大量條目
#
# no comment
xxx@gmail.com  gmail   1
```

以第一条 gmail 为例，根据文字、編碼、權重的先后顺序，按照每行一条的格式，输入你的自定义短语。注意，各个字段之间以**制表符（tab）**分隔，**不是空格！** 

最后，在开始菜单【小狼毫】重新部署即可。 

### 现在开始配置 Rime 皮肤

打开 weasel.custom.yaml 文件，若没有，则新建。

复制如下代码段到该文件。

```
customization:
distribution_code_name: Weasel
distribution_version: 0.14.3
generator: "Weasel::UIStyleSettings"
modified_time: "Thu Jun 27 17:32:21 2019"
rime_version: 1.5.3

patch:
"style/display_tray_icon": true
"style/horizontal": true #横排显示
"style/font_face": "Microsoft YaHei" #字体
"style/font_point": 13 #字体大小
"style/inline_preedit": true # 嵌入式候选窗单行显示

"style/layout/border_width": 0
"style/layout/border": 0
"style/layout/margin_x": 12 #候选字左右边距
"style/layout/margin_y": 12 #候选字上下边距
"style/layout/hilite_padding": 12 #候选字背景色色块高度 若想候选字背景色块无边界填充候选框，仅需其高度和候选字上下边距一致即可
"style/layout/hilite_spacing": 3 # 序号和候选字之间的间隔
"style/layout/spacing": 10 #作用不明
"style/layout/candidate_spacing": 24 # 候选字间隔
"style/layout/round_corner": 0 #候选字背景色块圆角幅度

"style/color_scheme": Micosoft
"preset_color_schemes/Micosoft":
 name: "Micosoft"
 author: "XNOM"
 back_color: 0xffffff #候选框 背景色
 border_color: 0xD77800 #候选框 边框颜色
 text_color: 0x000000 #已选择字 文字颜色
 hilited_text_color: 0x000000 #已选择字右侧拼音 文字颜色
 hilited_back_color: 0xffffff #已选择字右侧拼音 背景色
 hilited_candidate_text_color: 0xffffff #候选字颜色
 hilited_candidate_back_color: 0xD77800 #候选字背景色
 candidate_text_color: 0x000000 #未候选字颜色
```

重新部署，查看效果。该配色方案是近乎完全还原 Windows10 微软输入法皮肤。效果如下。 **XNOM 首发，转载请注明来源。**

![](https://cdn.wallleap.cn/img/pic/illustration/202302201523177.png)

Rime 微软拼音配色方案

## 如何使用 RIME

到此， Rime 基本配置完毕。如果更多定制化要求，请自行搜寻相关教程。

切换为 Rime 输入法后，按下 Ctrl+` 或 F4 快捷键，调出方案选单，第一次选择：选择所需方案，第二次选择：选择简体或繁体、半角或全角、中文或英文、中文标点或英文标点输出，即可开始使用。

![](https://cdn.wallleap.cn/img/pic/illustration/202302201523178.png)

方案选单 1

![](https://cdn.wallleap.cn/img/pic/illustration/202302201523179.png)

方案选单 2

开始您的自由输入之旅吧。

## 如何同步个人词典和配置方案

Rime 没有云同步功能，但有本地同步功能，我们可以借助坚果云、 onedrive 等第三方云实现个人词典和配置方案在不同电脑间的同步和备份。以坚果云举例：

首先，在你的坚果云同步文件夹内，这里举例为 'D:\Nutstore‘，新建一个 RimeSync 文件夹。其他第三方同步云请自行同理修改。 

其次，先打开用户资料夹，打开 installation.yaml 文件，在最下方添加如下代码：

```
sync_dir: 'D:\Nutstore\RimeSync'
```

其中， installation_id 后的一长串字段，可以自行修改为喜欢的名称。这里举例为 XNOM。

最后完成的样子如下： 

```
distribution_code_name: Weasel
distribution_name: "小狼毫"
distribution_version: 0.14.3
install_time: "Wed Jul 10 15:57:26 2019"
installation_id: "XNOM"
rime_version: 1.5.3
sync_dir: 'D:\Nutstore\RimeSync'
```

第三，在开始菜单找到【小狼毫】用户资料同步，(也可以点击托盘图标，选择用户资料同步)，完成后，你就能在 RimeSync 文件夹中找到 XNOM 文件夹，其中的内容就是你的个人词典文件和配置文件。 

第四，以后若在另外的电脑上使用 Rime，则按照相同的步骤，将 RimeSync 的内容同步即可。 

**关于同步功能的注意：**

- Rime 的同步功能，在个人词典是双向同步，在个人配置是单项同步。怎么理解呢？
- 个人词典双向同步，举例来说，甲电脑个人词典累积了词汇 ABC，乙电脑累积了词汇 DEF，那么，通过第三方云同步和 Rime 同步后，个人词典词汇会双向同步合并为 ABCDEF。
- 配置单向同步，是指将配置文件，单向地从「用户文件夹」同步进入「同步文件夹」。这也是为了保持配置的一致性的必须方案。因为，若这两个文件夹中的配置不一致时，必然产生混乱。因而必须由用户手动。
- 所以，上述的第四步，是将云端同步好的个人词典文件同步，而个人配置文件，需要你手动复制粘贴进入「用户资料夹」。

## 一些没能实现的功能

在 Rime，利用反斜杆可以实现输入特殊符号、颜文字等等强大功能。\
例如：输入 /xl，能输出希腊字母。\

![](https://cdn.wallleap.cn/img/pic/illustration/202302201523180.jpeg)

反斜杠特殊符号

输入 /wz，能输出常用网址。\

![](https://cdn.wallleap.cn/img/pic/illustration/202302201523181.jpeg)

反斜杠网址

这个功能，用家可以选用明月拼音方案进行体验。但在双拼方案中，似乎无法实现码表的对应，以至于该功能缺失。目前我还没有找到好的方法能够在双拼方案中实现反斜杠特殊符号功能。在此抛砖引玉，或许有用家能够实现呢？

## 参考资料及扩展阅读

1. [中州韵输入法引擎帮助文档](https://sspai.com/link?target=https%3A%2F%2Frime.im%2Fdocs%2F)
2. [鼠须管输入法傻瓜配置 基于rime_pro](https://sspai.com/link?target=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000005754706)
3. [致第一次安装RIME的你](https://sspai.com/link?target=https%3A%2F%2Fwww.zybuluo.com%2Feternity%2Fnote%2F81763)
4. [Rime自定义符号](https://sspai.com/link?target=https%3A%2F%2Fblog.csdn.net%2Fu013410771%2Farticle%2Fdetails%2F79422576)
5. [推荐一个神级输入法RIME](https://sspai.com/link?target=https%3A%2F%2Fwww.byvoid.com%2Fzhs%2Fblog%2Frecommend-rime%2F)
6. [一位匠人的中州韵——专访Rime输入法作者佛振](https://sspai.com/link?target=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000002424698)
7. [最新版 Rime 输入法使用](https://sspai.com/link?target=https%3A%2F%2Fjdhao.github.io%2F2019%2F02%2F18%2Frime_configuration_intro%2F)

## 按键

```
BackSpace	退格
Tab	水平定位符
Linefeed	换行
Clear	清除
Return	回車
Pause	暫停
Sys_Req	印屏
Escape	退出
Delete	刪除
Home	原位
Left	左箭頭
Up	上箭頭
Right	右箭頭
Down	下箭頭
Prior、Page_Up	上翻
Next、Page_Down	下翻
End	末位
Begin	始位
Shift_L	左Shift
Shift_R	右Shift
Control_L	左Ctrl
Control_R	右Ctrl
Meta_L	左Meta
Meta_R	右Meta
Alt_L	左Alt
Alt_R	右Alt
Super_L	左Super
Super_R	右Super
Hyper_L	左Hyper
Hyper_R	右Hyper
Caps_Lock	大寫鎖
Shift_Lock	上檔鎖
Scroll_Lock	滾動鎖
Num_Lock	小鍵板鎖
Select	選定
Print	列印
Execute	執行
Insert	插入
Undo	還原
Redo	重做
Menu	菜單
Find	蒐尋
Cancel	取消
Help	幫助
Break	中斷
space
exclam	!
quotedbl	"
numbersign	#
dollar	$
percent	%
ampersand	&
apostrophe	'
parenleft	(
parenright	)
asterisk	*
plus	+
comma	,
minus	-
period	.
slash	/
colon	:
semicolon	;
less	<
equal	=
greater	>
question	?
at	@
bracketleft	[
backslash	
bracketright	]
asciicircum	^
underscore	_
grave	`
braceleft	{
bar	|
braceright	}
asciitilde	~
KP_Space	小鍵板空格
KP_Tab	小鍵板水平定位符
KP_Enter	小鍵板回車
KP_Delete	小鍵板刪除
KP_Home	小鍵板原位
KP_Left	小鍵板左箭頭
KP_Up	小鍵板上箭頭
KP_Right	小鍵板右箭頭
KP_Down	小鍵板下箭頭
KP_Prior、KP_Page_Up	小鍵板上翻
KP_Next、KP_Page_Down	小鍵板下翻
KP_End	小鍵板末位
KP_Begin	小鍵板始位
KP_Insert	小鍵板插入
KP_Equal	小鍵板等於
KP_Multiply	小鍵板乘號
KP_Add	小鍵板加號
KP_Subtract	小鍵板減號
KP_Divide	小鍵板除號
KP_Decimal	小鍵板小數點
KP_0	小鍵板0
KP_1	小鍵板1
KP_2	小鍵板2
KP_3	小鍵板3
KP_4	小鍵板4
KP_5	小鍵板5
KP_6	小鍵板6
KP_7	小鍵板7
KP_8	小鍵板8
KP_9	小鍵板9
```
