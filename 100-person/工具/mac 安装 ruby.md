---
title: mac 安装 ruby
date: 2024-05-13 16:03
updated: 2024-05-13 16:32
---

## 安装 rvm

```sh
curl -L https://get.rvm.io | bash -s stable
source /Users/luwang/.rvm/scripts/rvm
rvm -v
```

查看源

```sh
gem sources -l
```

删除旧的 `ruby` 源

```sh
gem sources --remove https://rubygems.org/
```

添加国内的源, 添加完成后再查看一下是否安装成功

```sh
gem sources --add https://gems.ruby-china.com/
```

## 安装 ruby

`rvm` 是 `ruby` 的安装和版本管理工具. 当然也可以使用 `brew` 来安装 `ruby`

查看已知的 `ruby` 版本

```sh
rvm list known
```

查询已安装的 `ruby` 版本

```sh
rvm list
```

使用 `rvm` 安装指定版本的 `ruby`

```sh
rvm install 3.0.0
```

卸载一个已安装版本

```sh
rvm remove 3.0.0
```

查看版本

```sh
ruby -v
rvm use 3.0.0 --default
```

> 如果 `rvm` 没有安装成功, 可以使用最新版本的 `homebrew` 安装, 安装命令: `brew install ruby`

```sh
brew --prefix openssl@3
> /opt/homebrew/opt/openssl@3
rvm reinstall 3.0.0 --with-openssl-dir=/opt/homebrew/opt/openssl@3
```
