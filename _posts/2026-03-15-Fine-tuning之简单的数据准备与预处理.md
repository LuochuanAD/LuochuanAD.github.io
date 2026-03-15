---
layout:     post
title:      Fine-tuning之简单的数据准备与预处理
subtitle:   Fine-tuning
date:       2026-03-15
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- Fine-tuning

---

## 背景

> 私有LLM,最重要的是用私有数据对LLM进行微调(Fine-tuning).这篇文章先讲如何进行数据准备和预处理

## 数据准备

通过下载“斯坦福Alpaca训练所使用的数据集”得到如下格式的json文件:
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
## 数据预处理

Fine-tuning要求的数据格式(**基础的SFT数据结构**)如下:

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


通过一段python脚本将alpaca_data.json的数据格式转化为Fine-tuning要求的数据格式.
并且转化后的数据要有**训练数据集**和**验证数据集**2种. (训练:验证 = 7:3)

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

完整的代码链接如下:

[https://github.com/LuochuanAD/Fine-tuning-Learn](https://github.com/LuochuanAD/Fine-tuning-Learn)


## 参考

**数据来源:**

“斯坦福Alpaca训练所使用的数据集”:

[https://raw.githubusercontent.com/tatsu-lab/stanford_alpaca/refs/heads/main/alpaca_data.json](https://raw.githubusercontent.com/tatsu-lab/stanford_alpaca/refs/heads/main/alpaca_data.json) 
















