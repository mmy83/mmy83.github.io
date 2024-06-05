---
title: Stable Diffusion：CLIP模型介绍
author: mmy83
date: 2024-06-05 16:22:00 +0800
categories: [专题, "Stable Diffusion"]
tags: [AI, 人工智能, 'Stable Diffusion', 私有化, 部署, CLIP, 模型]
math: true
mermaid: true
image:
  path: /images/2024-06-05/Stable Diffusion：CLIP模型介绍/Stable Diffusion：CLIP模型介绍-00.png
  lqip: data:image/webp;base64,UklGRkoAAABXRUJQVlA4ID4AAADQAQCdASoIAAMAAUAmJYwCdAEO+yLXgAD+/XL150W219MD5hY5Aoa7wu8VGgOyCWWZiVDYG20V7UJb9+AAAA==
  alt: Stable Diffusion：CLIP模型介绍
---

## 前言

&emsp;&emsp;部署了一个私有化模型 Stable Diffusion ，在部署的时候总是报一个错误，```Can't load tokenizer for 'openai/clip-vit-large-patch14'```，查了一下，是因为网络原因，clip-vit-large-patch14 这个下载失败。然后手动下载，发现这个很大。下载完成后又下载模型文件，也很大。于是就有一个问题：这个文件是啥？经过查找资料发现，这个文件是一个CLIP模型。

## CLIP模型

&emsp;&emsp;CLIP全称Constrastive Language-Image Pre-training，是OpenAI推出的采用对比学习的文本-图像预训练模型。CLIP惊艳之处在于架构非常简洁且效果好到难以置信，在zero-shot文本-图像检索，zero-shot图像分类，文本→图像生成任务guidance，open-domain 检测分割等任务上均有非常惊艳的表现。

![CLIP模型](/images/2024-06-05/Stable%20Diffusion：CLIP模型介绍/Stable%20Diffusion：CLIP模型介绍-01.png)

&emsp;&emsp;通过上面的描述和图，我们可以知道CLIP模型是一个文本-图像预训练模型。可以把CLIP模型看作是一个解决文字和图像对应关系的模型，其中还包含了文本编码和图像编码的过程。

&emsp;&emsp;思考一下，为什么 Stable Diffusion 可以通过我们的一段文字描述，就可以生成一张图片呢？原因很简单，就是通过 CLIP 模型，把文字和图片对应起来了。只不过这种对应不是简单的一一对应，而是通过将文字和图片进行编码，然后通过对比学习，找到最相似的编码。

## 测试

&emsp;&emsp;查看代码 [https://github.com/openai/CLIP](https://github.com/openai/CLIP) 可以看到上面提供了一些测试代码，这里把上面第一个最简单的测试代码运行一下，看看效果。

### 环境安装

&emsp;&emsp;这里使用conda作为环境管理工具，如果没有安装，可以参考[conda安装与使用](/posts/conda安装与使用)

```bash
# 创建环境,python使用3.9版本，我开始用3.10版本，报错
conda create -n clip python=3.9
# 激活环境
conda activate clip
# 安装pytorch，这里使用cpu版本，也可以安装gpu版本，需要显卡支持
# 如果报torch.fx模块不存在错误，就是这里的问题
# 可以按照自己的环境选择需要的版本 https://pytorch.org/get-started/previous-versions/
conda install pytorch==1.7.1 torchvision==0.8.2 torchaudio==0.7.2 cpuonly -c pytorch
pip install ftfy regex tqdm
pip install git+https://github.com/openai/CLIP.git
```

### 测试代码

```python
import torch
import clip
from PIL import Image

device = "cuda" if torch.cuda.is_available() else "cpu"
model, preprocess = clip.load("ViT-B/32", device=device)

# CLIP.png图片就是上面那张图片，你可以换成自己的
image = preprocess(Image.open("CLIP.png")).unsqueeze(0).to(device)
text = clip.tokenize(["a diagram", "a dog", "a cat"]).to(device)

with torch.no_grad():
    image_features = model.encode_image(image)
    text_features = model.encode_text(text)
    
    logits_per_image, logits_per_text = model(image, text)
    probs = logits_per_image.softmax(dim=-1).cpu().numpy()

print("Label probs:", probs)  # prints: [[0.9927937  0.00421068 0.00299572]]
```

&emsp;&emsp;运行上面的代码，输出如下：[[0.9927937  0.00421068 0.00299572]]，这说明图片是一个 "a diagram" ,可以换成其他的图片试试。

## 相关资料

- 不同环境下pytorch版本的选择：[https://pytorch.org/get-started/previous-versions/](https://pytorch.org/get-started/previous-versions/)

- openai的CLIP模型：[https://github.com/openai/CLIP](https://github.com/openai/CLIP)

- 另一个CLIP模型[https://github.com/mlfoundations/open_clip/](https://github.com/mlfoundations/open_clip/)
