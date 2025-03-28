---
title: 003-使用 Docker 运行 MySQL 创建数据库和常用语句
date: 2025-03-13T11:14:54+08:00
updated: 2025-03-13T14:06:06+08:00
dg-publish: false
---

数据库 > 表 > 行 > 列 / 字段 ← 数据类型 值

## 数据库

存储各种数据的

常用数据库：MySQL、Postgres、MyOracel、SQL Server 等

数据库的常用操作：增删改查（Create、Delete、Update、Read）CURD

<https://mysql.com>

## 下载 Docker 客户端

- Windows 下载 Docker Desktop：<https://docker.com>
- macOS 下载 OrbStack：<https://orbstack.dev/>

Docker Desktop 可以添加阿里云镜像 Docker Engine

```json
{
  ...
	"registry-mirrors": [
		"https://xelrug2w.mirror.aliyuncs.com"
	]
}
```

## 创建 Docker Compose 运行 MySQL

在项目根目录新建 `docker-compose.yml` 文件

```yml
version: '3.8'

services:
  mysql:
    image: mysql:8.3.0
    # 下面这行是为了解决mysql8.0.23版本之后，默认的编码为utf8mb4，而express-admin-ui中默认使用的是utf8编码，所以需要修改为utf8mb4编码
    command: mysqld --character-set-server=utf8mb4 --collation-server=utf8mb4_general_ci
    environment:
      - MYSQL_ROOT_PASSWORD=express  # 默认root用户密码
      - MYSQL_LOWER_CASE_TABLE_NAMES=0  # 默认表名不区分大小写，设置为0，则区分大小写
    ports:
      - "12306:3306"  # 映射端口，前面端口是宿主机端口，后面端口是容器端口
    volumes:
        - ./data/mysql:/var/lib/mysql  # 数据持久化
```

接着运行命令

```sh
docker-compose up -d
```

如果网络访问不了，可以把 image 修改一下 `hub.vonne.me/image: mysql:8.3.0`

目录权限

```sh
sudo chmod -R 777 ./data/mysql
```

## MySQL 客户端

macOS 端可以下载 `Sequel Ace`

新建数据库连接

- Host 127.0.0.1
- Username root
- Password express
- Port 12306

Windows 下载 `Navicat Premium`

或者使用 VSCode 插件 Database

直接连接到 Docker 终端，然后使用命令

```sh
mysql -u root -p
```

之后输入密码，然后就能执行各种 SQL 语句了

退出的时候可以使用 `EXIT;` 或 `QUIT;` 或按 <kbd>Ctrl</kbd> + <kbd>d</kbd>

## 创建数据库与数据表

使用 SQL 语句创建数据库

```sql
CREATE DATABASE IF NOT EXISTS express_api_development
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_general_ci;
```

其中 `express_api_development` 为数据库名，编码为 `utf8mb4`，mb4 可以正确存储 emoji，字符集 `utf8mb4_general_ci` 不区分大小写和重音符号

在客户端中 Query 输入 SQL 语句然后点击执行

选中新创建的数据库，或使用 SQL 语句

查看已有数据库

```sql
SHOW DATABASES;
```

使用数据库

```sql
USE express_api_development;
```

在这个数据库中创建 Articels 表

```sql
CREATE TABLE Articles (
    id INT AUTO_INCREMENT PRIMARY KEY NOT NULL,
    title VARCHAR(255) NOT NULL,
    content TEXT,
    `create` DATETIME
);
```

- id INT 类型，主键，自增，不允许为空
- title
- content

由于用了 `create` 关键字，现在需要更改成 `create_at`，但由于之前创建了，可能会已经使用了 `create` 字段，所以新增字段并更新

```sql
ALTER TABLE Articles
ADD COLUMN created_at DATETIME;

UPDATE Articles
SET created_at = `create`;

-- 从 Articels 表中删除 create 字段（如果需要）
ALTER TABLE Articles
DROP COLUMN `create`;
```

查看已有表

