---
layout:     post
title: Design Philosophy of Workflow+limitedPlanning
subtitle: Workflow + Limited Planning
date:       2026-03-07
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- workflows
- structured


---

## Background

> In practical AI Agent systems, many teams adopt a hybrid architecture:
Workflow + Limited Planning, where an LLM performs lightweight planning followed by executing a predefined workflow.

```
Many real-world AI products use this architecture.
Most systems essentially implement Workflow + Limited Planning.

Examples:

LangGraph (state machine workflow)
Notion AI
Perplexity AI

These systems:

are not fully autonomous agents,
but rather:
structured workflows.
```

## Workflow + Limited Planning Design Concept

```
User Input
      ↓
Intent Detection
      ↓
Task Planner
      ↓
Workflow Selection
      ↓
Workflow Execution
      ↓
Result
```

## Detailed Process Flow (Complete Example)

Assume the user input:

> Help me write an AI industry report

System process:

**Step 1 Intent Detection**

The LLM first determines the user's intent:

> intent = research_report

For example, output:

```
{
 "intent": "research_report"
}
```

**Step 2 Planner (Lightweight Planning)**

Planner decides:

> which workflow is needed

For example:

> research_workflow

**Step 3 Workflow Selection**

The system selects a predefined workflow:

> Research Workflow

Structure:

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

Each step calls the LLM or tools:

```
Search → Web API
Extract → LLM
Summarize → LLM
Write → LLM
```

## Common Implementation Patterns of Limited Planning

Different systems implement this differently, but typically there are 3 patterns.

**Pattern 1 Router Agent (Most Common)**

The planner is only responsible for:

> selecting which Agent to engage

Structure:
```
User Input
      ↓
Router
  ├ Research Agent
  ├ Coding Agent
  └ Writing Agent
```

Example:

User input:

> Write a Python scraper

Router:

> coding_agent

**Pattern 2 Tool Selection**

Planner only chooses tools:

```
Search
Database
Code Interpreter
```

Example:

> User: Search AI companies’ funding

Planner:

> use web_search

**Pattern 3 Workflow Routing**

Planner selects a workflow:

```
content_workflow
analysis_workflow
coding_workflow
```

Then executes the fixed flow.

## Implementation at Code Level (Simplified)

A typical Python structure:

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

Workflow:
```
def research_workflow(query):

    data = search(query)

    info = extract(data)

    summary = summarize(info)

    report = write_report(summary)

    return report
```
Here:

> planning = router

not a full agent.

## References

ChatGPT