---
title: GitHub Markdown 语法
date: 2024-08-26T11:19:14+08:00
updated: 2024-08-26T11:51:01+08:00
dg-publish: false
---

[Basic writing and formatting syntax - GitHub Docs](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)

## 标题 heading

```markdown
# A first-level heading
## A second-level heading
### A third-level heading
```

## 文本格式

| Style                  | Syntax             | Keyboard shortcut                         | Example                                  | Output                                 |
| ---------------------- | ------------------ | ----------------------------------------- | ---------------------------------------- | -------------------------------------- |
| Bold                   | `** **` or `__ __` | Command+B (Mac) or Ctrl+B (Windows/Linux) | `**This is bold text**`                  | **This is bold text**                  |
| Italic                 | `* *` or `_ _`     | Command+I (Mac) or Ctrl+I (Windows/Linux) | `_This text is italicized_`              | *This text is italicized*              |
| Strikethrough          | `~~ ~~`            | None                                      | `~~This was mistaken text~~`             | ~~This was mistaken text~~             |
| Bold and nested italic | `** **` and `_ _`  | None                                      | `**This text is _extremely_ important**` | **This text is *extremely* important** |
| All bold and italic    | `*** ***`          | None                                      | `***All this text is important***`       | ***All this text is important***       |
| Subscript              | `<sub> </sub>`     | None                                      | `This is a <sub>subscript</sub> text`    | This is a <sub>subscript</sub> text    |
| Superscript            | `<sup> </sup>`     | None                                      | `This is a <sup>superscript</sup> text`  | This is a <sup>superscript</sup> text  |

## 引用

```markdown
> Text that is a quote
```

> Text that is a quote

## 代码、代码块

press the Command+E (Mac) or Ctrl+E (Windows/Linux) keyboard shortcut to insert the backticks for a code block within a line of Markdown.

```markdown
Use `git status` to list all new or modified files that haven't yet been committed.
```

Use `git status` to list all new or modified files that haven't yet been committed.

````markdown
Some basic Git commands are:
```
git status
git add
git commit
```
````

Some basic Git commands are:

```
git status
git add
git commit
```

## 颜色

```markdown
The background color is `#ffffff` for light mode and `#000000` for dark mode.
```

The background color is `#ffffff` for light mode and `#000000` for dark mode.

| Color | Syntax             | Example                    | Output                                                                                                                                                                                                        |
| ----- | ------------------ | -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| HEX   | `` `#RRGGBB` ``    | `` `#0969DA` ``            | ![Screenshot of rendered GitHub Markdown showing how HEX value #0969DA appears with a blue circle.](https://docs.github.com/assets/cb-1558/images/help/writing/supported-color-models-hex-rendered.png)       |
| RGB   | `` `rgb(R,G,B)` `` | `` `rgb(9, 105, 218)` ``   | ![Screenshot of rendered GitHub Markdown showing how RGB value 9, 105, 218 appears with a blue circle.](https://docs.github.com/assets/cb-1962/images/help/writing/supported-color-models-rgb-rendered.png)   |
| HSL   | `` `hsl(H,S,L)` `` | `` `hsl(212, 92%, 45%)` `` | ![Screenshot of rendered GitHub Markdown showing how HSL value 212, 92%, 45% appears with a blue circle.](https://docs.github.com/assets/cb-2066/images/help/writing/supported-color-models-hsl-rendered.png) |

## 链接

This site was built using [GitHub Pages](https://pages.github.com/).

use the keyboard shortcut Command+Shift+V.

```markdown
This site was built using [GitHub Pages](https://pages.github.com/).
```

## 图片

```md
![](https://cdn.wallleap.cn/img/pic/illustration/202408261126737.svg?imageSlim)
```

![](https://cdn.wallleap.cn/img/pic/illustration/202408261126737.svg?imageSlim)

| Context                                                     | Relative Link                                                          |
| ----------------------------------------------------------- | ---------------------------------------------------------------------- |
| In a `.md` file on the same branch                          | `/assets/images/electrocat.png`                                        |
| In a `.md` file on another branch                           | `/../main/assets/images/electrocat.png`                                |
| In issues, pull requests and comments of the repository     | `../blob/main/assets/images/electrocat.png?raw=true`                   |
| In a `.md` file in another repository                       | `/../../../../github/docs/blob/main/assets/images/electrocat.png`      |
| In issues, pull requests and comments of another repository | `../../../github/docs/blob/main/assets/images/electrocat.png?raw=true` |

适配暗黑模式的图片

```html
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://user-images.githubusercontent.com/25423296/163456776-7f95b81a-f1ed-45f7-b7ab-8fa810d529fa.png">
  <source media="(prefers-color-scheme: light)" srcset="https://user-images.githubusercontent.com/25423296/163456779-a8556205-d0a5-45e2-ac17-42d089e3c3f8.png">
  <img alt="Shows an illustrated sun in light mode and a moon with stars in dark mode." src="https://user-images.githubusercontent.com/25423296/163456779-a8556205-d0a5-45e2-ac17-42d089e3c3f8.png">
</picture>
```

## 列表

无序列表

```markdown
- George Washington
* John Adams
+ Thomas Jefferson
```

有序列表

```markdown
1. James Madison
2. James Monroe
3. John Quincy Adams
```

缩进

```markdown
1. First list item
   - First nested list item
     - Second nested list item
```

任务列表

```markdown
- [x] #739
- [ ] https://github.com/octo-org/octo-repo/issues/740
- [ ] Add delight to the experience when all tasks are complete :tada:
```

If a task list item description begins with a parenthesis, you'll need to escape it with \:

`- [ ] \(Optional) Open a followup issue`

## 艾特某个人或团队

```md
@github/support What do you think about these updates?
```

## emoji

```md
:EMOJICODE:
```

## 段落

You can create a new paragraph by leaving a blank line between lines of text.

```md
文本

上面留空行
```

## 脚注

```text
Here is a simple footnote[^1].

A footnote can also have multiple lines[^2].

[^1]: My reference.
[^2]: To add line breaks within a footnote, prefix new lines with 2 spaces.
  This is a second line.
```

## Alerts

```markdown
> [!NOTE]
> Useful information that users should know, even when skimming content.

> [!TIP]
> Helpful advice for doing things better or more easily.

> [!IMPORTANT]
> Key information users need to know to achieve their goal.

> [!WARNING]
> Urgent info that needs immediate user attention to avoid problems.

> [!CAUTION]
> Advises about risks or negative outcomes of certain actions.
```

> [!NOTE]
> Useful information that users should know, even when skimming content.

> [!TIP]
> Helpful advice for doing things better or more easily.

> [!IMPORTANT]
> Key information users need to know to achieve their goal.

> [!WARNING]
> Urgent info that needs immediate user attention to avoid problems.

> [!CAUTION]
> Advises about risks or negative outcomes of certain actions.

## 注释

```md
<!-- This content will not appear in the rendered Markdown -->
```

## 转义

```md
Let's rename \*our-new-project\* to \*our-old-project\*.
```

Let's rename \*our-new-project\* to \*our-old-project\*.
