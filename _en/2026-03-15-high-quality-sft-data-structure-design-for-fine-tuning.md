---
layout:     post
title: High-Quality SFT Data Structure Design for Fine-tuning
subtitle: High-Quality SFT Data Structure
date:       2026-03-15
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- Fine-tuning
- SFT

---

## Background

> Fine-tuning model capability = **70% Data Structure Design** + 20% Data Quality + 10% Training Parameters

This article focuses solely on "**Data Structure Design**."

## 5 Levels of High-Quality SFT Data

1. Level 1  Basic Q&A  
2. Level 2  System Prompt  
3. Level 3  Chain of Thought (CoT)  
4. Level 4  Multi-turn Dialogue  
5. Level 5  Tool / RAG

## Analysis

### Level 1 Basic Q&A

> Basic response capability, prone to hallucinations, lacks reasoning

```
{
 "messages":[
  {"role":"user","content":"What is XAI?"},
  {"role":"assistant","content":"XAI is the process..."}
 ]
}
```

### Level 2 System Prompt

> Adds a system prompt to train the model on role behavior and personality

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

### Level 3 Chain of Thought (CoT)

> Enhances reasoning ability

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

### Level 4 Multi-turn Dialogue

> Real conversations are multi-turn; the model learns context memory and continuous reasoning

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

### Level 5 Tool / RAG

> The model learns to select tools, invoke APIs, and return answers based on user queries

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

> The model uses provided Context

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

## Recommended High-Quality SFT Data Structure

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