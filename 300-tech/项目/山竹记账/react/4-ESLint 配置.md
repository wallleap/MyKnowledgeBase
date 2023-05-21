可以直接使用 antfu 的 eslint -config
地址：<https://github.com/antfu/eslint-config>

项目中安装
```zsh
# 局部安装
npm install -D eslint @antfu/eslint-config
```

添加配置文件 `.eslintrc`，输入一下内容，保存
```json
{
  "extends": "@antfu"
}
```

配置 VSCode 自动修复
创建文件 `.vscode/settings.json`

```json
{
  "prettier.enable": false,
  "editor.formatOnSave": false,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  }
}
```

不要用 [Prettier](https://antfu.me/posts/why-not-prettier).

在 VSCode 中安装 ESLint 插件

<kbd>Ctrl</kbd> + <kbd>S</kbd> 自动格式化，每次保存会自动修复代码毛边

之前可以直接用 `console.log()`，先在会飘红，可以加 `window`

```tsx
window.console.log(a)
```

浮上去，然后选择这行/这个文件不使用 ESLint
需要关闭可以到配置文件中加上，其他不需要的也像这样加到这里设置为 `off`
```
{
	"extends": "@antfu",
	"rules": {
		"@typescript-eslint/no-unused-vars": "off"
	}
}
```

如果没有安装插件，需要命令行使用和修复
`package.json` 中加入
```json
{
  "scripts": {
    "lint": "eslint .",
    "lint:fix": "eslint . --fix"
  }
}
```

在终端执行
```zsh
npm run lint
npm run lint:fix
```