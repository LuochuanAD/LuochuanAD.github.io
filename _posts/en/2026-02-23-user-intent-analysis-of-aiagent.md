---
layout:     post
title: User Intent Analysis of AIAgent
subtitle: Router
date:       2026-02-23
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags: 
- AI Agent
- Router

---

## Background
> User queries may involve simple information searches, tool invocations, casual chatting, and more. How can we perform intent analysis (or query classification) on user queries in the context of a specific AIAgent system?

## Scenario Overview of a Specific AIAgent

> The dataset of the RAG system in the specific AIAgent is as follows: (It is evident that this is a chunking of a resume)

```
resume_chunks = [
	basic_chunk,
	skills_chunk,
	introduction_chunk,
	certification_chunk,
	project_chunk_1,
	project_chunk_2,
	project_chunk_3,
	......
	
]
```

## Intent Analysis Using the Dataset of a Specific AIAgent

### Method 1: Keyword Extraction and Matching

In the following article, I have explained how to split datasets into specific chunks using a **keyword density algorithm**.
[https://strictfrog.com/2026/01/31/RAG%E4%B9%8BStructural-Chunks%E8%AE%BE%E8%AE%A1%E4%B8%8E%E6%80%9D%E8%80%83/](https://strictfrog.com/2026/01/31/RAG%E4%B9%8BStructural-Chunks%E8%AE%BE%E8%AE%A1%E4%B8%8E%E6%80%9D%E8%80%83/)

```
type_keywords = {
	"basic_chunk" = ["name", "first name", "age", "years old", "birthdate", "university", "college", "person"...],
	"skills_chunk" = ["python", "C++", "iOS", "Android", "Java", "PHP"...],
	"introduction_chunk" = ["eager to learn", "stress-resistant", "efficient", "agile development",...],
	"certification_chunk" = ["Computer Level 2", "CET-4", "CET-6", "IELTS", "AWS",...],
	"project_chunk" = ["project experience", "role", "duration",...]
}
```
**Example:**

```
UserQuery: I want to find a person named Louis with 5 years of Python development experience.

By iterating through type_keywords in Python, keywords like "name", "person", and "Python" were found in the User Query.

Intent analysis result:

The user intends to search for "basic_chunk" and "skills_chunk" in the specific AIAgent.

In plain terms: the user wants to look up personal information and skills of someone.
```

### Method 2: Intent Analysis Using LLM Prompting

Assume the **architecture of the specific AIAgent is as follows:**

```
Chat (casual chat)
|
FAQ (dataset lookup)
|-- basic
|-- skills
|-- introduction
|-- certification
|-- project
|-- other
Tools (tool invocation to execute tasks)
|- sendMails
|- getCurrentWeather
|- ......
```

**The architecture diagram is as follows:** (Intent analysis only uses the first layer: Intent Router)

For the complete architecture, you can refer to this article:
[https://strictfrog.com/2026/02/19/AIAgent%E4%B9%8B%E5%AE%8C%E6%95%B4%E7%9A%84%E7%B3%BB%E7%BB%9F%E7%B3%BB%E6%9E%B6%E6%9E%84/](https://strictfrog.com/2026/02/19/AIAgent%E4%B9%8B%E5%AE%8C%E6%95%B4%E7%9A%84%E7%B3%BB%E7%BB%9F%E7%B3%BB%E6%9E%B6%E6%9E%84/)

```
User Query
      ↓
1️⃣ Intent Router
      ↓
┌────────────┬────────────────────────────┬──────────────┐
│ Chat	     │		FAQ     	  │ Tool_Required│
│ (LLM only) │ (basic|skills|...|project) | (ReAct loop) │
└────────────┴────────────────────────────┴──────────────┴
                ↓
	     ......
```

**Router Prompt:**

From Method 1, the initial analysis result is:

**first_result** = "The user wants to search for 'basic_chunk' and 'skills_chunk' in the specific AIAgent."

functionCalling.json is the structured JSON file describing all tools.

```
prompt = ‘

	You are an intent analyst specialized in deep analysis of user queries.

	The preliminary analysis result is: {first_result}.

	The dataset features are known as: {The resume dataset is divided into the following categories: basic (personal info including name, age, birthdate, education, etc.), skills (skills such as Python, Java and other IT programming languages), introduction (self-introduction), certification (certificates including IT skill certifications and TOEFL, etc.), project (project experience)}.
	
	Known available Tools: {functionCalling.json}.

	User query: {query}.
 
	Analyze the "user query" into one or more of the following categories:
 
	1. Chat
	2. FAQ: basic
	3. FAQ: skills
	4. FAQ: introduction
	5. FAQ: certification
	6. FAQ: project
	7. tool_required
 
	Return JSON:
	{
  		"type": "..."
		......
	}
’
```

## Review of the "Router Prompt"

```
Integrating the specific AIAgent dataset and toolset:

1. Use keyword matching based on RAG’s chunking characteristics for preliminary analysis.

2. Pass dataset and tool features into the prompt to enable the LLM to perform more precise deep analysis.
```