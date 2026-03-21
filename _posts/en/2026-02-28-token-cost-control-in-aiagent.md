---
layout:     post
title: Token Cost Control in AIAgent
subtitle: Cost Management
date:       2026-02-28
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- AIAgent
- token

---

## Background

> When building AI products, as the number of users grows, every LLM call and the prompt tokens used incur costs. So how can we minimize unnecessary expenses without affecting the results?

## LLM Pricing Structure

> Example: OpenAI

![](https://raw.githubusercontent.com/LuochuanAD/BlogSourceImage/refs/heads/master/BlogSourceImage/BlogSourceImage2026/openAI_prices.png)

It can be observed that **the latest OpenAI API token pricing structure is:**

```
Input tokens: Priced at 10x Cached input tokens, and 1/10 of the Output tokens price.

Cached input tokens: The cheapest, priced at 1/10 of Input tokens and 1/100 of Output tokens.

Output tokens: The most expensive, priced at 10x Input tokens and 100x Cached input tokens.
```

For older LLM versions such as the gpt-4o series, the cached input token fees are waived.

## Recommended Strategies

### 1. Encourage users to input concise, high-certainty keywords

```
Note here we say "encourage users," not "restrict users."
For a complete enterprise AI Agent, you can’t limit the length of user input because the user's message must be fully preserved and sent to the LLM.

Encouraging concise inputs => effectively reduces the context length in tokens.

Encouraging high-certainty rather than vague language => improves model response accuracy and reduces hallucinations.

Encouraging keyword-based inputs => improves retrieval accuracy and further reduces context token length.
```

### 2. Limit LLM output length

```
Since output tokens are the most expensive, explicitly instruct the LLM in the prompt to produce fixed-length responses or use summarizing and concise phrasing.

=> This effectively reduces the number of output tokens.
```

### 3. Reduce LLM calls: Memory design is essential

```
While OpenAI automatically calculates cached input tokens to reduce costs per call, the uncertainty is high. Thus, saving conversation history is important.

Memory here involves short-term and long-term recall, which this article won’t cover. We focus only on memory design to reduce token usage.

For repeated user queries in a short timeframe, avoid calling the LLM each time. Search the conversation records instead => reduces LLM calls.

How to leverage cached input design to minimize token fees?

Cached input principle: Avoid changing anything unnecessary; ideally, don’t alter a single token.

1. Categorize memory conversation records; group similar queries together => improves retrieval accuracy and reduces LLM calls.

2. Summarize memory conversation records => facilitates their use as part of input for future conversations, shortening context length.
```

### 4. Use prompt compression algorithms

```
Structured prompts tend to be lengthy, which increases token costs.

A recommended approach is the prompt compression algorithm LLMLingua-2.

No detailed explanation here, but interested readers may explore it further.
```

### 5. Prompt writing: Make full use of cached input

**To illustrate, the prompt is written in Chinese here; preferably, write prompts in English.**

```
Every LLM call uses a prompt. The prompt content adjusts or supplements according to user input, RAG retrieval results, etc.

To fully leverage cached input and reduce costs, reorder the prompt so that unchanged content is cached.
```

**Example:**

```
prompt = “
	You are a business case analyst skilled in assessing risks based on financial statements.
	The following background info is known:
	{Background_msg}

	And the financial statements:
	{Financial_msg}
	
	According to the user question:
	{question}

	Provide the answer.
	Return format:
	return:
	{
		result:""
	}
”
At first glance, this prompt seems fine, but according to cached input rules, only the first two lines get cached.

Improved prompt reordered for caching:

cached_prompt = “
	You are a business case analyst skilled in assessing risks based on financial statements.
	Based on Background info and Financial statements, answer the user question.
	Return format:
	return:
	{
		result:""
	}

	Background: {Background_msg}
	Financials: {Financial_msg}
	User question: {question}
”
According to cached input rules, this improved cached_prompt caches the first seven lines, lowering costs to about 1/10 of input price.
```

### 6. Is calling the LLM always necessary?

**Example:**
```
When storing semi-structured or unstructured documents into a vector database, does calling an LLM to convert raw data into structured JSON improve accuracy?

Proposed alternatives:

1. Regular expression filtering,

2. Rule-based parsing,

3. XGBoost classification,

4. Keyword searching,

to convert semi/unstructured documents into structured data.
```

Refer to this article for more: "RAG Structural Chunks Design"

[https://strictfrog.com/2026/01/31/RAG%E4%B9%8BStructural-Chunks%E8%AE%BE%E8%AE%A1%E4%B8%8E%E6%80%9D%E8%80%83/](https://strictfrog.com/2026/01/31/RAG%E4%B9%8BStructural-Chunks%E8%AE%BE%E8%AE%A1%E4%B8%8E%E6%80%9D%E8%80%83/)

### 7. Can we switch LLM models without compromising output quality?

```
It’s recommended to switch between models within the GPT-5 series after extensive testing.

Generally, the latest model is the best, but different models may excel at different tasks. This requires AI engineers to validate and judge.
```

### 8. Is calling the LLM API always necessary?

```
For example: Calling the Whisper API for speech-to-text costs money.

Running Whisper locally is free.

TTS also has many free alternatives.
```

### 9. Restrict user input length: Max_tokens

```
For a specific AI Agent, decide:

1. Whether to set Max_tokens,

2. What the maximum Max_tokens should be.

My view:

For enterprise AI Agents used internally, do not set a limit as internal users typically won’t abuse tokens.

For external users, definitely set token limits and monitor usage.
```

### 10. Prefer writing prompts in English

```
On average,
One English word = 0.75 tokens,
One Chinese word = 1.0 token
```

### 11. manage the length of propmt

"Context Management and Filtering in Prompts"
[https://strictfrog.com/en/2026-03-20-context-management-and-filtering-in-prompts/](https://strictfrog.com/en/2026-03-20-context-management-and-filtering-in-prompts/)

## Summary

**A penny per call is small, but with 10,000 users calling 10 times, costs become significant.**