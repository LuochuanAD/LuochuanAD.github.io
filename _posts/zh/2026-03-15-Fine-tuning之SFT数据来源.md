---
layout:     post
title:      Fine-tuning之SFT数据来源
subtitle:   数据来源分析
date:       2026-03-15
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- Fine-tuning

---

## 背景

> 用于微调LM的SFT数据来源有哪些?


## 1 开源数据集

常见：

Stanford University Alpaca

[https://raw.githubusercontent.com/tatsu-lab/stanford_alpaca/refs/heads/main/alpaca_data.json](https://raw.githubusercontent.com/tatsu-lab/stanford_alpaca/refs/heads/main/alpaca_data.json) 


Hugging Face datasets

[https://huggingface.co/datasets](https://huggingface.co/datasets)

OpenAI OpenAssistant

[https://github.com/LAION-AI/Open-Assistant/tree/main/oasst-data](https://github.com/LAION-AI/Open-Assistant/tree/main/oasst-data)

## 2 用 LLM 自动生成数据（最常见）

```
import openai
import json

prompt = "Generate 50 math reasoning QA pairs with step-by-step reasoning."

def generate_data(prompt):
	
    response = openai.chat.completions.create(
        model="gpt-4",
        messages=[{"role":"user","content":prompt}]
    )

    return response.choices[0].message.content

```

## 3 爬取真实对话

来源：

- Reddit
- StackOverflow
- GitHub issues
- Wikipedia

真实用户问题质量很高。

所以需要需要写爬虫了,这里就不提供了.


## 4 私有数据库知识

> 企业RAG知识库, 文档, 网站



## 5 人工编写高质量数据

格式请参照下面这篇文章:
“Fine-tuning之高质量SFT数据结构设计”

[https://strictfrog.com/2026/03/15/Fine-tuning%E4%B9%8B%E9%AB%98%E8%B4%A8%E9%87%8FSFT%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E8%AE%BE%E8%AE%A1/](https://strictfrog.com/2026/03/15/Fine-tuning%E4%B9%8B%E9%AB%98%E8%B4%A8%E9%87%8FSFT%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E8%AE%BE%E8%AE%A1/)







