---
title: SD 提示词
date: 2023-11-29T03:32:00+08:00
updated: 2024-08-21T10:32:30+08:00
dg-publish: false
aliases:
  - SD 提示词
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

正面提示词后添加：

```
(masterpiece:1,2), best quality, masterpiece, highres, original, extremely detailed wallpaper, perfect lighting,(extremely detailed CG:1.2), drawing, paintbrush,
```

负面提示词后添加：

```
NSFW, (worst quality:2), (low quality:2), (normal quality:2), lowres, normal quality, ((monochrome)), ((grayscale)), skin spots, acnes, skin blemishes, age spot, (ugly:1.331), (duplicate:1.331), (morbid:1.21), (mutilated:1.21), (tranny:1.331), mutated hands, (poorly drawn hands:1.5), blurry, (bad anatomy:1.21), (bad proportions:1.331), extra limbs, (disfigured:1.331), (missing arms:1.331), (extra legs:1.331), (fused fingers:1.61051), (too many fingers:1.61051), (unclear eyes:1.331), lowers, bad hands, missing fingers, extra digit,bad hands, missing fingers, (((extra arms and legs))),
```

\* 广泛适用于二次元风格，可以考虑搭配不同模型使用！

找 tag 关键词网站：

可参考 Civitai | Stable Diffusion models, embeddings, hypernetworks and more 中优秀作品的提示词作为模板。

其他网站还有：

ChatGPT：[https://chat.openai.com/](https://link.zhihu.com/?target=https%3A//chat.openai.com/)

AI Creator：[https://ai-creator.net/arts](https://link.zhihu.com/?target=https%3A//ai-creator.net/arts)

NovelAI：[https://spell.novelai.dev](https://link.zhihu.com/?target=https%3A//spell.novelai.dev)

魔咒百科词典：[https://aitag.top](https://link.zhihu.com/?target=https%3A//aitag.top)

AI 咒术生成器：[https://tag.redsex.cc/](https://link.zhihu.com/?target=https%3A//tag.redsex.cc/)

AI 词汇加速器 AcceleratorI Prompt：

词图 PromptTool：[https://www.prompttool.com/NovelAI](https://link.zhihu.com/?target=https%3A//www.prompttool.com/NovelAI)

鳖哲法典：[http://tomxlysplay.com.cn/#/](https://link.zhihu.com/?target=http%3A//tomxlysplay.com.cn/%23/)

Danbooru tag：Tag Groups Wiki | Danbooru ([http://donmai.us](https://link.zhihu.com/?target=http%3A//donmai.us))

**Prompt 格式优化**

第一段：画质 tag，画风 tag

第二段：画面主体，主体强调，主体细节概括（主体可以是人、事、物、景）画面核心内容

第三段：画面场景细节，或人物细节，embedding tag。画面细节内容

第二段一般提供人数，人物主要特征，主要动作（一般置于人物之前），物体主要特征，主景或景色框架等

1\. 越靠前的 Tag 权重越大。

2\. 生成图片的大小会影响 Prompt 的效果，图片越大需要的 Prompt 越多，不然 Prompt 会相互污染。

3.Stable-diffusion 中，可以使用括号人工修改提示词的权重，方法如下：

(word) - 将权重提高 1.1 倍

((word)) - 将权重提高 1.21 倍（= 1.1 * 1.1）

[word] - 将权重降低至原先的 90.91%

(word:1.5) - 将权重提高 1.5 倍

(word:0.25) - 将权重减少为原先的 25%

请注意，权重值最好不要超过 1.5

4.\ Prompt 支持使用 emoji，可通过添加 emoji 达到表现效果。如 形容表情， 可修手。

5.\“+” ， “ AND” ， “|” 用法：“+”和“ AND ”都是用于连接短 Tag，但 AND 两端要加空格。"+" 约等于 " and "；“|” 为循环绘制符号（融合符号）(Prompt A: w1)|(Prompt B: w2)

以上表达适用于 WebUI，w1、w2 为权重。AI 会对 A、 B 两 Prompt 进行循环绘制。可往后无限加入 Prompt。

6.\tag 不一定是多么充满细节，只要模型稳定。小图 + 高分辨率重绘。800*400 的图变成 1600*800，初识小图减少崩坏概率。

7.\关键词最好具有特异性，譬如 Anime(动漫) 一词就相对泛化，而 Jojo 一词就能清晰地指向 Jojo 动漫的画风。措辞越不抽象越好，尽可能避免留下解释空间的措辞。
