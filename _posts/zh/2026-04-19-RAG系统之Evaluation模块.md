---
layout:     post
title:      RAG系统之Evaluation模块
subtitle:   Evaluation模块
date:       2026-04-19
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- RAG
- Evaluation


---

## 背景

> 在之前做过的几个AIAgent中,构建了RAG系统. 现在需要对这个RAG系统进行评估, 我会从以下几个维度来评估: 准确率评分(accuracy_score),精确率评分(precision_score),召回率评分(recall_score),平均覆盖率(avg_coverage),平均响应延迟的时间(avg_latency). 


## 构建数据源样例

**数据源结构:**

```
evalution_examples =
[
    {
		"query": "user_question",
		"context": [
			"RAG chunk1",
			"RAG chunk5",
			"Memory short 1",
			"Memory short 5",
			"Memory long 1",
			"Memory long 3",
			......
		],
		"generated_answer": "True answer or False answer",
		"reference_answer": "True answer or False answer"
	},
......
]
```

- query 用户输入
- context 上下文数据(RAG数据或者Memory数据)
- generated_answer LLM的回答
- reference_answer 实际的答案

## 评估的计算方法

|   | generated  | generated  |
|:----------|:----------|:----------|
| reference    | [1, 1]    | [1, 0]    |
| reference    | [0, 1]    | [0, 0]    |


- generated =》0: LLM的回答❌, 1: LLM的回答✅
- reference =》0: 实际的答案❌, 1: 实际的答案✅

**LLM的回答的对与错的标准:**
> 我根据数据源的结构, 通过”关键词重叠率“来计算 generated_answer 和 reference_answer的平均关键词重叠率

伪代码

```
		Count(Set(reference_keyWords)) & Count(Set(generated_keyWords))
avg_keyWords = ------------------------------------------------------------------
		   		Count(Set(reference_keyWords))

```

如果 avg_keyWords > 0.7 ,则认为LLM的回答✅.

**实际的答案的对与错的标准:**

在构建数据源时,通过人为的标记, 我就将数据源分为了实际答案✅ 和 实际答案❌的2种数据.


**accuracy_score（准确率）**

> 准确率: 预测正确的数据占总体数据的百分比.

伪代码

```
		         Count([1, 1]) + Count([0, 0])
accuracy_score = --------------------------------------------------------------
		   Count([1, 1]) + Count([1, 0]) + Count([0, 1]) + Count([1, 1])

```

- 如果LLM的回答和实际的答案都是对的,或者都是错的的. 代表着 这是一条预测正确的数据.
- 如果LLM的回答和实际的答案不一样. 说明出现了漏判和误判.




**precision_score (精确率)**

> 精确率: 在LLM回答是对的情况下, 有多少实际的答案也是对的.

伪代码

```
		         Count([1, 1])
precision_score = -------------------------------
		   Count([1, 1]) + Count([0, 1])

```
**recall_score (召回率)**

> 召回率: 在实际答案是对的情况下，有多少LLM的回答也是对的。

伪代码

```
		         Count([1, 1])
recall_score = -------------------------------
		   Count([1, 1]) + Count([1, 0])

```


**avg_coverage (平均覆盖率)**

> 我根据数据源的结构, 通过”关键词重叠率“来计算 context 和 generated_answer的平均关键词重叠率

伪代码

```
		     Count(Set(context_keyWords)) & Count(Set(generated_keyWords))
avg_coverage = Sum( ------------------------------------------------------------------ ) / Count(evalution_examples)
		   		Count(Set(generated_keyWords))

```

**avg_latency (平均响应时间)**

> 在RAG系统中, Total = T(embed) ​+ T(retrieve) ​+ T(rerank) ​+ T(llm)​. 但在这里我通过计算平均每条数据的响应时间来表示平均响应时间.


伪代码
```
		Sum(end_time - start_time)
avg_latency = ------------------------------
		Count(evalution_examples)
```




## F1 Score

> 在RAG评估中 accuracy_score往往不靠谱. precision_score 和 recall_score成反比. 为了平衡precision_score 和 recall_score, 这里使用了F1 Score.

伪代码
```
		2 * Count([1, 1])
f1_score = ---------------------------------------------------
	     2 * Count([1, 1]) + Count([0, 1]) + Count([1, 0])
```

## 未来

下一步,我会使用更高级的评估指标:

1. MRR（Mean Reciprocal Rank）
2. nDCG（排序质量）
3. Hit@K
4. BLEU / ROUGE（生成评估）
5. LLM-as-a-Judge（现在最主流）



















