---
layout:     post
title: Simple Data Preparation and Preprocessing for Fine-tuning
subtitle: Fine-tuning
date:       2026-03-15
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- Fine-tuning

---

## Background

> For private LLMs, the most important part is fine-tuning the LLM with private data. This article first explains how to prepare and preprocess the data.

## Data Preparation

By downloading the "Stanford Alpaca training dataset," you obtain a JSON file in the following format:  
**alpaca_data.json**

```
[
    {
        "instruction": "Give three tips for staying healthy.",
        "input": "",
        "output": "1.Eat a balanced diet and make sure to include plenty of fruits and vegetables. \n2. Exercise regularly to keep your body active and strong. \n3. Get enough sleep and maintain a consistent sleep schedule."
    },
.......
    {
        "instruction": "What are the three primary colors?",
        "input": "",
        "output": "The three primary colors are red, blue, and yellow."
    }
]
```

## Data Preprocessing

The data format required for fine-tuning (**basic SFT data structure**) is as follows:

```
{
	"message":[
		{
			"role": "user",
			"content": "xxxx_question_1"
		},
		{
			"role": "assistant",
			"content": "xxxx_answer_1"
		}
	]
}
```

A Python script is used to convert the format of the data in `alpaca_data.json` to the required format for fine-tuning.  
The converted data includes two datasets: **training dataset** and **validation dataset** with a 7:3 split.

```
import json
import random

def load_data():
    with open('alpaca_data.json','r',encoding='utf-8') as f:
        data=json.load(f)
    return data

def conversion_to_jsonl(data):

    random.shuffle(data)

    split=int(len(data)*0.7)

    train=data[:split]
    valid=data[split:]

    def convert(item):
        return {
            "messages":[
                {
                    "role":"user",
                    "content":item["instruction"]+"\n"+item.get("input","")
                },
                {
                    "role":"assistant",
                    "content":item["output"]
                }
            ]
        }

    with open("train.jsonl","w",encoding="utf-8") as f:
        for item in train:
            f.write(json.dumps(convert(item),ensure_ascii=False)+"\n")

    with open("valid.jsonl","w",encoding="utf-8") as f:
        for item in valid:
            f.write(json.dumps(convert(item),ensure_ascii=False)+"\n")

if __name__ == "__main__":
    original_data = load_data()
    conversion_to_jsonl(original_data)

```

The full code repository link is as follows:

[https://github.com/LuochuanAD/Fine-tuning-Learn](https://github.com/LuochuanAD/Fine-tuning-Learn)

## References

**Data source:**

"Stanford Alpaca training dataset":  

[https://raw.githubusercontent.com/tatsu-lab/stanford_alpaca/refs/heads/main/alpaca_data.json](https://raw.githubusercontent.com/tatsu-lab/stanford_alpaca/refs/heads/main/alpaca_data.json)