---
title: Docker-cli 常用命令
date: 2024-11-14T13:30:57+08:00
updated: 2024-11-14T13:31:10+08:00
dg-publish: false
---

Docker CLI 提供了强大的命令，可以显著提高生产力，简化工作流程，并使容器管理更加高效。以下是每个开发者都应知道的一些基本技巧和提示。

## **1. \**使用 `docker inspect` 提取容器信息\****

```
# 新 Docker 客户端语法
docker inspect --format='{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}'

# 旧 Docker 客户端语法
docker inspect --format='{{.NetworkSettings.IPAddress}}' <container_name>
```

- **提示：** 使用 `docker inspect` 访问有关容器、镜像和卷的详细信息，包括 IP 地址和挂载卷。使用 `--format` 与 `docker inspect` 一起提取特定字段

## **2. \**快速清理未使用的资源\****

```
docker system prune
```

- **输出示例：**

```
WARNING! This will remove:
  - all stopped containers
  - all networks not used by at least one container
  - all dangling images
  - all dangling build cache

Are you sure you want to continue? [y/N]
```

- **提示：** 随着时间的推移，未使用的容器、镜像和卷会累积并占用磁盘空间。`docker system prune` 移除所有未使用的内容，或者使用 `docker image prune`、`docker container prune` 和 `docker volume prune` 进行选择性清理。

## **3. \**在容器中运行一次性命令\****

- **提示：** 使用 `docker exec` 在运行中的容器内执行一次性命令，无需启动新容器。例如：

```
docker exec -it my_container /bin/bash
```

- 适用于在容器内进行调试和检查运行时配置。

## **4. \**限制容器的资源使用\****

```
# 命令
docker run --memory <memory_limit> --cpus <cpu_limit>

# 示例
docker run --memory=512m --cpus=1 my_image
```

- **提示：** 通过限制每个容器的内存和 CPU 使用量，防止资源占用过多。这确保了公平的资源分配：

## **5. \**实时查看容器日志\****

- **提示：** `-f` 选项显示实时日志输出，对于调试运行中的服务很有用。例如：

```
docker logs -f my_container
```

- 您还可以添加 `--tail` 仅查看最近的几行：

```
docker logs -f --tail 50 my_container
```

## **6. \**导出和导入镜像\****

```
# 导出
docker save -o <filename.tar> <image_name>

# 导入
docker load -i <filename.tar>
```

- **提示：** 在不重新下载的情况下，在机器之间导出和导入镜像。非常适合离线设置或网络受限的环境。

## **7. \**查看运行中的容器统计信息\****

```
docker stats
```

- **提示：** 使用 `docker stats` 监控 CPU、内存和网络使用的实时指标。这有助于诊断性能问题，并可视化每个容器的资源利用情况。

## **8. \**动态绑定端口\****

```
docker run -P <image_name>
```

- **提示：** 使用 `-P`，Docker 将暴露的容器端口映射到随机可用的主机端口。要查看映射的端口，请使用：

```
docker port <container_name>
```

- 或者，使用 `-p` 指定特定的主机端口（例如，`docker run -p 8080:80 my_image`）。

## **9. \**使用 `docker-compose` 快速构建和运行容器\****

```
docker-compose up -d
```

- **提示：** 对于多容器应用程序，`docker-compose` 简化了设置。`-d` 标志在后台运行服务，而 `docker-compose up` 仅重新创建修改过的容器，加快开发速度。

## **10. \**使用 `docker system df` 查看磁盘使用情况\****

```
docker system df
```

- **输出示例：**

```
TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
Images              11                  6                   2.866GB             1.171GB (40%)
Containers          17                  13                  3.4MB               2.145MB (63%)
Local Volumes       9                   5                   849.5MB             315.1MB (37%)
Build Cache         0                   0                   0B                  0B
```

- **提示：** 查看 Docker 资源（镜像、容器、卷）占用了多少磁盘空间。此命令提供了存储使用情况的概览，有助于资源管理。

------

## **结论**

掌握这些 Docker CLI 技巧可以显著改善您的工作流程，从高效管理资源到快速调试应用程序。
