---
layout:     post
title: Data Quality Management in Fine-tuning
subtitle: Data Cleaning
date:       2026-03-15
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- Fine-tuning

---

## Background

> Fine-tuning model performance = 70% data structure design + **20% data quality** + 10% training parameters

This article focuses solely on **data quality**, specifically data cleaning.

## Complete Data Cleaning Pipeline

```
Raw data
   ↓
Format standardization
   ↓
Simple deduplication
   ↓
Semantic deduplication
   ↓
Noise filtering
   ↓
Length control
   ↓
Language filtering
   ↓
Semantic consistency check
   ↓
Perplexity filtering
   ↓
Final SFT data
```

## Data Cleaning

### 1 Deduplication

> Real-world data often contains a lot of duplicate entries.

**Method 1: Simple text deduplication**

```
seen = set()

clean_data = []

for item in data:

    text = item["messages"][0]["content"]

    if text not in seen:
        seen.add(text)
        clean_data.append(item)
```

**Method 2: Semantic deduplication**

> Many questions are phrased differently but mean the same thing:

```
What is Python?
Explain Python.
Tell me about Python language.
```
Use **embedding similarity to deduplicate**.

For example, use:

- OpenAI embeddings
- Hugging Face sentence-transformers

Approach:
```
Compute embeddings
↓
cosine similarity
↓
>0.9 Remove duplicates
```
Refer to the article: “Incremental Vector Update Strategy for Embeddings”

[https://strictfrog.com/2026/03/14/Embedding%E4%B9%8B%E5%A2%9E%E9%87%8F%E5%90%91%E9%87%8F%E6%9B%B4%E6%96%B0%E7%AD%96%E7%95%A5/](https://strictfrog.com/2026/03/14/Embedding%E4%B9%8B%E5%A2%9E%E9%87%8F%E5%90%91%E9%87%8F%E6%9B%B4%E6%96%B0%E7%AD%96%E7%95%A5/)

### 2 Noise Filtering

**Method 1: Simple rules based on keywords**

```
bad_words = ["I don't know", "Sorry", "Maybe"]

def is_bad(text):

    for w in bad_words:
        if w in text:
            return True

    return False
```

**Method 2: Remove empty responses**

### 3 Format Standardization

> Normalize different data sources into a fixed JSON format.

For example, this article converts Alpaca data format into SFT data structure.

“Simple Data Preparation and Preprocessing for Fine-tuning”:

[https://strictfrog.com/2026/03/15/Fine-tuning%E4%B9%8B%E7%AE%80%E5%8D%95%E7%9A%84%E6%95%B0%E6%8D%AE%E5%87%86%E5%A4%87%E4%B8%8E%E9%A2%84%E5%A4%84%E7%90%86/](https://strictfrog.com/2026/03/15/Fine-tuning%E4%B9%8B%E7%AE%80%E5%8D%95%E7%9A%84%E6%95%B0%E6%8D%AE%E5%87%86%E5%A4%87%E4%B8%8E%E9%A2%84%E5%A4%84%E7%90%86/)

### 4 Length Control

Many training frameworks require: < 4096 tokens

Use tokenizer to calculate:
```
from transformers import AutoTokenizer

tokenizer = AutoTokenizer.from_pretrained("gpt2")

tokens = tokenizer.encode(text)

if len(tokens) > 4096:
    continue
```

### 5 Language Filtering

If the training model is Chinese, filter out English, Japanese, and other languages.

### 6 Semantic Quality Filtering

Example: Question and answer mismatch

Q: What is Python?  
A: Tokyo is the capital of Japan.

Solution: Use similarity scoring based on a fine-tuned model to evaluate relevance.

### 7 Perplexity Filtering

Idea:
```
Use a language model to calculate the perplexity.

If perplexity is too high:

It indicates poor text quality.
```

For example, nonsensical strings like asdlfjlasdjfl should be removed.

Common models:

- GPT-2
- KenLM

### 8 Conversation Completeness Check

> For multi-turn conversation data:

Ensure the turn order:
```
user
assistant
user
assistant
```
Avoid sequences like:
```
user
user
assistant
assistant
```
Check method:
```
roles = [m["role"] for m in messages]

for i in range(len(roles)-1):
    if roles[i] == roles[i+1]:
        return False
```

### 9 Data Quality Scoring

| Dimension   | Meaning           |
| --------- | ---------------- |
| Semantic Matching | Q/A relevance |
| Language Quality  | Grammar correctness |
| Length Reasonableness | Not overly long |
| Diversity         | Different topics |

Score range 0-1, remove if below 0.6

```
def score_data(question, answer):

    prompt = f"""
Rate the quality of this QA pair from 0-1.

Q: {question}
A: {answer}

Only output a number.
"""

    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role":"user","content":prompt}]
    )

    return int(response.choices[0].message.content)
```

## Data Scale Reference

For example:
```
Raw data 1,000,000
↓
After cleaning 100,000
↓
Final training 50,000
```

| Final Training Size | Effect       |
| ------------------ | ----------- |
| 1,000              | Slight improvement |
| 5,000              | Noticeable improvement |
| 10,000             | Quite good  |
| 50,000             | Near professional level |