---
layout:     post
title: SFT Data Sources for Fine-tuning
subtitle: Data Source Analysis
date:       2026-03-15
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- Fine-tuning

---

## Background

> What are the sources of SFT data for fine-tuning LMs?


## 1 Open Source Datasets

Common ones:

Stanford University Alpaca

[https://raw.githubusercontent.com/tatsu-lab/stanford_alpaca/refs/heads/main/alpaca_data.json](https://raw.githubusercontent.com/tatsu-lab/stanford_alpaca/refs/heads/main/alpaca_data.json) 


Hugging Face datasets

[https://huggingface.co/datasets](https://huggingface.co/datasets)

OpenAI OpenAssistant

[https://github.com/LAION-AI/Open-Assistant/tree/main/oasst-data](https://github.com/LAION-AI/Open-Assistant/tree/main/oasst-data)

## 2 Auto-Generatng Data with LLMs (Most Common)

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

## 3 Scraping Real Conversations

Sources:

- Reddit
- StackOverflow
- GitHub issues
- Wikipedia

Real user questions tend to have very high quality.

So you would need to write scrapers; examples are not provided here.


## 4 Private Database Knowledge

> Enterprise RAG knowledge bases, documents, websites



## 5 Manually Curated High-Quality Data

For formatting, refer to the following article:  
“High-Quality SFT Data Structure Design for Fine-tuning”

[https://strictfrog.com/2026/03/15/Fine-tuning%E4%B9%8B%E9%AB%98%E8%B4%A8%E9%87%8FSFT%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E8%AE%BE%E8%AE%A1/](https://strictfrog.com/2026/03/15/Fine-tuning%E4%B9%8B%E9%AB%98%E8%B4%A8%E9%87%8FSFT%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E8%AE%BE%E8%AE%A1/)