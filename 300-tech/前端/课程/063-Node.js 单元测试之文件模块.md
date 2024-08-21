---
title: 063-Node.js 单元测试之文件模块
date: 2024-04-30T07:56:00+08:00
updated: 2024-08-21T10:32:39+08:00
dg-publish: false
---

测试工具：jest

```sh
npm install -D jest
```

## 基础使用

例如有一个文件 `sum.js`

```js
function sum(a, b) {  
return a + b;  
}  
module.exports = sum;
```

那么可以新建一个 `sum.test.js`

```js
const sum = require('./sum');  
  
test('adds 1 + 2 to equal 3', () => {  
  expect(sum(1, 2)).toBe(3);  
});
```

接着在 `package.json` 中添加一个脚本

```json
{  
  "scripts": {  
    "test": "jest"  
  }  
}
```

现在运行 `npm run test` 就会进行测试

## 测试文件读写

上个项目只需要测试 `db.js` 即可

新建 `__test__/db.spec.js`，由于会操作 fs，但有个原则，最好不改变现有环境，所以需要进行 mock，新建 `__mocks__/fs.js`

```js
// fs.js
const fs = jest.genMockFromModule('fs')
const _fs = jest.requireActual('fs')

Object.assign(fs, _fs)

let readMocks = {}

fs.setReadMock = (path, error, data) => {
  readMocks[path] = [error, data]
}

fs.readFile = (path, options, callback) => {
  if (callback === undefined)
    callback = options
  if (path in readMocks) {
    callback(...readMocks[path])
  } else {
    _fs.readFile(path, options, callback)
  }
}

let writeMocks = {}

fs.setWriteMock = (file, fn) => {
  writeMocks[file] = fn
}

fs.writeFile = (file, data, options, callback) => {
  if (file in writeMocks) {
    writeMocks[file](file, data, options, callback)
  } else {
    _fs.writeFile(file, data, options, callback)
  }
}

fs.clearMocks = () => {
  readMocks = {}
  writeMocks = {}
}

module.exports = fs
```

进行测试

```js
// db.spec.js
const { afterEach } = require("node:test")
const db = require("../db")
const fs = require("fs")
jest.mock('fs')

describe("db", () => {
  afterEach(() => {
    fs.clearMocks()
  })
  it("can read", async () => {
    const data = [{title: 'hi', done: true}]
    fs.setReadMock('/xxx', null, JSON.stringify(data))
    const list = await db.read('/xxx')
    expect(list).toStrictEqual(data)
  })
  it("can write", async () => {
    let fakeFile
    fs.setWriteMock('/yyy', (file, data, callback) => {
      fakeFile = data
      callback(null)
    })
    const list = [{title: 'hi', done: true}]
    await db.write(list, '/yyy')
    expect(fakeFile).toBe(JSON.stringify(list) + '\n')
  })
})
```

测试步骤：

1. 选择测试工具 jest
2. 设计测试用例
3. 写测试，运行测试，改代码
4. 单元测试，功能测试，集成测试

重点：

- 单元测试不应该与外部打交道（那是集成测试要做的）
- 单元测试的对象是函数
- 功能测试的对象是模块
- 继承测试的对象是系统
