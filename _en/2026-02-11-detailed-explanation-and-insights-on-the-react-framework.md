---
layout:     post
title: Detailed Explanation and Insights on the ReAct Framework
subtitle: ReAct Prompt
date:       2026-02-11
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- Prompt 
- AI

---

## Background

> Various papers propose multiple reasoning frameworks for intelligent agents (also called frameworks): CoT, ToT, LLM+P, etc. Among them, the ReAct framework is used as the reasoning engine by several AI application development tools such as LangChain and LlamaIndex.


## ReAct Framework

> ReAct: Synergizing Reasoning and Acting in Language Models

![](https://raw.githubusercontent.com/LuochuanAD/BlogSourceImage/refs/heads/master/BlogSourceImage/BlogSourceImage2026/React_mind.png)



```
PromptTemplate(
	input_variables = ['agent_scratchpad', 'input', 'tool_name', 'tools'],
	template = 'Answer the folllowing questions as best you can.
		You have access to the folowing tools: \n\n{tools}\n\n
		Use the following format: \n\n
		Question: the input question you must answer \n
		Thought: you should always think about what to do \n
		Action: the action to take, should be one of [{tool_names}] \n
		Action input: the input to the action \n...\n
		Observation: the result of the action \n...\n
		(this Thought/ Action/ Action Input/ Observation can repeat 3 times) \n
		Thought: I now know the final answer \n
		Final Answer: the final answer to the original input questions \n\n Begain! \n\n
		Questions: {input} \n
		Thought: {agent_scratchpad}'
)

```
During instantiation, this prompt guides a large language model to answer questions in a specific format that includes Thought, Action, Action Input, and Observation. This cycle may repeat up to 3 times as needed to arrive at the final answer.


## References

LangChain official site: hwchase17/react