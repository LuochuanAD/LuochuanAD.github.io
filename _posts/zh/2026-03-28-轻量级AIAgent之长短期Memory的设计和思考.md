---
layout:     post
title:      轻量级AIAgent之长短期Memory的设计和思考
subtitle:   长短期Memory
date:       2026-03-28
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- Memory

---

## 背景

> 场景: 在类似于ChatGPT设计风格的对话框中,用户可以进行对话,文件分析,以及其他的一些文档生成,排版等等的**轻量级ChatAgent**. 

**注意:此特定的AIgent属于即时性对话和分析,本文不涉及RAG和长任务规划**

**要解决的问题:**

- 1, 通过用户意图分析(intent router),进行长短期Memory设计
- 2, 用户通过引用一段文字进行追问时, 短期记忆的设计
- 3, 解决长期记忆的缺点: (用户是谁, 历史行为, 偏好, 长期目标) | (系统角色性格,回答的风格).

## 设计思路

- 1, 我需要设计一个历史记录数据库,实时将每一次和ChatAgent的对话历史保存为一整个数据条.
- 2, 我需要设计一个用户意图分析(intent router), 判断LLM是否可以直接通用function calling进行回答, 判断是否需要短期Memory, 判断是否需要长期memory, 判断是否再次分析文件来回答, 判断是否修改ChatAgent的系统角色的性格,回答风格等等.
- 3, 我需要设计一个Agent,根据用户的要求,生成符合用户要求的系统角色性格,回答风格等等的prompt词条, 写入或修改prompt_system.txt. 这样chatAgent在每次回复用户问题时,都能调用这个文件来调整.
- 3, 根据用户意图分析(intent router),将涉及到文件分析的对话保存为长期Memory.
- 4, 正常情况将用户的最近5轮对话保存为短期Memory, 可以通过历史记录数据库检索最近5次对话记录.
- 5, 当用户引用了一段文字进行追问时, 在历史记录数据库中检索这段文字出现的位置, 然后将当前位置的3轮对话保存为短期记忆.


## 架构图

![](https://raw.githubusercontent.com/LuochuanAD/BlogSourceImage/refs/heads/master/BlogSourceImage/BlogSourceImage2026/ChatAgent.png)


## 架构图详解

### Intent Layer (LLM)

> 针对用户的输入,用LLM进行意图分析,分为:QA问题,需要文件分析,需要查找记忆模块, 系统角色设置.

结构化输出json为:

```
intent_layer = {
	type: "qa / file_analysis / memory_query / config_change",
	need_memory: true/false,
	need_file: true/false
}
```

### Control Layer

> 基于规则的决策,调用功能模块

伪代码如下:

```
intent = analyze_intent(user_input)

if intent["need_memory"]:
    memory = retrieve()

if intent["need_file"]:
    file = analyze()
......

```

**QA问题:**

可以控制是否调用工具?调用什么工具, 然后

- 1, 用LLM + function calling来回答用户的问题.
- 2, 用Tool Registry + 规则匹配 (工具intent和用户输入来选择tool)


**file_analysis**

在openAI中,可以直接将File和用户输入作为参数调用API.

**memory_query**

> 查找memory模块中的长短期记忆.

- 一般来说,短期记忆是最近3-5轮对话,
- 用户通过引用一段文字进行追问时, 在历史记录数据库中检索该文本出现位置 → 取3轮上下文
- 短期记忆取多少轮上下文,都是要基于tokens限制的,如: tokens < 3000

**config_change** System_Agent (LLM)

> 这里需要调用系统Agent, 对用户的要求,提炼出相应的的性格,回答方式等等.实时更新System_prompt中的动态提示词部分. System_Prompt = base_prompt + dynamic_profile.

LLM结构化输出json为:

```
dynamic_profile = {
  "tone": "formal",
  "style": "concise",
  "persona": "technical expert"
}
```

## Contex Builder Layer

> prompt的上下文管理, 动态改变prompt结构和大小,有助与LLM降噪和节约Tokens

## Memory Update Layer

1, 将用户的问题和LLM的response 保存到历史记录数据库中.
2, 根据用户的总结语句,或者文件分析结果都保存到长期Memory的数据库中.
3, 短期记忆是从历史记录数据库中动态且临时获取的.


### 改善

1, 将历史记录数据库,改为向量数据库,方便检索. 同时可以扩展到RAG

2, 添加日志输出,在debug时,输出如下

```
【Intent】
类型：file_followup

【Memory命中】
命中：第23条记录

【使用上下文】
- 最近5轮对话
- 文件摘要

【最终回答来源】
基于文件内容 + 历史问题
```













