---
title: faker.js 教程
date: 2024-08-26T10:50:12+08:00
updated: 2024-08-26T10:50:40+08:00
dg-publish: false
source: https://www.kancloud.cn/apachecn/zetcode-zh/1950573
---

faker.js 教程显示了如何使用 faker.js 库在 JavaScript 中生成伪造数据。

## faker.js

faker.js 是用于生成伪造数据的 JavaScript 库。 伪数据在构建和测试我们的应用时很有用。 faker.js 可以为各个领域生成伪造数据，包括地址，商业，公司，日期，财务，图像，随机数或名称。

在本教程中，我们在 Node 应用中使用 faker.js。

## 安装 faker.js

首先，我们安装 faker.js。

```js
$ node -v
v11.5.0

```

我们使用 Node 版本 11.5.0。

```js
$ npm init -y

```

我们启动一个新的 Node 应用。

```js
$ npm i faker

```

我们使用 `nmp i faker` 安装 faker.js 模块。

## 伪造名称

在第一个示例中，我们伪造了与用户名有关的数据。

`names.js`

```js
const faker = require('faker');

let firstName = faker.name.firstName();
let lastName = faker.name.lastName();

let jobTitle = faker.name.jobTitle();
let prefix = faker.name.prefix(); 
let suffix = faker.name.suffix();
let jobArea = faker.name.jobArea();

let phone = faker.phone.phoneNumber();

console.log(`Employee: ${prefix} ${firstName} ${lastName} ${suffix}`);
console.log(`Job title: ${jobTitle}`);
console.log(`Job area: ${jobArea}`);
console.log(`Phone: ${phone}`);

```

该示例创建一个随机的名字，姓氏，职务，姓名前缀和后缀，职务区域和电话号码。

```js
const faker = require('faker');

```

我们需要伪造者模块。

```js
let firstName = faker.name.firstName();

```

我们使用 `firstName()` 函数生成了虚假的名字。 该函数位于名称对象中。

```js
$ node names.js
Employee: Mrs. Vernice Abernathy III
Job title: Customer Data Associate
Job area: Program
Phone: 1-516-716-9832

```

这是一个示例输出。

## 伪造日期

在第二个示例中，我们生成假日期。

`dates.js`

```js
const faker = require('faker');

let futureDate = faker.date.future();
let recentDate = faker.date.recent();
let weekday = faker.date.weekday();

console.log(futureDate);
console.log(recentDate);
console.log(weekday);

```

该示例选择了将来和最近的日期以及某个工作日。

```js
$ node dates.js
2019-04-13T19:09:43.672Z
2019-01-06T12:28:29.089Z
Saturday

```

这是一个示例输出。

## 伪造随机值

伪造者允许生成随机值，例如整数，uuid 或单词。

`random_values.js`

```js
const faker = require('faker');

let number = faker.random.number();
console.log(number);

let uuid = faker.random.uuid();
console.log(uuid);

let word = faker.random.word();
console.log(word);

let words = faker.random.words(6);
console.log(words);

```

该示例生成随机数，uuid，单词和一组六个单词。

```js
$ node random_values.js
630
1470356a-1197-4955-b3c1-30302fd1db10
Facilitator
dot-com connect Practical Checking Account Mandatory real-time

```

这是一个示例输出。

## 伪造语言环境数据

伪造者在某种程度上支持本地化数据。 请注意，语言环境已完成各种级别。

`locale_faker.js`

```js
const faker = require('faker/locale/ru');

let firstName = faker.name.firstName();
let lastName = faker.name.lastName();

console.log(`Pаботник: ${firstName} ${lastName}`);

let month = faker.date.month();
let recentDate = faker.date.recent();
let rectentDate = faker.date.weekday();

console.log(month);
console.log(recentDate);
console.log(rectentDate);

```

该示例使用俄语生成伪造数据。

```js
$ node locale_faker.js
Pаботник: Антон Васильева
май
2019-01-06T19:24:01.283Z
Пятница

```

