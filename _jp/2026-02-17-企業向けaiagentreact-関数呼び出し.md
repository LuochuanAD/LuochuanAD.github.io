---
layout:     post
title: 企業向けAIAgent：ReAct + 関数呼び出し
subtitle: ReActフレームワーク
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

> 完全なAIエージェントでは、ツールの呼び出しにChatGPTのFunction Callingのコンセプトを採用し、「**Tools Schema**」を定義してLLMとツールの呼び出しを連携させています。

## アーキテクチャ構成
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

## 1. まずTools Schemaを定義する

> ここで使うツールは、Search Database、Send Mail、Get Weatherなどです。これらのツールはカスタムツールメソッドの名前に該当します。


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


## 2. LLMが次のアクションを決定する (ReActフレームワーク)


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

## 評価: (ReActフレームワーク + Function Calling 推奨度:🌟🌟🌟🌟)
```
メリット:

1. 動的プランニング
2. 自動条件判断
3. ブランチロジック対応
4. 自己修正機能
5. 人間の思考に近い：ReAct
```