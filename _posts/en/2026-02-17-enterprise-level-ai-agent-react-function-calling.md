---
layout:     post
title: Enterprise-level AI Agent for ReAct + Function Calling
subtitle: ReAct Framework
date:       2026-02-17
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- FunctionTools 
- AIAgent
- ReAct

---

## Background

> In a complete AI Agent, tool invocation adopts the Function Calling concept from ChatGPT, connecting LLM and tool calls through defining a "**Tools Schema**."

## Architecture
```
User Query
    ↓
Agent (LLM with tool access)
    ↓
Decide:
    - Call Tool?
    - Or Final Answer?
    ↓
Tool Result
    ↓
Append to Context
    ↓
Repeat

```

## 1. Define the Tools Schema first

> The tools used here: Search Database, Send Mail, Get Weather, etc. These tools correspond to the names of custom tool methods.

```
tools = [

	{
		"type": "function",
		"function": {
			"name": "Search Database",
			"description": "Retrieve profile from database",
			"parameters":{
				"type": "object",
				"properties": {
					"name": {"type": "string"}
				},
				"required": ["name"]
			}
		}
	},
	{
		"type": "function",
		"function": {
			"name": "Send Mail",
			"description": "Send Mail to Gmail",
			"parameters":{
				"to": {"type": "string"},
				"subject": {"type": "string"},
				"body": {"type": "string"}
			}
		}
	},
	{
		"type": "function",
		"function": {
			"name": "Get Weather",
			"description": "Get Weather from location",
			"parameters":{
				"type": "object",
				"properties": {
					"location": {"type": "string"}
				},
				"required": ["location"]
			}
		}
	},
......

]

```

## 2. LLM decides the next action (ReAct framework)

**Agent Prompt:**
```
prompt = ‘
	You are an intelligent task agent.

	User query: {Query}
	At each step:
	- Decide whether to call a tool.
	- If needed, call exactly one tool.
	- If enough information is gathered, provide final answer.
	- Always reason step-by-step.
 
	Available tools:
	1. Search Database
	2. Send Mail
	3. Get Weather
’
```

```

messages = [{"role": "user", "content": user_query}]
 
while True:
 
    response = client.chat.completions.create(
        model="gpt-4o",
        messages=messages,
        tools=tools
    )
 
    message = response.choices[0].message
 
    if message.tool_calls:
        tool_call = message.tool_calls[0]
        result = execute_tool(tool_call)
 
        messages.append(message)
        messages.append({
            "role": "tool",
            "tool_call_id": tool_call.id,
            "content": json.dumps(result)
        })
 
    else:
        final_answer = message.content
        break

```

## Evaluation: (ReAct framework + Function Calling recommended rating: 🌟🌟🌟🌟)
```
Advantages:

1. Dynamic planning 
2. Automatic condition judgment
3. Supports branching logic
4. Self-correction
5. More aligned with human thinking: ReAct

```