---
title: 001-Docker 简介及安装
date: 2024-10-27T02:26:51+08:00
updated: 2024-10-27T02:39:57+08:00
dg-publish: false
---

<https://docker.easydoc.net/>

## Docker 是什么

Docker 是一个应用**打包**、**分发**、**部署**的工具

你也可以把它理解为一个轻量的虚拟机，它只虚拟你软件需要的**运行环境**，多余的一点都不要，

而普通虚拟机则是一个完整而庞大的系统，包含各种不管你要不要的软件。

> 本文档课件配套 [视频教程](https://www.bilibili.com/video/BV11L411g7U1)

## 跟普通虚拟机的对比

| 特性   | 普通虚拟机                                                   | Docker                                               |
| ------ | ------------------------------------------------------------ | ---------------------------------------------------- |
| 跨平台 | 通常只能在桌面级系统运行，例如 Windows/Mac，无法在不带图形界面的服务器上运行 | 支持的系统非常多，各类 windows 和 Linux 都支持       |
| 性能   | 性能损耗大，内存占用高，因为是把整个完整系统都虚拟出来了     | 性能好，只虚拟软件所需运行环境，最大化减少没用的配置 |
| 自动化 | 需要手动安装所有东西                                         | 一个命令就可以自动部署好所需环境                     |
| 稳定性 | 稳定性不高，不同系统差异大                                   | 稳定性好，不同系统都一样部署方式                     |

## 打包、分发、部署

**打包**：就是把你软件运行所需的依赖、第三方库、软件打包到一起，变成一个**安装包**

**分发**：你可以把你打包好的“安装包”上传到一个镜像**仓库**，其他人可以非常方便的获取和安装

**部署**：拿着“安装包”就可以一个命令**运行**起来你的应用，自动模拟出一摸一样的运行环境，不管是在 Windows/Mac/Linux。

![](https://cdn.wallleap.cn/img/pic/illustration/202410271427507.png?imageSlim)

## Docker 部署的优势

常规应用开发部署方式：自己在 Windows 上开发、测试 --> 到 Linux 服务器配置运行环境部署。

> 问题：我机器上跑都没问题，怎么到服务器就各种问题了

用 Docker 开发部署流程：自己在 Windows 上开发、测试 --> 打包为 Docker 镜像（可以理解为软件安装包） --> 各种服务器上只需要一个命令部署好

> 优点：确保了不同机器上跑都是一致的运行环境，不会出现我机器上跑正常，你机器跑就有问题的情况。

例如 [易文档](https://easydoc.net/)，[SVNBucket](https://svnbucket.com/) 的私有化部署就是用 Docker，轻松应对客户的各种服务器。

## Docker 通常用来做什么

- 应用分发、部署，方便传播给他人安装。特别是开源软件和提供私有部署的应用
- 快速安装测试/学习软件，用完就丢（类似小程序），不把时间浪费在安装软件上。例如 Redis / MongoDB / ElasticSearch / ELK
- 多个版本软件共存，不污染系统，例如 Python2、Python3，Redis4.0，Redis5.0
- Windows 上体验/学习各种 Linux 系统

## 重要概念：镜像、容器

**镜像**：可以理解为软件安装包，可以方便的进行传播和安装。

**容器**：软件安装后的状态，每个软件运行环境都是独立的、隔离的，称之为容器。

## 安装

桌面版：<https://www.docker.com/products/docker-desktop>

服务器版：<https://docs.docker.com/engine/install/#server>

> macOS 可以使用 OrbStack 代替桌面版

## 启动报错解决

报错截图

![](https://cdn.wallleap.cn/img/pic/illustration/202410271427455.png?imageSlim)

**解决方法**：

控制面板 ->程序 ->启用或关闭 windows 功能，开启 Windows 虚拟化和 Linux 子系统（WSL2)

![](https://cdn.wallleap.cn/img/pic/illustration/202410271432522.png?imageSlim)

**命令行安装 Linux 内核**

`wsl.exe --install -d Ubuntu`

> 你也可以打开微软商店 Microsoft Store 搜索 Linux 进行安装，选择一个最新版本的 Ubuntu 或者 Debian 都可以

> 上面命令很可能你安装不了，微软商店你也可能打不开，如果遇到这个问题，参考：<https://blog.csdn.net/qq_42220935/article/details/104714114>

**设置开机启动 Hypervisor**

`bcdedit /set hypervisorlaunchtype auto`

> 注意要用管理员权限打开 PowerShell

**设置默认使用版本 2**

`wsl.exe --set-default-version 2`

**查看 WSL 是否安装正确**

`wsl.exe --list --verbose`

应该如下图，可以看到一个 Linux 系统，名字你的不一定跟我的一样，看你安装的是什么版本。

并且 VERSION 是 2

![](https://cdn.wallleap.cn/img/pic/illustration/202410271427447.png?imageSlim)

**确保 BIOS 已开启虚拟化，下图检查是否已开启好**

> 如果是已禁用，请在开机时按 F2 进入 BIOS 开启一下，不会设置的可以网上搜索下自己主板的设置方法，Intel 和 AMD 的设置可能稍有不同

![](https://cdn.wallleap.cn/img/pic/illustration/202410271427459.png?imageSlim)

**出现下图错误，点击链接安装最新版本的 WSL2**

<https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi>

![](https://cdn.wallleap.cn/img/pic/illustration/202410271427513.png?imageSlim)

## 镜像加速源

| 镜像加速器          | 镜像加速器地址                       |
| ------------------- | ------------------------------------ |
| Docker 中国官方镜像 | <https://registry.docker-cn.com>       |
| DaoCloud 镜像站     | <http://f1361db2.m.daocloud.io>        |
| Azure 中国镜像      | <https://dockerhub.azk8s.cn>           |
| 科大镜像站          | <https://docker.mirrors.ustc.edu.cn>   |
| 阿里云              | <https://ud6340vz.mirror.aliyuncs.com> |
| 七牛云              | <https://reg-mirror.qiniu.com>         |
| 网易云              | <https://hub-mirror.c.163.com>         |
| 腾讯云              | <https://mirror.ccs.tencentyun.com>    |

```
"registry-mirrors": ["https://registry.docker-cn.com"]
```

![](https://cdn.wallleap.cn/img/pic/illustration/202410271427510.png?imageSlim)
