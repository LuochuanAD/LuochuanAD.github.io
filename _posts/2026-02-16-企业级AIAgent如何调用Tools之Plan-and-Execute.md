---
layout:     post
title:      企业级AIAgent如何调用Tools之Plan-and-Execute
subtitle:   Plan-and-Execute框架
date:       2026-02-16
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- Function Tools 
- AI Agent

---

## 背景

> 在完整的AI Agent中,工具的调用这里采用ChatGPT的Function Calling的思想,通过定义“**Tools Schema**”来连接LLM和Tools的调用.


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
## 2, 循环执行Execute (Plan-and-Execute框架)


**Planner Prompt:**
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



```
import json
 
# 1️⃣ 让 Planner 生成计划
planner_response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": planner_prompt},
        {"role": "user", "content": user_query}
    ]
)
 
plans = json.loads(planner_response.choices[0].message.content)["plans"]
 
# 2️⃣ 逐步执行
context_memory = {}
 
for step in plans:
    tool_name = step["tool"]
    args = step["arguments"]
 
    result = execute_tool(tool_name, args, context_memory)
 
    # 把结果加入上下文
    context_memory[f"step_{step['step_id']}"] = result

```

```
def execute_tool(tool_name, args, memory):
 
    if tool_name == "Search Database":
        return search_Database(args["name"])
 
    elif tool_name == "Send Mail":
        
        return send_Mail(args["to"],args["subject"],args["body"])
 
    elif tool_name == "Get Weather":
        return get_Weather(args["location"])
 
    else:
        raise Exception("Unknown tool")

```

## 评价: (Plan-and-Execute框架 + Function Calling 推荐指数:🌟🌟)
```
优点:

1, 任务分解: 通过将大任务分解为小任务,可以有效管理和解决复杂任务.
2, 详细指导: 通过提供详细的指导来改善推理步骤的质量和准确性.
3, 适应性: 可以根据不同类型的任务进行调整,在各种复杂问题中表现出色.

缺点:

1, Planner Prompt一次性生成规划,步骤可能会丢失,乱序,中间状态无法感知
2, 循环执行Execute 过于机械,不具备条件跳转,分支,动态决策,失败重试
3, Planner 不知道Execute的结果,是否为空等等

适合场景:

1, 步骤固定,流程稳定
2, 不需要动态决策,不需要分支逻辑

不适合场景:

1, 决策型任务
2, 条件跳转,
3, 数据驱动推理
4, 长链条任务
```
