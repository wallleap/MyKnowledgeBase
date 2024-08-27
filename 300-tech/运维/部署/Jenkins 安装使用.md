---
title: Jenkins 安装使用
date: 2024-08-22T01:52:37+08:00
updated: 2024-08-22T01:55:43+08:00
dg-publish: false
source: https://github.com/peng-yin/note/issues/21
---

## Jenkins Install

___

**On Ubuntu**

### **执行如下代码：**

```
    wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
    sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ &gt; /etc/apt/sources.list.d/jenkins.list'
    sudo apt-get update
    sudo apt-get install jenkins
```

### **Jenkins 的运行**

服务启动|停止|重启：`sudo service jenkins start|stop|restart`

### **Jenkins 的初次配置**

> 1.  浏览器中访问： `http://IP:8080`， 使用 `cat /var/lib/jenkins/secrets/initialAdminPassword` 查看初始密码并在浏览器中登录
> 2.  安装推荐或自选插件 (老手可自行配置，新手推荐默认方式)
> 3.  创建管理员
> 4.  更改 8080 端口：jenkins 默认使用 8080 端口，较容易冲突，更改方式如下：
>
> > 4.1. 编辑 `/etc/default/jenkins` 文件，将 `HTTP_PORT=8080` 更改为自定义端口
> > 4.2. 编辑 `/etc/init.d/jenkins` 文件，将唯一的 8080 更改为自定义端口

参看官方文档：

1. [Installing Jenkins on Ubuntu](https://wiki.jenkins.io/display/JENKINS/Installing+Jenkins+on+Ubuntu)

___

## Simple Use

**Plugin Install**

安装插件，1、2 插件使用 Tag 进行版本发布与回滚，3 插件将代码部署到远端

> 1.  Git Parameter Plug-In
> 2.  Stash Branch Parameter Plug-In
> 3.  Git Tag Message Plugin
> 4.  Publish Over SSH

这里以 Web 工程为例，安装了相关插件，如下：

> NodeJS Plugin

### 新建任务

选择简单配置，可参考文档使用其他类型任务

![](https://cdn.wallleap.cn/img/pic/illustration/202408221354563.png?imageSlim)

#### 1.General 配置

勾选 `参数化构建过程`， 添加参数选择 `Git Parameter`，填写参数名相关配置。

![](https://cdn.wallleap.cn/img/pic/illustration/202408221354254.png?imageSlim)

![](https://cdn.wallleap.cn/img/pic/illustration/202408221354255.png?imageSlim)

#### 2.源码管理

勾选 `Git`，填写相关配置

![](https://cdn.wallleap.cn/img/pic/illustration/202408221355657.png?imageSlim)

填写入代码库地址，推荐使用已 ssh 地址，**`凭证`** 为获取远程库代码的权限校验，若没有对应选项，点击 `添加` 新建一个凭证即可。

##### 添加凭证

类型这里使用 `SSH Username with private key`,只需在 `private Key` 中添加对应远程库的 ***公钥对应的私钥*** 即可，请注意，这里需要填写 `对应的私钥`

![](https://cdn.wallleap.cn/img/pic/illustration/202408221355648.png?imageSlim)

#### 3.构建环境

对于不同的工程，选择对应的构建环境，这里以 Web 工程为例，所以勾选 `Provide Node & npm bin/ folder to PATH` 了，默认即可。

注：若无 NodeJs 选择可选，保存现有配置，跳转到 `Jenkins` -> `全局工具配置`，新增一个 NodeJs 即可，对于 node 版本选择项目对应的版本即可。

#### 3.构建

**该步骤为当代码 Ready 后，如何进行构建**
勾选 `执行shell` 了，在命令中输入对应构建命令即可
Web 工程构建为例：

```shell
npm install
npm run build
// 打包构建文件
zip -qr dist.zip dist/
```

#### 4.构建后操作

**该步骤关注点为如何进行部署：构建完成后，通过 SSH 传递到目标服务器，并在目标服务器上执行部署脚本，完成部署**

首先需要解决的是通过 ssh 能无密码登录到目标服务器

进入 `系统管理` -> `系统设置`

> 1.  在 `Publish over SSH` 类目中点击 `新增`
> 2.  在 `key` 中填入对应私钥（注：该秘钥对应的公钥此时在目标服务器的授权认证目录下）
> 3.  在 `SSH Server` 中输入对应配置即可， `Name Username Hostname`
> 4.  点击 `Test Configuration`，出现 `Success` 表示可 Jenkins 可无密码访问

回到 Test 任务的配置页面，继续之前配置，如下图

![](https://cdn.wallleap.cn/img/pic/illustration/202408221355658.png?imageSlim)

在 `Name` 中选择对应 SSH Server 选项

Transfer 中根据实际情况填写对应目录与文件

Exec command 中可写入对应 bash 命令，命令在目标服务器上执行
