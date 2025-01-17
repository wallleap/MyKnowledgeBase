---
title: Git 忽略大小写以及正常大小写修改
date: 2024-08-01T10:41:00+08:00
updated: 2024-08-21T10:32:06+08:00
dg-publish: false
---

正常情况大小写如果提交了，需要修改则需要先重命名为其他，例如

`Work` 先修改成 `fuck` 再修改成 `word`

## 1. 配置 Git 的 core.ignorecase 选项

Git 默认是大小写敏感的，但可以通过配置来使其在对待文件名时忽略大小写。

1. **设置 core.ignorecase：**

		bashCopy Code
		
		`git config core.ignorecase true`
		
		这会告诉 Git 在文件名比较时忽略大小写。
		
2. **重新索引现有文件：** 如果你已经在仓库中有大小写不同但只有大小写区别的文件，可以使用以下命令来重新索引它们：

		bashCopy Code
		
		`git rm -r --cached . git reset --hard`
		
		这会删除当前的索引并重新建立，使新的 ignorecase 配置生效。

## 2. 使用 `.git/config` 文件手动配置

有时候，直接修改 `.git/config` 文件也是一个可行的方法。

1. **手动编辑 `.git/config` 文件：** 打开项目中的 `.git/config` 文件，找到 `[core]` 部分，并添加或修改以下行：

		Copy Code
		
		`[core]     ignorecase = true`
		
		这样做会手动设置 Git 忽略大小写。

## 注意事项

- **一致性问题：** 修改大小写敏感性设置后，确保团队中的所有成员都采用相同的设置，以避免不一致性问题。
- **已经跟踪的文件：** 如果已经在 Git 中跟踪了大小写不同的文件，并且想要忽略大小写，你可能需要先使用 `git rm --cached` 将这些文件从 Git 中删除（但保留在工作目录中），然后再提交这些更改。
- **跨平台注意事项：** 在 Windows 和 macOS/Linux 之间工作时，文件名大小写可能会有不同的行为。配置 Git 忽略大小写可以减少这些问题。

通过这些方法，你可以根据项目和团队的需求来设置 Git 的文件名大小写处理方式，以达到更好的版本控制和协作效果。
