---
title: 前言
date: 2024-08-26T09:54:56+08:00
updated: 2024-08-26T09:56:40+08:00
dg-publish: false
source: https://www.sqlsec.com/2019/12/pyenv.html
---

有时候开发需要在不同版本的 Python 中切换，这个时候就需要神器 pyenv 来简化我们的操作了，下面国光就来简单介绍一下入门操作，大道至简。

## 前言

我们可以经常会问 Python2 还是 Pyton3 ？？或者 Python 3.6 还是 Python 3.8 ？？小孩子才做选择，我们当然是…

pyenv 是 Python 版本管理工具，可以简化我们在不同的版本之间切换的过程，Python 开发必备神器。

项目地址：[https://github.com/pyenv/pyenv](https://github.com/pyenv/pyenv)

## 教程

### 安装

macOS 下可以使用神器 Homebrew 来安装 pyenv

```bash
brew update
brew install pyenv
```

- **zsh 用户**

```bash
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(pyenv init -)"' >> ~/.zshrc
zsh
```

- **bash 用户**

```bash
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init -)"' >> ~/.bashrc
bash
```

### 版本安装

```bash
# 查看已经安装的Python版本
pyenv versions

# 查看当前的 Python 版本
pyenv version

# 查看可安装的版本
pyenv install -l

# 安装与卸载 python 3.6.6
pyenv install 3.6.6
pyenv uninstall 3.6.6
```

这里如果安装的时候可能会比较卡，可以考虑配合 proxychains 来使用：

```bash
PYTHON_CONFIGURE_OPTS="--disable-ipv6" proxychains4 pyenv install <python版本>
```

安装的版本会在 `~/.pyenv/versions` 目录下

### 版本切换

```bash
# global 全局设置 一般不建议改变全局设置
pyenv global <python版本>

# shell 会话设置 只影响当前的shell会话
pyenv shell <python版本>
# 取消 shell 会话的设置
pyenv shell --unset

# local 本地设置 只影响所在文件夹
pyenv local <python版本>
```

pyenv 的 global、local、shell 的优先级关系是：shell > local > global

## 加速

### 卡顿日常

```bash
➜  3.6.6 pyenv install pypy3.6-7.3.0
Downloading pypy3.6-v7.3.0-osx64.tar.bz2...
-> https://bitbucket.org/pypy/pypy/downloads/pypy3.6-v7.3.0-osx64.tar.bz2

failed to download pypy3.6-v7.3.0-osx64.tar.bz2
BUILD FAILED (OS X 10.15.3 using 0000000000)
Status Legend:
(INPR):download in-progress.

aria2 will resume download if the transfer is restarted.
If there are any errors, then see the log file. See '-l' option in help/man page for details.
```

可以看到下载 [https://bitbucket.org/pypy/pypy/downloads/pypy3.6-v7.3.0-osx64.tar.bz2](https://bitbucket.org/pypy/pypy/downloads/pypy3.6-v7.3.0-osx64.tar.bz2) 这个压缩包的时候卡顿了，实际上我们浏览器走一下代理可以很快下载下来。

### 手工加速

pyenv 安装 Python 版本的时候下载速度缓慢卡顿，这个时候我们得手动来加速一下：

```bash
# 新建cache缓存目录
➜ mkdir ~/.pyenv/cache

# 将下载好的 压缩包 移动到 cache目录下
➜ mv ~/Downloads/pypy3.6-v7.3.0-osx64.tar.bz2 ~/.pyenv/cache

# 再次安装 就会很快 秒装了
➜ pyenv install pypy3.6-7.3.0
Installing pypy3.6-v7.3.0-osx64...
Installed pypy3.6-v7.3.0-osx64 to /Users/sqlsec/.pyenv/versions/pypy3.6-7.3.0

# 安装完成手动清空cache目录
➜ rm -rf ~/.pyenv/cache/*
zsh: sure you want to delete the only file in /Users/sqlsec/.pyenv/cache [yn]? y

# 查看是否安装成功
➜ pyenv versions
* system (set by /Users/sqlsec/.python-version)
  3.6.6
  3.8.0
  pypy3.6-7.3.0
```

## 总结

pyenv Python 开发者必备，强烈建议大家自己安装体验一下。
