---
title: 028-使用 Prettier 格式化代码
date: 2025-03-27T15:28:11+08:00
updated: 2025-03-27T15:37:07+08:00
dg-publish: false
---

安装

```sh
npm install prettier -D
```

`.prettierrc.js`

```js
module.exports = {
  printWidth: 100, // 每行的最大字符数，超过会自动换行
  singleQuote: true, // 使用单引号代替双引号
  trailingComma: 'es5', // 在 ES5 中有效的地方添加尾随逗号（对象、数组等）
  useTabs: false, // 使用空格代替制表符
  tabWidth: 2, // 每个缩进级别使用的空格数
  semi: false, // 在语句末尾添加分号
  bracketSpacing: true, // 在对象文字中的括号之间添加空格
  arrowParens: 'always', // 箭头函数的参数始终使用括号
  endOfLine: 'lf', // 使用换行符 `\n` 作为行结束符
  quoteProps: 'as-needed', // 仅在需要时为对象的属性添加引号
}
```

`package.json` 的 scripts 中添加

```json
"format:check": "prettier --check \"**/*.{js,json,md}\"",
"format": "prettier --write \"**/*.{js,json,md}\""
```

然后执行 `npm run format:check` 检查、`npm run format` 格式化
