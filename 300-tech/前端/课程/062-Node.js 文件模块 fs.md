---
title: 062-Node.js 文件模块 fs
date: 2024-04-30T03:05:00+08:00
updated: 2024-08-21T10:32:39+08:00
dg-publish: false
---

目标：一个基于文件的 Todo 工具

## 需求

功能：

- 可以列出所有的 todo
- 可以新增、编辑、删除 todo
- 可以标记 todo 为已完成/未完成

命令：

`t`、`t add 任务名`、`t clear`

## 创建项目

```sh
mkdir node-todo-demo
cd node-todo-demo
npm init -y
```

修改生成的 `package.json`

```json
{
  "name": "todo-demo",
  "version": "0.0.1",
  "main": "index.js",
  "license": "MIT"
}
```

新建 `index.js` 入口文件

```js
console.log('Hi, Node.js')
```

在命令行/终端运行 `node index`，将打印出 Hi, Node.js

## 添加库

[commander - npm (npmjs.com)](https://www.npmjs.com/package/commander) 完整的 node.js 命令行解决方案

安装：

```sh
npm install commander@3.0.2
```

修改 `index.js` 文件，先抄文档中的代码，再进行修改

```js
const program = require('commander');

program
  .option('-x, --xxx', 'what is x')
  .parse(process.argv)

console.log('执行了代码')
```

直接执行 `ndoe index` 会打印 `执行了代码`，运行 `node index -h` 则输出帮助命令

修改一下内容

```js
const program = require('commander');

program
  .option('-x, --xxx', 'what is x') // 选项
  .parse(process.argv)

console.log(program.xxx)
```

这个时候运行 `node index -x` 或 `node index --xxx` 则输出 `true`

```js
const program = require('commander');

program
  .option('-x, --xxx', 'what is x') // 选项
  .description('create a new project') // 描述命令

program
  .command('add <taskName>') // 子命令
  .description('Add a task')
  .action(taskName => { // 子命令相应动作
    console.log(taskName)
  })

program.parse(process.argv) // 解析参数
```

`add` 获取后面的内容

- 发现后面的都是 action 这个函数的参数
- 空格分开每个参数，需要获取所有的参数，使用 JS 语法 `...args`
- 发现最后一个是 `Comman` 对象，不需要，进行删除，之后拼接

```js
program
  .command('add')
  .description('Add a task')
  .action((...args) => {
    const words = args.slice(0, -1).join(' ');
    console.log(words)
  })
```

添加 `clear` 子命令

```js
program
  .command('clear')
  .description('Clear all tasks')
  .action(() => {
    console.log('clear')
  })
```

在实现功能之前，规范一下文件名

将 `index.js` 重命名为 `cli.js`，同时新建一个 `index.js`

## 实现 add 功能

所有的功能都放在 `index.js` 中，并且导出

```js
module.exports.add = (title) => {
  console.log(title)
}
```

在 `cli.js` 中导入并使用

```js
const { add } = require('./index.js')

program
  .command('add')
  .description('Add a task')
  .action((...args) => {
    const words = args.slice(0, -1).join(' ');
    add(words)
  })
```

获取用户 Home 目录

```js
const homedir = require('os').homedir()
```

可能用户设置了 Home 变量 `process.env.HOME`，所以

```js
const home = process.env.HOME || homedir
```

读取文件

```js
fs.readFile(
  目录, // 文件路径
  { 
    encoding: 'utf8', // 编码，默认为utf8，可省略
    flag: 'a+' // 读写模式，默认为只读 r，可省略，a+ 为创建并追加
  },
  callback // 回调函数
)
```

目录通过如下代码拼接

```js
const p = require('path')
const dbPath = p.join(home, '.todo')
```

写文件

```js
fs.writeFile(dbPath, JSON.stringify(list) + '\n', (err) => {
  if (err) {
    console.error(err)
  } else {
    console.log('添加成功')
  }
})
```

实现功能后的代码：

```js
const fs = require('fs')
const homedir = require('os').homedir()
const home = process.env.HOME || homedir
const p = require('path')
const dbPath = p.join(home, '.todo')

module.exports.add = (title) => {
  fs.readFile(dbPath, { encoding: 'utf8', flag: 'a+' }, (err, data) => {
    if (err) {
      console.error(err)
    } else {
      const list = data ? JSON.parse(data) : []
      const task = {
        title,
        completed: false
      }
      list.push(task)
      fs.writeFile(dbPath, JSON.stringify(list) + '\n', (err) => {
        if (err) {
          console.error(err)
        } else {
          console.log('添加成功')
        }
      })
    }
  })
}
```

封装优化代码：

优化后只应该有三行代码

```js
module.exports.add = (title) => {
  // 读取之前的任务
  const list = db.read()
  // 往里面添加一个任务
  list.push({
    title,
    completed: false
  })
  // 存储任务到文件
  db.write()
}
```

新建 `db.js`，实现功能

- 异步需要返回内容使用 Promise
- if-else 提前 return

```js
const fs = require('fs')
const homedir = require('os').homedir()
const home = process.env.HOME || homedir
const p = require('path')
const dbPath = p.join(home, '.todo')

const db = {
  read(path = dbPath) {
    return new Promise ((resolve, reject) => {
      fs.readFile(path, { encoding: 'utf8', flag: 'a+' }, (err, data) => {
        if (err) {
          console.error(err)
          return reject(err)
        }
        const list = data ? JSON.parse(data) : []
        resolve(list)
      })
    })
  },
  write(list, path = dbPath) {
    return new Promise((resolve, reject) => {
      fs.writeFile(path, JSON.stringify(list) + '\n', (err) => {
        if (err) {
          console.error(err)
          return reject(err)
        }
        console.log('添加成功')
        resolve()
      })
    })
  }
}

module.exports = db
```

这样使用的时候

```js
const db = require('./db')

module.exports.add = async (title) => {
  const list = await db.read()[

视频

实现创建功能



](https://xiedaimala.com/tasks/856af3c7-a715-4ac9-81c8-93c0cdb70a08/video_tutorials/3decbbcd-bdbd-4df8-8534-1d54f7e3fe52)
  list.push({
    title,
    completed: false
  })
  await db.write(list)
}
```

## 实现 clear 功能

```js
module.exports.clear = async (title) => {
  await db.write([])
}
```

`cli.js` 中

```js
program
  .command('clear')
  .description('Clear all tasks')
  .action(() => {
    clear()
  })
```

## 实现其他功能

没有参数的时候展示所有内容

```js
if (program.args.length === 0) {
  void api.showAll()
}
```

打印

```js
module.exports.showAll = async (title) => {
  const list = await db.read()
  list.forEach((task, index) => {
    console.log(`${task.completed ? '[x]' : '[ ]'} ${index + 1}. ${task.title}`)
  })
}
```

实现操作列表，使用库 [inquirer - npm (npmjs.com)](https://www.npmjs.com/package/inquirer)

```sh
npm i inquirer
```

基本用法

```js
inquirer.prompt({
    type: 'list',
    name: 'index',
    message: '选择要操作的任务',
    choices: [{name: '↗ 退出', value: '-1'}]
  }).then(answer => {
    const index = parseInt(answer.index)
  })
```

实现代码：

```js
const actions = {
  markAsDone(list, index) {
    list[index].completed = true
    db.write(list)
  },
  markAsUndone(list, index) {
    list[index].completed = false
    db.write(list)
  },
  remove(list, index) {
    list.splice(index, 1)
    db.write(list)
  },
  updateTitle(list, index) {
    inquirer.prompt({
      type: 'input', name: 'title', message: '新标题',
      default: list[index].title
    }).then(answer3 => {
      list[index].title = answer3.title
      db.write(list)
    })
  }
}

function askForAction(list, index) {
  inquirer.prompt({
    type: 'list', name: 'action', message: '选择操作', choices: [
      {value: 'quit', name: '↗ 退出'},
      {value: 'markAsDone', name: '已完成'},
      {value: 'markAsUndone', name: '未完成'},
      {value: 'updateTitle', name: '改标题'},
      {value: 'remove', name: '删除'},
    ]
  }).then(answer => {
    const action = actions[answer.action]
    action && action(list, index)
  })
}

function askForCreateTask(list) {
  inquirer.prompt({
    type: 'input', name: 'title', message: '任务名称'
  }).then(async answer => {
    const todo = {title: answer.title, completed: false}
    list.push(todo)
    await db.write(list)
  })
}

function printTasks(list) {
  const choices = [{name: '↗ 退出', value: '-1'}].concat(list.map((item, index) => {
    return {
      name: `${item.completed ? '[x]' : '[ ]'} ${index + 1} ${item.title}`,
      value: index.toString()
    }
  }), {name: '+ 创建任务', value: '-2'})

  inquirer.prompt({
    type: 'list',
    name: 'index',
    message: '选择要操作的任务',
    choices
  }).then(answer => {
    const index = parseInt(answer.index)
    if (index >= 0) {
      askForAction(list, index)
    } else if (index === -2) {
      askForCreateTask(list)
    }
  })
}

module.exports.showAll = async (title) => {
  const list = await db.read()
  printTasks(list)
}
```

## 发布到 npm

修改 `package.json`

```json
{
  "name": "node-todo-demo",
  "bin": {
    "t": "cli.js"
  },
  "files": ["*.js"],
  "version": "0.0.1",
  "main": "index.js",
  "license": "MIT",
  "dependencies": {
    "commander": "^3.0.2",
    "inquirer": "^7.0.0"
  }
}
```

- name 修改一下，不要和已有包同名
- version 每次发包之前需要修改版本号
- bin 最后生成的命令行是什么，例如叫 `t`，对应文件 `cli.js`
- files 声明有用的文件（发布的文件，`package.json` 默认会上传），可以一个个列出，这里所有 js 文件都需要上传

最好在 `cli.js` 文件最上面添加 shebang

```js
#!/usr/bin/env node
```

让 `cli.js` 为可执行文件

```sh
chmod +x cli.js
```

切换到官方源

```sh
nrm use npm
```

登录

```sh
npm adduser
# 输入用户名 密码 邮箱
```

发布

```sh
npm publish
```

会先把文件打包，然后上传发布

其他用户使用

```sh
npm install -g node-todo-demo
t
```

修改 `cli.js` 文件，读取版本

```js
const pkg = require('./package.json')

program
  .version(pkg.version)
```

可以使用 `node cli -V` 或 `node cli --version` 查看版本

修改版本号，重新发布，使用 `t` 查看版本

```sh
npm install -g node-todo-demo@latest
t --version
```
