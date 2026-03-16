---
layout:     post
title: Building and Reflections on a Financial Statement Analysis AI Agent: ReAct RAG Agent
subtitle: Architecture
date:       2026-02-21
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- Function Tools 
- AI Agent
- ReAct

---

## Background
> Without complex RAG design, simply using LlamaIndex’s ReAct RAG Agent framework to analyze financial statements can achieve a level comparable to a junior business analyst. However, there are many limitations. Personally, I think it is only suitable for micro and small enterprises.

## Financial Report Analysis Architecture

![](https://raw.githubusercontent.com/LuochuanAD/BlogSourceImage/refs/heads/master/BlogSourceImage/BlogSourceImage2026/%E8%B4%A2%E6%8A%A5%E5%88%86%E6%9E%90%E8%BF%87%E7%A8%8B%E5%9B%BE.png)

## 1. Load Financial Report Files for Companies A, B, ...

```python
from llama_index.core import SimpleDirectoryReader

A_doc = SimpleDirectoryReader(
	imput_file = ["./A公司财务报表文件_2025.pdf"]
).load_data()

......

```

## 2. Build Vector Data Based on Financial Report and Save Locally

```python
from llama_index.core import VectorStoreIndex

A_index = VectorStoreIndex.frome_documents(A_doc)

......

frome llama_index.core import StorageContext

A_index.storage_context.persist(persist_dir="./storage/A")

......

```
**Local Vector Data Directory Structure:**

```
A
|- default_vectory_store.json
|- docstore.json
|- graph_store.json
|- image_vectore_store.json
|- index_store.json
B
|- default_vectory_store.json
|- docstore.json
|- graph_store.json
|- image_vectore_store.json
|- index_store.json

......

```

## 3. Configure Query Tools: (A/B/...)

```python
from llama_index.core.tools import QueryEngineerTool, ToolMetadata

query_engineer_tools= [
	QueryEngineerTool(
		query_engineer = A_engineer,
		metadata = ToolMetadata(
			name = "A_finance",
			description= ("Used to provide financial report information for company A"),
		),
	),

	......

]
```

## 4. Create LLM: ReAct RAG Agent

```python
from llama_index.core.agent import ReActAgent

agent = ReActAgent.from_tools(query_tools, llm=llm, verbose=True)

```

## Reflection: (ReAct RAG Agent)

```
Advantages:
1. Only need to provide the company’s financial report files.
2. Simple and fast, leveraging existing LlamaIndex to quickly build a ReAct RAG Agent.
3. The analysis results reach the level of a junior business analyst.

Disadvantages:
1. No detailed RAG process design; performance depends solely on LlamaIndex capabilities.
2. No design for the ReAct process; again, dependent on LlamaIndex itself.
3. No fine-tuning of the large model, so analysis results are limited to junior business analyst level.
4. No human-in-the-loop thinking added, which limits maximizing cognitive synergy: AI + Human.

```
**If the above four drawbacks are addressed, the system could reach the level of a top-tier business analyst. Based on this, I am very optimistic about the future development and application of AI.  
Because developing such a system would take at most six months and could embody the expertise of a top business analyst with 20 years of experience. This is humanity’s greatest leverage.**

## References

Example article source: *Hands-on AI Agent* by Huang Jia