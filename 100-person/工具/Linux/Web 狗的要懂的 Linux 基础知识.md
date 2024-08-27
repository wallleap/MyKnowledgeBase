---
title: 更新源
date: 2024-08-26T09:25:06+08:00
updated: 2024-08-26T09:49:06+08:00
dg-publish: false
source: https://www.sqlsec.com/2019/10/linux.html
---

Linux 知识众多，发行版又广，为了避免每次着急时查资料的尴尬情况出现，特此将我平时可能用到的命令都记录下来，以方便后面使用。

## 更新源

### Ubuntu

```bash
# 备份更新源
mv /etc/apt/sources.list /etc/apt/sources.list.bak

# 22.04 中科大源
echo "deb https://mirrors.ustc.edu.cn/ubuntu/ jammy main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb-src https://mirrors.ustc.edu.cn/ubuntu/ jammy main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb https://mirrors.ustc.edu.cn/ubuntu/ jammy-security main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb-src https://mirrors.ustc.edu.cn/ubuntu/ jammy-security main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb https://mirrors.ustc.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb-src https://mirrors.ustc.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb https://mirrors.ustc.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb-src https://mirrors.ustc.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse" >> /etc/apt/sources.list

# 20.04 中科大源
echo "deb https://mirrors.ustc.edu.cn/ubuntu/ focal main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb-src https://mirrors.ustc.edu.cn/ubuntu/ focal main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb https://mirrors.ustc.edu.cn/ubuntu/ focal-security main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb-src https://mirrors.ustc.edu.cn/ubuntu/ focal-security main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb https://mirrors.ustc.edu.cn/ubuntu/ focal-updates main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb-src https://mirrors.ustc.edu.cn/ubuntu/ focal-updates main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb https://mirrors.ustc.edu.cn/ubuntu/ focal-backports main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb-src https://mirrors.ustc.edu.cn/ubuntu/ focal-backports main restricted universe multiverse" >> /etc/apt/sources.list

# 20.04 清华源
echo "deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse" >> /etc/apt/sources.list

# 19.04 清华源
echo "deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ disco main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ disco main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ disco-updates main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ disco-updates main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ disco-backports main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ disco-backports main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ disco-security main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ disco-security main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ disco-proposed main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ disco-proposed main restricted universe multiverse" >> /etc/apt/sources.list

# 18.04 阿里源
echo "deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse" >> /etc/apt/sources.list

# 18.04 清华源
echo "deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse" >> /etc/apt/sources.list

# 16.04 清华源
echo "deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted universe multiverse" >> /etc/apt/sources.list

# 14.04 阿里源
echo "deb http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb-src http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb-src http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb-src http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb-src http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb-src http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse" >> /etc/apt/sources.list
```

### Kali

```bash
# 备份更新源
mv /etc/apt/sources.list /etc/apt/sources.list.bak

# 中科大源
echo "deb http://mirrors.ustc.edu.cn/kali kali-rolling main non-free contrib" >> /etc/apt/sources.list
echo "deb-src http://mirrors.ustc.edu.cn/kali kali-rolling main non-free contrib" >> /etc/apt/sources.list

# 浙大源
echo "deb http://mirrors.zju.edu.cn/kali kali-rolling main contrib non-free" >> /etc/apt/sources.list
echo "deb-src http://mirrors.zju.edu.cn/kali kali-rolling main contrib non-free" >> /etc/apt/sources.list

# 东软大学
echo "deb http://mirrors.neusoft.edu.cn/kali kali-rolling/main non-free contrib" >> /etc/apt/sources.list
echo "deb-src http://mirrors.neusoft.edu.cn/kali kali-rolling/main non-free contrib" >> /etc/apt/sources.list
```

### Debian

