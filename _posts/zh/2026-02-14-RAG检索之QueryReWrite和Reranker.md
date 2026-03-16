---
layout:     post
title:      RAG检索之QueryReWrite和Reranker
subtitle:   Query ReWriting
date:       2026-02-14
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- RAG
- QueryReWriting
- Reranker

---

## 背景

> 在设计RAG系统的过程中,我通过设计了Structural Chunks和Structural Prompt后, 检索出来的结果是符合理想的准确性. 但我想更加提高准确率,需要Query Rewrite和Reranker双重检索.

## Query Rewrite

### 方案1: 同义词扩展

**例如:**

```
用户query: python

改写成: 
python语言
python开发
python版本

```
评价: 快,成本低, 覆盖有限

### 方案2: LLM Rewrite (推荐)

> 最常见的是利用LLM生成3个子问题, 将3次检索出来的结果合并在一起.

**注意:这里为了方便说明用中文写的prompt,最好用英文**

```
prompt = “将用户查询重写为多个搜索查询,返回3个不同的查询结果“
```
流程:
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
### 方案3: HyDE (慎重)

> 利用LLM本身的知识将用户query生成一个预期的答案, 再利用这个预期答案在向量数据库中进行语义搜索.

流程:

```
User Query
    ↓
LLM 生成预期答案
    ↓
Embedding
    ↓
Vector Search

```
**注意:因为LLM本身的知识和RAG的知识差异比较大,并且受限于LLM本身的知识有效期,所以会出现预期答案错误,过时等. 所以慎重要使用HyDE**

### 方案4: 多语言翻译 (推荐)

> RAG知识库是英文和日文,用户Query是中文怎么办? 

**例如:**

```
用户query: 输出日志的保留时间是多少?

先翻译为英文和日文:

What is the retention period for the output logs?

出力ログの保存期間はどれくらいですか?

然后将这3条query 向量数据库搜索.

```
流程:
```
User Query
    ↓
LLM 翻译
    ↓
Query 1 (中文)
Query 2 (英文)
Query 3 (日语)
    ↓
Vector Search
    ↓
Merge results

```
## Reranker重排序

- “Vector Search”向量检索只是初步检索, 保证召回(recall)多, TopK一般为20-50
- Reranker精炼排序, 保证精确度(precision)高, TopK一般为3-5

### 方案1:基于LLM的Reranker

> 使用LLM对初步检索出来的文档进行评分或选择

```
prompt = “
	Given the query and the following documents, rank with which document is most relevant.
	Query: {query}
	Documents:
	{chunk_1}
	{chunk_2}
	{chunk_3}
	...
	Return the IDs of the most relevant documents.

”
```
评价: 消耗tokens多,慢

### 方案2: 基于相似度微调模型 (推荐)

> 将初步检索出来的Chunk+用户Query =》直接输入模型 =》relevance score 相关性评分

```
英文:
ms-marco-MiniLM-L-6-v2
ms-marco-MiniLM-L-12-v2
bge-reranker-base

日语:
japanese-reranker-cross-encoder-xsmall-v1
japanese-reranker-cross-encoder-base-v1
japanese-reranker-cross-encoder-large-v1

中文:
bge-reranker-large
bge-reranker-base
m3e-reranker

跨语言Cross-Encoder模型:

BAAI/bge-reranker-v2-m3
Alibaba-NLP/gte-multilingual-reranker-base
cross-encoder/ms-marco-MiniLM-L6-v2
```
评价: 精度高