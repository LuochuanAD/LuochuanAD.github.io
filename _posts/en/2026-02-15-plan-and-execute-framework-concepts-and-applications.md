---
layout:     post
title: Plan-and-Execute Framework for Concepts and Applications
subtitle: Plan-and-Solve
date:       2026-02-15
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- Prompt 
- AI

---

## Background

> LangChain's Plan-and-Execute framework is inspired by the Plan-and-Solve paper. Plan-and-Execute is well-suited for more complex long-term planning by breaking down complicated problems into smaller sub-tasks and tackling them one by one. Although this involves frequent calls to large models, it avoids the issue of overly long prompts during the ReAct Agent loop.

## Plan-and-Solve Strategy

![](https://raw.githubusercontent.com/LuochuanAD/BlogSourceImage/refs/heads/master/BlogSourceImage/BlogSourceImage2026/COT%E5%92%8CPlan-and-Solve%E5%AF%B9%E6%AF%94.png)

The above diagram compares zero-shot COT and Plan-and-Solve. Essentially, Plan-and-Solve concretizes COT by breaking it into small tasks and then solving these tasks one by one. The Plan-and-Execute framework shares the same philosophy as Plan-and-Solve, but in practice, Plan-and-Execute is more focused on AI practical implementation.

## Example Application of Plan-and-Execute Framework

> Suitable for Question Rewriting design, multi-agent coordination work, and more.

Scenario: Within a complete AI agent system built on a RAG system.

**For example:**

**Question**: Find personal information about Louis and Liu Yifei, then generate a wedding invitation based on their information and send the invitation content to the email worldXXTest@gmail.com.

Plan-and-Execute decomposes the task into: Plan, Execute

Use LLM to break down the question into about 5 **Plans**:
```
|-1- Search the database for personal information about Louis
|-2- Search the web for personal information about Liu Yifei
|-3- Search for wedding invitation templates online
|-4- Generate the wedding invitation content for Louis and Liu Yifei
|-5- Send the invitation content to the email worldXXTest@gmail.com
```
Use the ReAct framework's LLM to handle these 5 plans one by one in the **Execute** phase:
```
|-1- "Louis’s personal info: iOS and AI Engineer, male, good temperament, gentle, kind, humorous..."
|-2- Use the Browser tool, obtain: “Liu Yifei’s personal info: Qianxi, female, film and TV actress, both beauty and talent, likes freedom...”
|-3- Wedding invitation template: “We are getting married. Dear Mr./Ms. xxx, you are welcome to join us on xx/xx/2026 at...”
|-4- Wedding invitation content for Louis and Liu Yifei: “We are getting married. Dear Mr./Ms. xxx, you are welcome to attend our grand wedding on February 31, 2026, at the North Pole Palace on the Moon. We will dispatch the interstellar ship LL001 to pick you up. Please board the ship at midnight on February 30 at the top floor of the Oriental Pearl Tower in Shanghai.”
|-5- Call tool sendMail to send the wedding invitation content
```
**Summary: Call LLM once to generate 5 plans, then call the ReAct Agent 5 times to handle each plan.**

#### Planner Prompt:
```
prompt = ‘
	You are a task planner in a multi-step AI system.
	User query: {Query}
	Your job:
	1, Break down the user query into executable steps.
	2, Each step must be atomic and tool-executable.
	3, Steps should be ordered logically.
	4, Do NOT execute anything.
	5, Only return JSON.
’
```

**For example:**

Query: I want to find someone who holds an English CET-6 certificate and has 3 years of Python development experience. Then send their personal information to luochuanad@gmail.com

The LLM(**Planner Prompt**) outputs:
```
json
[
 {
	"step": 1,
	"action": "Search Database",
	"description": "Query the database to find individuals with an English CET-6 certificate 					and 3 years of Python development experience.",
	"parameters": {
		"certificate": "CET-6",
		"experience": {
			"Language": "Python",
			"years": 3
		}
	}
 },
 {
	"step": 2,
	"action": "Retrieve Personal Information",
	"description": "Extract the personal information of the individual(s) found in the 					previous step.",
	"parameters": {
		"fields": ["name", "email", "phone", "address"]
	}
 },
 {
	"step": 3,
	"action": "Format Email",
	"description": "Prepare an email containing the personal information of the individual(s) 					found.",
	"parameters": {
		"recipient": "luochuanad@gmail.com",
		"subject": "Candidate Information",
		"body": "Include the personal information retrieved in step 2."
 	}
 },
 {
 	"step": 4,
	"action": "Send Email",
	"description": "Send the formatted email to the specified email address.",
	"parameters": {
		"recipient": "xxx@gmail.com",
		"content": "Use the email content prepared in step 3."
 	}
 }
]
```

## References

Lei Wang et al.'s paper “Plan-and-Solve Prompting: Improving Zero-Shot Chain-of-Thought Reasoning by Large Language Models” [https://aclanthology.org/2023.acl-long.147.pdf](https://aclanthology.org/2023.acl-long.147.pdf)