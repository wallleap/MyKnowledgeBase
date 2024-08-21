---
title: release 后缀
date: 2024-03-22T11:24:00+08:00
updated: 2024-08-21T10:32:06+08:00
dg-publish: false
---

- **darwin**, 可以理解成 MacOS
- **armv7**, 32 位 arm
- **armv8**, 32/64 位 arm
- **armv9**, 64 位 arm
- **x86** / **i386** / **386**, 32 位 intel / AMD
- **x86_64** / **amd64** / **intel64**, 64 位 intel / AMD
- Riscv64, s390x ......

其他

- **foss**："Free and Open Source Software"
- **universal**: 按理说这个词表示该版本具有“普遍的兼容性”，你可以理解成它能在 x86 、amd64 、armv7 、armv8 下运行（跨系统还是不行），但由于没法保证开发者对“普遍”的理解是一致的，所以很难评价
- **portable**，便携版 / 绿色版。不需要安装、不修改注册表、不写入系统文件、不依赖外部组件，点击即用，可以直接从 u 盘里运行（快捷方式不行）。缺点是可能不包含安装版软件的所有功能
- **setup**，安装版。会将软件安装到指定目录、创建快捷方式、添加启动项等，依赖系统中的组件，如果缺依赖会无法运行
- **src**，源代码，通常用户不需要关心
- **debug**，用于调试的版本，通常用户不需要关心
- **minimal**，最小化发行版，通常只包含最少组件、最核心功能
- **full**，和上面对应，完整版
- **lts**，long term support ，意味着这个版本被允诺受到长期维护
- **alpha**，早期测试版本
- **beta**，比 alpha 稍微 beta 一些的测试版本
- **rc**，release candidate ，比 beta 更 beta ，接近 release 的测试版本
