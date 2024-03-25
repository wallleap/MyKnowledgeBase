---
title: 在 CentOS 上搭建 Git 服务器
date: 2024-03-16 09:54
updated: 2024-03-16 09:57
---

## [安装 Git](https://www.xiaojun.im/posts/2018-09-20-centos7-git#安装-git)

将可能已安装的 Git 卸掉

```sh
yum remove -y git
```

安装依赖

```sh
yum install -y curl-devel expat-devel gettext-devel openssl-devel zlib-devel asciidoc
yum install -y gcc perl-ExtUtils-MakeMaker
```

从这里选择一个 Git 版本，我选择的是 2.9.5 <https://mirrors.edge.kernel.org/pub/software/scm/git/>

```sh
cd /usr/local/src/
wget https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.9.5.tar.gz
tar -vxf git-2.9.5.tar.gz
cd git-2.9.5
make prefix=/usr/local/git all
make prefix=/usr/local/git install
echo "export PATH=$PATH:/usr/local/git/bin" >> /etc/profile
source /etc/profile
# 验证是否安装成功
git --version
```

## [创建 Git 用户](https://www.xiaojun.im/posts/2018-09-20-centos7-git#创建-git-用户)

```sh
adduser git
passwd git
# 输入两次密码
```

## [创建 Git 仓库](https://www.xiaojun.im/posts/2018-09-20-centos7-git#创建-git-仓库)

为了方便管理，所有的 git 仓库都置于同一目录下，假设为 /srv/gitLibrary

```sh
mkdir /srv/gitLibrary
# 创建一个名字为sample.git的裸库
git init --bare /srv/gitLibrary/sample.git，
chown -R git:git /srv/gitLibrary
```

## [配置 SSH](https://www.xiaojun.im/posts/2018-09-20-centos7-git#配置-ssh)

*配置 ssh 的目的是为了能免密码访问。*

创建 authorized_keys 文件

```sh
cd /home/git
mkdir .ssh
chmod 755 .ssh
touch .ssh/authorized_keys
chmod 644 .ssh/authorized_keys
chown -R git:git /home/git
```

在客户端上生成密钥

```sh
ssh-keygen -t rsa -C "your_email"
```

一路回车后会产生两个文件: `id_rsa` 和 `id_rsa.pub`，控制台会打印这俩文件的路径，先找到它们，等下要用到。

编辑 authorized_keys 文件

```sh
vim /home/git/.ssh/authorized_keys
```

然后把 `id_rsa.pub` 文件中的内容拷贝进去， *你也可以上传多个密钥，比如，家里一个电脑，公司一个电脑，每个密钥占一行即可。*

至此，远程仓库算是搭建完成了，接下来用客户端测试一下能否将这个仓库拉到本地

```sh
git clone ssh://git@x.x.x.x:x/srv/gitLibrary/sample.git
```
