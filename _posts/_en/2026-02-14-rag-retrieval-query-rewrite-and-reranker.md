---
layout:     post
title: RAG Retrieval: Query ReWrite and Reranker
subtitle: Query ReWriting
date:       2026-02-14
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- RAG
- QueryReWriting
- Reranker

---

## Background

> In designing the RAG system, after implementing Structural Chunks and Structural Prompt, the retrieval results achieved the desired accuracy. However, to further improve accuracy, dual retrieval with Query Rewrite and Reranker is necessary.

## Query Rewrite

### Approach 1: Synonym Expansion

**For example:**

```
User query: python

Rewrite as: 
python language
python development
python version

```
Evaluation: Fast, low cost, limited coverage

### Approach 2: LLM Rewrite (Recommended)

> The most common method is to use an LLM to generate 3 sub-queries, then merge the results retrieved from 3 searches.

**Note: For clarity, prompts are shown in Chinese here, but English is preferred.**

```
prompt = “Rewrite the user query into multiple search queries and return 3 different query results“
```
Process:
```
User Query
    ↓
LLM Rewrite
    ↓
Query 1
Query 2
Query 3
    ↓
Vector Search
    ↓
Merge results

```
### Approach 3: HyDE (Use With Caution)

> Use the LLM’s knowledge to generate an expected answer from the user query, then perform semantic search in the vector DB using that answer.

Process:

```
User Query
    ↓
LLM generates expected answer
    ↓
Embedding
    ↓
Vector Search

```
**Note: Because the knowledge in the LLM and the RAG knowledge base can differ significantly and LLM knowledge is limited by its update period, the expected answer may be incorrect or outdated. Use HyDE cautiously.**

### Approach 4: Multilingual Translation (Recommended)

> What if the RAG knowledge base is in English and Japanese, but the user query is in Chinese?

**For example:**

```
User query: 输出日志的保留时间是多少?

First translate to English and Japanese:

What is the retention period for the output logs?

出力ログの保存期間はどれくらいですか?

Then search the vector DB with all three queries.

```
Process:
```
User Query
    ↓
LLM Translation
    ↓
Query 1 (Chinese)
Query 2 (English)
Query 3 (Japanese)
    ↓
Vector Search
    ↓
Merge results

```
## Reranker Re-ranking

- “Vector Search” only performs preliminary retrieval to ensure high recall, typically with TopK of 20-50.
- Reranker refines ranking to ensure high precision, typically with TopK of 3-5.

### Approach 1: LLM-based Reranker

> Use an LLM to score or select the top documents from preliminary retrieval.

```
prompt = “
	Given the query and the following documents, rank the documents by relevance.
	Query: {query}
	Documents:
	{chunk_1}
	{chunk_2}
	{chunk_3}
	...
	Return the IDs of the most relevant documents.

”
```
Evaluation: High token consumption, slow

### Approach 2: Similarity Fine-tuned Model (Recommended)

> Feed the retrieved Chunk + user query directly into the model to get a relevance score.

```
English:
ms-marco-MiniLM-L-6-v2
ms-marco-MiniLM-L-12-v2
bge-reranker-base

Japanese:
japanese-reranker-cross-encoder-xsmall-v1
japanese-reranker-cross-encoder-base-v1
japanese-reranker-cross-encoder-large-v1

Chinese:
bge-reranker-large
bge-reranker-base
m3e-reranker

Cross-lingual Cross-Encoder models:

BAAI/bge-reranker-v2-m3
Alibaba-NLP/gte-multilingual-reranker-base
cross-encoder/ms-marco-MiniLM-L6-v2
```
Evaluation: High accuracy