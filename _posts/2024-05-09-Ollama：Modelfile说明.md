---
title: Ollama：Modelfile说明
author: mmy83
date: 2024-05-09 22:09:00 +0800
categories: [IT技术, AI]
tags: [AI, 人工智能, Llama, ollama, 私有化部署, Modelfile]
math: true
mermaid: true
image:
  path: /images/2024-05-09/Ollama：Modelfile说明/Ollama：Modelfile说明-00.png
  lqip: data:image/webp;base64,UklGRlwAAABXRUJQVlA4IFAAAACwAQCdASoIAAUAAUAmJQBOgB6SSnegAP75MHyG/wP34tKiBQgeTFvXzOel/pQ77hOVx8FfO28ueyZ6JGWoGwoOK69nJuvkCroDK1STL4AAAA==
  alt: Ollama：Modelfile说明
---

## Modelfile

&emsp;&emsp;在Ollama中，模型文件（Model File）是一个蓝图，它用于创建和共享模型。

## 实例

```shell
{%raw%}
FROM ./Llama3-8B-Chinese-Chat.q6_k.GGUF
TEMPLATE """{{ if .System }}<|start_header_id|>system<|end_header_id|>

{{ .System }}<|eot_id|>{{ end }}{{ if .Prompt }}<|start_header_id|>user<|end_header_id|>

{{ .Prompt }}<|eot_id|>{{ end }}<|start_header_id|>assistant<|end_header_id|>

{{ .Response }}<|eot_id|>"""
PARAMETER stop "<|start_header_id|>"
PARAMETER stop "<|end_header_id|>"
PARAMETER stop "<|eot_id|>"
PARAMETER stop "<|reserved_special_token"

{%endraw%}
```

> 注意: Modelfile 语法还在开发中
{: .prompt-tip }

## 格式

命令 参数

|命令|	描述|
|---|---|
|FROM (必需的)	|定义使用的基模型|
|PARAMETER(参数)	|设置Ollama运行模型的参数|
|TEMPLATE(提示词模板)	|于发送给模型的完整提示模板|
|SYSTEM	|指定将在模板中设置的系统消息|
|ADAPTER	|定义适用于模型的（Q）LoRA适配器|
|LICENSE	|Specifies the legal license.|
|MESSAGE	|指定消息历史|

## 查看模型Modelfile

```shell
ollama show --modelfile llama3:latest
```

## 语法

### FROM

```shell

FROM <model name>:<tag>
# 例如：
FROM llama2
# 或者通过 bin 文件进行构建
FROM ./ollama-model.bin
```


### PARAMETER

```shell
PARAMETER <parameter> <parametervalue>
```

|Parameter(参数名)|	描述|	值的类型	|使用示例|
|---|---|---|---|
|mirostat|	启用Mirostat算法以控制困惑度(perplexity)。 Mirostat算法可以有效减少结果中重复的发生。perplexity是指对词语预测的不确定性 (default: 0, 0 = disabled, 1 = Mirostat, 2 = Mirostat 2.0)	|int|	mirostat 0|
|mirostat_eta	|它影响算法对生成文本反馈的响应速度。学习率较低会导致调整更慢，而较高的学习率则会使算法反应更加迅速。 (Default: 0.1)	|float|	mirostat_eta 0.1|
|mirostat_tau	|控制输出的连贯性和多样性之间的平衡。较低的值会使得文本更集中和连贯，而较高的值则会带来更大的多样性。 (Default: 5.0)	|float	|mirostat_tau 5.0|
|num_ctx|	设置生成下一个token时使用的上下文窗口大小。(Default: 2048)	|int	|num_ctx 4096|
|repeat_last_n	|设定了模型需要回顾多少信息来以防止重复。 (Default: 64, 0 = disabled, -1 = num_ctx)	|int|	repeat_last_n 64|
|repeat_penalty|	设定了重复惩罚的强度。较高的值（例如，1.5）会更强烈地处罚重复，而较低的值（如0.9）则会宽容一些. (Default: 1.1)|	float	|repeat_penalty 1.1|
|temperature|	模型的温度。 temperature通常用于控制随机性和多样性，提高温度意味着更高的随机性，可能导致更出乎意料但可能更有创意的答案。(Default: 0.8)	|float	|temperature 0.7|
|seed|	设置了生成时使用的随机数种子。设置特定的数值将使得模型对于相同的提示会生成相同的文本。(Default: 0)	|int|	seed 42|
|stop|	设置停止序列。当模型遇到这个模式时，会停止生成文本并返回。可以通过在Modelfile中指定多个独立的stop参数来设置多个停止模式。	|string	|stop “AI assistant:”|
|tfs_z|	尾部自由采样被用来减少不那么可能的token对输出的影响。较高的值（例如，2.0）会更大幅度地减小这种影响，而设置为1.0则禁用此功能。(default: 1)	|float|	tfs_z 1|
|num_predict|	生成文本时预测的最大token数量。 (Default: 128, -1 = infinite generation(无限制), -2 = fill context(根据上下文填充完整fill the context to its maximum))	|int|	num_predict 42|
|top_k|	减少生成无意义内容的概率。较高的值（例如，100）会使答案更加多样，而较低的值（如，10）则会更为保守。 (Default: 40)	|int|	top_k 40|
|top_p|	top-k协同工作。较高的值（例如，0.95）将导致更丰富的文本多样性，而较低的值（如，0.5）则会生成更聚焦和保守的内容。(Default: 0.9)|	float|	top_p 0.9|

