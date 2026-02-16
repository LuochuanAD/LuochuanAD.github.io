---
layout:     post
title:      Plan-and-Execute框架思路与应用
subtitle:   Plan-and-Solve
date:       2026-02-15
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- Prompt 
- AI

---

## 背景

> LangChain的Plan-and-Execute框架受到关于Plan-and-Solve的论文的启发.Plan-and-Execute非常适合更复杂的长期规划,把复杂问题拆分成一个个子任务,逐个击破. 这会频繁的的调用大模型,但可以避免ReAct Agent循环过程中产生提示词过长的问题.


## Plan-and-Solve 策略

![](https://raw.githubusercontent.com/LuochuanAD/BlogSourceImage/refs/heads/master/BlogSourceImage/BlogSourceImage2026/COT%E5%92%8CPlan-and-Solve%E5%AF%B9%E6%AF%94.png)

上图是无样本COT和Plan-and-Solve的对比. Plan-and-Solve本质上将COT具体化为一个个小任务,然后根据这一个个小任务去一个个的解决任务. Plan-and-Execute框架和Plan-and-Solve的思路是一样的,但在执行上,Plan-and-Execute框架更偏向于AI实操.


## Plan-and-Execute框架的应用举例

> 适合Question ReWriting设计和多Agent协调工作等等

场景: 在一个构建了RAG系统的完整AI Agent系统中

**Question**: 查找Louis和刘亦菲的个人信息, 并且根据他们的个人信息生成一个结婚请柬,将此结婚请柬的内容发送到worldXXTest@gmail.com邮箱中.

Plan-and-Execute将任务分解为:Plan,Execute

利用LLM将Question分解为大概5个**Plan**:
```
|-1- 在数据库中查找Louis的个人信息
|-2- 在网络查找刘亦菲的个人信息
|-3- 在网络上查找结婚贺卡请柬的模版
|-4- 生成Louis和刘亦菲的结婚贺卡请柬内容
|-5- 发送内容到worldXXTest@gmail.com邮箱
```
利用ReAct框架的LLM一一处理这5个Plan, **Execute**:
```
|-1- ”Louis的个人信息:iOS and AI Engineer,男, 性格好,温和,善良,幽默......“
|-2- 调用Tool工具Browser,得到:“刘亦菲的个人信息: 茜茜, 女,影视和电影明星,颜值与实力并存,爱好自由......”
|-3- 结婚请柬的模版:“我们结婚啦, 您好xxx先生/女士,欢迎您在2026年xx月xx日,参加我们在...”
|-4- Louis和刘亦菲的结婚请柬内容:“我们结婚啦, 您好xxx先生/女士,欢迎您在2026年2月31日,参加我们在月球的北极宫殿举行的太阳历大婚礼,届时我们会派遣星际飞船LL001号去接您,请在2月30日晚上12点上海的东方明珠塔顶楼乘坐飞船.”
|-5- 调用工sendMail,发送结婚请柬内容
```
**总结:调用一次LLM生成5个plan, 调用5次 ReAct Agent处理这5个Plan.**

##企业级AI gent如何调用Tools(Plan-and-Execute框架)

[https://strictfrog.com/2026/02/15/%E4%BC%81%E4%B8%9A%E7%BA%A7AI-gent%E5%A6%82%E4%BD%95%E8%B0%83%E7%94%A8Tools(Plan-and-Execute%E6%A1%86%E6%9E%B6)/](https://strictfrog.com/2026/02/15/%E4%BC%81%E4%B8%9A%E7%BA%A7AI-gent%E5%A6%82%E4%BD%95%E8%B0%83%E7%94%A8Tools(Plan-and-Execute%E6%A1%86%E6%9E%B6)/)

## 参考

Lei Wang等人的论文“Plan-and-Solve Prompting: Improving Zero-Shot Chain-of-Thought Reasoning by Large Language Models” [https://aclanthology.org/2023.acl-long.147.pdf](https://aclanthology.org/2023.acl-long.147.pdf)










