---
title: Docker Network
date: 2024-11-14T13:28:21+08:00
updated: 2024-11-14T13:28:28+08:00
dg-publish: false
source: https://www.mdnice.com/writing/41ba20a5bbb64403bfa405556fd94858
---

Docker 通过容器化应用程序，彻底改变了我们构建、分发和运行应用程序的方式。然而，有效使用 Docker 的一个关键方面是理解容器如何相互通信以及与外界通信。

## **1. 什么是 Docker 网络？**

Docker 网络允许容器相互通信以及与外部资源通信。默认情况下，每个容器都是隔离的，这意味着除非连接到共享网络，否则它们不会自动与其他容器通信。Docker 提供了一个灵活且强大的网络模型，简化了容器之间的连接。

简而言之，Docker 网络处理：

- 容器之间的通信
- 容器与外界的通信
- 为容器化应用程序定义隔离边界

## **2. Docker 网络类型**

Docker 提供了几种网络驱动程序，每种都适用于不同的用例。了解这些网络类型对于为您的应用程序选择正确的网络至关重要。

### **桥接网络（默认）**

桥接网络是 Docker 在创建容器时使用的默认网络驱动程序。同一桥接网络上的容器可以使用 IP 地址或容器名称相互通信。

**用例：** 这通常用于需要在单个主机上内部通信的多容器应用程序，例如开发环境中的微服务。

```
# 列出默认桥接网络上的容器
docker network inspect bridge
```

### **主机网络**

在主机网络中，容器共享主机的网络命名空间。这意味着容器可以直接使用主机的 IP 地址和端口。

**用例：** 这适用于需要避免网络开销的性能关键型应用程序，但您会失去容器与主机之间的隔离。

```
docker run --network host <image>
```

### **无网络**

顾名思义，`none` 网络驱动程序禁用了容器的所有网络功能。当容器不需要任何外部或内部网络时，这很有用。

**用例：** 用于不需要任何网络通信的容器，例如特定的后台任务。

```
docker run --network none <image>
```

### **覆盖网络**

覆盖网络用于多主机 Docker 设置，其中不同主机上运行的容器需要相互通信。它需要 Docker Swarm 或 Kubernetes 设置。

**用例：** 对于跨多个 Docker 主机的分布式系统至关重要。

```
docker network create --driver overlay my-overlay-network
```

------

## **3. Docker 网络实战**

现在，让我们探索如何在现实世界场景中使用 Docker 网络。

### **创建和管理网络**

默认情况下，Docker 在运行容器时会创建一个桥接网络。但是，您可以为更高级的设置创建自定义网络。

```
# 创建自定义桥接网络
docker network create my-custom-network
```

您可以使用以下命令检查您的网络：

```
docker network ls  # 列出所有网络
docker network inspect my-custom-network  # 关于特定网络的详细信息
```

### **将容器连接到网络**

创建自定义网络后，您可以将容器附加到它。

```
# 运行一个容器并将其附加到自定义网络
docker run -d --name container1 --network my-custom-network nginx

# 运行另一个容器并将其附加到同一网络
docker run -d --name container2 --network my-custom-network redis
```

同一网络上的容器可以通过容器名称相互通信。

```
# 在 container1 内部，您可以通过名称访问 container2
ping container2
```

### **将服务暴露给主机**

如果您希望容器内运行的服务能够从主机或外部世界访问，您需要暴露其端口。

```
docker run -d -p 8080:80 --name webserver nginx
```

在这种情况下，容器内运行的 Nginx Web 服务器将可以通过主机机器上的 `localhost:8080` 访问。

为确保容器间通信高效且安全，优化 Docker 网络配置是关键。以下是一些推荐的优化策略：

### **选择合适的网络模式**

Docker 提供的不同网络模式适用于不同的场景，合理选择可以提高应用程序的性能和安全性。

- 桥接网络：适合单主机、多容器之间的通信，典型的使用场景是开发环境的微服务。使用自定义桥接网络能够灵活命名容器，便于容器名称解析。
- 主机网络：适合高性能应用，因为它消除了 NAT（网络地址转换）带来的网络开销。但使用主机网络时容器会共享主机的网络命名空间，因此隔离性会降低，适用于信任容器的场景。
- 覆盖网络：用于多主机 Docker Swarm 或 Kubernetes 集群中，可以跨多主机的容器间通信。分布式系统或集群应用建议选择覆盖网络。
- 无网络模式：适合无需网络的容器，例如单纯的数据处理或备份任务等。

使用 docker network ls 可以列出当前主机上的所有网络类型，从中选择合适的网络配置。

### **限制网络带宽**

Docker 支持使用 --network-opt 设置自定义网络的带宽上限，帮助控制容器的网络带宽。对于共享资源的多租户系统，网络限速可以避免个别容器过度占用带宽，影响整体系统的稳定性。

```
docker network create \
  --opt com.docker.network.driver.mtu=1200 \
  --opt com.docker.network.window=30 \
  custom_network
```

### **使用自定义 DNS 配置**

在微服务架构中，DNS 解析是容器间通信的关键。Docker 默认使用主机的 DNS 配置，但在某些情况下，配置自定义 DNS 服务器更可靠。使用 --dns 参数为容器指定 DNS 服务器，以确保解析过程不受主机配置影响。

```
docker run --dns 8.8.8.8 --dns 8.8.4.4 --network my-custom-network my_container
```

也可以通过在 /etc/docker/daemon.json 文件中添加 "dns" 配置，设置 Docker Daemon 的默认 DNS。

```
{
  "dns": ["8.8.8.8", "8.8.4.4"]
}
```

### **网络命名空间隔离**

对于隔离要求高的应用，建议将生产和测试环境分别置于不同的网络命名空间。例如，您可以创建不同的自定义网络来隔离环境，确保敏感数据或服务不会意外暴露或被干扰。

## **4. Docker 网络常见问题**

在使用 Docker 网络时，开发者可能会遇到一些常见问题。以下列出几种常见问题及对应的解决方案。

### **问题 1：容器之间无法通信**

原因：容器未连接到同一个网络，或网络配置不当。

解决方案：确保容器位于相同的自定义网络上，可通过 `docker network connect` 将容器添加到目标网络，或使用自定义网络创建容器。检查 `docker network inspect <network_name>`，查看是否所有容器均正确连接。

```
docker network connect my-custom-network container1
```

### **问题 2：桥接网络内通信不通**

原因：默认桥接网络或防火墙阻止了容器间通信。

解决方案：使用 docker network inspect bridge 检查网络设置，确保桥接网络的设置未被更改。如果网络仍不通，可能需要检查主机防火墙规则，确保允许 Docker 的默认端口和 IP 范围。

### **问题 3：服务端口映射失败**

原因：容器内服务未绑定到正确的 IP 或端口映射未设置。

解决方案：确保服务监听 0.0.0.0，而非 localhost。启动容器时使用 -p 参数设置端口映射，如 docker run -d -p 8080:80 my_container。

------

## **结论**

Docker 网络是一个强大的功能，允许容器与彼此和外部世界无缝交互。通过了解不同的 Docker 网络类型以及如何配置它们，开发者可以高效地构建和部署容器化应用程序。无论您是在开发一个小型应用程序还是复杂的多容器系统，Docker 网络都能实现这一切。
