---
title: SQLite
date: 2024-01-29 10:37
updated: 2024-01-29 11:24
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - SQLite
rating: 1
tags:
  - SQLite
category: 数据库
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: 
url: //myblog.wallleap.cn/post/1
---

SQLite 是一种轻量级的嵌入式关系型数据库管理系统，可以在大多数操作系统上运行。

在线运行：[SQL OnLine IDE](https://sqliteonline.com/)

## 使用 SQLite 的内置命令

使用 SQLite 提供的命令行工具或编程语言中的相关库，连接到 SQLite 数据库文件。例如，在命令行中输入 `sqlite3` 命令可以连接到一个 SQLite 数据库。

```sh
sqlite3
# 创建数据库
sqlite3 <database_name>.db
```

接着可以使用各种命令，如

```sqlite
# 显示帮助
.help
# 打开数据库文件 如 .open notes.db
.open FILENAME 
# 显示当前连接的所有数据库
.databases
# 显示当前数据库中的所有表
.tables
```

## CRUD

### 创建表

使用 SQL 语句创建表格以存储数据。例如，使用以下 SQL 语句创建一个名为 "users" 的表格（不要漏掉末尾分号）：

```sql
CREATE TABLE users (
  id INTEGER PRIMARY KEY,
  name TEXT,
  age INTEGER
);
```

结构为 列名 数据类型 约束条件

在 SQLite 中，通常推荐使用具有以下特点的**列名**：

1. 有描述性：列名应该能清楚地描述所存储的数据的含义，使其他人易于理解和使用。
2. 简洁明了：列名应该尽量简洁明了，避免过长或复杂的名称，提高可读性。
3. 使用下划线或小写字母：SQLite 不区分大小写，但约定俗成地使用小写字母和下划线作为列名的分隔符，以增加可读性。
4. 避免关键字和保留字：避免使用 SQLite 的关键字和保留字作为列名，以免引起命名冲突（SELECT、FROM、WHERE、ORDER BY、GROUP BY、JOIN、UPDATE、INSERT、DELETE、CREATE、DROP、ALTER、INDEX、UNION、AND、OR、NOT、EXISTS、NULL、INTEGER、REAL、TEXT、BLOB、PRIMARY KEY、UNIQUE、NOT NULL、CHECK、FOREIGN KEY）。

举例来说，如果创建一个用于存储用户信息的表，可以使用类似以下的列名：

- id: 用户唯一标识符
- username: 用户名
- email: 邮箱地址
- age: 年龄
- created_at: 创建时间

另外，在使用列名时，还应避免使用过于模糊或不易理解的缩写，尽量保持一致性和规范性，以提高代码的可维护性和可读性。

总之，选择一个具有描述性、简洁明了、遵循命名规范的列名是良好的实践。

SQLite 中常用的**数据类型**包括：

1. INTEGER: 用于存储整数值，可以是 1、2、3、4、6 或 8 字节大小的有符号整数。
2. REAL: 用于存储浮点数值，即带有小数部分的数字。
3. TEXT: 用于存储字符串文本，可以存储任意长度的文本数据。
4. BLOB: 用于存储二进制数据，如图片、音频、视频等非文本数据。
5. NULL: 表示一个空值。

这些数据类型可以满足大多数的数据存储需求，你可以根据具体情况选择合适的数据类型来定义表结构中的列。

在 SQLite 中，常见的**约束条件**包括：

1. PRIMARY KEY: 用于指定一个列作为主键，主键的值在表中必须唯一且不能为空。
2. UNIQUE: 用于确保列中的值在表中是唯一的。
3. NOT NULL: 用于确保列中的值不为空。
4. CHECK: 用于定义列中的值必须满足的条件，可以使用表达式或函数进行自定义约束。
5. FOREIGN KEY: 用于定义与其他表的关联关系，确保引用完整性。需要注意的是，默认情况下 SQLite 不会强制执行外键约束，但可以通过启用外键支持来实现。

这些约束条件可以在创建表时通过列定义来应用，确保数据的完整性和一致性。

### 插入数据

使用 INSERT INTO 语句将数据插入表格中。例如，使用以下 SQL 语句将一条用户记录插入到 "users" 表格中：

```sql
INSERT INTO users (name, age) VALUES ('John', 25);
```

### 查询数据

使用 SELECT 语句从表格中检索数据。

例如，使用以下 SQL 语句查询 "users" 表格中的所有记录：

```sql
SELECT * FROM users;
```

**查询所有数据**，将检索表中所有的数据记录

```sql
SELECT * FROM 表名;
```

**查询特定列的数据**，将列出所需的列名，以逗号分隔

```sql
SELECT 列1, 列2 FROM 表名;
```

**使用条件查询**，可以在 WHERE 子句中使用条件来筛选满足特定条件的数据记录

```sql
SELECT * FROM 表名 WHERE 条件;
```

**使用排序**，可以使用 ORDER BY 子句按特定列的升序（ASC）或降序（DESC）对结果进行排序

```sql
SELECT * FROM 表名 ORDER BY 列名 ASC/DESC;
```

### 更新数据

使用 UPDATE 语句更新表格中的数据。例如，使用以下 SQL 语句将 "users" 表格中 id 为 1 的记录的年龄更新为 30：

```sql
UPDATE users SET age = 30 WHERE id = 1;
```

### 删除数据

使用 DELETE FROM 语句从表格中删除数据。例如，使用以下 SQL 语句删除 "users" 表格中 id 为 1 的记录：

```sql
DELETE FROM users WHERE id = 1;
```
