---
title: 给你的 npm 包增加更新检测
date: 2023-08-03T03:50:00+08:00
updated: 2024-08-21T10:32:30+08:00
dg-publish: false
aliases:
  - 给你的 npm 包增加更新检测
author: Luwang
category: web
comments: true
cover: //cdn.wallleap.cn/img/post/1.jpg
description: 文章描述
image-auto-upload: true
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
rating: 1
source: #
tags:
  - Node
  - web
url: //myblog.wallleap.cn/post/1
---

**本文参加了由**[公众号@若川视野](https://link.juejin.cn/?target=https%3A%2F%2Flxchuan12.gitee.io "https://link.juejin.cn/?target=https%3A%2F%2Flxchuan12.gitee.io") **发起的每周源码共读活动**， [点击了解详情一起参与。](https://juejin.cn/post/7079706017579139102 "https://juejin.cn/post/7079706017579139102")

这是源码共读的 [**第6期**](https://juejin.cn/post/7084985851347730446 "https://juejin.cn/post/7084985851347730446") | update-notifier 检测 npm 包是否更新

## 前言

我们在开发一个轮子或者 CLI 工具时，大部分情况都不会去考虑后续用户使用时的版本检测操作，`update-notifier` 非常精简的实现了版本的自动检测和更新提示，接下来会细细道来其中的实现原理~

## 使用

使用 `npm init -y` 创建一个库，接着将库的名字改为 `public-ip` 用于测试。 然后将以下代码执行两次就可以看到回显

```js
import updateNotifier from "update-notifier";
//import packageJson from './package.json' assert {type: 'json'};
//手动读取 packageJson
import fs from "node:fs";

const pkgFile = fs.readFileSync("./package.json", { encoding: "utf-8" });
const pkgJson = JSON.parse(pkgFile.toString());
new updateNotifier({ pkg: pkgJson, updateCheckInterval: 0 }).notify();
```

## 入口

老规矩，在 `package.json` 中 我们可以明确其执行路径为： `"exports": "./index.js"`

```js
import UpdateNotifier from "./update-notifier.js";

export default function updateNotifier(options) {
const updateNotifier = new UpdateNotifier(options);
updateNotifier.check();
return updateNotifier;
}
```

### 初始化

```js
constructor(options = {}) {
    //省略若干初始化参数代码
    
    this.#updateCheckInterval =
    typeof options.updateCheckInterval === "number"
            ? options.updateCheckInterval
            : ONE_DAY;
if (!this.#isDisabled) {
        try {
                this.config = new ConfigStore(`update-notifier-${this._packageName}`, {
                        optOut: false,
                        // Init with the current time so the first check is only
                        // after the set interval, so not to bother users right away
                        lastUpdateCheck: Date.now(),
                });
        } catch {
                // Expecting error code EACCES or EPERM
                const message =
                        chalk.yellow(format(" %s update check failed ", options.pkg.name)) +
                        format("\n Try running with %s or get access ", chalk.cyan("sudo")) +
                        "\n to the local update config store via \n" +
                        chalk.cyan(
                                format(" sudo chown -R $USER:$(id -gn $USER) %s ", xdgConfig)
                        );

                process.on("exit", () => {
                        console.error(boxen(message, { textAlignment: "center" }));
                });
        }
}
}
```

这里可以看到熟悉的库 `ConfigStore`，上一期也讲过这个库，其作用就是数据本地文件持久化。 这一段代码会将按照我们传递要检测的包作为文件名，然后将执行时的时间作为最后一次检测时间存储进去。

> 1.  如果是第一次使用这个库执行检测更新，那么不会有任何结果，仅仅是创建了这个文件并存储
> 2.  另外如果你没有指定 updateCheckInterval 参数，那么它默认只有在一天之后再会去做比较执行
> 3.  所以你可以通过改变本地时间来达到提前检测的目的 (嘿嘿)

接着回到入口处它会接着执行 `updateNotifier.check();`

### check

```js
check() {
    if (!this.config || this.config.get("optOut") || this.#isDisabled) {
            return;
    }

    this.update = this.config.get("update");

    if (this.update) {
            // Use the real latest version instead of the cached one
            this.update.current = this.#packageVersion;

            // Clear cached information
            this.config.delete("update");
    }
    //如果现在时间减去上一次存储的时间 小于 检测间隔则啥也不干
    // Only check for updates on a set interval
    if (
            Date.now() - this.config.get("lastUpdateCheck") <
            this.#updateCheckInterval
    ) {
            return;
    }
    // spawn 子进程执行命令 process.execPath 得到node执行路径 即node命令 然后执行文件是 当前目录下的 check.js 执行参数
    // Spawn a detached process, passing the options as an environment property
    spawn(
            process.execPath,
            [path.join(__dirname, "check.js"), JSON.stringify(this.#options)],
            {
                    detached: true,
                    stdio: "ignore", //不输出 执行中的结果 抛出到控制台
            }
    ).unref(); //unref 父级的事件循环不将子级包括在其引用计数中
}
```

继续

```js
//取出被转为Json字符串的  对象参数 并转回对象作为参数使用
const updateNotifier = new UpdateNotifier(JSON.parse(process.argv[2]));

try {
// Exit process when offline
setTimeout(process.exit, 1000 * 30);

const update = await updateNotifier.fetchInfo();

// Only update the last update check time on success
updateNotifier.config.set("lastUpdateCheck", Date.now());

if (update.type && update.type !== "latest") {
updateNotifier.config.set("update", update);
}

// Call process exit explicitly to terminate the child process,
// otherwise the child process will run forever, according to the Node.js docs
process.exit();
} catch (error) {
console.error(error);
process.exit(1);
}
```

跟着会调用 `fetchInfo` 方法，会去获取最新的一个版本号作比较并返回构造的对象信息

> semverDiff、semver 库都是基于 semver 版本号规范的轮子，用于版本号的比较等

```js
async fetchInfo() {
    const { distTag } = this.#options;
    const latest = await latestVersion(this._packageName, { version: distTag });

    return {
            latest,
            current: this.#packageVersion,
            //更新的版本类型 是 major、patch 还是什么默认的latest
            type: semverDiff(this.#packageVersion, latest) || distTag,
            name: this._packageName,
    };
}
```

当得到返回信息后，会将检测时间进行更新，接着根据返回的版本类型 `(latest即最新的)` 是不是最新的 而决定更新 `update` 这个字段

### notify

而 notify 方法则非常简单，就是调用控制台输出的库，根据当前使用的包管理工具决定输出的信息，最终只看这个判断的执行然后就可以看到控制台的输出了

```js
if (
    !process.stdout.isTTY ||
    suppressForNpm ||
    !this.update ||
    !semver.gt(this.update.latest, this.update.current)
) {
    return this;
}
```

在 `check` 函数中已经更新了 `this.update.latest` 的值，而在初始化中更新了 `this.update.current` 的值，两者通过比较，以及其他条件的判断进行输出

### 一些其他补充

`is-ci` 默认的导出依赖了 `ci-info` 这个包， 其原理就是通过预先定义好的各种 CI 环境信息去做 process.env 的匹配，当前是否处于 CI 服务器环境下

`process.stdout.isTTY` 用于判断命令执行是否在终端环境 `suppressForNpm` 是否为 Npm Yarn `process.execPath` 执行 node 的环境变量

[devdocs.io/node~16\_lts…](https://link.juejin.cn/?target=https%3A%2F%2Fdevdocs.io%2Fnode~16_lts%2Fchild_process%23child_process.spawn() "https://devdocs.io/node~16_lts/child_process#child_process.spawn()")

`child_process.spawn(command[, args][, options])` 执行的命令 传递参数 命令参数

## 总结

梳理下最终可以得到这样一个流程：

1. 先执行初始化，创建本地文件更新第一次存储的时间
2. 每次 `check` 函数中 会先比较一次 本地时间和持久化文件中的时间，条件符合则更新一次存储时间，并请求最新的版本返回用于比较
3. 根据前面得到的版本信息进行比较，再根据当前环境决定提示信息的拼装，最后控制台输出更新提示信息
