---
title: 自动化发布 npm 包及生成 Github Changelog
date: 2024-08-22T03:58:20+08:00
updated: 2024-08-22T03:58:33+08:00
dg-publish: false
---

手动维护npm包容易出现一些问题：忘记编译、多提交一些无关文件、网络不通等。自动化发布能够极大地简化这个过程。

最近刚调整过一次自动化流程，可以自动发布版本，及根据Commit消息生成Changelog。实现如下图所示的效果：

[![Demo效果图](https://user-images.githubusercontent.com/2416493/80095052-e2d01680-8599-11ea-9978-e9da8dc65a9a.png)](https://user-images.githubusercontent.com/2416493/80095052-e2d01680-8599-11ea-9978-e9da8dc65a9a.png)

下面介绍下具体的配置过程。

## 前置要求

在开始之前，需要准备如下的材料：

1.  npm token，用于发布。获取方式参见：[https://docs.npmjs.com/creating-and-viewing-authentication-tokens](https://docs.npmjs.com/creating-and-viewing-authentication-tokens)

## 步骤列表

1.  设置[约定式提交 Conventional Commits](https://www.conventionalcommits.org/zh-hans)生成Changelog
2.  设置Github Actions自动发布版本
3.  设置自动生成Github Release及Changelog

### 设置约定式提交Conventional Commits生成Changelog

[约定式提交 Conventional Commits](https://www.conventionalcommits.org/zh-hans) 是一种Commit消息规范，它要求开发者按照规范编写Git Commit消息，并可根据Git Commit消息自动生成Changelog。

本文中Changelog的生成也是根据此规范自动处理的。

在npm工程中，要设置Conventional Commits，需要做如下几个方面的处理

#### 设置npm version命令自动生成Changelog

可以在`package.json`的**scripts**中添加一个**version**命令，内容如下：

```json
 "scripts": {
   "version": "conventional-changelog -p angular -i CHANGELOG.md -s && git add CHANGELOG.md"
 },
```

同时，安装下相应的依赖**conventional-changelog-cli**，全局安装或在项目中安装均可。我用的是全局安装

```shell
npm i -g conventional-changelog-cli
```

这样配置以后，在执行`npm version patch|minor|major`时，会相应地自动更新Changelog文件。

#### 设置Git hook

上面的conventional-changelog设置好了机制，另外还需要提交的时候遵守[约定式提交 Conventional Commits](https://www.conventionalcommits.org/zh-hans)规范才能实现自动生成Changelog的功能。

为了避免写出不规范的Commit消息，可考虑添加Git hook，在提交代码时检查Commit消息是否合法。

首先安装**husky**用来管理git hook：

再添加husky的配置文件`.huskyrc`，添加提交消息的git hook

```json
{
  "hooks": {
    "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
  }
}
```

这里我们使用了[commitlint](https://commitlint.js.org/#/)进行commit消息的规范校验。

一般来说，使用Angular团队推荐的Conventional Commits规范格式即可，有需要的可适当自定义。

我使用的commitlint配置文件（commitlint.config.js）如下：

```js
module.exports = {
  extends: ['@commitlint/config-conventional']
}
```

相应地，还需要安装`commitlint`的命令行工具及规范包，推荐还是全局安装即可：

```shell
npm install -g @commitlint/cli @commitlint/config-conventional
```

如上步骤配置完之后，就有了一套可用的changelog生成机制了。

Git commit之后，试试`npm version patch`，看看是不是发布了新版本，并自动添加了CHANGELOG.md文件？

### 设置Github Actions自动发布版本

上一节讲的还都是一些本地操作，本地使用`npm version patch|minor|major`发布版本时自动生成Changelog。下面要讲的就是怎么跟Github结合，以及如何使用Github Actions发布版本。

#### 设置postversion script

在package.json中添加**postversion**脚本，可在`npm version patch|minor|major`之后，自动将代码和tag推送到Github上。

设置方式为：

```json
 "scripts": {
     "postversion": "git push --follow-tags",
 },
```

如果不添加这种脚本，自己每次手动push也行。

#### 设置项目的Github Secrets

自动化发布过程中，涉及到npm身份的校验。我们是用的NPM\_TOKEN来解决这个问题。（NPM\_TOKEN的生成见[https://docs.npmjs.com/creating-and-viewing-authentication-tokens）](https://docs.npmjs.com/creating-and-viewing-authentication-tokens%EF%BC%89)

但是NPM\_TOKEN是不能直接写在代码中的，安全性太低，需要保存在Github Secrets中，再在Actions中再使用。

关于Secrets的创建，可以参阅Github上的指南：[https://help.github.com/cn/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets。](https://help.github.com/cn/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets%E3%80%82)

1.  在 GitHub 上，导航到仓库的主页面。
2.  在仓库名称下，单击 **Settings（设置）**。
3.  在左侧边栏中，单击 **Secrets（密码）**。
4.  在 "Name"（名称）输入框中输入密码的名称。
5.  输入密码的值。
6.  单击 **Add secret（添加密码）**。

#### 设置自动发布

在上面的步骤中，我们使用`npm version patch|minor|major`等创建了Tag，并使用`postversion`中的脚本自动将其推送到了Github仓库。

现在我们可以通过检测新tag，自动执行工作流来进行`npm publish`的操作。Github上可用的CI/CD服务很多，这里我选择的是其自带的[Github Actions](https://help.github.com/cn/actions)。

要使用Github Actions，只需要在相应的仓库中，创建一个 `.github/workflows/`目录，并在其中加入一些`.yml`后缀的配置文件即可。

我们先添加一个用于`npm publish`的配置文件`.github/workflows/publish.yml`，文件名是随意定义的。

```yaml
on:
  push:
    tags:
    - 'v*'  # 这段的意思是仅在出现名为 v 字符开头的tag时，触发此任务，如v1.2.1
name: Publish
jobs:
  # test: # test任务可选，视情况决定是否添加
  #   runs-on: ubuntu-latest
  #   steps:
  #   - uses: actions/checkout@v2
  #   - uses: actions/setup-node@v1
  #     with:
  #       node-version: 10
  #   - run: npm i
  #   - run: npm test

  publish:
    # needs: test
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - uses: actions/setup-node@v1
      with:
        node-version: 10
        registry-url: https://registry.npmjs.org/
    - run: npm i
    - run: npm publish # 如果发布之前需要构建，可添加prepublishOnly的生命周期脚本
      env:
        NODE_AUTH_TOKEN: ${{secrets.NPM_TOKEN}} # 上面在Github Secrets中设置的密钥
```

如果发布之前需要执行构建 ，可在 `- run: npm publish`之前加一段类似于`- run: npm run build`之类的指令，或者在`package.json`中添加`prepublishOnly`的脚本：

```json
{
  "scripts": {
    "version": "conventional-changelog -p angular -i CHANGELOG.md -s && git add CHANGELOG.md",
    "postversion": "git push --follow-tags",
    "prepublishOnly": "npm run build"
 }
}
```

按上面的方式设置之后，每次执行`npm version patch|minor|major`之后，会自动地在Github Actions中创建一个名为`publish`的任务，可在Git仓库的 actions 路径下找到它。

现在应已经能正常地执行npm仓库的发布了。

### 设置自动生成Github Release

上面已经有了自动化的npm版本发布，且也已经自动地在CHANGELOG.md文件中生成了changelog。不过它现在还实现不了

[![Demo效果图](https://user-images.githubusercontent.com/2416493/80095052-e2d01680-8599-11ea-9978-e9da8dc65a9a.png)](https://user-images.githubusercontent.com/2416493/80095052-e2d01680-8599-11ea-9978-e9da8dc65a9a.png)

这样的效果。

需要调用下Github Release方面的API，生成对应的Release才可以。

因为我们已经在CHANGELOG.md中自动填充上了changelog，所以此处不再需要conventional commits的解析了。我选择的是用一个解析changelog.md生成Github Release的工具：[github-release-from-changelog](https://github.com/MoOx/github-release-from-changelog)。

下面我们再修改下 `.github/workflows/publish.yml`，添加上对Github Release的支持。

```yaml
on:
  push:
    tags:
    - 'v*'
name: Publish
jobs:
  # test:
  #   runs-on: ubuntu-latest
  #   steps:
  #   - uses: actions/checkout@v2
  #   - uses: actions/setup-node@v1
  #     with:
  #       node-version: 10
  #   - run: npm i
  #   - run: npm test

  publish:
    # needs: test
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - uses: actions/setup-node@v1
      with:
        node-version: 10
        registry-url: https://registry.npmjs.org/
    - run: npm i
    - run: npm publish
      env:
        NODE_AUTH_TOKEN: ${{secrets.NPM_TOKEN}}

  github-release:
    needs: publish
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - uses: actions/setup-node@v1
      with:
        node-version: 10
        registry-url: https://registry.npmjs.org/
    - run: npm i -g github-release-from-changelog
    - run: github-release-from-changelog
      env:
        GITHUB_TOKEN: ${{secrets.GITHUB_TOKEN}}
```

上面的`github-release`任务即是本次新增的Github Release任务，它的主要内容就是安装和执行[github-release-from-changelog](https://github.com/MoOx/github-release-from-changelog)。

过程中会用到名为`GITHUB_TOKEN`的密钥，用于访问Github相关的API，不过这个私钥是Github Actions默认提供的，不需要额外设置。

上面的步骤完成之后，整个事情就大功告成了！

## 总结

总结下上面进行的改动，有如下几点：

1.  获取NPM\_TOKEN，用于发布。获取方式参见：[https://docs.npmjs.com/creating-and-viewing-authentication-tokens](https://docs.npmjs.com/creating-and-viewing-authentication-tokens)
    
2.  将上面获取到的NPM\_TOKEN添加到Github Secrets中，设置变量名为NPM\_TOKEN。设置方式参见：[https://help.github.com/cn/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets。](https://help.github.com/cn/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets%E3%80%82)
    
3.  全局安装一些工具包，或安装在项目的devDependencies中
    
    ```shell
    npm i -g @commitlint/cli @commitlint/config-conventional conventional-changelog-cli
    
    # 或 npm i -D @commitlint/cli @commitlint/config-conventional conventional-changelog-cli
    ```
    
4.  在项目中devDependencies中安装如下包：
    
5.  在`package.json`中添加如下的**scripts**：
    
    ```json
    {
      "scripts": {
        "version": "conventional-changelog -p angular -i CHANGELOG.md -s && git add CHANGELOG.md",
        "postversion": "git push --follow-tags",
        "prepublishOnly": "npm run build"
     }
    }
    ```
    
    其中build命令为工程自己配置，不需要的话可以删除`prepublishOnly`脚本。
    
6.  在项目中添加如下的配置文件：
    
    1.  `.huskyrc` 用于设置 git hooks
        
        ```json
        {
          "hooks": {
            "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
          }
        }
        ```
        
    2.  `commitlint.config.js` 用于设置Git Commit消息规范
        
        ```js
        module.exports = {
          extends: ['@commitlint/config-conventional']
        }
        ```
        
    3.  `.github/workflows/publish.yml`用于设置Github Actions
        
        ```yaml
        on:
          push:
            tags:
            - 'v*'
        name: Publish
        jobs:
          # test:
          #   runs-on: ubuntu-latest
          #   steps:
          #   - uses: actions/checkout@v2
          #   - uses: actions/setup-node@v1
          #     with:
          #       node-version: 10
          #   - run: npm i
          #   - run: npm test
        
          publish:
            # needs: test
            runs-on: ubuntu-latest
            steps:
            - uses: actions/checkout@v2
            - uses: actions/setup-node@v1
              with:
                node-version: 10
                registry-url: https://registry.npmjs.org/
            - run: npm i
            - run: npm publish
              env:
                NODE_AUTH_TOKEN: ${{secrets.NPM_TOKEN}}
        
          github-release:
            needs: publish
            runs-on: ubuntu-latest
            steps:
            - uses: actions/checkout@v2
            - uses: actions/setup-node@v1
              with:
                node-version: 10
                registry-url: https://registry.npmjs.org/
            - run: npm i -g github-release-from-changelog
            - run: github-release-from-changelog
              env:
                GITHUB_TOKEN: ${{secrets.GITHUB_TOKEN}}
        ```