```bash
# 备份更新源
mv /etc/apt/sources.list /etc/apt/sources.list.bak

# Debian 11 bullseye 中科大源
echo "deb https://mirrors.ustc.edu.cn/debian/ bullseye main contrib non-free " >> /etc/apt/sources.list
echo "deb-src https://mirrors.ustc.edu.cn/debian/ bullseye main contrib non-free " >> /etc/apt/sources.list
echo "deb https://mirrors.ustc.edu.cn/debian/ bullseye-updates main contrib non-free " >> /etc/apt/sources.list
echo "deb-src https://mirrors.ustc.edu.cn/debian/ bullseye-updates main contrib non-free " >> /etc/apt/sources.list
echo "deb https://mirrors.ustc.edu.cn/debian/ bullseye-backports main contrib non-free " >> /etc/apt/sources.list
echo "deb-src https://mirrors.ustc.edu.cn/debian/ bullseye-backports main contrib non-free " >> /etc/apt/sources.list
echo "deb https://mirrors.ustc.edu.cn/debian-security/ bullseye-security main contrib non-free " >> /etc/apt/sources.list
echo "deb-src https://mirrors.ustc.edu.cn/debian-security/ bullseye-security main contrib non-free " >> /etc/apt/sources.list

# Debian 10 buster 网易源
echo "deb http://mirrors.163.com/debian/ buster main contrib non-free" >> /etc/apt/sources.list
echo "deb http://mirrors.163.com/debian/ buster-updates main contrib non-free" >> /etc/apt/sources.list
echo "deb http://mirrors.163.com/debian/ buster-backports main contrib non-free" >> /etc/apt/sources.list
echo "deb http://mirrors.163.com/debian-security buster/updates main contrib non-free" >> /etc/apt/sources.list


# Debian 9 stretch 网易源
echo "deb http://mirrors.163.com/debian/ stretch main contrib non-free" >> /etc/apt/sources.list
echo "deb http://mirrors.163.com/debian/ stretch-updates main contrib non-free" >> /etc/apt/sources.list
echo "deb http://mirrors.163.com/debian/ stretch-backports main contrib non-free" >> /etc/apt/sources.list
echo "deb http://mirrors.163.com/debian-security stretch/updates main contrib non-free" >> /etc/apt/sources.list

# Debian 8 jessie 网易源
echo "deb http://mirrors.163.com/debian/ jessie main contrib non-free" >> /etc/apt/sources.list
echo "deb http://mirrors.163.com/debian/ jessie-updates main contrib non-free" >> /etc/apt/sources.list
echo "deb http://mirrors.163.com/debian/ jessie-backports main contrib non-free" >> /etc/apt/sources.list
echo "deb http://mirrors.163.com/debian-security jessie/updates main contrib non-free" >> /etc/apt/sources.list
```

### CentOS

```bash
# 备份更新源
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak

# 阿里云的CentOS-Base.repo 到/etc/yum.repos.d/
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

# 刷新源
yum update

# 生成缓存
yum makecache
```

> 这里我用的是 CentOS7 的更新源，其他源参考如下：

```bash
# CentOS 5 国内源替换
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-5.repo

# CentOS 6 国内源替换
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
```

## proxychains

### 安装

#### Ubuntu/Debian

```bash
apt install  proxychains
```

#### CentOS

```bash
# 下载到指定目录
cd /usr/local/src
git clone https://github.com/rofl0r/proxychains-ng.git
cd proxychains-ng

# 安装GCC
yum install gcc

# 手动编译
./configure
make && make install
make install-config
```

### 配置

#### Kali/Ubuntu/Debian

```bash
vim /etc/proxychains.conf
```

#### CentOS

```bash
vim /usr/local/etc/proxychains.conf
```

### 测试

```bash
proxychains curl https://www.google.com
```

## zsh

### Ubuntu/Kali/Debian 安装

```bash
# 安装zsh
apt install zsh

# 更改默认shell为zsh
chsh -s /bin/zsh

# 安装oh-my-zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

# 下载命令补全插件
git clone https://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions

# zshrc配置文件中修改如下内容
plugins=(git zsh-autosuggestions)

# 重新加载zsh配置
zsh
```

