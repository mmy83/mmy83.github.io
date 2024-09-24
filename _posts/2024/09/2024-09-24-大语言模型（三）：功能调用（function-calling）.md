---
title: 大语言模型（三）：功能调用（function-calling）
author: mmy83
date: 2024-09-24 14:10:00 +0800
categories: [IT技术, AI]
tags: [AI, 智能体, 大模型, 人工智能, rag, 知识库, 插件, function-calling, 工作流]
math: true
mermaid: true
image:
  path: /images/2024/09/2024-09-24/大语言模型（三）：功能调用（function-calling）/大语言模型（三）：功能调用（function-calling）-00.png
  lqip: data:image/webp;base64,UklGRl4AAABXRUJQVlA4IFIAAADQAQCdASoIAAgAAUAmJYgCdADz22dTEAD+9f4TC1aXVnMD6E7Ot76TGz/m4VhxCJitbhwH+fGDFIMDZoV7R8eVH280Mjbwts/8NCGHu1KegAAA
  alt: 大语言模型（三）：功能调用（function-calling）
---

## 介绍

&emsp;&emsp;上一篇的[大语言模型（二）：RAG技术](/posts/大语言模型-二-rag技术/)中，介绍了简单的RAG技术。但是RAG技术只是大模型的一种技术。这里就简单介绍下function-calling功能调用。

+ RAG：RAG技术是通过构建知识库，然后在知识库中查询出问题相关的资料，然后提交给大模型的过程。这个技术其实是在调用大模型前就把问题答案相关的资料在知识库中查询出来，然后让大模型帮我们重新组织答案润色的过程。

+ 功能调用：功能调用（function-calling）则是把我们的问题交给大模型，并且给大模型一些可以调用的外部工具，让大模型帮我们分析调用哪个功能可以解决我们的问题的过程。

&emsp;&emsp;如果用新招的毕业生来工作的场景来理解就是，这个毕业生目前还只是个跑腿的，老板说你算一下公司业绩，他就跑财务部门让财务算一个，然后把结果给老板看。老板说你去写个代码吧，他就找技术部门写个代码，然后给老板。其实他啥也没做，就是跑腿的，但是他只是谁能做好这个事情。不管是这个角色还是这个处理问题的思路是很重要的。

## 功能调用

&emsp;&emsp;首先要说的是不是所有模型都支持功能调用的，比如```llama3```之前就不支持。用```Ollama```运行```qwen2```也是不支持的，但是```qwen2.5```是支持的。对于公共大模型现在基本都支持，比如腾讯的混元就有 ```hunyuan-pro```、```hunyuan-functioncall``` 都支持。而且 ```hunyuan-functioncall``` 效果更好，成本更低。

&emsp;&emsp;不光是模型的支持问题，```Langchain```框架也支持功能调用。但是有些模型也是不支持的。比如腾讯的混元，```Langchain```框架可以支持聊天，但是却不支持功能调用。但是可以支持```Ollama```。

&emsp;&emsp;功能调用需要我们自己写插件，然后给大模型调用。但是他有一个好处，我们可以在插件代码里更好地做权限管理等。

![功能调用](/images/2024/09/2024-09-24/大语言模型（三）：功能调用（function-calling）/大语言模型（三）：功能调用（function-calling）-01.png)

## 参考资料

1、[Function Calling: Integrate Your GPT Chatbot With Anything](https://semaphoreci.com/blog/function-calling)

2、[Ollama: 使用Langchain的OllamaFunctions](https://zhuanlan.zhihu.com/p/701616275)