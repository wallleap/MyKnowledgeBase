---
title: Stable Diffusion 安装
date: 2023-11-29T03:03:00+08:00
updated: 2024-08-21T10:32:30+08:00
dg-publish: false
aliases:
  - Stable Diffusion 安装
author: Luwang
category: AI
comments: true
cover: //cdn.wallleap.cn/img/post/1.jpg
description: 文章描述
image-auto-upload: true
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
rating: 1
source: 
tags:
  - AI
  - 图片
url: //myblog.wallleap.cn/post/1
---

扫盲： <https://www.bilibili.com/video/BV11m4y12727>

项目地址：[AUTOMATIC1111/stable-diffusion-webui: Stable Diffusion web UI (github.com)](https://github.com/AUTOMATIC1111/stable-diffusion-webui)

安装参考：[AUTOMATIC1111/stable-diffusion-webui: Stable Diffusion web UI (github.com)](https://github.com/AUTOMATIC1111/stable-diffusion-webui#installation-and-running)

模型下载：[huggingface.co/](https://huggingface.co/)、[civitai.com/](https://civitai.com/)、[备用站](https://www.4b3.com)

这是别人整理的：<https://pan.baidu.com/s/10rzgzIjzad7AKmj-w8zO_w?pwd=nely#list/path=%2F>

在 mac 下使用 brew 下载需要的软件

```sh
brew install cmake protobuf rust python@3.10 git wget
```

克隆项目

```sh
git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui
```

将下载的模型 <https://pan.baidu.com/s/1FzIL5tgdKiuLpxHMINUhYw?pwd=i6dp> （文件后缀为 `.ckpt` 或 `.safetensor`、`.pt`）放到 `stable-diffusion-webui/models/Stable-diffusion` 目录

运行 `./webui.sh`，等待下载完成依赖，之后再运行 `./webui.sh`

```sh
cd stable-diffusion-webui
unset all_proxy
./webui.sh
```

> 在之前是运行了代理的，会有点问题，因此运行之前最好 `unset all_proxy` 如果不想取消命令行代理可以在 `webui.sh` 文件中的 `KEEP_GOING=1` 行之前，加一行 `pip install socksio`（有其他问题就把错误复制下来到谷歌/仓库 Issue 搜索）

![](https://cdn.wallleap.cn/img/pic/illustration/202311291524619.png)

运行成功的话会自动打开浏览器，如果没有自动打开浏览器但是看到命令行界面中显示

```
Running on local URL:  http://127.0.0.1:7860
```

可以复制 `http://127.0.0.1:7860` 到浏览器打开

安装中文：点击 Extensions-Available 之后将复选框全部取消，点击 Load from

Ctrl + F 搜索 `zh_cn` 安装

![](https://cdn.wallleap.cn/img/pic/illustration/202311291622567.png)

点击 Stettings-User interface

![](https://cdn.wallleap.cn/img/pic/illustration/202311291625516.png)

## 后续更新

如果有更新可以使用 Git 拉取新的代码

- **`git fetch --all`**：获取远程仓库的最新信息
- **`git reset --hard origin/master`**：强制将本地分支重置为远程 master 分支，并放弃本地未提交的更改
- **`git pull`**：从远程仓库拉取更新并合并到当前本地分支

> 注：如果有较长时间没有更新，可以将根目录下的 venv 文件夹删除，再运行脚本，让它重新下载各项依赖。

## 模型讲解

**1.Checkpoint**

体积较大，也被称为大模型，不同的大模型使用不同的图片训练而成，对应不同的风格，相当于最底层的引擎。有时候需要大模型 +VAE+emb+Lora 联合搭配使用以达到需要的效果。

下载的大模型可放置于 SD 文件夹 `/models/Stable-diffusion` 内。

**2.Lora**

Lora 是特征模型，体积较小，是基于某个确定的角色、确定的风格或者固定的动作训练而成的模型，可使用权重控制，确定性要远强于 embedding。embedding 和 Lora 有功能交集的部分，也有互相不可取代的地方。

在 ckpt 大模型上附加使用，对人物、姿势、物体表现较好。在 webui 界面的 Additional Networks 下勾线 Enable 启用，然后在 Model 下选择模型，并可用 Weight 调整权重。权重越大，该 Lora 的影响也越大。不建议权重过大（超过 1.2），否则很容易出现扭曲的结果。

多个 Lora 模型混合使用可以起到叠加效果，譬如一个控制面部的 Lora 配合一个控制画风的 Lora 就可以生成具有特定画风的特定人物。因此可以使用多个专注于不同方面优化的 Lora，分别调整权重，结合出自己想要实现的效果。

LoHA 模型是一种 LORA 模型的改进。

LoCon 模型也一种 LORA 模型的改进，泛化能力更强。

下载的 Lora 可放置于 SD 文件夹 `/models/Lora` 内。

**3.VAE**

VAE 模型类似滤镜，对画面进行调色与微调，一般需要搭配相应的模型一起使用。（如果图片比较灰，颜色不太靓丽，就可能是没加载 vae)

下载的 VAE 可放置于 SD 文件夹 `/models/VAE` 内。

**4.Textual inversion（embedding）**

关键词预设模型，即关键词打包，即等于预设好一篮子关键词 a,b,c 打包，进而来指代特定的对象/风格。也可以通过下载 Textual inversion 进行使用。

下载的 embedding 可放置于 SD 文件夹 `/embeddings` 内。
