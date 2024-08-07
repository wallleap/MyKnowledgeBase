---
title: 排除已经提交到 Git 版本控制的文件
date: 2024-08-01 10:43
updated: 2024-08-01 10:44
---

## 方法一：使用 `.gitignore` 文件

1. **创建 `.gitignore` 文件：** 如果还没有 `.gitignore` 文件，可以在项目的根目录创建一个。这个文件用来列出你希望 Git 忽略的文件或目录。

2. **添加需要忽略的文件：** 在 `.gitignore` 文件中添加你想要排除的文件或目录。例如，如果你想忽略名为 `file.txt` 的文件，只需在 `.gitignore` 中添加一行：

		Copy Code
		
		`file.txt`
		
3. **更新索引并提交 `.gitignore` 文件：** 执行以下命令更新 Git 索引并提交 `.gitignore` 文件，确保忽略的文件不再被跟踪：

		bashCopy Code
		
		`git rm --cached file.txt git commit -m "Add file.txt to .gitignore and untrack it"`

## 方法二：使用 `git update-index`

如果你只想单独排除某个文件，而不是创建 `.gitignore` 文件来忽略多个文件，可以使用 `git update-index` 命令：

1. **停止跟踪文件：** 使用以下命令停止跟踪已经提交的文件，但保留在工作目录中：

		bashCopy Code
		
		`git rm --cached file.txt`
		
2. **更新索引并提交：** 更新 Git 索引以停止跟踪该文件，并提交更改：

		bashCopy Code
		
		`git commit -m "Stop tracking file.txt"`

这两种方法都能够有效地排除已经提交到 Git 的文件，具体选择取决于你是希望排除单个文件还是一组文件。
