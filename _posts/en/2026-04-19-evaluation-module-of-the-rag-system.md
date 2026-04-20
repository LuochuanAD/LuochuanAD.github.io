---
layout:     post
title: Evaluation Module of the RAG System and Memory Layer
subtitle: Evaluation Module
date:       2026-04-19
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- RAG
- Memory
- Evaluation


---

## Background

> In several previous AI Agent projects, I built RAG systems and Memory Layer. Now, I need to evaluate this RAG system and Memory Layer from the following dimensions: accuracy_score, precision_score, recall_score, average coverage (avg_coverage), and average response latency (avg_latency).

## Sample Data Source Construction

**Data Source Structure:**

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

- `query`: user input  
- `context`: contextual data (RAG data or Memory data)  
- `generated_answer`: LLM's response  
- `reference_answer`: the ground truth answer  

## Evaluation Metrics Calculation

|   | generated  | generated  |
|:----------|:----------|:----------|
| reference    | [1, 1]    | [1, 0]    |
| reference    | [0, 1]    | [0, 0]    |


- generated => 0: LLM's answer is incorrect, 1: LLM's answer is correct  
- reference => 0: ground truth is incorrect, 1: ground truth is correct  

**Criteria for LLM's answer correctness:**  
> Based on the data source structure, I calculate the average keyword overlap rate between `generated_answer` and `reference_answer`.

Pseudocode:

```
		Count(Set(reference_keyWords)) & Count(Set(generated_keyWords))
avg_keyWords = ------------------------------------------------------------------
		   		Count(Set(reference_keyWords))

```

If `avg_keyWords > 0.7`, the LLM's answer is considered correct.

**Criteria for ground truth correctness:**  

During data source construction, manual labeling was done to categorize data into ground truth correct and incorrect classes.

**accuracy_score**

> Accuracy: The percentage of correctly predicted samples over the total dataset.

Pseudocode:

```
		         Count([1, 1]) + Count([0, 0])
accuracy_score = --------------------------------------------------------------
		   Count([1, 1]) + Count([1, 0]) + Count([0, 1]) + Count([0, 0])

```

- If both LLM’s answer and the ground truth are correct or both are incorrect, it is counted as a correct prediction.  
- If LLM’s answer and the ground truth differ, it signals either a false negative or false positive.

**precision_score**

> Precision: Of all cases where the LLM's answer is correct, how many are actually correct in the ground truth.

Pseudocode:

```
		         Count([1, 1])
precision_score = -------------------------------
		   Count([1, 1]) + Count([0, 1])

```

**recall_score**

> Recall: Of all actually correct cases, how many did the LLM answer correctly.

Pseudocode:

```
		         Count([1, 1])
recall_score = -------------------------------
		   Count([1, 1]) + Count([1, 0])

```

**avg_coverage**

> Based on the data source structure, I calculate the average keyword overlap rate between `context` and `generated_answer`.

Pseudocode:

```
		     Count(Set(context_keyWords)) & Count(Set(generated_keyWords))
avg_coverage = Sum( ------------------------------------------------------------------ ) / Count(evalution_examples)
		   		Count(Set(generated_keyWords))

```

**avg_latency**

> In a RAG system, Total = T(embed) + T(retrieve) + T(rerank) + T(llm). Here, I measure average response time by averaging the latency per data item.

Pseudocode:

```
		Sum(end_time - start_time)
avg_latency = ------------------------------
		Count(evalution_examples)
```


## F1 Score

> In RAG evaluation, accuracy_score is often unreliable. Precision and recall usually trade off against each other. To balance precision and recall, F1 Score is used here.

Pseudocode:

```
		2 * Count([1, 1])
f1_score = ---------------------------------------------------
	     2 * Count([1, 1]) + Count([0, 1]) + Count([1, 0])
```

## Code URL

[https://github.com/LuochuanAD/Evaluation_RAG_Memory](https://github.com/LuochuanAD/Evaluation_RAG_Memory)

## Future Directions

Next, I will use more advanced evaluation metrics:

1. MRR (Mean Reciprocal Rank)  
2. nDCG (Ranking Quality)  
3. Hit@K  
4. BLEU / ROUGE (Generation Evaluation)  
5. LLM-as-a-Judge (Currently the mainstream approach)