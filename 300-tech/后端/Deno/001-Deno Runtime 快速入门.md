---
title: 001-Deno Runtime 快速入门
date: 2025-02-17T10:11:55+08:00
updated: 2025-02-17T10:12:19+08:00
dg-publish: false
---

<https://docs.denohub.com/deno/runtime/manual>

[Deno](https://www.deno.com/) ([/ˈdiːnoʊ/](http://ipa-reader.xyz/?text=%CB%88di%CB%90no%CA%8A)，发音为 `dee-no`) 是一个具有安全默认设置和出色开发者体验的 JavaScript、TypeScript 和 WebAssembly 运行时。它构建在 [V8](https://v8.dev/)、[Rust](https://www.rust-lang.org/) 和 [Tokio](https://tokio.rs/) 之上。

Deno 是自由且开源的软件，采用 [MIT 许可证](https://github.com/denoland/deno/blob/main/LICENSE.md)。

让我们在不到五分钟内创建并运行您的第一个 Deno 程序，并为您介绍一些运行时的关键特性。

## 安装 Deno[​](https://docs.denohub.com/deno/runtime/manual#%E5%AE%89%E8%A3%85-deno "安装 Deno的直接链接")

使用以下终端命令之一在系统上安装 Deno 运行时。

- macOS
- Windows
- Linux

```sh
curl -fsSL https://x.deno.js.cn/install.sh | sh
```

[其他安装选项可以在这里找到](https://docs.denohub.com/deno/runtime/manual/getting_started/installation)。安装后，您应该在系统路径上找到 `deno` 可执行文件。您可以通过在终端中运行以下命令来确认这一点：

```sh
deno --version
```

## 创建并运行一个 TypeScript 程序 [​](https://docs.denohub.com/deno/runtime/manual#%E5%88%9B%E5%BB%BA%E5%B9%B6%E8%BF%90%E8%A1%8C%E4%B8%80%E4%B8%AA-typescript-%E7%A8%8B%E5%BA%8F "创建并运行一个 TypeScript 程序的直接链接")

虽然您可以使用纯 JavaScript，但 Deno 还内置了对 [TypeScript](https://www.typescriptlang.org/) 的支持。在终端中，创建一个名为 `hello.ts` 的新文件，并包含以下代码。

hello.ts

```ts
interface Person {  firstName: string,  lastName: string}function sayHello(p: Person): string {  return `Hello, ${p.firstName}!`;}const ada: Person = {  firstName: "Ada",  lastName: "Lovelace"};console.log(sayHello(ada));
```

此程序声明了一个用于表示个人信息的 [接口](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#interfaces)，并定义了一个使用这种数据类型向控制台打印消息的函数。您可以使用 `deno run` 命令来执行此示例中的代码。

```text
deno run -A hello.ts
```

您可以在 [这里了解有关在 Deno 中使用 TypeScript 的更多信息](https://docs.denohub.com/deno/runtime/manual/advanced/typescript/overview)。

## 内置 Web API 和 Deno 命名空间 [​](https://docs.denohub.com/deno/runtime/manual#%E5%86%85%E7%BD%AE-web-api-%E5%92%8C-deno-%E5%91%BD%E5%90%8D%E7%A9%BA%E9%97%B4 "内置 Web API 和 Deno 命名空间的直接链接")

Deno 旨在提供一个类似浏览器的编程环境，[实现了存在于前端 JavaScript 中的 Web 标准 API](https://docs.denohub.com/deno/runtime/manual/runtime/web_platform_apis)。例如，[`fetch`](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) API 在全局范围内可用，就像在浏览器中一样。要查看这个示例，请将 `hello.ts` 的内容替换为以下代码。

```ts
const site = await fetch("https://www.deno.com");console.log(await site.text());
```

然后运行它：

```text
deno run -A hello.ts
```

对于不存在于 Web 标准中的 API（例如从系统环境中访问变量或操作文件系统的 API），这些 API 会在 [`Deno` 命名空间](https://docs.denohub.com/deno/runtime/manual/runtime/builtin_apis) 中公开。将 `hello.ts` 的内容替换为以下代码，它将在 [localhost:8000](http://localhost:8000/) 上启动 [一个 HTTP 服务器](https://deno.land/api?s=Deno.serve)。

```ts
Deno.serve((_request: Request) => {  return new Response("Hello, world!");});
```

使用以下命令运行上面的脚本：

```text
deno run -A hello.ts
```

了解更多关于内置在 Deno 中的 [Web 标准 API](https://docs.denohub.com/deno/runtime/manual/runtime/web_platform_apis) 以及 [`Deno` 命名空间 API](https://docs.denohub.com/deno/runtime/manual/runtime/builtin_apis) 的信息。

## 运行时安全性 [​](https://docs.denohub.com/deno/runtime/manual#%E8%BF%90%E8%A1%8C%E6%97%B6%E5%AE%89%E5%85%A8%E6%80%A7 "运行时安全性的直接链接")

Deno 的一个主要特性是默认提供 [运行时安全性](https://docs.denohub.com/deno/runtime/manual/basics/permissions)，这意味着您作为开发者必须明确允许您的代码访问潜在敏感的 API，比如文件系统访问、网络连接和环境变量访问。

到目前为止，我们一直在使用 `-A` 标志运行所有脚本，这会授予我们的脚本所有运行时功能的访问权限。这是运行 Deno 程序的最宽松模式，但通常您只希望授予您的代码运行所需的权限。

为了看到这一点的效果，让我们再次将 `hello.ts` 的内容替换为之前的 `fetch` 示例。

```ts
const site = await fetch("https://www.deno.com");console.log(await site.text());
```

不使用 `-A` 标志运行此程序 - 然后会发生什么？

```bash
deno run hello.ts
```

如果没有传递任何权限标志，您将看到类似以下内容的安全提示：

```text
justjavac@justjavac-deno scratchpad % deno run index.ts✅ Granted net access to "www.deno.com".┌ ⚠️  Deno requests net access to "deno.com".├ Requested by `fetch()` API.├ Run again with --allow-net to bypass this prompt.└ Allow? [y/n/A] (y = yes, allow; n = no, deny; A = allow all net permissions) >
```

在提示中，您可能已经注意到它提到了需要使用权限来访问网络的 CLI 标志 - `--allow-net` 标志。如果您使用此标志再次运行脚本，您将不会被提示交互式地授予网络访问权限：

```bash
deno run --allow-net hello.ts
```

为简单起见，有时我们会展示使用 `deno run -A ...` 的示例，但在可能的情况下（在生产或 CI 环境中），我们鼓励您利用 Deno 的完整套 [可配置的运行时安全选项](https://docs.denohub.com/deno/runtime/manual/basics/permissions)。

## 导入 JavaScript 模块 [​](https://docs.denohub.com/deno/runtime/manual#%E5%AF%BC%E5%85%A5-javascript-%E6%A8%A1%E5%9D%97 "导入 JavaScript 模块的直接链接")

大多数情况下，您会希望将程序拆分为多个文件。再次强调 Web 标准和类似浏览器的编程模型，Deno 通过 [ECMAScript 模块](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) 支持这一点。考虑我们之前展示的 TypeScript 示例：

hello.ts

```ts
interface Person {  firstName: string,  lastName: string}function sayHello(p: Person): string {  return `Hello, ${p.firstName}!`;}const ada: Person = {  firstName: "Ada",  lastName: "Lovelace"};console.log(sayHello(ada));
```

您可能希望将此程序拆分，使 `Person` 接口和 `sayHello` 函数位于一个单独的模块中。为此，在相同目录下创建一个名为 `person.ts` 的新文件，并包含以下代码：

person.ts

```ts
export default interface Person {  firstName: string,  lastName: string}export function sayHello(p: Person): string {  return `Hello, ${p.firstName}!`;}
```

这个模块为 `sayHello` 函数创建了一个 [命名导出](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules#exporting_module_features)，并为 `Person` 接口创建了一个 [默认导出](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules#default_exports_versus_named_exports)。

回到 `hello.ts`，您可以使用 `import` 关键字来使用这个模块。

hello.ts

```ts
import Person, { sayHello } from "./person.ts";const ada: Person = {  lastName: "Lovelace",  firstName: "Ada",};console.log(sayHello(ada));
```

导入时需要文件扩展名

请注意，导入模块时**需要文件扩展名** - 在 Deno 中的导入逻辑与浏览器中的导入逻辑相同，您需要包含导入的完整文件名。

[您可以在此了解有关 Deno 模块系统的更多信息](https://docs.denohub.com/deno/runtime/manual/basics/modules)。

## 远程模块和 Deno 标准库 [​](https://docs.denohub.com/deno/runtime/manual#%E8%BF%9C%E7%A8%8B%E6%A8%A1%E5%9D%97%E5%92%8C-deno-%E6%A0%87%E5%87%86%E5%BA%93 "远程模块和 Deno 标准库的直接链接")

Deno 支持从 URL 加载和执行代码，就像在浏览器中使用 `<script>` 标签一样。在 Deno 1.x 中，[标准库](https://deno.land/std) 和大多数 [第三方模块](https://deno.land/x) 都是通过 HTTPS URL 分发的。

为了看到这一点的效果，让我们为我们上面创建的 `person.ts` 模块创建一个测试。Deno 提供了一个 [内置的测试运行器](https://docs.denohub.com/deno/runtime/manual/basics/testing)，它使用通过 HTTPS URL 分发的断言模块。

person_test.ts

```ts
import { assertEquals } from "https://deno.land/std@0.208.0/assert/mod.ts";import Person, { sayHello } from "./person.ts";Deno.test("sayHello function", () => {  const grace: Person = {    lastName: "Hopper",    firstName: "Grace",  };  assertEquals("Hello, Grace!", sayHello(grace));});
```

使用以下命令运行此测试：

```text
deno test person_test.ts
```

输出应该类似于以下内容：

```text
just java c@justjavac-deno scratchpad % deno test person_test.tsCheck file:///Users/justjavac/dev/denoland/scratchpad/person_test.tsrunning 1 test from ./person_test.tssayHello function ... ok (4ms)ok | 1 passed | 0 failed (66ms)
```

还有很多要探索的内容，[标准库](https://deno.land/std) 和 [第三方模块](https://deno.land/x) - 一定要去了解它们！

## 使用 deno.json 配置项目 [​](https://docs.denohub.com/deno/runtime/manual#%E4%BD%BF%E7%94%A8-denojson-%E9%85%8D%E7%BD%AE%E9%A1%B9%E7%9B%AE "使用 deno.json 配置项目的直接链接")

Deno 项目默认不需要配置文件，但有时将设置、管理脚本和依赖项配置存储在一个众所周知的位置是方便的。在 Deno 中，这个文件是 [`deno.json` 或 `deno.jsonc`](https://docs.denohub.com/deno/runtime/manual/getting_started/configuration_file)。这个文件在某种程度上类似于 Node.js 中的 `package.json` 文件。

你可以使用 `deno.json` 来配置 [导入映射](https://docs.denohub.com/deno/runtime/manual/basics/import_maps)，它可以让你为经常使用的模块设置别名。

为了演示，让我们将我们项目中想要使用的标准库版本固定为 `0.208.0`。

创建一个包含以下内容的 `deno.jsonc` 文件。

deno.jsonc

```js
{  "imports": {    // "std" 前面的美元符号不是特殊符号 - 它是导入映射中设置的别名的可选约定    "$std/": "https://deno.land/std@0.208.0/"  }}
```

现在，打开之前的测试文件，并更改以使用这个导入别名。

person_test.ts

```ts
import { assertEquals } from "$std/assert/mod.ts";import Person, { sayHello } from "./person.ts";Deno.test("sayHello 函数", () => {  const grace: Person = {    lastName: "Hopper",    firstName: "Grace",  };  assertEquals("Hello, Grace!", sayHello(grace));});
```

使用 `deno test person_test.ts` 运行测试应该与之前一样工作，但你可能会注意到 Deno 下载了一些额外的文件并生成了一个 `deno.lock` 文件，其中指定了你的代码所依赖的一组文件。`deno.jsonc` 和 `deno.lock` 都可以检入到源代码控制中。

了解更多关于 [配置你的项目的信息](https://docs.denohub.com/deno/runtime/manual/getting_started/configuration_file)。

## Node.js API 和 npm 包 [​](https://docs.denohub.com/deno/runtime/manual#nodejs-api-%E5%92%8C-npm-%E5%8C%85 "Node.js API 和 npm 包的直接链接")

Deno 提供了一个兼容层，使您的代码能够使用 [Node.js 内置模块和 npm 的第三方模块](https://docs.denohub.com/deno/runtime/manual/node)。在代码中使用 Node 和 npm 模块看起来很像使用标准的 Deno 模块，唯一的区别是在导入 Node 内置模块或 npm 模块时，您将分别使用 `node:` 或 `npm:` 修饰符。

为了了解它是如何工作的，创建一个名为 `server.js` 的文件，并包含以下内容 - 一个使用流行的 [Express](https://expressjs.com/) 框架创建的简单 HTTP 服务器。

```js
import express from "npm:express@4";const app = express();app.get("/", (request, response) => {  response.send("Hello from Express!");});app.listen(3000);
```

使用 `node:` 和 `npm:` 指示符，您可以将 Node.js 生态系统的优势带入 Deno 中。[了解更多关于 Node 和 npm 支持的信息](https://docs.denohub.com/deno/runtime/manual/node)。

## 配置你的 IDE[​](https://docs.denohub.com/deno/runtime/manual#%E9%85%8D%E7%BD%AE%E4%BD%A0%E7%9A%84-ide "配置你的 IDE的直接链接")

Deno 开发在多个主要 IDE 中都得到了支持。其中一个流行的选项是 **Visual Studio Code**，它由 justjavac 创建目前由 Deno 团队维护着 [官方扩展](https://docs.denohub.com/deno/runtime/manual/references/vscode_deno)。 [安装扩展](https://marketplace.visualstudio.com/items?itemName=denoland.vscode-deno)，并在 VS Code 工作区中启用它，选择命令面板中的 `Deno: Initialize Workspace Configuration` 选项。

![命令面板设置](https://docs.denohub.com/deno/assets/images/command_palette-dd2a85f903f65636307b8a9d526cd773.png)

如果你不是 VS Code 的用户，可以在 [这里](https://docs.denohub.com/deno/runtime/manual/getting_started/setup_your_environment) 找到适合你喜欢的编辑器的集成。

## Web 应用程序框架 [​](https://docs.denohub.com/deno/runtime/manual#web-%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F%E6%A1%86%E6%9E%B6 "Web 应用程序框架的直接链接")

Deno 的一个常见用例是构建数据驱动的 Web 应用程序。通常，这需要使用更高级别的 Web 框架，在 Deno 生态系统中存在许多选择。以下是一些最受欢迎的选择。

### Deno 本地框架 [​](https://docs.denohub.com/deno/runtime/manual#deno-%E6%9C%AC%E5%9C%B0%E6%A1%86%E6%9E%B6 "Deno 本地框架的直接链接")

- [Deno Fresh](https://fresh.deno.dev/) - Fresh 是专为 Deno 设计的 Web 框架。页面默认由服务器呈现，还可以包含在客户端上运行 JavaScript 的交互式岛屿选项。如果您刚开始使用 Deno 并寻找一个起点，我们建议首先尝试 Fresh！
- [Hono](https://hono.dev/getting-started/deno) - Hono 是一款轻量级 Web 框架，类似于 [Express](https://expressjs.com/)。非常适用于 API 服务器和简单的 Web 应用程序。

### Deno 兼容框架 [​](https://docs.denohub.com/deno/runtime/manual#deno%E5%85%BC%E5%AE%B9%E6%A1%86%E6%9E%B6 "Deno兼容框架的直接链接")

- [Astro](https://astro.build/) - Astro 是一个现代的 Web 框架，最初是为 Node.js 设计的，但在 Deno 上也运行得很好。我们建议从 [这个模板](https://github.com/denoland/deno-astro-template) 开始。
- [SvelteKit](https://kit.svelte.dev/) - SvelteKit 是另一个更适用于运行时的 Web 框架，可以与 Deno 一起使用。我们建议从 [这个模板](https://github.com/denoland/deno-sveltekit-template) 开始。
- [Nuxt (Vue)](https://nuxt.com/) - Nuxt 是一个混合 SSR 和客户端框架，可以 与 Deno 一起使用。我们建议从 [这个模板](https://github.com/denoland/deno-nuxt-template) 开始。

除了上面列出的框架外，还有更多支持 Deno 的框架，但我们建议从这些开始作为一个很好的起点。

## 部署到生产环境 [​](https://docs.denohub.com/deno/runtime/manual#%E9%83%A8%E7%BD%B2%E5%88%B0%E7%94%9F%E4%BA%A7%E7%8E%AF%E5%A2%83 "部署到生产环境的直接链接")

当你准备进入生产环境时，最简单的选择将是 [Deno Deploy](https://docs.denohub.com/deno/deploy/manual)。Deno Deploy 使您能够轻松创建使用 Deno 的快速、全球分布的
