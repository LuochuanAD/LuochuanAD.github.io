---
layout:     post
title: The Design Philosophy of Self-Improving
subtitle: Self-Improving
date:       2026-03-13
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- SelfImproving
- AIAgent


---

## Background

> This article explains the design concept of the Self-Improving Agent, which adds self-improvement capabilities on top of the Autonomous Agent architecture.

"Autonomous Agent architecture":
[https://strictfrog.com/2026/03/07/AutoGPT%E5%88%86%E6%9E%90%E4%B8%8EAutonomous%E6%80%9D%E8%80%83/](https://strictfrog.com/2026/03/07/AutoGPT%E5%88%86%E6%9E%90%E4%B8%8EAutonomous%E6%80%9D%E8%80%83/)


## Self-Improving Agent Design Concept

**Overall Architecture:**

```
User Task
   ↓
Planner
   ↓
Executor
   ↓
Result
   ↓
Evaluator
   ↓
* Reflection
   ↓
Policy Update
   ↓
Agent Memory
```
This can be understood as two loops:

**First Loop: Task Loop**
```
Goal
 ↓
Plan
 ↓
Execute
 ↓
Evaluate
```
**Second Loop: Self-Improvement Loop**
```
Performance Data
 ↓
Reflection
 ↓
Strategy Update
 ↓
Agent Update
```

## Three Technical Approaches to Self-Improving Agents

**Approach 1: Prompt Self-Improvement**

> The agent automatically rewrites the prompt.

Process:
```
Task
 ↓
Run Prompt
 ↓
Evaluate Result
 ↓
Improve Prompt
```

**Paper: “Reflexion: Language Agents with Verbal Reinforcement Learning”**

> Uses multiple LLMs responsible for evaluation, reflection, and generation respectively.

**Paper: “Self-Refine: Iterative Refinement with Self-Feedback”**

> Uses human feedback for iterative learning.

**Approach 2: Tool Strategy Learning**

Example:

Poor strategy:

> search → summarize

Improved strategy:

> search → filter → summarize

Agent updates:

> tool policy

**Approach 3: Code Self-Improvement**

> The agent modifies its own code.

Process:
```
Run code
 ↓
Test
 ↓
Bug detected
 ↓
Rewrite code
 ↓
Retest
```

## Key Mechanisms of Self-Improving Agents

**1 Memory**

The agent needs to remember:
```
past failures
past successes
```
Common Memory types:
```
vector database
experience replay
```

**2 Experience Dataset**

The agent accumulates experiences:
```
task
action
result
score
```
Example:
```
task: research AI market
action: search → summarize
score: 0.6
```
Then it optimizes its strategy.

**3 Reflection Prompt**

Typical prompt:
```
Analyze the failure.

Why did the plan fail?
What should be improved?
```
The LLM generates:
```
lessons learned
```

## Limitations

1. Evaluation is challenging.
2. Learning from errors can degrade performance.
3. Credit assignment problem: which step led to success?
4. Cost issue: requires extensive trial and error.