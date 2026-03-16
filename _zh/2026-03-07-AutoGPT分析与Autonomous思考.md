---
layout:     post
title:      AutoGPT分析与Autonomous思考
subtitle:   Auto-GPT
date:       2026-03-07
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- Autonomous 
- AutoGPT

---

## 背景

> Auto-GPT是能够根据用户的目标输入,制定计划,解析计划,拆解计划,执行计划,评估执行结果,并且不断循环产生结果并结合外部资源执行相应操作,逐步达到达到目的.

> 我对Auto-GPT的prompt提示词会自动更新这一点非常感兴趣.

## Auto-GPT原文讲解

![](https://raw.githubusercontent.com/LuochuanAD/BlogSourceImage/refs/heads/master/BlogSourceImage/BlogSourceImage2026/autoGpt_mind.png)

```
用户的目标输入+初始化系统提示词 =》

检索Memory记忆库 =》

得到以前的Memory =》

制定计划 =》生成(首轮/下一轮)提示词 =》

调用LLM(ReAct框架)执行计划 =》

更新当前的状态(用于生产下一轮提示词) =》

调用工具或组件执行计划 =》

根据当前结果判断是否继续回到“制定计划”执行下一步,还是直接Done


```

## AutoGPT评价

由于LLM对任务分解的不稳定, 每次循环制定计划时,会出现prompt提示词爆炸增长.生成的计划也会出现不可控的变化. 在商业Agent上几乎不会使用Auto-GPT框架.



## 稳定的Autonomous Agent结构



```
Goal
 ↓
Planner
 ↓
Task Queue
 ↓
Executor
 ↓
Evaluator
 ↓
Memory
 ↓
Loop
```

## Autonomous Agent评价

```
问题:	说明:
成本	LLM调用次数多
稳定性	任务规划不稳定
可控性	难调试
安全	可能执行错误操作

```


## 参考

“Auto-GPT for Online Decision Making:Benchmarks and Additional Opinions”
[https://www.semanticscholar.org/reader/3b8871e4c25d3aaca2bee6606c07bc870337253c](https://www.semanticscholar.org/reader/3b8871e4c25d3aaca2bee6606c07bc870337253c)


"Auto-GPT"
[https://github.com/Significant-Gravitas/AutoGPT](https://github.com/Significant-Gravitas/AutoGPT)






