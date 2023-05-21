---
title: RIME 小鹤双拼快速设置
date: 2023-02-20 15:24
updated: 2023-02-20 15:34
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - RIME 小鹤双拼快速设置
rating: 1
tags:
  - tool
category: tool
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: null
url: //myblog.wallleap.cn/post/1
---

如果你也使用 Linux 操作系统，那么 RIME 输入法基本是必备的。它开源、跨平台，支持各种输入方案；无服务器，不会泄漏你的输入数据。但同时相比各个商业公司的输入法方案来说，它的配置略显麻烦，没有图形界面，我曾经也一度不会配置；另外就是它默认词汇很少，也不会实时更新，这样会对我们的输入效率有一定影响。

不过不用担心，本篇文章中我们就来快速设置一个 RIME 的小鹤双拼输入方案。这里，我们先就 RIME 的配置文件做个简单的介绍，然后介绍一下如何快速配置极简的小鹤双拼，以及如何增加词库。

本文以 Linux 平台为例进行说明，其他平台大同小异，不做额外说明；如果你发现有啥叙述不恰当的，欢迎留言指正。

本文使用的 RIME 版本是 1.7.3 ， ibus-rime 版本 1.5.0

## 配置文件格式及惯例说明

RIME 的配置文件采用 [YAML](https://zh.m.wikipedia.org/zh/YAML) 格式，它的内容以“树状”形式来组织；当某个节点需要包含多个配置时，就需要多一级节点进行配置，语法上通过在当前节点末尾添加英文冒号 `:` 来表示多一层缩进。例如这样：

```yaml
schema_list: # 启用两个输入方案
  - schema: double_pinyin_flypy # 小鹤
  - schema: luna_pinyin         # 明月拼音
```


RIME 的配置虽然都是 YAML 格式，但是为了区分，它对文件名和内容有一些惯例，或者说约定。

它要求用户自定义的配置文件需要在扩展名之外添加 `.custom` ，即以 `.custom.yaml` 结尾。比如我们想更改默认配置文件 `default.yaml` ，那么我们应该修改 `default.custom.yaml` 这个文件，默认的配置文件不要去动它。

自定义配置除了文件名有差异之外，它的配置项也比默认的配置多一个层级，需要统一放在 `patch` 节点之下。比如默认的 `schema_list` 需要变成 `patch/schema_list` ，如下所示：

```yaml
patch:
  schema_list:
    - schema: double_pinyin_flypy # 只使用一个两个输入方案
```

RIME 支持很多的输入方案，每个方案有自己的 id ，也有自己的配置文件，文件名格式为 `$id.yaml` ；相应的自定义配置文件就是 `$id.custom.yaml` 。

每个输入方案与它的 id 对应如下（需要记住自己习惯的输入方案 id 在后面我们才知道应该动哪个配置文件）：

| 输入方案      | id                  |
| --------- | ------------------- |
| 朙月拼音      | luna_pinyin         |
| 朙月拼音·简化字  | luna_pinyin_simp    |
| 朙月拼音·臺灣正體 | luna_pinyin_tw      |
| 朙月拼音·語句流  | luna_pinyin_fluency |
| 自然碼雙拼     | double_pinyin       |
| 智能ABC雙拼   | double_pinyin_abc   |
| 小鶴雙拼      | double_pinyin_flypy |
| MSPY雙拼    | double_pinyin_mspy  |

另外，字典文件的结尾为 `xxx.dict.yaml` ，其中的 xxx 名字可以自己取，只要引用这个字典处的配置（ `"translator/dictionary"` 配置行）能够对应上即可。

Linux 上当前用户的 RIME 配置文件在目录 `~/.config/ibus/rime/` 中，为方便后文叙述，记此目录为 USER_CONFIG 。

目录中有如下文件和目录：

- `default.custom.yaml` 默认配置（入口配置）的自定义文件
- `ibus_rime.custom.yaml` Linux 平台上 ibus 的自定义配置，比如以横排还是竖排的形式展现输入候选词
- `double_pinyin_flypy.custom.yaml` 小鹤双拼这个输入方案的自定义配置
- `luna_pinyi.userdb` 用户使用过程中产生的词库？总之不需要动
- `installation.yaml` 安装信息，不需要动
- `user.yaml` 从内容看像是存放用户的最新部署时间等信息，总之也不需要动
- `build/` 目录，每次重新对 RIME 进行部署，就会更新这个目录。通常我们不动它。

总而言之，想要基于 RIME 快速配置一个可用的小鹤双拼输入法，需要改这 2 个配置文件： `default.custom.yaml` 和 `double_pinyin_flypy.custom.yaml` ，前者用于指定输入方案，后者用于定制小鹤双拼以及引入词典等。

## 指定使用小鹤双拼

`default.custom.yaml` 如下：

```yaml
patch:
  switcher:
    hotkeys:
      - "Control+grave"            # 输入选项菜单 Control + `

  menu:
    page_size: 7                   # 候选词数量

  schema_list:
    - schema: double_pinyin_flypy  # 使用小鹤双拼
```

## 定制小鹤双拼以及导入词典

`double_pinyin_flypy.custom.yaml`:

```yaml
patch:
  "translator/dictionary": luna_pinyin.extended # 使用额外的词库，对应的文件是 luna_pinyin.extended.dict.yaml
  switches:
    - name: ascii_mode
      states: ["中文", "西文"]
    - name: full_shape
      states: ["半角", "全角"]
    - name: simplification
      reset: 1                  # 默认简体
      states: ["漢字", "汉字"]
    - name: ascii_punct
      states: ["。，", "．，"]
```

### 导入词典

词典这块 GitHub 上已经有不少同学做了整理了，我目前在用的是 <https://github.com/rime-aca/dictionaries/tree/master/luna_pinyin.dict> 这个目录下的词典，把其中所有的 *.dict.yaml 文件丢到 USER_CONFIG 目录中即可。

需要注意的是，其中一个词典文件的名字需要与上述 YAML 配置中的 `"translator/dictionary"` 值一致才行。如果你从别的地方下载词典，需要确保他们一致才行。

## （可选）更改候选词展示

如果你和我一样，比较习惯横排候选词的方式，那么我们还需要更改下 `ibus_rime.custom.yaml` ，内容如下：

```yaml
patch:
  style:
    horizontal: true            #  是否横排展示候选词
```

## 重新部署

有了上述配置之后，我们就可以重新部署看看了：在系统托盘中左击 RIME 图标，在弹出菜单中选“部署”即可。
