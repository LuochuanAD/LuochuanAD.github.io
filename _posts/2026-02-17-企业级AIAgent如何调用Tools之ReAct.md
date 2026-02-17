---
layout:     post
title:      企业级AIAgent如何调用Tools之ReAct
subtitle:   ReAct框架
date:       2026-02-17
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- Function Tools 
- AI Agent
- ReAct

---

## 背景

> 在完整的AI Agent中,工具的调用这里采用ChatGPT的Function Calling的思想,通过定义“**Tools Schema**”来连接LLM和Tools的调用.

## 架构结构
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

## 1,先定义Tools Schema

> 这里用到的工具: Search Database, Send Mail, Get Weather等等. 这些工具也就是自定义工具方法的名字.


```
tools = [

	{
		"type": "function",
		"function": {
			"name": "Search Database",
			"description": "Retrive profile from database",
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


## 2, 由LLM决定下一步的行动 (ReAct框架)


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

## 评价: (ReAct框架 + Function Calling 推荐指数:🌟🌟🌟🌟)
```
优点:

1, 动态规划 
2, 自动条件判断
3, 支持分支逻辑
4, 自我修正
5, 更符合人类思维:ReAct

```
