---
title: 创建 plist 文件
date: 2024-10-21T02:13:12+08:00
updated: 2024-10-21T02:13:42+08:00
dg-publish: false
---

Linux 系统（如 Debian 10）上可以方便的使用 systemctl 开启守护进程，关闭守护进程，那么在 Mac 上如何设置自定义守护进程并开启、关闭它呢？本篇以 JupyterHub 为例，演示在 Mac 上开启 JupyterHub 守护进程. 本篇假设已经完成了 Miniconda，JupyterHub 和 configurable-http-proxy 的安装。

## 创建 plist 文件

创建以 `com.jupyterhub.app.plist` 命名的文件，放在 `/Users/jinzhongxu/Library/LaunchAgents` 目录下，文件内容如下

```
MAKEFILE
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
    <dict>
        <key>Label</key>
        <string>com.jupyterhub.app</string>
        <key>ProgramArguments</key>
        <array>
            <string>/usr/local/miniconda/bin/jupyterhub</string>
            <string>-f</string>
            <string>/etc/jupyterhub/config.py</string>
        </array>
        <key>KeepAlive</key>
        <true/>
        <key>StandardErrorPath</key>
        <string>/tmp/jupyterhuberr.log</string>
        <key>StandardOutPath</key>
        <string>/tmp/jupyterhubout.log</string>
        <key>EnvironmentVariables</key>
        <dict>
          <key>PATH</key>
            <string><![CDATA[/usr/local/bin:/usr/bin:/bin:/usr/local/miniconda/bin:/usr/local/node/bin]]></string>
        </dict>
		<key>WorkingDirectory</key>
    <string>/Users/jinzhongxu</string>
    </dict>
</plist>
```

各条目解释：

```
BASH
Label：对应的需要保证全局唯一性;
Program：要运行的程序;
ProgramArguments：命令语句;
StartCalendarInterval：运行的时间，单个时间点使用dict，多个时间点使用 array <dict>;
StartInterval：时间间隔，与StartCalendarInterval使用其一，单位为秒;
StandardInPath、StandardOutPath、StandardErrorPath：标准的输入输出错误文件，这里建议不要使用 .log 作为后缀，会打不开里面的信息;
定时启动任务时，如果涉及到网络，但是电脑处于睡眠状态，是执行不了的，这个时候，可以定时的启动屏幕就好了。
```

注意：

1. configurable-http-proxy 的路径需添加到 PATH 里面；
2. WorkingDirectory 需指定用户能够创建 jupyterhub_cookie_secret 的地方；
3. 如果启动不成功，可通过 jupyterhuberr.log 查看原因。

plist 可以放的目录包括

```
BASH
~/Library/LaunchAgents 由用户自己定义的任务项
/Library/LaunchAgents 由管理员为用户定义的任务项
/Library/LaunchDaemons 由管理员定义的守护进程任务项
/System/Library/LaunchAgents 由 Mac 为用户定义的任务项
/System/Library/LaunchDaemons 由 Mac 定义的守护进程任务项

/System/Library 与 /Library 以及 ~/Library 目录的区别？
/System/Library 目录是存放 Apple 自己开发的软件。
/Library 目录是系统管理员存放的第三方软件。
~/Library 是用户自己存放的第三方软件。

LaunchDaemons 和 LaunchAgents 的区别？
LaunchDaemons 是用户未登陆前就启动的服务（守护进程）。
LaunchAgents 是用户登陆后启动的服务（守护进程）。
```

## 开启守护进程

```
BASH
cd
launchctl load Library/LaunchAgents/com.jupyterhub.app.plist
```

## 关闭守护进程

```
BASH
# 如果运行 load 再次运行 load 将会出错，类似下面的错误
# Load failed: 5: Input/output error
# Try running `launchctl bootstrap` as root for richer errors.
# 此时，需要先退出，方法如下：
cd
launchctl unload Library/LaunchAgents/com.jupyterhub.app.plist
```

## 查看守护进程是否启动成功

```
BASH
# 当返回的结果第一个位置有数字而不是“-”时，说明程序正常运行
launchctl list | grep jupyterhub

# or 当列出程序进程信息时，说明程序正常运行
pstree | grep jupyterhub ｜ grep -v grep
```

## 参考链接

1. [Mac执行定时任务之Launchctl](https://www.jianshu.com/p/b65c1d339eec)
