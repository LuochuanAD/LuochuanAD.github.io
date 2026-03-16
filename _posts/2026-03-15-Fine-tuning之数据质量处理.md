---
layout:     post
title:      Fine-tuning之数据质量处理
subtitle:   数据清洗
date:       2026-03-15
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- Fine-tuning

---

## 背景

> 微调模型能力 = 70% 数据结构设计 + **20% 数据质量** + 10% 训练参数

这篇文章只讲**数据质量**, 也就是数据清洗


## 完整数据清洗 Pipeline

```
原始数据
   ↓
格式统一
   ↓
简单去重
   ↓
语义去重
   ↓
垃圾过滤
   ↓
长度控制
   ↓
语言过滤
   ↓
语义匹配检测
   ↓
困惑度过滤
   ↓
最终SFT数据
```

## 数据清洗

### 1 去重复

> 真实数据包含很多重复数据

**方法1: 简单文本去重**

```
seen = set()

clean_data = []

for item in data:

    text = item["messages"][0]["content"]

    if text not in seen:
        seen.add(text)
        clean_data.append(item)
```

**方法2：语义去重**

> 很多问题表达不同但意思一样：

```
What is Python?
Explain Python.
Tell me about Python language.
```
可以用 **embedding 相似度去重。**

例如使用：

- OpenAI embedding
- Hugging Face sentence-transformers


思路：
```
计算 embedding
↓
cosine similarity
↓
>0.9 删除
```
可以参考以下文章: “Embedding之增量向量更新策略”

[https://strictfrog.com/2026/03/14/Embedding%E4%B9%8B%E5%A2%9E%E9%87%8F%E5%90%91%E9%87%8F%E6%9B%B4%E6%96%B0%E7%AD%96%E7%95%A5/](https://strictfrog.com/2026/03/14/Embedding%E4%B9%8B%E5%A2%9E%E9%87%8F%E5%90%91%E9%87%8F%E6%9B%B4%E6%96%B0%E7%AD%96%E7%95%A5/)


### 2 去垃圾数据

**方法1: 简单规则: 基于关键词**

```

bad_words = ["I don't know", "Sorry", "Maybe"]

def is_bad(text):

    for w in bad_words:
        if w in text:
            return True

    return False
```

**方法2: 删除空回答**

### 3 统一格式

> 将不同的数据来源格式,统一成固定格式json.

如下面这篇文章,将alpaca的数据格式 转换成SFT的数据结构.

“Fine-tuning之简单的数据准备与预处理”:

[https://strictfrog.com/2026/03/15/Fine-tuning%E4%B9%8B%E7%AE%80%E5%8D%95%E7%9A%84%E6%95%B0%E6%8D%AE%E5%87%86%E5%A4%87%E4%B8%8E%E9%A2%84%E5%A4%84%E7%90%86/](https://strictfrog.com/2026/03/15/Fine-tuning%E4%B9%8B%E7%AE%80%E5%8D%95%E7%9A%84%E6%95%B0%E6%8D%AE%E5%87%86%E5%A4%87%E4%B8%8E%E9%A2%84%E5%A4%84%E7%90%86/)

### 4 长度控制

很多训练框架要求：< 4096 tokens

利用tokenizer计算:
```
from transformers import AutoTokenizer

tokenizer = AutoTokenizer.from_pretrained("gpt2")

tokens = tokenizer.encode(text)

if len(tokens) > 4096:
    continue
```

### 5 语言过滤

如果训练模型是中文,那就要过滤英语,日语等等

### 6 语义质量过滤

例如：问题和答案不匹配

Q: What is Python?
A: Tokyo is the capital of Japan.

解决方法: 基于相似度微调模型计算相关性评分

### 7 困惑度过滤（Perplexity Filtering）

思路：
```
用一个语言模型计算 困惑度（Perplexity）。

如果困惑度太高：

说明文本质量差。
```

例如： asdlfjlasdjfl 这种会被删除。

常用模型：

- GPT-2
- KenLM

### 8 对话完整性检查

> 如果是 多轮对话数据：

需要保证：
```
user
assistant
user
assistant
```
不能出现：
```
user
user
assistant
assistant
```
检查方法：
```
roles = [m["role"] for m in messages]

for i in range(len(roles)-1):
    if roles[i] == roles[i+1]:
        return False
```

### 9 数据质量评分

| 维度   | 含义    |
| ---- | ----- |
| 语义匹配 | Q/A相关 |
| 语言质量 | 语法正确  |
| 长度合理 | 不过长   |
| 多样性  | 不同主题  |

评分纬度0-1, 低于0.6删除

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


## 数据规模参照

例如:
```
原始数据 100万
↓
清洗后 10万
↓
最终训练 5万
```

| 最终训练   | 效果   |
| ----- | ---- |
| 1000  | 轻微改善 |
| 5000  | 明显改善 |
| 10000 | 很不错  |
| 50000 | 接近专业 |











