---
layout:     post
title:      企业级AIAgent:Router + ReAct + Function Calling
subtitle:   Architecture
date:       2026-02-18
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- Function Tools 
- AI Agent
- ReAct

---

## 背景
> 用户的Query既有简单的信息搜索又有工具调用,那么就需要设计一个通用可复用的成熟的系统架构


## 架构结构 Architecture
```
User Query
      ↓
1️⃣ Intent Router
      ↓
┌───────────────┬──────────────────┐
│ Simple QA     │ Tool Required     │
│ (LLM only)    │ (ReAct loop)      │
└───────────────┴──────────────────┘
                      ↓
            2️⃣ ReAct Tool Loop
                      ↓
            3️⃣ Structured Tool Layer
                      ↓
            4️⃣ Guardrails & Policies
                      ↓
                 Final Output
```

## 1, 第一层:Intent Router

> Router 需要判断是否调用Tools

**Router Prompt:**

```
prompt = ‘

	You are a task classifier.
	User query: {query}
 
	Classify the user query into one of:
 
	1. simple_qa
	2. retrieval
	3. tool_call
	4. multi_step
 
	Return JSON:
	{
  		"type": "..."
	}
’
```
## 2, 第二层:ReAct Tool Core

```
1. simple_qa ==》LLM直接回答

2. retrieval ==》调用向量数据库直接检索/调用Browser检索

3. tool_call  ==》进入ReAct循环
4. multi_step ==》进入ReAct循环

```
### ReAct循环的架构:

```
LLM decides next action
↓
Call one tool
↓
Append tool result
↓
Repeat

```
**Sample Code:**

```
while True:
 
    response = llm(messages, tools=tool_schema)
 
    if response.tool_calls:
        result = execute_tool(...)
        messages.append(tool_result)
 
    else:
        break

```

## 3, 第三层:Structured Tool Layer

> 这里采用ChatGPT的Function Calling的思想,通过定义“**Tools Schema**”来连接LLM和Tools的调用.

企业级AIAgent:ReAct + Function calling:
[https://strictfrog.com/2026/02/17/%E4%BC%81%E4%B8%9A%E7%BA%A7AIAgent%E5%A6%82%E4%BD%95%E8%B0%83%E7%94%A8Tools%E4%B9%8BReAct/](https://strictfrog.com/2026/02/17/%E4%BC%81%E4%B8%9A%E7%BA%A7AIAgent%E5%A6%82%E4%BD%95%E8%B0%83%E7%94%A8Tools%E4%B9%8BReAct/)

## 4, 第四层:Guardrails & Policies

**最大步骤限制:**

```
max_steps = 15
```
**不允许发送敏感邮件:**
```
if tool = send_mail:
	require confirmation
```
## 评价: (ReAct框架 + Function Calling 推荐指数:🌟🌟🌟🌟🌟)
```
优点:

1, 通用性    极高
2, 可控性    高
3, 扩展性    极高
4, 复杂任务	极高
5, 简单任务效率 极高

```


