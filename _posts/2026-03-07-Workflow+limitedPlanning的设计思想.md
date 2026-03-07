---
layout:     post
title:      Workflow+limitedPlanning的设计思想
subtitle:   Workflow+limitedPlanning
date:       2026-03-07
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- workflows
- structured


---

## 背景

> 在实际 AI Agent 系统中，很多团队采用一种折中架构：
Workflow + Limited Planning，即用 LLM 做轻量规划，再执行预定义 workflow。

```
真实世界很多 AI 产品都用这种架构
很多系统其实就是 Workflow + Limited Planning。

例如：

LangGraph（状态机 workflow）
Notion AI
Perplexity AI

这些系统：

不是完全 autonomous agent
而是：
structured workflows
```

## Workflow + Limited Planning 的设计思想

```
User Input
      ↓
Intent Detection  用户意图分析
      ↓
Task Planner
      ↓
Workflow Selection
      ↓
Workflow Execution
      ↓
Result
```

## 具体运行流程（完整例子）

假设用户输入：

> 帮我写一份AI行业报告

系统流程：

**Step 1 Intent Detection**

LLM 先判断用户意图：

> intent = research_report


例如输出：

```
{
 "intent": "research_report"
}
```

**Step 2 Planner（轻量规划）**

Planner 判断：

> 需要哪种 workflow


例如：

> research_workflow

**Step 3 Workflow Selection**

系统选择预定义流程：

> Research Workflow

结构：

```
Search Data
↓
Extract Key Info
↓
Summarize
↓
Write Report
```

**Step 4 Workflow Execution**

每一步调用 LLM 或工具：

```
Search → Web API
Extract → LLM
Summarize → LLM
Write → LLM
```
## Limited Planning 的几种实现方式

不同系统实现不同，但通常有 3 种模式。

**模式1 Router Agent（最常见）**

Planner 只负责：

> 选择哪个 Agent

结构：
```
User Input
      ↓
Router
  ├ Research Agent
  ├ Coding Agent
  └ Writing Agent
```
例如：

用户输入：

> 写 Python 爬虫

Router：

> coding_agent


**模式2 Tool Selection**

Planner 只选择工具：

```
Search
Database
Code Interpreter
```

例如：

> User: 查AI公司融资

Planner：

> use web_search

**模式3 Workflow Routing**

Planner 选择 workflow：

```
content_workflow
analysis_workflow
coding_workflow
```

然后执行固定流程。

## 代码层面的实现（简化版）

一个典型 Python 结构：

```
def router(user_input):

    intent = llm_intent_detection(user_input)

    if intent == "research":
        return research_workflow(user_input)

    elif intent == "coding":
        return coding_workflow(user_input)

    elif intent == "summary":
        return summary_workflow(user_input)
```

workflow：
```
def research_workflow(query):

    data = search(query)

    info = extract(data)

    summary = summarize(info)

    report = write_report(summary)

    return report
```
这里：

> planning = router

而不是完整 Agent。

## 参考

ChatGPT