### CentOS 安装

```bash
# 安装zsh
yum install zsh
```

除了安装使用 yum 以外，其他配置方法和上面其他发行版系统一样，这里就不再赘述了。

### zsh 插件

CentOS7 安装 zsh 一些插件和 macOS 下安装相差不多，这里就简单提一下吧：

#### autojump

快捷目录跳转插件

```bash
yum install autojump autojump-zsh -y
```

在 `~/.zshrc` 中配置

```
plugins=(其他的插件 autojump)
```

输入 `zsh` 命令生效配置后即可正常使用 `j` 命令，下面是简单的演示效果：

```bash
# root @ centos in ~/test/23333 [18:45:14]
$ pwd
/root/test/23333

# root @ centos in ~/test/23333 [18:45:16]
$ cd ~


# root @ centos in ~ [18:45:17] 直接 j 2 就跳转到历史目录了
$ j 2
/root/test/23333

# root @ centos in ~/test/23333 [18:45:20]
$ pwd
/root/test/23333
```

#### zsh-syntax-highlighting

命令输入正确会绿色高亮显示，可以有效地检测命令语法是否正确

```bash
cd ~/.oh-my-zsh/custom/plugins
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git
```

在 `~/.zshrc` 中配置

```
plugins=(其他的插件 zsh-syntax-highlighting)
```

输入 `zsh` 命令生效配置

#### autosuggestions

终端下自动提示接下来可能要输入的命令，这个实际使用效率还是比较高的

```bash
cd ~/.oh-my-zsh/custom/plugins
git clone https://github.com/zsh-users/zsh-autosuggestions.git
```

在 `~/.zshrc` 中配置

```
plugins=(其他的插件 zsh-autosuggestions)
```

输入 `zsh` 命令生效配置

## SSH

### 安装 SSH

#### Ubuntu

```bash
# 直接安装报错
apt install openssh-server
```

居然报错了，内容如下：

```verilog
下列软件包有未满足的依赖关系：
 openssh-server : 依赖: openssh-client (= 1:7.6p1-4)
                  依赖: openssh-sftp-server 但是它将不会被安装
                  推荐: ssh-import-id 但是它将不会被安装
E: 无法修正错误，因为您要求某些软件包保持现状，就是它们破坏了软件包间的依赖关系。
```

手动解决依赖，这里是因为 openssh-server 是依赖于 openssh-clien 的，那我们手动安装下指定的版本的 openssh-client

```bash
# 手动解决依赖
apt install openssh-client=1:7.6p1-4

# 重新安装openssh-server
apt install openssh-server
```

#### CentOS

```bash
# 安装SSH
yum install openssh-server
```

#### Kali/Debian

```bash
apt install openssh-server
```

### SSH 自启

```bash
# 开机自动启动ssh命令
sudo systemctl enable ssh

# 关闭ssh开机自动启动命令
sudo systemctl disable ssh
```

### 允许 root 远程登陆

#### Ubuntu

```bash
# 允许root远程登陆
echo "PermitRootLogin yes" >> /etc/ssh/sshd_config

# SSH开机自启
systemctl enable ssh

# 开启SSH
/etc/init.d/ssh start
```

#### CentOS/Debian/Kali

```bash
# 允许root远程登陆
echo "PermitRootLogin yes" >> /etc/ssh/sshd_config

# SSH开机自启
systemctl enable sshd

# 开启SSH
systemctl start sshd
```

### ssh 连接非 22 端口

```bash
ssh -p 端口 x.x.x.x
```

### ssh 使用公私钥登陆

```bash
# 生成公私钥
ssh-keygen

#     到ssh公私钥放的文件夹
cd /root/.ssh/

# 将公钥的内容拷贝到authorized_keys文件中
cat id_rsa.pub >> authorized_keys

# 关闭密码登陆
echo "PasswordAuthentication no" >> /etc/ssh/sshd_config

# 重启ssh
/etc/init.d/ssh restart
```

