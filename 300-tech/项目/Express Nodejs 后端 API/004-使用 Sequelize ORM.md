---
title: 004-使用 Sequelize ORM
date: 2025-03-13T14:05:29+08:00
updated: 2025-03-13T15:01:56+08:00
dg-publish: false
---

写 SQL 语句非常容易出错，并且需要记的东西有点多

在日常开发中，可以使用 ORM 来操作数据库

ORM 是在数据库与编程语言之间建立一种映射关系，这样可以用简单的代码来实现各种数据库的操作

## Sequelize ORM

<https://sequelize.org/>

> Sequelize is a modern TypeScript and Node.js ORM for Oracle, Postgres, MySQL, MariaDB, SQLite and SQL Server, and more. Featuring solid transaction support, relations, eager and lazy loading, read replication and more.

映射

- `Article.all()` → `SELECT * FROM Articles;`
	- Article 是模型，单数，对应的表是复数，会自动找到对应的表，`all()` 查询所有
- `Article.findByPk(3)` → `SELECT * FROM Articles WHERE id=3;`
	- Pk 就是 Primary Key，即主键的缩写，即通过 id 查找数据

## 安装

```sh
npm install -g sequelize-cli
npm install sequelize mysql2
```

可用命令

```sh
Sequelize CLI [Node: 10.21.0, CLI: 6.0.0, ORM: 6.1.0]

sequelize <command>

Commands:
  sequelize db:migrate                        Run pending migrations
  sequelize db:migrate:schema:timestamps:add  Update migration table to have timestamps
  sequelize db:migrate:status                 List the status of all migrations
  sequelize db:migrate:undo                   Reverts a migration
  sequelize db:migrate:undo:all               Revert all migrations ran
  sequelize db:seed                           Run specified seeder
  sequelize db:seed:undo                      Deletes data from the database
  sequelize db:seed:all                       Run every seeder
  sequelize db:seed:undo:all                  Deletes data from the database
  sequelize db:create                         Create database specified by configuration
  sequelize db:drop                           Drop database specified by configuration
  sequelize init                              Initializes project
  sequelize init:config                       Initializes configuration
  sequelize init:migrations                   Initializes migrations
  sequelize init:models                       Initializes models
  sequelize init:seeders                      Initializes seeders
  sequelize migration:generate                Generates a new migration file      [aliases: migration:create]
  sequelize model:generate                    Generates a model and its migration [aliases: model:create]
  sequelize seed:generate                     Generates a new seed file           [aliases: seed:create]

Options:
  --version  Show version number                                                  [boolean]
  --help     Show help                                                            [boolean]

Please specify a command
```

## 使用

接着运行

```sh
sequelize init
```

创建了

- `config/config.json` 连接到数据的配置文件
- `models/` 模型，每个模型文件对应数据库中的一张表
- `migrations/` 迁移，对数据库新增表、修改字段、删除表等操作在这添加迁移文件
- `seeders/` 种子，需要添加到数据表的测试数据存放在这

`config/config.json` 现在在开发环境，只关心开发环境的配置

```json
{
  "development": {
    "username": "root",
    "password": "express",
    "database": "express_api_development",
    "host": "127.0.0.1",
    "dialect": "mysql",
    "timezone": "+08:00"
  },
  "test": {
    "username": "root",
    "password": null,
    "database": "express_api_test",
    "host": "127.0.0.1",
    "dialect": "mysql"
  },
  "production": {
    "username": "root",
    "password": null,
    "database": "express_api_production",
    "host": "127.0.0.1",
    "dialect": "mysql"
  }
}
```

现在把之前创建了的表直接删除

```sql
DROP TABLE IF EXISTS Articles;
```

使用命令建表

```sh
sequelize model:generate --name Article --attributes title:string,content:text
```

创建了

- `models/article.js` 模型文件
- `migrations/20250313064038-create-article.js` 迁移文件

模型文件

```js
'use strict';
const {
  Model
} = require('sequelize');
module.exports = (sequelize, DataTypes) => {
  class Article extends Model {
    /**
     * Helper method for defining associations.
     * This method is not a part of Sequelize lifecycle.
     * The `models/index` file will call this method automatically.
     */
    static associate(models) {
      // define association here
    }
  }
  Article.init({
    title: DataTypes.STRING,  // 对应 varchar
    content: DataTypes.TEXT
  }, {
    sequelize,
    modelName: 'Article',
  });
  return Article;
};
```

迁移文件 创建修改表

```js
'use strict';
/** @type {import('sequelize-cli').Migration} */
module.exports = {
  async up(queryInterface, Sequelize) {
    await queryInterface.createTable('Articles', {  // 创建表
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER
      },
      title: {
        type: Sequelize.STRING,
        allowNull: false  // 加上不为空
      },
      content: {
        type: Sequelize.TEXT
      },
      // 自动加了 id 和新增和修改时间的
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE
      },
      updatedAt: {
        allowNull: false,
        type: Sequelize.DATE
      }
    });
  },
  async down(queryInterface, Sequelize) {
    await queryInterface.dropTable('Articles');
  }
};
```

执行迁移命令

```sh
sequelize db:migrate
```

`Articles` 表就创建成功了，并且同时创建了一个 `SequelizeMeta` 表，用于保存迁移文件数据，判断该迁移文件是否已经执行过

现在没有数据，可以使用种子文件创建一些测试数据

生成种子文件

```sh
sequelize seed:generate --name article
```

`seeders/20250313065256-article.js`

```js
'use strict';

/** @type {import('sequelize-cli').Migration} */
module.exports = {
  async up (queryInterface, Sequelize) {
    const articles = []
    const count = 100

    for (let i = 1; i <= count; i++) {
      const article = {
        title: `Article ${i}`,
        content: `Content ${i}`,
        createdAt: new Date(),
        updatedAt: new Date()
      }
      articles.push(article)
    }

    await queryInterface.bulkInsert('Articles', articles, {});
  },

  async down (queryInterface, Sequelize) {
    await queryInterface.bulkDelete('Articles', null, {});
  }
};
```

`up` 用来填充数据，`down` 则是反向操作，用来删除数据

接着运行命令 后面的是种子文件名

```sh
sequelize db:seed --seed 20250313065256-article
```

## 总结

1. 创建模型和迁移文件 `sequelize model:generate --name Article --attributes title:string,content:text`
2. 调整迁移文件
3. 运行迁移，生成数据表 `sequelize db:migrate`
4. 新建种子文件 `sequelize seed:generate --name article`
5. 修改代码，填充数据
6. 运行种子文件，把数据填充到数据表中 `sequelize db:seed --seed 种子文件名`
