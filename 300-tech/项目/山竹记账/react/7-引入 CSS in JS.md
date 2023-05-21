使用以下几种：
- CSS Modules
- styled components
- unocss/windicss/tailwind

## CSS Modules
由于使用的 vite，所以默认就可以使用 CSS Modules
这里来实现一下 welcome 页面样式
在 tsx 同级目录下创建，`文件名.module.scss` 
`Welcome1.tsx`：
```tsx
import s from './Welcome1.module.scss'
export const Welcome1: React.FC = () => {
	return (
		<div className={s.wrapper}>山竹记账</div>
	)
}
```

自动提示类名的插件可能是：`CSS Pick`
报错：没有 scss，直接安装
```zsh
npm install scss
```

类名会是 `_类名_随机字串`，并且只能用 class，hover 写法可以就用 CSS 写法或用 SCSS（`&:hover`）
多个类：
```tsx
<div className={[s.wrap, s.blue].join()}>山竹</div>
/* 或者用插件 */
import c from 'classnames'
……
<div className={(s.wrap, s.blue)}>山竹</div>
```

也可以直接加字符串，但是 module.scss 中不能直接给它加样式，可以：
```scss
:global(.noran) { /* 全局的就不会加随机串后缀 */
	background: rgba(222, 222, 11, .3);
}
```

##  styled components
主页：www. styled-components.com

安装：
```zsh
npm install styled-components@5.3.6
# ts
npm install --save-dev @type/styled-components@5.3.6
```

`Welcome2.tsx`：
```tsx
import styled from 'styled-components'
const Bordered = styled.div`
border: 1px solid red;
&:hover {
	background: rgba(222, 222, 11, .3);
}
`
export const Welcome2: React.FC = () => {
	return (
		<Bordered> Hi </Bordered>
	)
}
```

类名全随机，所以适合应用类开发
提示插件：vscode-styled-components
样式复用：
```tsx
const Box = styled(Box)`
	color: blue;
`

/* 甚至可以用组件复用 */
const Box = styled(Welcome2)`
	color: blue;
`
```

## unocss
仓库：

Tailwind.css——推出
```tsx
<div class="b-1 b-red text-xxl flex grid bg-red">
```

它这个是事先把这些类和样式写好

WindiCSS——优化
用到了才生成（JIT），Tailwind.css 随后也推出了这个

unocss——创新
antfu 也是 WindiCSS 的，Unocss 下个版本将作为 WindiCSS 的引擎

安装：
```zsh
# vite
npm i -D unocss
```

配置：
```ts
# vite.config.ts
import Unocss fom 'unocss/vite'
// 用了 React 应该把这个放在后面，只有用了插件才可以把 unocss 放前面
export default {
	plugin: 
}
```

或者加个文件 `uno.config.ts`

`main.tsx` 中引入空的 `uno.css` 文件
```tsx
// js
import 'uno.css'

// ts
import 
```

`Welcome3.tsx`
```tsx
import * as React from 'react'
export const Welcome3: React.FC = () => {
	return (
		<div flex>
			<header></header>
			<main></main>
			<footer></footer>
		</div>
	)
}
```

直接使用 `flex` 会飘红，需要加个文件 `src/shims.d.ts`
```ts
import type {} from ''

```

插件：colorize 可以把有颜色的展示出来
缩写参考 windi css 官网和 <https://unocss.antfu.me>