### TEMPLATE(模板)

&emsp;&emsp;模型接收到的完整提示模板 (TEMPLATE)。它可以包含（可选地）系统消息、用户的消息以及模型的响应。注意：语法可能取决于特定的模型。模板使用Go语言编写。

Template Variables(模板变量)

{%raw%}

|Variable(变量)	|描述|
|---|---|
|{{ .System }}	|用于指定自定义行为的系统消息。|
|{{ .Prompt }}	|用户的提示消息。|
|{{ .Response }}	|来自模型的回应。在生成响应时，这部分之后的文本会被忽略。|

* __{{ if .System }}__: 这是一个条件语句，如果.System变量存在并且为真（即非空），将会执行接下来的代码块。
* __{{ .System }}__: 如果.System变量被定义且包含内容，这部分的内容将被插入到文本中。这里的.通常是上下文中变量的引用符号。
* __{{ end }}__: 当前条件语句或代码块结束的标记。
* __{{ if .Prompt }}__: 同样是一个条件语句，如果.Prompt（可能是指用户的提示信息）存在且非空，将会执行接下来的代码段。
* __{{ .Prompt }}__: 如果.Prompt变量有值，这部分内容也会被插入到输出文本中。
总的来说，这段模板语法用于根据.System和.Prompt变量的是否存在或内容决定是否包含在最终生成的文本中。

{%endraw%}

### SYSTEM

```shell
SYSTEM """<system message>"""
```

### ADAPTER（模型适配器）

&emsp;&emsp;在LLM中，ADAPTER 指令是一个可选的指令，它指定应应用到基础模型上的轻量级（LoRA）适配器。这个值应该是一个绝对路径或者相对于模文件的相对路径，而且必须是GGML格式的文件。如果没有对基础模型进行适当的调优，则使用该适配器的行为是未定义的。

```shell
ADAPTER ./ollama-lora.bin
```

### LICENSE

&emsp;&emsp;LICENSE 指令允许你指定与这个模文件关联使用的模型所采用的法律许可协议。这通常涉及到如何根据开源软件或AI模型的标准许可（如MIT、Apache、GNU等）来分享或分发模型，以确保在使用过程中遵守相关法律法规。

```shell
LICENSE """
<license text>
"""
```

### MESSAGE(消息)

&emsp;&emsp;MESSAGE 指令允许你为模型设置一个消息历史，以便在生成响应时参考。通过多次使用 MESSAGE 命令，你可以构建一段对话，引导模型以类似的方式进行回答，模拟真实的对话流程。这通常用于训练模型理解和维持上下文，使其生成的回复更自然、连贯。

```shell
MESSAGE <role> <message>
```

Valid roles

|Role(角色)	|描述|
|---|---|
|system|	提供给模型的系统消息替代方式。|
|user|	一个用户可能会提出的问题的示例。|
|assistant|	模型应如何响应的一个示例。|

```shell
MESSAGE user Is Toronto in Canada?
MESSAGE assistant yes
MESSAGE user Is Sacramento in Canada?
MESSAGE assistant no
MESSAGE user Is Ontario in Canada?
MESSAGE assistant yes
```

> 注：
>
> 在Modelfile中是不会区分字母大小写的. 为了便于识别，示例中采用了大写字母形式的指令。
>
> 指令可以按照任意顺序放置。在示例中，通常会将 FROM 指令放在最前面，以保持清晰易读性。这可能是因为 FROM 指令常常用于指示信息的来源或者模文件的初始设置，然后跟随的是其他类型的命令等，这种顺序有助于读者快速地理解整个模板或程序流程。
{: .prompt-tip }
