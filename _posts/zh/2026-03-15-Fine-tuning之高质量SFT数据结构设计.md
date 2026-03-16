---
layout:     post
title:      Fine-tuning之高质量SFT数据结构设计
subtitle:   高质量SFT数据结构
date:       2026-03-15
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- Fine-tuning
- SFT

---

## 背景

> 微调模型能力 = **70% 数据结构设计** + 20% 数据质量 + 10% 训练参数

这篇文章只讲 ”**数据结构设计**“.

## 高质量SFT数据的5个层级

1. Level 1  基础问答
2. Level 2  System Prompt
3. Level 3  思考链 (CoT)
4. Level 4  多轮对话
5. Level 5  Tool / RAG

## 分析

### Level 1 基础问答

> 基础应答能力,会胡说,不会推理

```
{
 "messages":[
  {"role":"user","content":"What is XAI?"},
  {"role":"assistant","content":"XAI is the process..."}
 ]
}
```

### Level 2  System Prompt

> 增加了 system, 让模型学习角色行为,并且形成某种人格

```
{
 "messages":[
  {
   "role":"system",
   "content":"You are a professional biology teacher."
  },
  {
   "role":"user",
   "content":"Explain photosynthesis"
  },
  {
   "role":"assistant",
   "content":"Photosynthesis is..."
  }
 ]
}
```

### Level 3  思考链 (CoT)

> 提升推理能力

```
{
 "messages":[
  {"role":"user","content":"If I have 20 apples and give away 7, how many remain?"},
  {"role":"assistant","content":
   "Let's think step by step.\n
    1. Start with 20 apples\n
    2. Give away 7 apples\n
    3. 20 - 7 = 13\n
    Therefore, 13 apples remain."
  }
 ]
}
```

### Level 4  多轮对话

> 真实聊天是多轮会话的,模型会学会上下文记忆和连续推理

```
{
 "messages":[
  {"role":"system","content":"You are a math tutor."},
  {"role":"user","content":"What is 12 * 8?"},
  {"role":"assistant","content":"12 × 8 = 96."},
  {"role":"user","content":"What about 96 ÷ 3?"},
  {"role":"assistant","content":"96 ÷ 3 = 32."}
 ]
}
```

### Level 5  Tool / RAG

> 模型会根据用户问题,学会选择工具,调用API,返回答案

```
{
 "messages":[
  {"role":"user","content":"What's the weather in Tokyo?"},
  {
   "role":"assistant",
   "content":"I will check the weather API.",
   "tool_call":{
      "name":"get_weather",
      "arguments":{"city":"Tokyo"}
   }
  },
  {
   "role":"tool",
   "content":"Temperature: 20°C"
  },
  {
   "role":"assistant",
   "content":"The temperature in Tokyo is 20°C."
  }
 ]
}
```

> 模型会读取上下文Context

```
{
 "messages":[
  {
   "role":"system",
   "content":"Use the provided context to answer."
  },
  {
   "role":"user",
   "content":"Context: Tokyo is the capital of Japan.\nQuestion: What is the capital of Japan?"
  },
  {
   "role":"assistant",
   "content":"The capital of Japan is Tokyo."
  }
 ]
}
```
## 推荐高质量的SFT数据结构如下

```
{
 "messages":[
  {"role":"system","content":"You are a helpful AI."},

  {"role":"user","content":"Calculate 25 * 14"},

  {"role":"assistant","content":
  "Let's solve step by step.\n
   25 × 14 = (25 × 10) + (25 × 4)\n
   = 250 + 100\n
   = 350\n
   Therefore the answer is 350."
  }
 ]
}
```