这是一个示例输出。

## 使用 JSON 服务器提供虚假数据

在下一个示例中，我们生成 JSON 数据并将其写入文件。 该文件由 JSON 服务器提供。

```js
$ npm i g json-server

```

我们安装 `json-server` 模块。

`generate_users.js`

```js
const faker = require('faker');
const fs = require('fs')

function generateUsers() {

  let users = []

  for (let id=1; id <= 100; id++) {

    let firstName = faker.name.firstName();
    let lastName = faker.name.lastName();
    let email = faker.internet.email();

    users.push({
        "id": id,
        "first_name": firstName,
        "last_name": lastName,
        "email": email
    });
  }

  return { "data": users }
}

let dataObj = generateUsers();

fs.writeFileSync('data.json', JSON.stringify(dataObj, null, '\t'));

```

该示例生成一百个用户，并将它们写入 `data.json` 文件。

```js
$ json-server --watch data.json --port 3004

```

我们启动 JSON 服务器。 服务器提供来自生成的 JSON 文件的数据。

```js
$ curl localhost:3004/data/3/
{
  "id": 3,
  "first_name": "Sheila",
  "last_name": "Bayer",
  "email": "Moshe.Walsh32@yahoo.com"
}

```

我们使用 `curl` 工具使用 ID 3 检索用户。

我们展示了如何使用 Node `request` 模块在 JavaScript 中生成 HTTP GET 请求。

```js
$ npm i request

```

我们安装模块。

`get_data.js`

```js
const request = require('request');

request('http://localhost:3004/data?_start=4&_end=8', (err, resp, body) => {

    if (err) {
        console.error('request failed');
        console.error(err);
    } else {
        console.log(body);
    }
});

```

该程序从 JSONServer 获取数据，数据从索引 4（不包括索引）开始，到索引 8（不包括索引）结束。

```js
$ node get_data.js
[
  {
    "id": 5,
    "first_name": "Cheyanne",
    "last_name": "Ernser",
    "email": "Amber.Spinka62@yahoo.com"
  },
  {
    "id": 6,
    "first_name": "Jeff",
    "last_name": "Bogisich",
    "email": "Bettie.Ritchie60@hotmail.com"
  },
  {
    "id": 7,
    "first_name": "Simone",
    "last_name": "Zemlak",
    "email": "Dorris49@gmail.com"
  },
  {
    "id": 8,
    "first_name": "Demond",
    "last_name": "Barrows",
    "email": "Nestor81@yahoo.com"
  }
]

```

这是一个示例输出。

## Faker CLI

`faker-cli` 是一个节点模块，是 `faker.js` 的包装。 它允许从命令行工具生成假数据。

```js
$ npm install -g faker-cli

```

我们安装 `faker-cli` 模块。

```js
$ faker-cli -d future
"2019-07-22T06:37:19.032Z"
$ faker-cli -d recent
"2019-01-06T21:46:50.788Z"
$ faker-cli -n firstName
"Tevin"
$ faker-cli -n lastName
"Sawayn"

```

我们使用 `faker-cli` 程序生成一些虚假数据。

在本教程中，我们使用了 faker.js 在 JavaScript 中生成假数据。

您可能也对以下相关教程感兴趣： [Moment.js 教程](https://www.kancloud.cn/javascript/momentjs/)， [JSONServer 教程](https://www.kancloud.cn/javascript/jsonserver/)，[从 JavaScript 中的 URL 读取 JSON](https://www.kancloud.cn/articles/javascriptjsonurl/) ， [JavaScript 贪食蛇教程](https://www.kancloud.cn/javascript/snake/) ， [JQuery 教程](https://www.kancloud.cn/web/jquery/)， [Node Sass 教程](https://www.kancloud.cn/javascript/nodesass/)， [Lodash 教程](https://www.kancloud.cn/javascript/lodash/)。
