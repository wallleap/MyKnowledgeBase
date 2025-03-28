---
title: 保姆级 ComfyUI + Flux 使用说明书
date: 2025-02-21T15:10:07+08:00
updated: 2025-02-27T16:49:59+08:00
dg-publish: false
---

<https://www.bilibili.com/video/BV1JWwdeQEv4/?vd_source=a0b343ce9dc91e8404ea304068710b8a>

## 介绍

SD 1.5 → SDXL → SD 3.5、Flux

Flux 模型由黑森林工作室开发的一款全新的大模型，该组织成员基本来自之前从 stability AI 离职的核心成员

Flux 之所以热度这么高，原因就是效果相比其他模型提升巨大

- SD3 medium：训练参数 2B（20 亿）
- SD3 Large：训练参数 8B（80 亿）
- Flux：训练参数 12B（120 亿）

Flux 模型有三种：

- Flux.1 pro：效果最好，开源但不支持独立下载本地文件
- Flux.1 dev：效果略逊，支持本地下载使用
- Flux.1 schnell：8 步左右出图，速度快，体积小，效果略逊于 dev 版模型

可以在 liblib 的高级生图体验

Flux 模型的优势：

- 照片级画质
- 极大优化了手的准确生成能力
- 字体生成排版效果好（不支持中文）
- 训练参数大，风格多样
- 分辨率弹性好
- embeddings 通用性好
- 不需要输入负面提示词

Flux 七种模型的区别：

- Pro：仅 API 调用
- Dev FP16：正常能用到的质量最好的模型，但至少需要 4090 及以上的显卡
- Dev FP8：相比 FP16 容量减半，但整体效果并不差太多
- schnell FP16：相比 Dev 版本，需要的采样步数更少
- schnell FP8：相比 Dev 版本，需要的采样布更少，但是生成 1080p 图片，依然需要 14-16G 显存，非顶级显卡依然难以使用
- GCUF：划分了多个版本，Q2-Q3 版本，适用于 6G 以下的显存，Q4-Q5 版本适用于 8G 显存，Q 后面的数字越大，需要的显存越高，效果也越好
	- 下载地址：<https://huggingface.co/city96/FLUX.1-dev-gguf/tree/main>
- NF4：整合了 Clip、VAE 和 T5 文本编辑器（可以不用搭建这些节点），8G 显存就能用，生成速度也不错

## Flux 模型的三种部署方法

### webui 中部署

使用 forge 版：<https://github.com/lllyasviel/stable-diffusion-webui-forge>

模型放到 `models/Stable-diffusion` 中，重启 webui，页面左上方选择 Flux 模式，大模型选择 Flux NF4 模型（<https://huggingface.co/ZhenyaYang/flux_1_dev_hyper_8steps_nf4/tree/main>），CFG 调整为 3.5

### ComfyUI 中部署

使用 ComfyUI 最新版：<https://github.com/comfyanonymous/ComfyUI>

下载整合包，把模型和插件放到对应位置

如果想使用 GGUF 和 NF4 模型，需要先下载对应相似名称的 ComfyUI 插件

- <https://github.com/city96/ComfyUI-GGUF>
- <https://github.com/comfyanonymous/ComfyUI_bitsandbytes_NF4>

例如使用 GGUF：

先下载上面的 ComfyUI-GGUF 插件，再下载下面的模型

- unet：<https://huggingface.co/city96/FLUX.1-dev-gguf/tree/main> 下载 `flux1-dev-Q3_K_S.gguf`
- vae：<https://huggingface.co/black-forest-labs/FLUX.1-dev> 需要先登录同意再点到对应的文件去下载 `vae/diffusion_pytorch_model.safetensors` 下载完重命名为 `flux1_dev_vae.safetensors`
- clip：
	- <https://huggingface.co/openai/clip-vit-large-patch14/tree/main> 下载 `model.safetensors` 重命名为 `flux_clip_model.safetensors`
	- <https://huggingface.co/comfyanonymous/flux_text_encoders/tree/main> 下载 `t5xxl_fp8_e4m3fn.safetensors`
- lora：<https://huggingface.co/XLabs-AI/flux-RealismLora/blob/main/lora.safetensors> 下载完后重命名为 `flux_realism_lora.safetensors`

搭建简易的 gguf 工作流：

双击搜索 `gguf` 找到 `Unet Loader(GGUF)` 使用 `flux1-dev-Q3_K_S.gguf` 模型，添加 `双CLIP加载器` 使用 `flux_clip_model.safetensors` 和 `t5xxl_fp8_e4m3fn.safetensors` 类型选 `flux` 或使用 Clip 文本编码 Flux，添加一个 K 采样器 把模型连上去，添加一个 `空Latent图像` 设定尺寸连到 K 采样器上，双 Clip 可以添加 Clip 文本编码框，之后链接到 VAE 解码，添加一个 加载 VAE 的，最后保存图像

### 直接在 liblib 上使用 flux pro

直接在主页进入在线工作流

## Flux 模型基础工作流

### flux dev 标准文生图工作流

相比于 SD 标准文生图工作流，Flux 标准文生图工作流，K 采样器、VAE 解码、空 Latent、保存图像节点都是相同的，不同之处在于：

- 大模型加载使用的是 UNET 加载器，而不是 Checkpoint 加载器
	- Flux 选择对应的 dev 模型，fp16 和 fp8 之前已经说过显卡需求
- 由于没有负面提示词，负面提示词节点被替换成了条件零化节点/空的 CLIP 文本编码器
- 由于 UNET 加载器中没有独立的 VAE 原点及 CLIP 原点，所以需要创建双 CLIP 加载器，大模型分别可以搭配的为：
	- SDXL：clip-l、clip-g
	- SD3：clip-l、clip-g 或 clip-l、t5 或 clip-g、t5
	- Flux：clip-l、t5
- 以及 VAE 加载器
	- 选择适配 Flux 的 VAE 模型

### flux + controlnet 工作流

加载图像 → Aux 集成预处理器

ControlNet 加载器 → 设置 UnionControlNet 类型

｜

正向提示词 条件零化

↓

ControlNet 应用

↓

K 采样器

> 注意：目前 Flux 针对 ControlNet 没有传统 SD 所适配的效果好，因此可以先用 SD + ControlNet 生成需要的图，然后使用 Flux 图生图优化

### GGUF 工作流

Unet 加载器更换成 Unet Loader GGUF（需要先下载自定义节点）

### NF4 工作流

Checkpoint 加载器 NF4，集成了 VAE 和 CLIP，可以不需要另外的 CLIP 和 VAE
