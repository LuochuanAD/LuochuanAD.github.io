---
layout:     post
title: Enterprise-level AI Agent: Plan-and-Execute + Function Calling
subtitle: Plan-and-Execute Framework
date:       2026-02-16
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- Function Tools 
- AI Agent

---

## Background

> In a complete AI Agent, the invocation of tools here follows the concept of ChatGPT's Function Calling, connecting the LLM with tool calls by defining a "**Tools Schema**."

## 1. Define the Tools Schema First

> The tools used here include: Search Database, Send Mail, Get Weather, etc. These tools correspond to the names of custom tool methods.

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

## 2. Loop Execution of Execute (Plan-and-Execute Framework)

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
 
# 1️⃣ Let the Planner generate the plan
planner_response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": planner_prompt},
        {"role": "user", "content": user_query}
    ]
)
 
plans = json.loads(planner_response.choices[0].message.content)["plans"]
 
# 2️⃣ Execute step-by-step
context_memory = {}
 
for step in plans:
    tool_name = step["tool"]
    args = step["arguments"]
 
    result = execute_tool(tool_name, args, context_memory)
 
    # Add the result to context
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

## Evaluation: (Plan-and-Execute Framework + Function Calling Recommendation: 🌟🌟)

```
Advantages:

1. Task Decomposition: Breaking down large tasks into smaller ones enables effective management and resolution of complex tasks.
2. Detailed Guidance: Providing detailed instructions improves the quality and accuracy of reasoning steps.
3. Adaptability: Can be adjusted for different types of tasks and performs well on various complex problems.

Disadvantages:

1. The Planner Prompt generates the plan all at once; steps may be missed, unordered, and intermediate states are not perceivable.
2. The Execute loop is too mechanical, lacking conditional jumps, branching, dynamic decisions, and failure retries.
3. The Planner does not know the execution results, such as whether they are empty or not.

Suitable Scenarios:

1. Fixed steps and stable workflows
2. No need for dynamic decision-making or branching logic

Unsuitable Scenarios:

1. Decision-driven tasks
2. Conditional jumps
3. Data-driven reasoning
4. Long-chain tasks
```