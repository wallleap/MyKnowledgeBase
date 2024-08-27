---
title: mac 本地 Git 服务器
date: 2023-05-03T10:07:07+08:00
updated: 2024-08-21T11:18:34+08:00
dg-publish: false
---

[mac搭建gitosis服务器_gitosis 服务器搭建-CSDN博客](https://blog.csdn.net/qq_37156624/article/details/109593373)

重点注意 Python 版本是 2

## 一、创建 git 账户

1.点击 系统编号设置 -> 用户与群组，添加一个名为 git 的管理员账户，按步骤填写就行。

![](https://cdn.wallleap.cn/img/pic/illustration/202408211117586.png?imageSlim)

2.转到新创建的 git 用户，设置远程登录权限：系统偏好设置 -> 共享 -> 远程登录，这里选择的是所有用户。

![](https://cdn.wallleap.cn/img/pic/illustration/202408211117588.png?imageSlim)

注意：添加权限设置的目的是为了让其他电脑通过 ssh 登录 git 账户来连接 git 服务器，后面会有相关步骤。

## 二、下载安装 gitosis

1.Mac OSX Yosemite 默认已经为我们安装了 Git 和 Python，直接命令行下载 gitosis。

```cobol
yourname:~ git$ git clone git://github.com/res0nat0r/gitosis.git

Cloning into gitosis

remote: Counting objects: 614, done.

remote: Compressing objects: 100% (183/183), done.

remote: Total 614 (delta 434), reused 594 (delta 422)

Receiving objects: 100% (614/614), 93.82 KiB | 45 KiB/s, done.

Resolving deltas: 100% (434/434), done.
```

2.进入 gitosis 目录，使用命令  "sudo python setup.py install" 来 执行 python 脚本 来安装 gitosis。

```cobol
yourname:~ git$ cd gitosis/

yourname:gitosis git$ sudo python setup.py install

running install

running bdist_egg

running egg_info

creating gitosis.egg-info

……

Using /Library/Python/2.6/site-packages/setuptools-0.6c9-py2.6.egg

Finished processing dependencies for gitosis==0.2
```

## 三、制作 ssh rsa 公钥

1.回到 client 机器上，制作 ssh 公钥。在这里我的使用同一台机器上的另一个账户作为  client 。如果作为  client  的机器与作为 server 的机器不是同一台，也是类型的流程：制作公钥，放置到服务的 /tmp 目录下。只不过在同一台机     器上，我们可以通过开启另一个 terminal，使用 su 切换到 li 账户就可以同时操作两个账户。

```cobol
yourname:~git$ su li

Password:

bash-3.2$ cd ~

bash-3.2$ ssh-keygen -t rsa

Generating public/private rsa key pair.

Enter file in which to save the key (/Users/li/.ssh/id_rsa):

Enter passphrase (empty for no passphrase):

Enter same passphrase again:

Your identification has been saved in /Users/li/.ssh/id_rsa.

Your public key has been saved in /Users/li/.ssh/id_rsa.pub.

bash-3.2$ cd .ssh

bash-3.2$ ls

id_rsa id_rsa.pub

bash-3.2$ cp id_rsa.pub /tmp/

bash-3.2$ cd /tmp/

bash-3.2$ mv id_rsa.pub yourame.pub
```

在上面的命令里，首先通过 su 切换到 local 账户（只有在同一台机器上才有效），然后进入到 li 账户的 home 目录，使用 ssh-keygen -t rsa 生成 id\_rsa.pub，最后将该文件拷贝放置到  /tmp/yourname.pub，这样 git 账户就可以访问 yourname.pub 了，在这里改名是为了便于在 git 中辨识多个 client。

## 四、使用 ssh 公钥初始化 gitosis

1.不论你是以那种方式（邮件，usb 等等）拷贝 yourname.pub 至服务器的 /tmp/yourname.pub。下面的流程都是一样，登入服务器机器的 git 账户，进入先前提到 gitosis 目录，进行如下操作初始化 gitosis，初始化完成后，会在 git       的 home 下创建 repositories 目录。

```cobol
yourname:gitosis git$ sudo -H -u git gitosis-init < /tmp/yourname.pub

Initialized empty Git repository in /Users/git/repositories/gitosis-admin.git/

Reinitialized existing Git repository in /Users/git/repositories/gitosis-admin.git/
```

在这里，会将该 client 当做认证受信任的账户，因此在 [Git](http://lib.csdn.net/base/git) 的 home 目录下会有记录，文件 authorized\_keys 的内容与 yourname.pub 差不多。

```crystal
yourname:~git$ cd ~/.ssh/

yourname:.ssh git$ ls

authorized_keys
```

我们需要将 authorizd\_keys 稍做修改，用文本编辑器打开它，删除里面的 " command="gitosis-serve yourname",no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty  " 这一行：

```crystal
yourname:.ssh git$ vim authorized_keys
```

然后，我们对 post-update 赋予可写权限，以便 client 端可以提交更改。

```cobol
yourname:gitosis git$ sudo chmod -R 755 /Users/git/repositories//gitosis-admin.git/hooks/post-update

Password:

yourname:.ssh git$ cd ~

yourname:~ git$ cd repositories/

yourname:repositories git$ ls

gitosis-admin.git

yourname:repositories git$
```

在上面的命令中可以看到，gitosis 也是作为仓库的形式给出，我们可以在其他账户下 checkout，然后对 gitosis 进行配置管理等等，而无需使用服务器的 git 账户进行。

最后一步，修改 git 账户的 PATH 路径。

```cobol
yourname:gitosis git$ touch ~/.bashrc

yourname:gitosis git$ echo PATH=/usr/local/bin:/usr/local/git/bin:\$PATH>.bashrc

yourname:gitosis git$ echo export PATH >> .bashrc

yourname:gitosis git$ cat .bashrc

PATH=/usr/local/bin:/usr/local/git/bin:$PATH

export PATH
```

至此，服务器的配置完成。

## 五、client 配置

1.回到 li 账户，首先在 terminal 输入如下命令修改 li 账户 的 git 配置：

```cobol
bash -3.2$ git config --global user.name "li"

bash -3.2$ git config --global user.email "li@qq.com"
```

这里 ”li“ 表示你的 gitname，”li@qq.com“ 表示你的 git 账号

2.测试服务器是否连接正确，将 10.73.250.202 换成你服务的名称或服务器地址即可。

```cobol
yourname:~li$ ssh git@10.73.250.202

Last login: Mon Nov 7 13:11:38 2011 from 10.73.250.202
```

3.在本地 clone 服务器仓库，下面以 gitosis-admin.git 为例：

```cobol
bash-3.2$ git clone git@10.73.250.202:repositories/gitosis-admin.git

Cloning into gitosis-admin

remote: Counting objects: 5, done.

remote: Compressing objects: 100% ( 5/ 5), done.

remote: Total 5 (delta 0), reused 5 (delta 0)

Receiving objects: 100% ( 5/ 5), done.

bash- 3.2$ ls

Desktop InstallApp Music Sites

Documents Library Pictures gitosis - admin

Downloads Movies Public

bash-3.2$git
```

在上面的输出中可以看到，我们已经成功 clone 服务器的 gitosis-admin 仓库至本地了。

4.在本地管理 gitosis-admin：

进入 gitosis-admin 目录，我们来查看一下其目录结构：gitosis.conf 文件是一个配置文件，里面定义哪些用户可以访问哪些仓库，我们可以修改这个配置；keydir 是存放 ssh 公钥的地方。

```cobol
bash-3.2$ cd gitosis-admin/

bash-3.2$ ls

gitosis.conf keydir

bash-3.2$ cd keydir/

bash-3.2$ ls

yourname.pub
```

到这里说明我们已经创建好了，接下来就要根据自己的需求来创建仓库添加其他机器了。

在服务器 终端进入到 repositories 目录

```bash
cd ~/repositories

mkdir ssm.git

cd ssm.git

git --bare init
```

然后就可以 git clone git@10.73.250.202:repositories/ssm.git
