---
title: Stable Diffusion：部署
author: mmy83
date: 2024-06-05 11:43:00 +0800
categories: [专题, "Stable Diffusion"]
tags: [AI, 人工智能, 'Stable Diffusion', 图片, webui, 私有化, 部署]
math: true
mermaid: true
image:
  path: /images/2024/06/2024-06-05/Stable Diffusion：部署/Stable Diffusion：部署-00.png
  lqip: data:image/webp;base64,UklGRl4AAABXRUJQVlA4IFIAAADwAQCdASoIAAUAAUAmJagCdLoAArdxJjAA/ub4PGugsGehnaBj5/1k25pvuPB6ozm2dp/v9JvCTRui3/ahw7DXJUryE5a/9MGsRGzG8xOqhQAA
  alt: Stable Diffusion：部署
---

## 前言

&emsp;&emsp;随着 ChatGPT 的出现，AI 变得异常火爆，如果你不知道 AI ，好像就和这个高速发展的时代脱节了。所有最近也一直在研究 AI 相关的技术，之前也做过 [私有化部署Llama](/posts/私有化部署Llama)，用来生成文本，部署过程很简单的，增加了不少信心，今天打算再部署一个 Stable Diffusion，用于生成图片。

## 环境

&emsp;&emsp;部署之前看了不少资料，都是说可以不用显卡，只使用CPU也可以，就是生成速度慢。资源有限，先用虚拟机部署试试。

|系统|CPU|内存|磁盘|ip|备注|
|---|---|---|---|---|---|
|ubuntu-22.04-server|4核|8G|200G|192.168.1.248|kvm虚拟机|

## 安装

&emsp;&emsp;安装Stable Diffusion工具有三个，分别是：