效果如下：

![](https://dn-coding-net-tweet.codehub.cn/photo/2019/4506c862-c539-4ebb-9b38-9dd78a31ecd3.jpg)

将服务器的 rsa 私钥下载下来，然后拷贝到自己的机器上。此时必须使用私钥匙登陆才可以成功：

```bash
# 将拷贝下来的私钥设置600权限
chmod 600 ~/Downloads/test_rsa

# 使用私钥登陆
ssh -i ~/Downloads/test_rsa root@10.211.55.9
```

![](https://dn-coding-net-tweet.codehub.cn/photo/2019/2eebb0ab-5c06-4333-8e23-7e08b77cfa5d.jpg)

### 一台电脑保存多 SSH KEY

假设有这样一个场景

| 服务器 IP  | 私钥位置            |
| :------ | :-------------- |
| 1.1.1.1 | ~/.ssh/id_rsa_a |
| 2.2.2.2 | ~/.ssh/id_rsa_b |

想要自己的电脑保存这两台服务器的私钥的话，可以这样配置：

在 `~/.ssh` 目录下创建 `config` 文件：

```bash
vim ~/.ssh/config
```

输入以下信息：

```bash
Host 1.1.1.1
    IdentityFile ~/.ssh/id_rsa_a

Host 2.2.2.2
    IdentityFile ~/.ssh/id_rsa_b
```

然后设置私钥的权限：

```bash
chmod 600 ~/.ssh/id_rsa_a
chmod 600 ~/.ssh/id_rsa_b
```

### SSH 取消公私钥登陆

```bash
# 清空.ssh文件夹
rm -rf /root/.ssh/*

# 允许密码登陆
echo "PasswordAuthentication yes" >> /etc/ssh/sshd_config

# 重启SSH服务
/etc/init.d/ssh restart

# CentOS 服务器这样重启
systemctl restart sshd
```

### SCP 拷贝文件

```bash
scp 本地文件 用户名@服务器地址:要拷贝的路径
```

![](https://dn-coding-net-tweet.codehub.cn/photo/2019/8f43f3dc-2117-4171-9295-a126dbb05205.jpg)

### SCP 拷贝非 22 端口文件

```bash
scp -P 端口 本地文件 用户名@服务器地址:要拷贝的路径
```

## FTP

### 连接 FTP

**最基本的连接方式**:

```bash
ftp ip
```

**FTP 连接非默认端口**：

```bash
ftp ip port
```

或者：

```bash
➜  ~ ftp
ftp> open ip port
```

一个完整的登录示例：

```bash
➜  ~ ftp 116.xx.xx.xx 21
Connected to 116.xx.xx.xx.
220 Welcome to www.net.cn FTP service.
Name (116.xx.xx.xx:sqlsec): hyuxxxxxxxxxx
331 Please specify the password.
Password:
230 Login successful.
ftp>
```

### 基本操作

#### 目录相关

```bash
pwd               # 显示远程计算机上的当前目录
ls/dir            # 列出当前远程目录的内容，可以使用该命令在Linux下的任何合法的ls选项
cd                # 移动到cd 后的目录
cdup/cd ..        # 返回上一级目录
lcd               # 列出当前本地目录路径

mkdir             # 在远程系统中创建目录
rname             # 重命名一个文件或目录
redir             # 删除远程目录
delete            # 删除远程文件
mdelete           # 删除多个远程文件
```

#### 文件上传下载

```bash
binary            # 用于二进制文件传送（图像文件等）
ascii             # 用于文本文件传送
get/mget          # 在当前远程目录下复制（一个/多个）文件到本地文件系统当前目录
put/mput          # 从当前目录把文件复制到当前远程目录
```

#### 退出

```bash
!                 # 临时退出ftp模式，返回本地Linux Shell模式，键入exit返回
close             # 关闭当前连接
bye/quit          # 关闭连接并退出ftp命令模式
```

## JDK

### OracleJDK

因为 JDK 官网需要登录验证才可以下载的原因，不能直接通过 wget 来下载 tar.gz 安装包了，得自己手动 copy 一下。

```bash
# 解压到/usr/lib/目录下
sudo tar -zxvf jdk-8u221-linux-x64.tar.gz -C /usr/lib/

# 编辑/etc/profile 配置文件
sudo vim /etc/profile
```

末尾添加如下内容：

```bash
JAVA_HOME=/usr/lib/jdk1.8.0_221
PATH=$JAVA_HOME/bin:$PATH
CLASSPATH=$JAVA_HOME/jre/lib/ext:$JAVA_HOME/lib/tools.jar
export PATH JAVA_HOME CLASSPATH
```

执行命令

```bash
# 重新载入配置文件
source /etc/profile

# 系统注册此 JDK 300 是一个整数，在自动模式下，这个数字越高的选项，其优先级也就越高
sudo update-alternatives --install /usr/bin/java java /usr/lib/jdk1.8.0_221/bin/java 300

# 验证是否安装成功
java -version
```

### OpenJDK

```bash
# 搜索软件列表里的 JDK
apt search openjdk

# 安装 JDK
apt install openjdk-11-jdk

# 验证是否安装成功
java -version
```

### 版本切换

安装了多个版本的 JDK，你可以通过以下命令在这些版本之间切换：

```bash
sudo update-alternatives --config java
```

[![](https://image.3001.net/images/20201011/16024064392787.png)](https://image.3001.net/images/20201011/16024064392787.png)

## Python

### Centos 7

Centos7 自带 Python2 版本和 PIP2，所以我们只要直接安装 Python3 即可：

```bash
# 搜索可用的Python3版本
yum search python3

# 安装Python3.6版本
yum install python36

# 查看Python版本
python3 -V
Python 3.6.8

# 查看pip版本
pip3 -V
pip 9.0.3 from /usr/lib/python3.6/site-packages (python 3.6)

# 升级pip版本
pip install --upgrade pip
pip3 install --upgrade pip

# 安装ipython
pip install ipython
pip3 install ipython
```

#### 安装 pyenv

pyenv 是 Python 多版本管理神器，安装起来也很简单：

```bash
# 安装基础依赖
yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel -y

# 下载pyenv
git clone https://github.com/pyenv/pyenv.git ~/.pyenv

# 配置pyenv
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(pyenv init -)"' >> ~/.zshrc
zsh
```

基础入门使用可以参考我的这篇文章 [macOS pyenv 入门使用记录](https://www.sqlsec.com/2019/12/pyenv.html#toc-heading-4)

### Ubuntu

**pip3 安装**

```bash
sudo apt update
sudo apt install python3-pip
```

## 搜索替换

### grep 搜索文件内容

```bash
# 搜索/home/sqlsec/Desktop/BBS/中的所有 ph p后缀中的 password 关键词
grep -r "password" --include="*.php"  /home/sqlsec/Desktop/BBS/

# 可以添加多个后缀
grep -r "关键词" --include="*.后缀1"  --include="*.后缀2" 目标路径

# 搜索/home/sqlsec/Desktop/BBS/中的所有php后缀中的password关键词
# 不搜索js后缀
grep -r "password" --include="*.php" --exclude="*.js"  /home/sqlsec/Desktop/BBS/
```

### find 查找文件名

```bash
# 查找当前目录下所有 .php 的文件
find . -name "*.php"

# 查找所有文件中的 log 文件
find / -name "*.log"

# 目前目录及其子目录下所有最近 20 天内更新过的文件列出
find . -ctime -20

# 查找系统中所有文件长度为0的普通文件，并列出它们的完整路径
find / -type f -size 0 -exec ls -l {} \
```

### 查找内容并替换

```bash
# 将 file_name文件中的 old 全部替换为 new
sed -i 's/old/new/g'  file_name

# 修改 MySQL 的默认端口为 3307
sed -i 's/3306/3307/g' my.cnf

# 批量替换文件夹内容
sed -i "s/old/new/g" `grep "old" -rl floder_path`
```

## 用户

### 修改用户密码

```bash
# 修改当前用户密码
passwd

# 修改指定用户密码
passwd 用户名

# 修改root密码
sudo passwd
```

### adduser 新建用户

```bash
adduser test 
```

[![](https://dn-coding-net-tweet.codehub.cn/photo/2019/650df41a-b6fc-43fd-a982-24d268faac0a.jpg)](https://dn-coding-net-tweet.codehub.cn/photo/2019/650df41a-b6fc-43fd-a982-24d268faac0a.jpg)

### useradd 新建用户

```bash
# 创建test用户并自动生成主目录且指定shell为bash
useradd -d /home/test -m test -s /bin/bash

# 设置test用户密码
passwd test
```

### 删除用户

```bash
# 只删除用户
sudo userdel 用户名

# 连同用户主目录一块删除
sudo userdel -r 用户名
```

## MySQL

### Ubuntu

```bash
# 安装MySQL客户端
apt install mysql-client

# 安装MySQL服务端
apt install mysql-server

# 启动MySQL
/etc/init.d/mysql start

# 初始化MySQL
sudo mysql_secure_installation
```

### Debian/Kali

```bash
# 安装MySQL客户端
apt install mariadb-client

# 安装MySQL服务端
apt install mariadb-server

# 启动MySQL
/etc/init.d/mysql start

# 初始化MySQL
sudo mysql_secure_installation
```

### CentOS

```bash
# 安装客户端和服务端 默认root密码为空
yum install mariadb-server

# 启动 mariadb
sudo systemctl start mariadb

# 设置开机自启
sudo systemctl enable mariadb

# 查看mariadb状态
sudo systemctl status mariadb

# 初始化MySQL
sudo mysql_secure_installation
```

### 允许 root 远程登陆

```SQL
# 允许root外部访问连接
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '这里是root密码' WITH GRANT OPTION;

# 刷新权限
FLUSH PRIVILEGES; 
```

查看此时的用户状态：

```sql
mysql> select user, host from mysql.user;
+------+-----------+
| user | host      |
+------+-----------+
| root | %         |
| root | 127.0.0.1 |
| root | ::1       |
| root | localhost |
+------+-----------+
```

### 关闭 root 远程登录

```bash
# 删除 host 为 % 的用户
DELETE FROM mysql.user WHERE User= "root" and Host="%";

# 刷新权限
FLUSH PRIVILEGES; 
```

查看此时的用户状态：

```sql
mysql> select user, host from mysql.user;
+------+-----------+
| user | host      |
+------+-----------+
| root | 127.0.0.1 |
| root | ::1       |
| root | localhost |
+------+-----------+
```

### 修改密码

```bash
# 修改 root 密码的 SQL语句
use mysql;
set password for 'root'@'localhost' = password('你设置的密码');

# 刷新权限 并退出
flush privileges;
quit; 
```

或者直接命令行下修改：

```bash
mysqladmin -uroot -poldpassword password newpassword
```

### 连接非 3306 默认端口

```bash
mysql -h 目标IP地址 -u 用户名 -p 密码 -P 端口
mysql -h 10.211.55.9 -u root -p password -P 33060
```

### 导出 SQL 文件

```bash
# 导出所有数据库
mysqldump -uroot -proot --all-databases >/tmp/all.sql

# 导出 db1、db2 两个数据库的所有数据
mysqldump -uroot -proot --databases db1 db2 >/tmp/user.sql

# 导出 db1 中的 a1、a2 表
mysqldump -uroot -proot --databases db1 --tables a1 a2  >/tmp/db1.sql

# 导出 db1 表 a1 中 id=1 的数据
mysqldump -uroot -proot --databases db1 --tables a1 --where='id=1'  >/tmp/a1.sql

# 只导出表结构不导出数据
mysqldump -uroot -proot --no-data --databases db1 >/tmp/db1.sql
```

### 导入 SQL 文件

```bash
mysql -uroot -proot < databases.sql
```

### 创建用户

```mysql
CREATE USER 'test'@'%' IDENTIFIED BY 'test用户的密码';
GRANT ALL PRIVILEGES ON `test`.* TO 'test'@'%';
```

## 进程

### 查看进程

```bash
ps -a
ps -au
ps -aux
ps -ef
```

### 杀掉进程

```bash
kill -9 pid号
```

### 杀掉端口进程

```bash
# 手工查找 3306 的 PID
netstat -nlp | grep :3306

# 读取 3306 进程的 PID 号
netstat -nlp | grep :3306 | awk '{print $7}' | awk -F"/" '{ print $1 }'

# 杀掉 PID
kill -9 PID
```

也可以使用下面 1 条命令解决：

```bash
kill [']netstat -nlp | grep :3306 | awk '{print $7}' | awk -F"/" '{ print $1 }'[']
```

## Docker

### 安装

#### CentOS 7

以下所有操作都是以 root 用户执行

```bash
# 更新到最新 yum 包
yum update -y

# 安装Docker以及Docker-compose
yum install -y yum-utils device-mapper-persistent-data lvm2 \
&& yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo \
&& yum install  -y docker-ce docker-ce-cli containerd.io \
&& systemctl start docker \
&& yum -y install epel-release \
&& yum -y install python-pip \
&& pip install --upgrade pip \
&& pip install docker-compose


# 查看Docker版本
$ docker version
Client: Docker Engine - Community
 Version:           19.03.5
 API version:       1.40
 Go version:        go1.12.12
 Git commit:        633a0ea
 Built:             Wed Nov 13 07:25:41 2019
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.5
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.12.12
  Git commit:       633a0ea
  Built:            Wed Nov 13 07:24:18 2019
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.2.10
  GitCommit:        b34a5c8af56e510852c35414db4c1f4fa6172339
 runc:
  Version:          1.0.0-rc8+dev
  GitCommit:        3e425f80a8c931f88e6d94a8c831b9d5aa481657
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683

# 查看 Docker-compose 版本
$ docker-compose version
docker-compose version 1.25.4, build unknown
docker-py version: 4.1.0
CPython version: 3.6.8
OpenSSL version: OpenSSL 1.0.2k-fips  26 Jan 201
```

#### Ubuntu

使用官方自动安装脚本，美滋滋：

```bash
sudo apt instal curl

# 自动安装脚本
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun

# 将当前用户添加到 docker 用户组里
sudo usermod -aG docker `whoami`

# 重启生效
reboot

# 检验版本输出是否正常
$ docker version
Client: Docker Engine - Community
 Version:           19.03.12
 API version:       1.40
 Go version:        go1.13.10
 Git commit:        48a66213fe
 Built:             Mon Jun 22 15:45:44 2020
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.12
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.13.10
  Git commit:       48a66213fe
  Built:            Mon Jun 22 15:44:15 2020
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.2.13
  GitCommit:        7ad184331fa3e55e52b890ea95e65ba581ae3429
 runc:
  Version:          1.0.0-rc10
  GitCommit:        dc9208a3303feef5b3839f4323d9beb36df0a9dd
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683
```

Docker 安装完成后那么开始准备安装 Docker Compose 了，首先去官网查看最新的稳定版为多少：

[https://github.com/docker/compose/releases](https://github.com/docker/compose/releases)

然后手动下载最新版本 赋予执行权限即可：

```bash
# 下载我写这篇文章的最新版本 1.27.2
sudo curl -L "https://github.com/docker/compose/releases/download/1.27.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# 赋予执行权限
sudo chmod +x /usr/local/bin/docker-compose

# 查看 Docker-compose 版本
$ docker-compose version
docker-compose version 1.27.2, build 18f557f9
docker-py version: 4.3.1
CPython version: 3.7.7
OpenSSL version: OpenSSL 1.1.0l  10 Sep 2019
```

国内 Docker 加速还是很有必要配置一下的，编辑一下配置文件：

```bash
sudo vim /etc/docker/daemon.json
{
  "registry-mirrors": ["https://xxxxxxxx.mirror.aliyuncs.com"]
}
```

内容如下，修改完后重启一下 Docker 服务：

```bash
# 配置完成重启 Docker 服务 让配置生效
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 入门使用

[Docker 温故知新](https://www.sqlsec.com/2019/10/docker2.html)

## 压缩与解压

### 压缩

```bash
# tar
tar zcvf FileName.tar DirName

# tar.bz2
tar jcvf x.tar xxxx

# gz
gzip FileName

# Z
compress FileName

# tar.Z
tar Zcvf FileName.tar.Z DirName

# zip
zip -r FileName.zip DirName

# war 将 shell.jsp 压缩为 shell.war
jar cvf shell.war shell.jsp
```

### 解压

```bash
# tar
tar zxvf FileName.tar

# tar.bz2
bzip2 -d FileName.bz2
bunzip2 FileName.bz2

# gz
gzip -d FileName.gz
gunzip FileName.gz

# Z
uncompress FileName.Z

# tar.Z
tar Zxvf FileName.tar.Z

# zip
unzip FileName.zip
```

## 常见案例

### CentOS 7 升级 OpenSSH

CentOS 自带的 SSH 版本为 OpenSSH 7.4 可以升级到 8 点几的版本。

#### 下载 SSH 包

OpenSSH 软件包：[https://ftp.riken.jp/pub/OpenBSD/OpenSSH/portable/](https://ftp.riken.jp/pub/OpenBSD/OpenSSH/portable/)

```bash
wget https://ftp.riken.jp/pub/OpenBSD/OpenSSH/portable/openssh-8.3p1.tar.gz
```

#### 编译安装

解压到 `/usr/local/src/` 目录下

```bash
tar -zxvf openssh-8.3p1.tar.gz -C /usr/local/src
```

然后进入到 OpenSSH 解压后的目录下 编译安装：

```bash
# 进入到目录
cd /usr/local/src/openssh-8.3p1

# 编译安装
./configure --prefix=/usr/local/openssh --with-ssl-dir=/usr/local/ssl --without-openssl-header-check
make && make install
```

#### 修改配置

```bash
# 添加 SSH 配置
echo "PermitRootLogin yes" >> /usr/local/openssh/etc/sshd_config
echo "PubkeyAuthentication yes" >> /usr/local/openssh/etc/sshd_config
echo "PasswordAuthentication yes" >> /usr/local/openssh/etc/sshd_config                                                                

# 拷贝配置文件
cp /usr/local/src/openssh-8.3p1/contrib/redhat/sshd.init /etc/init.d/sshd

# SSH 服务
chkconfig --add sshd

mv /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
cp /usr/local/openssh/etc/sshd_config /etc/ssh/sshd_config

# 提示覆盖 覆盖掉
cp /usr/local/openssh/sbin/* /usr/sbin/sshd -f

# 这一步提示覆盖 都覆盖掉
cp /usr/local/openssh/bin/* /usr/bin/
```

#### 重启验证

```bash
# 重启服务器
reboot

# 注意是大写V
ssh -V
```

[![](https://image.3001.net/images/20201021/16032852194860.png)](https://image.3001.net/images/20201021/16032852194860.png)

## 支持一下

本文可能实际上也没有啥技术含量，但是写起来还是比较浪费时间的，在这个喧嚣浮躁的时代，个人博客越来越没有人看了，写博客感觉一直是用爱发电的状态。如果你恰巧财力雄厚，感觉本文对你有所帮助的话，可以考虑打赏一下本文，用以维持高昂的服务器运营费用（域名费用、服务器费用、CDN 费用等）
