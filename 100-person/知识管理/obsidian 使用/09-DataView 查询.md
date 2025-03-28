---
title: 09-DataView 查询
date: 2023-05-03T10:07:00+08:00
updated: 2024-08-21T10:58:35+08:00
dg-publish: false
---

安装并启用 DataView 插件

具体使用：

[Dataview (blacksmithgu.github.io)](https://blacksmithgu.github.io/obsidian-dataview/#/queries)

[[Obs#30] 更多的Dataview： 动态查询笔记资料 – 简睿随笔 (jdev.tw)](http://jdev.tw/blog/6610)

[210604_111749 dataview入门教程 - 飞书云文档 (feishu.cn)](https://c94n0azlfu.feishu.cn/docs/doccnTu0YTtoiC7C70HaUOyPBFh)

## 文件 Front-metter

在 Obsidian 资料库中的每个文件都应该在开头加上 Meta 信息，格式为 YAML 的

````YAML
---
title: 文件名称
date: {{date}}
tags: [标签1, 标签2, 标签3]
aliases:
- 文章别名1
- 文章别名2
rating: 5
---
````

之后可以利用 DataView 进行查询

## DataView 语法

````sql
```dataview
TABLE|LIST|TASK <field> [AS "Column Name"], <field>, ..., <field> FROM <source> (like #tag or "folder")
WHERE <expression> (like 'field = value')
SORT <expression> [ASC/DESC] (like 'field ASC')
... other data commands
```
````

关键词：

- `dataview` 语法
- 可选显示格式为：表格 `table`、列表 `list`、任务列表 `task`
- `field`
- `from` 可以从 `#tag`、`"文件夹"`、`[[双链]]` 中进行查询
- `where` 条件
- 其他命令参数
	- `sort by` 根据字段进行排序
	- `group by` 分组

例如：

**查询**

除了可以根据 metaData 进行查询还可以使用内联字段（`字段名::值`）：这两个可以直接使用

````sql
---
date: 2022-11-11
rating: 5
---

#每天

起床:: 8:30am

```dataview
table 起床, date, rating from #每天
```
````

以及页面自己生成的原数据：这个就是下面这些

![](https://cdn.wallleap.cn/img/pic/illustrtion/202211231100529.png)

dataview 能自动的对每个页面添加大量的元数据。

- `file.name`: 该文件标题 (字符串)。
- `file.folder`: 该文件所在的文件夹的路径 (字符串)。
- `file.path`: 该文件的完整路径 (字符串)。
- `file.link`: 该文件的一个链接 (链接)。
- `file.size`: 该文件的大小 (bytes)(数字)
- `file.ctime`: 该文件的创建日期 (日期和时间)。
- `file.cday`: 该文件的创建日期 (仅日期)。
- `file.mtime`: 该文件最后编辑日期 (日期和时间)。
- `file.mday`: 该文件最后编辑日期 (仅日期)。
- `file.tags`: 笔记中所有标签组成的数组。子标签按每个级别进行细分，所以 `#Tag/1/A` 将会在数组中储存为 `[#Tag, #Tag/1, #Tag/1/A]`。
- `file.etags`: 笔记中所有显式标签组成的数组；不同于 `file.tags`，不包含子标签。
- `file.outlinks`: 该文件所有外链 (outgoing link) 组成的数组。
- `file.aliases`: 笔记中所有别名组成的数组。

如果文件的标题内有一个日期（格式为 yyyy-mm-dd 或 yyyymmdd），或者有一个 Date 字段/inline 字段，它也有以下属性:

- `file.day`: 一个该文件的隐含日期

````sql
```dataview
list file.tags as "标签"
from "文件夹"
```
````

## Obsidian 文件属性
