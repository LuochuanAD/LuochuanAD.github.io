---
layout:     post
title:      ChatGPT之AIAgent系统架构
subtitle:   Architecture
date:       2026-03-10
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- AIAgent
- Architecture
---

## 背景

> 上一篇文章说了ChatGPT的界限, 为了弥补ChatGPT做不到的事情,才有了这篇文章

## 完整 AI Agent 系统架构

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

## 模块1：Agent Controller（系统大脑）

作用：

> 控制整个 AI 任务流程

职责：

1.  接收用户请求
1.  调用 Planner
1.  执行 Tool
1.  更新 Memory
1.  返回结果

**流程：**

```
User Query
   │
   ▼
Controller
   │
   ▼
Plan → Execute → Observe → Loop

```
这其实是经典 Agent Loop：
```
while not done:
    think
    act
    observe
```
这种思想来自：

ReAct Agent

## 模块2：Planner（任务规划器）

Planner 的作用：

> 把复杂任务拆分成多个步骤

例如用户问：

> 帮我分析最近5个AI公司融资情况并写报告

Planner生成：

Plan:
1.  搜索AI融资新闻
1.  提取公司名称
1.  查询融资金额
1.  汇总数据
1.  生成报告

输出：

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
Planner 可以用：

> LLM

或者：

> State Machine

## 模块3：Tool Router（工具路由）

作用：

> 决定调用哪个工具

例如：

用户问题：东京天气

Router：

> weather_api


用户问题：查用户订单

Router：

> database_query

Router策略：

方法1
```
LLM Router

LLM判断tool
```
方法2
```
Embedding Router

query embedding
→ tool embedding
→ 相似度
```
方法3
```
规则 Router

if "weather"
→ weather tool
```

**生产系统通常**：

> LLM + rule

## 模块4：Tool System（工具系统）

> 工具是 AI 系统最重要的能力。

工具分类：

**1 API工具**

例如：
```
weather API
stock API
crypto API
```
**2 数据库工具**

例如：
```
SQL query
Vector search
```
**3 系统工具**

例如：
```
send email
create calendar
file read
```
**4 计算工具**

例如：
```
Python
code interpreter
```
典型 Tool schema：

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
## 模块5：Skills（技能系统）

Skills 是：

> 工具组合

例如：

Skill：

> Research Report

内部流程：
```
search
extract
analyze
summarize
```

Skill其实是：

> mini workflow

例如：

> skill_generate_report()

Skill优点：

1. 提高复用
1. 降低Agent复杂度

## 模块6：Memory（记忆系统）

> AI系统必须有长期记忆。

Memory分三种：

**1 Short Term Memory**

当前对话。

例如：

> conversation history

**2 Long Term Memory**

> 用户信息：用户偏好, 用户背景, 历史行为

存储：

1. Redis
1. Database

**3 Semantic Memory**

知识记忆：

> embedding, vector db

例如：

```
文档

笔记

知识库
```
常用：

```
Pinecone

Qdrant

Weaviate
```

## 模块7：RAG（知识检索）

优势: 减少 hallucination

> 详细请看我之前的文章

## 模块8：Workflow Engine

复杂任务需要：

> 工作流引擎

例如：

```
Step1
Step2
Step3
```
支持：

1. retry
1. timeout
1. branch
1. parallel

开源方案：

```
Temporal

Apache Airflow

Prefect
```
## 模块9：MCP（工具标准化）

未来趋势：

> Model Context Protocol

作用：

> 统一工具接口

例如：
```
filesystem
browser
database
```
所有工具都用 MCP 暴露。

Agent调用：

> MCP Client
