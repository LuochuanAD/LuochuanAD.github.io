---
layout:     post
title: AIAgent System Architecture of ChatGPT
subtitle: Architecture
date:       2026-03-10
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- AIAgent
- Architecture
---

## Background

> The previous article discussed the limitations of ChatGPT. This article aims to address the tasks ChatGPT cannot accomplish.

## Complete AI Agent System Architecture

```
                User
                 │
                 ▼
        ┌─────────────────┐
        │   API Gateway   │
        └─────────────────┘
                 │
                 ▼
        ┌─────────────────┐
        │ Agent Controller│
        └─────────────────┘
                 │
   ┌─────────────┼─────────────┐
   ▼             ▼             ▼
*Planner     *Tool Router    *Memory
   │             │             │
   ▼             ▼             ▼
*Workflow    Tool System     Storage
Engine           │               │
   │             │               │
   ▼             ▼               ▼
 *Skills      MCP Tools      Vector DB
                 │
                 ▼
            External APIs
            
```

## Module 1: Agent Controller (System Brain)

Purpose:

> Controls the entire AI task workflow

Responsibilities:

1. Receive user requests  
2. Call the Planner  
3. Execute Tools  
4. Update Memory  
5. Return results

**Process:**

```
User Query
   │
   ▼
Controller
   │
   ▼
Plan → Execute → Observe → Loop

```
This is essentially the classic Agent Loop:
```
while not done:
    think
    act
    observe
```
This concept is inspired by:

ReAct Agent

## Module 2: Planner (Task Planner)

Role of Planner:

> Breaks down complex tasks into multiple steps

For example, user query:

> Help me analyze the recent funding rounds of 5 AI companies and write a report

Planner generates:

Plan:
1. Search AI funding news  
2. Extract company names  
3. Query funding amounts  
4. Aggregate data  
5. Generate report

Output:

```
{
 "steps":[
  "search_news",
  "extract_companies",
  "get_funding_data",
  "generate_report"
 ]
}
```
Planner can use:

> LLM

or:

> State Machine

## Module 3: Tool Router

Function:

> Decides which tool to invoke

Example:

User question: Tokyo weather

Router returns:

> weather_api

User question: Check user orders

Router returns:

> database_query

Router strategies:

Method 1
```
LLM Router

LLM determines tool
```
Method 2
```
Embedding Router

query embedding
→ tool embedding
→ similarity
```
Method 3
```
Rule-based Router

if "weather"
→ weather tool
```

**Production systems typically**:

> use LLM + rules

## Module 4: Tool System

> Tools are the most critical capability of AI systems.

Tool categories:

**1 API Tools**

For example:
```
weather API
stock API
crypto API
```
**2 Database Tools**

For example:
```
SQL query
Vector search
```
**3 System Tools**

For example:
```
send email
create calendar
file read
```
**4 Computing Tools**

For example:
```
Python
code interpreter
```
Typical Tool schema:

```
{
"name": "search_news",
"description": "search latest news",
"parameters": {
 "type":"object",
 "properties":{
   "query":{"type":"string"}
 }
}
}
```
## Module 5: Skills System

Skills are:

> Tool compositions

For example:

Skill:

> Research Report

Internal process:
```
search
extract
analyze
summarize
```

Skill is essentially:

> a mini workflow

For example:

> skill_generate_report()

Skills advantages:

1. Improves reusability  
2. Reduces Agent complexity

## Module 6: Memory System

> AI systems must have long-term memory.

Memory types:

**1 Short Term Memory**

Current conversation.

For example:

> conversation history

**2 Long Term Memory**

> User info: preferences, profile, behavior history

Storage:

1. Redis  
2. Database

**3 Semantic Memory**

Knowledge memory:

> embeddings, vector DB

For example:

```
Documents

Notes

Knowledge base
```
Common tools:

```
Pinecone

Qdrant

Weaviate
```

## Module 7: RAG (Retrieval-Augmented Generation)

Benefits: Reduces hallucinations

> See my previous article for details

## Module 8: Workflow Engine

Complex tasks require:

> a workflow engine

For example:

```
Step1
Step2
Step3
```
Supports:

1. retry  
2. timeout  
3. branching  
4. parallelism

Open-source options:

```
Temporal

Apache Airflow

Prefect
```
## Module 9: MCP (Tool Standardization)

Future trend:

> Model Context Protocol

Purpose:

> Unify tool interfaces

For example:
```
filesystem
browser
database
```
All tools expose interfaces via MCP.

Agent calls:

> MCP Client