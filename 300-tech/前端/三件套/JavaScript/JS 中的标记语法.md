---
title: JS 中的标记语法
date: 2024-09-13T01:44:26+08:00
updated: 2024-09-13T01:44:40+08:00
dg-publish: false
---

## 循环

给循环起个名字，能 `break` 它

```js
out: for...
break out
```

## 标记模版字符串

```js
const name = 'TOM'
const hi = tag`Hi ${name}, I'm xxx.`
function tag() {
  console.log(arguments)  // [ 'Hi ', ', I'm xxx.', 'TOM' ]]
  let result = ''
  for (let i = 1; i <= arguments.length; i++) {
    const first = arguments[0]
    result += first[i - 1]
    if (i < arguments.length) {
      result += arguments[i]
    }
  }
  return result  // 还可以 return 其他任意东西，例如函数 return tag
}
console.log(hi)
```

在模版字符串前面加了个标记，相当于调用了这个函数，并传了参数，返回一个结果

里面打印参数可以发现，会根据变量进行拆分，纯字符串放到第一个数组中，变量依次往后面添加

**命令式编程变成声明式编程**，有点像 styled-components

从 `el.style.xxx=xxx` 变成

```js
const a = document.createElement('a')
document.body.appendChild(a)

const github = {
  title() {
    return Math.random().toString(36).substring(7)
  },
  link: 'https://github.com/jaredpalmer/razzle',
  target: '_blank',
  rel: 'noopener noreferrer'
}
const style = {
  getRandomColor() {
    return '#' + Math.random().toString(16).slice(2, 8)
  },
  padding: 10,
  borderRadius: 16
}
a.styles `
  color: ${style.getRandomColor()};
  font-size: 2em;
  text-align: center;
  background-color: ${style.getRandomColor()};
  border-bottom: 2px solid #000;
  padding: ${style.padding}px ${style.padding * 2}px;
  margin: 0;
  text-decoration: none;
  border-radius: ${style.borderRadius}px;
  font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;
`.props `
  href: ${github.link}
  target: ${github.target}
  rel: ${github.rel}
`.content `Link to ${github.title()}`
```

现在来实现它

```js
!(() => {
  function generateString(args) {
    let result = ''
    for (let i = 1; i <= args.length; i++) {
      const first = args[0]
      result += first[i - 1]
      if (i < args.length) {
        result += args[i]
      }
    }
    return result
  }

  HTMLElement.prototype.styles = function () {
    const styles = generateString(arguments)
    let curStyle = this.getAttribute('style')
    if (curStyle) {
      curStyle += styles
    } else {
      curStyle = styles
    }
    this.style = curStyle
    return this
  }
  HTMLElement.prototype.props = function () {
    const propString = generateString(arguments)
    propString
      .split('\n')
      .map(it => {
        const parts = it.trim().split(':')
        const key = parts[0].trim()
        let value = parts.splice(1).join(':').trim()
        if (value.indexOf(';') === value.length - 1) {
          value = value.substring(0, value.length - 1)
        }
        return [key, value]
      })
      .forEach(([k, v]) => {
        if (!k) {
          return
        }
        this[k] = v
      })
    return this
  }
  HTMLElement.prototype.content = function () {
    const text = generateString(arguments)
    this.textContent = text
    return this
  }
})()
```


