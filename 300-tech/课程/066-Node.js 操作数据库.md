---
title: 066-Node.js 操作数据库
date: 2024-05-07 10:52
updated: 2024-05-07 16:07
---

## 用 Docker 安装数据库

### 安装 Docker

<https://hub.docker.com> 注册，下载安装包，安装

```sh
docker run heallo-world
docker ps
docker kill <id>
docker rm <names>
docker system prune # 清空
```

### Docker 安装 MySQL

进入 Docker 上 MySQL 的主页 [mysql - Official Image | Docker Hub](https://hub.docker.com/_/mysql)

- 选择版本
- 使用 docker run 命令启动容器
- name 是容器的名字
- MYSQL_ROOT_PASSWORD 后是密码
- tag 是版本号
- 再加一个端口映射 `-p 3306:3306`
- `-d` 可以让它在后台运行（守护进程模式）

```sh
docker run --name mysql1 -e MYSQL_ROOT_PASSWORD=123456 -p 3306:3306 -d mysql:5.7.28
```

命令

- `docker ps` 查看容器运行状态
- `docker kill mysql1` 关掉容器
- `docker container start mysql1` 开启关闭的容器
- `docker rm mysql1` 删掉容器（可以加 `-f` 选项）
- `docker run` 启动新容器

用 Docker 运行的容器，默认不会持久化，也就是说如果容器被删掉了，那么数据也没了，如果需要持久化可以搜索“docker mysql 数据目录”

使用下面命令进入容器

```sh
docker exec -it mysql1 bash
```

容器里有一个 Linux 系统，然后就可以在这个系统里运行 MySQL

连接 MySQL

```sh
mysql -u root -p
# 回车后输入密码 123456
```

MySQL 命令（分号结尾再回车，或输入完回车后再分号回车）

- `show databases;` 查看数据库列表
- `use xxx;` 使用 xxx 数据库
- `show tables;` 查看该数据库下的所有表
- `select * from CHARACTER_SETS;` 查看表内容

按 <kbd>Ctrl</kbd> + <kbd>C</kbd> 中断，<kbd>Ctrl</kbd> + <kbd>D</kbd> 退出

## 概念

### 数据库

大量数据的集合称为数据库 Database

- 关系数据库 —— 使用最广泛
- 面向对象数据、XML 数据库、键值存储系统、层次数据库

### 数据库管理系统 DBMS

用来管理数据库的系统，如 MySQL、**PostgreSQL**、SQL Server、DB2、Oracle

![](https://cdn.wallleap.cn/img/pic/illustration/202405071130327.png)

我们使用的 mysql 命令就是一个客户端

而 mysql 背后还有一个 server 在 24 小时不断运行着

## Node.js 连接数据库

安装依赖

```sh
mkdir mysql-1
cd mysql-1
npm init -y
npm install mysql2 # mysql 好像有点问题
```

新建 `test.js` 复制文档上的示例代码进行修改

```js
var mysql      = require('mysql2');
var connection = mysql.createConnection({
  host     : 'localhost',
  user     : 'root',
  password : '123456',
  // database : 'my_db'
});

connection.connect();

connection.query('CREATE DATABASE IF NOT EXISTS fang DEFAULT CHARSET utf8mb4 COLLATE utf8mb4_unicode_520_ci;', function (error, results, fields) {
  if (error) throw error;
  console.log('The solution is: ', results);
});

connection.query('use fang;', function (error, results, fields) {
  if (error) throw error;
  console.log('使用 fang ', results);
});

//  DEFAULT CHARSET=utf8mb4
connection.query(`CREATE TABLE IF NOT EXISTS user (
  id int(11) NOT NULL AUTO_INCREMENT,
  name varchar(255) DEFAULT NULL,
  age int(11) DEFAULT NULL,
  PRIMARY KEY (id)
) ENGINE=InnoDB AUTO_INCREMENT=1;`, function (error, results, fields) {
  if (error) throw error;
  console.log('创建表 ', results);
});

connection.end();
```

MySQL 一定不要使用 utf-8

进入命令行查看是否创建成功

```sh
docker exec -it mysql1 bash
mysql -u root -p
mysql> show databases;
mysql> use fang;
mysql> show tables;
mysql> describe user;
mysql> insert into user (name, age) values ('Tom', 18);
mysql> update user set age=30 where name='Tom';
mysql> select * from user;
mysql> select count(*) from user;
mysql> select name from user limit 10 order by desc;
mysql> delete from user where name = 'Tom';
mysql> DROP table user;
mysql> DROP database fang;
```

推荐文档：<https://devdocs.io> PostgreSQL

## MySQL 数据类型

五大类

- 数字类型
	- bit(1) 1-64 默认为 1，表示每个值的位数
	- tinyint
		- 非常小的整数
		- 后面加 UNSIGNED 范围为 0-255，默认 -128-127
	- bool, boolean
		- TINYINT(1) 的同义词
		- 0 表示 false
		- 其他非 0 表示 true
	- smallint
	- mediumint
	- int
	- bigint
	- decimal
	- float
	- double
	- serial 等价于 BIGINT UNSIGNED NOT NULL AUTO_INCREMENT UNIQUE
- 字符串类型
	- char(100) 固定
	- varchar(100) 可变
	- binary(1024) 二进制 下面三个可存图片
	- varbinary(1024)
	- blob
	- text 长文本
	- enum('v1', 'v2')
	- set('v1', 'v2')
- 事件和日期类型
	- date 年月日
	- time 时分秒
	- datetime
	- timestamp
	- year
- JSON 类型
- 其他特殊类型

[MySQL :: MySQL 8.0 Reference Manual :: 13.1.1 Numeric Data Type Syntax](https://dev.mysql.com/doc/refman/8.0/en/numeric-type-syntax.html)

IOS8601 格式

sequelizejs

[Node.js学习指南 (poetries.top)](https://blog.poetries.top/node-learning-notes/)

[koa2 进阶学习笔记 · GitBook (chenshenhai.com)](https://chenshenhai.com/koa2-note/)