```sql
SHOW TABLES;
```

删除数据表命令

```sql
DROP TABLE IF EXISTS Articles;
```

## 常用数据类型

整数

|     类型      | 字节大小 | 有符号范围（Signed）          | 无符号范围（Unsigned）  |
| :----------- | ---- | ---------------------- | ---------------- |
|   TINYINT   | 1    | -128~127               | 0~255            |
|  SMALLINT   | 2    | -32768~32767           | 0~65535          |
|  MEDIUMINT  | 3    | -8388608~8388607       | 0~16777215       |
| INT/INTEGER | 4    | -2147483648~2147483647 | 0~4294967295     |
|   BIGINT    | 8    | -9223372036854775808   | 0~18446744073709 |

字符串

| 类型       | 说明               | 使用场景                    |
| :------- | ---------------- | ----------------------- |
| CHAR     | 固定长度，小型固定长度的数据   | 身份证号、手机号、电话、密码          |
| VARCHAR  | 可变长度，小型数据        | 姓名、地址、品牌、型号、用户的评论、文章的标题 |
| TEXT     | 可变长度，字符个数大于 4000 | 存储文章正文                  |
| LONGTEXT | 可变长度，超大型文本数据     | 存储超大型文本数据               |

时间

| 类型        | 字节大小 | 示例                                                    |
| :-------- | ---- | ----------------------------------------------------- |
| DATE      | 4    | '2020-01-01'                                          |
| TIME      | 3    | '12:29:59'                                            |
| DATETIME  | 8    | ''2020-01-01 12:29:59''                               |
| YEAR      | 1    | '2017'                                                |
| TIMESTAMP | 4    | '1970-01-01 00:00:01' UTC ~ '2038-01-01 00:00:01' UTC |

最常用的数据类型

| 类型                 | 含义  | 说明                                              |
| :----------------- | --- | ----------------------------------------------- |
| int                | 整数  | 需要设定长度                                          |
| decimal            | 小数  | 金额常用，需要设定长度，如 decimal(10, 2) 表示共存 10 位数，其中小数占 2 位 |
| char、varchar       | 字符串 | 文字类的常用，需要设定长度                                   |
| text               | 文本  | 存储大文本，无需设定长度                                    |
| date、time、datetime | 日期  | 记录时间                                            |

## 常用 SQL 语句

### 增加

插入数据，id 不需要手动写，会自增

```sql
INSERT INTO Articles (title, content, create_at)
VALUES ('test', 'test content', '2025-03-13 13:39:57');
```

多条数据

```sql
INSERT INTO Articles (title, content, create_at)
VALUES ('将敬酒', '天生我材必有用，千金散尽还复来', '2025-03-13 13:40:51'),
	('行路难', '长风破浪会有时，直挂云帆济沧海', '2025-03-13 13:42:07'),
	('望庐山瀑布', '飞流直下三千尺，疑是银河落九天', '2025-03-13 13:43:00');
```

### 修改

更新数据

```sql
UPDATE Articles SET title='黄鹤楼送孟浩然之广陵', content='故人西辞黄鹤楼，烟花三月下扬州' WHERE id=1;
```

注意需要加 `where` 条件

### 删除

删除数据

```sql
DELETE FROM Articles WHERE id=2;
```

### 查询

查询 `*` 全部

```sql
SELECT * FROM Articles;
```

查询需要的字段

```sql
SELECT id, title FROM Articles;
```

条件查询

```sql
SELECT * FROM Articles WHERE id=3;

-- 查询 id 大于 3 的
SELECT * FROM Articles WHERE id>3;
```

排序

- ASC 正序，从小到大
- DESC 倒序/降序，从大到小

```sql
SELECT id, title FROM Articles ORDER BY id DESC;
```

控制返回行数

```sql
-- 返回前 10 个
SELECT * FROM Articles LIMIT 10;

-- 返回从第 30 行开始的 5 个
SELECT * FROM Articles LIMIT 30, 5;
```
