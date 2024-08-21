---
title: Docker 安装部署 PostgreSQL 数据库
date: 2024-08-16T10:58:00+08:00
updated: 2024-08-21T10:32:34+08:00
dg-publish: false
source: //juejin.cn/post/7239990044333801533
---

PostgreSQL 是一款功能丰富的关系型数据库，类似于 MySQL，它也是受欢迎程度非常高的。

今天就来简单分享一下使用 Docker 安装部署 PostgreSQL 的过程。

## 1，拉取镜像

拉取 PostgreSQL 的最新镜像即可：

```sh
$ docker pull postgres
```

默认的镜像版本 (`latest`) 是基于 Debian 镜像的，如果你希望拉取体积更小的 PostgreSQL 镜像可以拉取基于 Alpine 的：

```sh
$ docker pull postgres:alpine
```

## 2，创建数据卷

我们最好是创建一个数据库以持久化 PostgreSQL 的数据：

```sh
$ docker volume create postgre-data
```

其他命令

```shell
$ docker volume list            # 列出 Docker 卷
$ docker volume rm pgdata       # 删除 Docker 卷

$ docker volume create pgdata   # 创建 Docker 卷
$ docker volume inspect pgdata  # 检查 Docker 卷
```

## 3，创建并运行容器

例如

```shell
docker run -it \
	--name postgres \
	--restart always \
	-e TZ='Asia/Shanghai' \
	-e POSTGRES_PASSWORD='abc123' \
	-e ALLOW_IP_RANGE=0.0.0.0/0 \
	-v /home/postgres/data:/var/lib/postgresql \
	-p 55435:5432 \
	-d postgres
```

| 名称                          | 解释                        |
| --------------------------- | ------------------------- |
| --name                      | 自定义容器名称                   |
| --restart always            | 设置容器在 docker 重启或崩溃时自动启动容器 |
| -e POSTGRES_PASSWORD        | Postgresql 数据库密码          |
| -e ALLOW_IP_RANGE=0.0.0.0/0 | 表示允许所有 IP 访问              |
| -e TZ='Asia/Shanghai'       | 设置时区                      |
| -v [path] : [path]          | 本地目录映射 (本地目录 : 容器内路径)     |
| -p 55435:5432               | 端口映射 (主机端口 : 容器端口)        |
| -d postgres                 | 镜像名称                      |

- `-it`: 以交互模式启动容器，允许你与容器进行交互。
- `-e` 设置环境变量
- `-d`: 以后台模式运行容器。

执行下列命令：

```sh
$ docker run -id --name=postgresql -v postgre-data:/var/lib/postgresql/data -p 5432:5432 -e POSTGRES_PASSWORD=123456 -e LANG=C.UTF-8 postgres
```

上述设定了数据卷，以及暴露了端口 `5432`，这是 PostgreSQL 的默认端口。

除此之外，我们还设定了下列环境变量：

- `POSTGRES_PASSWORD` 设定 PostgreSQL 的超级用户的密码，这里设定为 `123456`，PostgreSQL 容器的超级用户**用户名**为 `postgres`
- `LANG` 设定语言环境为 `C.UTF-8` 以支持中文

除此之外，还可以设定环境变量 `POSTGRES_USER` 来**指定超级用户的用户名**，上述没有指定这个环境变量则默认是 `postgres`。

参考：

- PostgreSQL 镜像主页：[传送门](https://link.juejin.cn/?target=https%3A%2F%2Fhub.docker.com%2F_%2Fpostgres "https://hub.docker.com/_/postgres")

## 连接到你的主机系统

假设你有安装 postgresql 客户端,您可以使用主机端口映射测试。您需要使用 `docker ps` 找出映射到本地主机端口:

```sh
$ docker ps
```

## 如何通过 bash 连接访问容器

让我们使用以下命令使用 bash 连接到容器。将会看到有一些 Postgres 进程在后台运行 (checkpointer、walwriter、stats collector 等等)

```sh
$ docker exec -it 67a4705c263c /bin/bash  # 67a4705c263c 是容器 id 或者容器名字
```

进入终端后查看进程

```sh
$ ps -ef | grep postgres
```

可以在容器中直接登录

```sh
$ psql -U postgres
```

连接到数据库的另一种方法是在连接到 Postgres 容器本身时使用 psql。

```sh
$ docker exec -it 67a4705c263c psql -U postgres
```

连接数据库后让我们使用以下查询命令：

```sh
postgres=# select now();
```