- [StableSwarmUI](https://github.com/Stability-AI/StableSwarmUI.git): 官方提供的WebUI，使用起来比较简单。
- [Stable Diffusion WebUI](https://github.com/AUTOMATIC1111/stable-diffusion-webui.git)：使用的比较多
- [ComfyUI](https://github.com/comfyanonymous/ComfyUI.git) ：也比较常见，采用流程化的界面

&emsp;&emsp;因为之前用百度飞桨部署过 Stable Diffusion WebUI，觉得挺好用的，所以这次私有化部署还是采用Stable Diffusion WebUI。毕竟第一次部署，选择熟悉的可以避免踩坑。另外两种部署方式，以后有时间再研究。

### 创建环境

&emsp;&emsp;创建环境，这里采用conda来管理环境。如果没有conda环境，可以参考[conda安装与使用](/posts/conda安装与使用)。

```bash
# 创建一个名为 stable-diffusion 的环境，python版本为 3.10
conda create -n stable-diffusion python=3.10
# 激活环境
conda activate stable-diffusion

# 下载项目
git clone https://github.com/Stability-AI/stable-diffusion-webui.git
# 进入项目
cd stable-diffusion-webui 
# 切换到 v1.9.4 版本
git checkout v1.9.4

# 安装依赖
pip install -r requirements.txt
```

### 下载模型

#### 下载CLIP模型文件

&emsp;&emsp;CLIP模型文件本来是不需要下载的，但是因为网络原因，往往下载失败，需要配合魔法才能下载成功，所以这里单独下载。下载地址：

- 需要魔法 [clip-vit-large-patch14](https://huggingface.co/openai/clip-vit-large-patch14/tree/main)。

- 百度网盘分享 链接: [https://pan.baidu.com/s/1axJdZlchmRScb7klwi6_LA?pwd=2q2h](https://pan.baidu.com/s/1axJdZlchmRScb7klwi6_LA?pwd=2q2h) 提取码: 2q2h

&emsp;&emsp;下载后，将文件解压后放到项目目录下的 stable-diffusion-webui/openai 文件夹下，如果没有openai文件夹，就手动创建。

```cosole
├── stable-diffusion-webui
│   ├── openai
│   │   └── clip-vit-large-patch14
│   │       ├── config.json
│   │       ├── flax_model.msgpack
│   │       ├── gitattributes.txt
│   │       ├── ILSVRC2013_devkit.tgz
│   │       ├── merges.txt
│   │       ├── model.safetensors
│   │       ├── preprocessor_config.json
│   │       ├── pytorch_model.bin
│   │       ├── README.md
│   │       ├── setuptools-40.8.0-py2.py3-none-any.whl
│   │       ├── setuptools-45.2.0-py3-none-any.whl
│   │       ├── special_tokens_map.json
│   │       ├── tf_model.h5
│   │       ├── tokenizer_config.json
│   │       ├── tokenizer.json
│   │       └── vocab.json
```

> 注：
> 这一步其实是可以省略从，但是因为国内网络环境问题，自动下载往往失败，所以这里单独下载。
> 这些文件并不是都有用，但是第一次安装，还是都下载，避免失败。
{: .prompt-tip }

#### 下载模型文件

&emsp;&emsp;Stable Diffusion WebUI 提供了多种模型，可以根据需要选择，这里先下载一个2.1版本的模型体验一下。
[https://huggingface.co/webui/stable-diffusion-2-1/tree/main](https://huggingface.co/webui/stable-diffusion-2-1/tree/main)

![stable-diffusion-2-1](/images/2024/06/2024-06-05/Stable%20Diffusion：部署/Stable%20Diffusion：部署-01.png)

&emsp;&emsp;下载下来后，将文件解压到 stable-diffusion-webui/models 文件夹下。

```console
stable-diffusion-webui
    ├── models
    │   ├── Stable-diffusion
    │   │   ├── Put Stable Diffusion checkpoints here.txt
    │   │   └── v2-1_768-ema-pruned.safetensors
```

> 注：
> 这一步也可以启动后进行，启动时没有模型，会自动下载一个，但是因为网络环境原因，可能失败，所有这里先下载好。
{: .prompt-tip }

### 启动前准备

&emsp;&emsp;启动前，需要根据自己的环境，修改配置文件。我是通过conda来管理环境，所以需要修改一下 webui.sh 文件。

```bash
# vi webui.sh

# 因为我是通过conda来管理环境，所以需要把use_venv=1改为use_venv=0
use_venv=0
# 如果你是通过root运行，还需要把can_run_as_root=0改为can_run_as_root=1
can_run_as_root=1

# 因为我的机器没有GPU，所以需要在最后添加 export CUDA_VISIBLE_DEVICES=-1
export CUDA_VISIBLE_DEVICES=-1
# 添加启动参数，但是我这里添加好像没起啥作用
# export COMMANDLINE_ARGS="--listen --gradio-auth mmy83:mmy83 --use-cpu all --lowvram --no-half --precision full --skip-python-version-check --skip-torch-cuda-test --enable-insecure-extension-access"
```

## 启动服务

```bash
# --listen 监听服务
# --gradio-auth mmy83:mmy83 开启认证并设置账号密码
# --use-cpu all 使用cpu
# --lowvram 使用低画质
# --skip-torch-cuda-test 跳过CUDA检查
# --enable-insecure-extension-access  这个参数用于允许安装插件

./webui.sh --listen --gradio-auth mmy83:mmy83 --use-cpu all --lowvram --skip-torch-cuda-test --precision full --no-half
```

![启动成功](/images/2024/06/2024-06-05/Stable%20Diffusion：部署/Stable%20Diffusion：部署-02.png)

> 注：
> 如果是本机安装，不需要```--gradio-auth mmy83:mmy83```参数，也不需要登录，但是访问地址只能是本机访问```http://127.0.0.1:7860```。

## 问题解决

```console
ImportError: libGL.so.1: cannot open shared object file: No such file or directory
```

&emsp;&emsp;这个问题是因为缺少libGL.so.1，安装即可解决。

```bash
pip uninstall opencv-python -y
pip install opencv-python-headless -i https://pypi.tuna.tsinghua.edu.cn/simple
```

## 访问服务

&emsp;&emsp;访问服务，使用浏览器访问 [http://192.168.1.248:7860](http://192.168.1.248:7860)

![页面](/images/2024/06/2024-06-05/Stable%20Diffusion：部署/Stable%20Diffusion：部署-03.png)

## 测试

### 提示词

```txt
正面提示词:
1 girl a 24 y o woman, blonde, dark theme, soothing tones, muted colors, high contrast, look at at viewer, contrasty , vibrant , intense, stunning, captured in the late afternoon sunlight, using a Canon EOS R6 and a 16-35mm to capture every detail and angle, with emphasis on the lighting and shadows, late afternoon sunlight, 8K
反面提示词:
(deformed, distorted, disfigured, doll:1.3), poorly drawn, bad anatomy, wrong anatomy, extra limb, missing limb, floating limbs, (mutated hands and fingers:1.4), disconnected limbs, mutation, mutated, ugly, disgusting, blurry, amputation, 3d, illustration, cartoon, flat , dull , soft, (deformed, distorted, disfigured:1.3), poorly drawn, bad anatomy, wrong anatomy, extra limb, missing limb, floating limbs,
```

### 效果

![效果](/images/2024/06/2024-06-05/Stable%20Diffusion：部署/Stable%20Diffusion：部署-04.png)

## 模型下载站点

网上有很多人分享了很多的模型，可以下载使用，这里推荐两个站点：

- 国外站点（需要魔法）： [https://huggingface.co/](https://huggingface.co/)
- 国外站点（需要魔法）： [https://civitai.com/](https://civitai.com/)
- 国内站点（不需要魔法）： [https://www.liblib.art/](https://www.liblib.art/)

> 注：
> 下载的时候注意，这些站点提供的模型的类型分为：CHECKPOINT 、LORA，上面下载的是CHECKPOINT，而不是LORA。
> 整体安装下来其实并不麻烦，主要是网络环境问题，下载失败，需要魔法。而且模型文件都很大。
> 由于使用CPU，生成速度确实很慢，5分钟左右才可以生成一张。
{: .prompt-tip }
