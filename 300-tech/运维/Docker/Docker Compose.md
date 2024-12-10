---
title: Docker Compose
date: 2024-11-14T13:31:37+08:00
updated: 2024-11-14T13:31:49+08:00
dg-publish: false
---

Docker Compose 是 Docker 官方编排工具，用于定义和运行多容器 Docker 应用程序。它是一个轻量级的工具，用于快速配置和启动应用程序的不同服务。

## **Docker Compose 是什么**

Docker Compose 最初是由 Docker 公司开发，并于 2014 年 6 月首次发布。它的前身是一个名为 `fig` 的开源项目，由 Piston Cloud Computing 开发。Docker 公司看到了 `fig` 的潜力，并决定将其集成到 Docker 生态系统中，从而诞生了 Docker Compose。Docker Compose 的设计目标是简化多容器应用程序的部署和管理，让用户能够通过一个简单的 YAML 文件来定义和管理整个应用程序。

这使得开发、测试和生产环境的一致性得以保证，同时也简化了应用程序的部署和管理过程。

Docker Compose 随着 Docker 的发展经历了多个版本的迭代。以下是一些重要的版本更新：

- **1.x 版本**：最初的版本，提供了基本的编排功能。
- **2.x 版本**：引入了对 Docker Engine API 的支持，允许更紧密地与 Docker Engine 集成。
- **3.x 版本**：引入了对服务依赖性的支持，允许定义服务之间的启动顺序。
- **4.x 版本**：引入了对环境变量和命令行参数的支持，使得配置更加灵活。
- **5.x 版本**：引入了对 Docker Stacks 的支持，允许在 Docker Swarm 模式下部署应用程序。

## **Docker Compose 使用方式**

### **安装 Docker Compose**

在大多数系统上，Docker Compose 可以作为 Python 的包来安装。以下是安装步骤：

```
# 安装pip（Python的包管理工具）
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python get-pip.py

# 使用pip安装Docker Compose
pip install docker-compose
```

### **定义服务**

创建一个名为 `docker-compose.yml` 的 YAML 文件来定义服务。以下是一个基本的 `docker-compose.yml` 文件示例：

```
version: '3'
services:
  web:
    image: "nginx:alpine"
    ports:
      - "80:80"
    volumes:
      - "web-data:/var/www"
  database:
    image: "postgres:latest"
    volumes:
      - "db-data:/var/lib/postgresql/data"

volumes:
  web-data:
  db-data:
```

在这个示例中，定义了两个服务：`web` 和 `database`。`web` 服务使用 `nginx:alpine` 镜像，并映射端口 80。`database` 服务使用 `postgres:latest` 镜像。

### **常用 CLI 命令**

以下是一些常用的 Docker Compose CLI 命令：

1. **启动服务**：

```
docker-compose up
```

这个命令将启动所有服务，并在前台运行。

1. **后台运行服务**：

```
docker-compose up -d
```

使用 `-d` 标志，服务将在后台运行。

1. **停止服务**：

```
docker-compose down
```

这个命令将停止所有服务，并移除容器。

1. **查看服务状态**：

```
docker-compose ps
```

这个命令将列出所有服务的状态。

1. **重建服务**：

```
docker-compose up --build
```

这个命令将重建服务，并重新创建容器。

1. **查看日志**：

```
docker-compose logs
```

这个命令将显示所有服务的日志。

1. **进入容器**：

```
docker-compose exec [service-name] /bin/bash
```

这个命令将进入指定服务的容器，并启动一个 bash 会话。

1. **停止单个服务**：

```
docker-compose stop [service-name]
```

这个命令将停止指定的服务。

1. **启动单个服务**：

```
docker-compose start [service-name]
```

这个命令将启动指定的服务。

1. **重启服务**：

```
docker-compose restart
```

这个命令将重启所有服务。

### **高级使用**

Docker Compose 还支持许多高级功能，如环境变量、扩展、网络配置等。以下是一些高级用法示例：

1. **环境变量**：

```
version: '3'
services:
  web:
    image: "nginx:alpine"
    env_file:
      - web.env
```

在这个示例中，`web` 服务将从 `web.env` 文件中读取环境变量。

1. **扩展**：

```
version: '3'
services:
  web:
    image: "nginx:alpine"
    deploy:
      replicas: 3
```

在这个示例中，`web` 服务将被扩展为 3 个副本。这意味着 Docker Compose 将启动 3 个相同的 web 服务实例，它们将作为一个集群运行，以提供高可用性和负载均衡。

1. **网络配置**：

```
version: '3'
services:
  web:
    image: "nginx:alpine"
    networks:
      - webnet
networks:
  webnet:
```

这段配置文件定义了一个名为 web 的服务，它将使用 nginx:alpine 镜像，并连接到一个名为 webnet 的网络。这个网络也是在同一个配置文件中定义的。这样的配置允许 web 服务在 Docker 容器中运行，并且可以通过 webnet 网络与其他服务进行通信。

## **结论**

Docker Compose 是一个强大的工具，用于定义和运行多容器 Docker 应用程序。它简化了应用程序的部署和管理，提高了开发效率。
