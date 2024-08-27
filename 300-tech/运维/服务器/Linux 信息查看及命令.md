---
title: Linux 信息查看及命令
date: 2024-08-22T01:50:01+08:00
updated: 2024-08-22T01:50:35+08:00
dg-publish: false
source: https://github.com/peng-yin/note/issues/8
---

- 如何查看 linux 版本和 centos 版本号
- 如何查看内存配额
- 如何查看 CPU 核心数量以及频率
- 如何查看服务器的平均负载 (附: 什么是平均负载)
- 如何获取服务器的公网 IP 以及私网 IP
- 如何查看服务器登录的所有用户

> linux 版本和 centos 版本

```
# 查看 linux 版本
$ uname -a
Linux shanyue 3.10.0-957.21.3.el7.x86_64 #1 SMP Tue Jun 18 16:35:19 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux

# 查看 centos 版本号
$ cat /etc/centos-release
CentOS Linux release 7.6.1810 (Core)

```

> IP

```
# 公网IP
$ curl ifconfig.me
59.110.216.155

# 公网IP，上个地址的连通性不太好
$ curl icanhazip.com
59.110.216.155

# 私网IP
$ ifconfig eth0
eth0: flags=4163&lt;UP,BROADCAST,RUNNING,MULTICAST&gt;  mtu 1500
        inet 172.17.68.39  netmask 255.255.240.0  broadcast 172.17.79.255
        ether 00:16:3e:0e:01:d8  txqueuelen 1000  (Ethernet)
        RX packets 416550  bytes 505253322 (481.8 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 194374  bytes 67561825 (64.4 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```

> 登录用户

```
$ who -u
who -u
root     pts/0        Oct 18 15:04 04:25       16860 (124.200.184.74)
root     pts/2        Oct 18 18:10 01:22        2545 (124.200.184.74)
root     pts/5        Oct 18 19:33   .         24952 (124.200.184.74)

$ last -a | head -6
root     pts/5        Fri Oct 18 19:33   still logged in    124.200.184.74
root     pts/2        Fri Oct 18 18:10   still logged in    124.200.184.74
root     pts/2        Fri Oct 18 18:10 - 18:10  (00:00)     124.200.184.74
root     pts/2        Fri Oct 18 17:54 - 18:10  (00:16)     124.200.184.74
root     pts/2        Fri Oct 18 17:49 - 17:53  (00:03)     124.200.184.74
root     pts/2        Fri Oct 18 16:49 - 17:25  (00:36)     124.200.184.74

```

> 查看还有多少内存，available 指还有多少可用内存

```
# -h 指打印可视化信息
$ free -h
              total        used        free      shared  buff/cache   available
Mem:           3.7G        154M        2.1G        512K        1.5G        3.3G
Swap:            0B          0B          0B

```

> 平均负载 ( load average) 指单位时间内的平均进程数

```
$ uptime
 16:48:09 up 2 days, 23:43,  2 users,  load average: 0.01, 0.21, 0.20

```

> 查看日志

- cat service.log
- tail -f service.log
- vim serivice.log
- cat service.log | grep xxx

cat 查看小文件的文本内容

vim serivice.log

- 按 G 跳转到文件的末尾
- 按? + 关键字搜索对应的记录
- 按 n 往上查询，按 N 往下查询

> 查进程和端口

- ps -ef
- ps aux

上面两个命令都是列出所有的进程，我们还是通过 |管道和 grep 来过滤掉想要查的进程，比如说：ps -ef |grep java

把进程查出来干嘛？知道它的进程 ID 了，我们可以把他给杀掉。

- kill -9 processId：杀掉某个进程

查端口也是一个很常见的操作，常见命令：netstat -lntup：

> grep "some string" file

grep 命令在每个文件中搜索模式。它还会寻找由换行符分隔的模式，并且 grep 打印与模式匹配的每一行。

```js
#  -i选项使我们能够在给定文件中搜索字符串时不区分大小写
grep -i "REact" file
# 我们可以使用-c（count）标志找到与给定的字符串/模式匹配的行数。
grep -c "react" index.js
```

> tail 命令读取文件并输出文件的最后部分（尾巴）。

```js
tail -n 25 somefile1 somefile2
```

> find

```

//  使用find命令可以快速查找文件或目录

find path -name filename

eg: find . -name somefile

// 搜索特定类型的文件

使用find命令还可以在目录（及其子目录）中搜索相同类型的文件。例如，以下命令将搜索当前工作目录中的所有.js文件。

find . -name "*.js"

```
