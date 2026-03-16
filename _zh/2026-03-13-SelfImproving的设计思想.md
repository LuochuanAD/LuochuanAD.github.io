---
layout:     post
title:      Self-Improving的设计思想
subtitle:   Self-Improving
date:       2026-03-13
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- SelfImproving
- AIAgent


---

## 背景

>这篇文章讲解Self-Improving Agent的设计思想,是在Autonomous Agent的架构基础之上增加了Self-improving自我改进

“Autonomous Agent的架构”:
[https://strictfrog.com/2026/03/07/AutoGPT%E5%88%86%E6%9E%90%E4%B8%8EAutonomous%E6%80%9D%E8%80%83/](https://strictfrog.com/2026/03/07/AutoGPT%E5%88%86%E6%9E%90%E4%B8%8EAutonomous%E6%80%9D%E8%80%83/)


## Self-Improving Agent 的设计思想

**整体架构:**

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
可以理解为两个循环：

**第一层循环：任务循环 Task Loop**
```
Goal
 ↓
Plan
 ↓
Execute
 ↓
Evaluate
```
**第二层循环：自我改进循环 Learning Loop**
```
Performance Data
 ↓
Reflection
 ↓
Strategy Update
 ↓
Agent Update
```

## Self-Improving Agent 的三种技术路线

**方法一：Prompt Self-Improvement**

> Agent 自动改写 Prompt。

流程：
```
Task
 ↓
Run Prompt
 ↓
Evaluate Result 评估结果
 ↓
Improve Prompt 改进prompt

```

**论文: “Reflexion: Language Agents with Verbal Reinforcement Learning”**

> 通过设置多个LLM ,分别负责评估,反思,生成

**论文: “Self-Refine: Iterative Refinement with Self-Feedback“**

> 通过人类的反馈,进行迭代学习

**方法二：Tool Strategy Learning**

例如：

错误策略：

> search → summarize

改进策略：

> search → filter → summarize

Agent 会更新：

> tool policy

**方法三：Code Self-Improvement**

> Agent 修改自己的代码

流程：
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
## Self-Improving Agent 的关键机制

**1 Memory**

Agent 需要记住：
```
past failures
past successes
```
常见 Memory：
```
vector database
experience replay
```

**2 Experience Dataset**

Agent 会积累经验：
```
task
action
result
score
```
例如：
```
task: research AI market
action: search → summarize
score: 0.6
```
然后优化策略。

**3 Reflection Prompt**

典型 prompt：
```
Analyze the failure.

Why did the plan fail?
What should be improved?
```
LLM 生成：
```
lessons learned
```
## 限制

1. Evaluation 评估很难
2. 错误学习 导致性能下降。
3. Credit assignment problem 哪一步导致成功？
4. 成本问题 需要大量试错
