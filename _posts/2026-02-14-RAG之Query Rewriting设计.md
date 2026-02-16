---
layout:     post
title:      RAG之Query ReWriting设计
subtitle:   Query ReWriting
date:       2026-02-14
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- RAG
- Query ReWriting
---

## 背景

> 在设计RAG系统的过程中,我通过设计了Structural Chunks和Structural Prompt后, 检索出来的结果是完全符合理想的准确性. 但我想针对复杂的多Agent系统更加准确和高效率的提升检索效果, 我想设计了一个结合Structural Chunks的Query ReWriting.

## 选择

> 我有哪些方案?

### 方案1: 
无脑的直接写提示词告诉LLM: 将用户的Query重新生成AI能够理解的提示词并适当的进行补充形成一个完整的结构化的Query.

### 方案2:
结合Structural Chunks的设计 [https://strictfrog.com/2026/01/31/RAG%E4%B9%8BStructural-Chunks%E8%AE%BE%E8%AE%A1%E4%B8%8E%E6%80%9D%E8%80%83/](https://strictfrog.com/2026/01/31/RAG%E4%B9%8BStructural-Chunks%E8%AE%BE%E8%AE%A1%E4%B8%8E%E6%80%9D%E8%80%83/), 对用户的Query进行分类.


## 取舍

> 我为什么选方案2, 不选方案1

方案1 只适合通用型的Query ReWriting, 有效果,但没有结合特定的RAG来设计

方案2 我将用户的Query进行分类(或者说意图分析),2个大类 FAQ和Tools的调用, **这篇文章详解FAQ, Tools的调用在另外一篇文章详解.
**

## 方案2详解

### 1, 先分析RAG系统中的 Structural Chunk是怎么设计的.
通过查看RAG的Chunk切割分析,得出每一篇文档都分为 Basic(基础的个人信息:如姓名,年纪,等等), Skills(个人技能:如python,Java,Go等等),Introduction(自我介绍:一大串字符串),Certification(个人证书:如英语6级,日语1级,AWS证书等等),Project(做过的项目:如个人情感陪伴APP等等),Other

```
resume_chunks = [
	basic_chunk,
	skills_chunk,
	introduction_chunk
	certification_chunk,
	project_chunk_1,
	project_chunk_2,
	project_chunk_3,
......
	other
]
```
### 2, 使用关键词密度算法来判断
> 在Structural Chunks的设计这篇文章 [https://strictfrog.com/2026/01/31/RAG%E4%B9%8BStructural-Chunks%E8%AE%BE%E8%AE%A1%E4%B8%8E%E6%80%9D%E8%80%83/](https://strictfrog.com/2026/01/31/RAG%E4%B9%8BStructural-Chunks%E8%AE%BE%E8%AE%A1%E4%B8%8E%E6%80%9D%E8%80%83/),我们就是使用关键词密度来Chunks切割, 这里我们同样通过关键词密度算法来分类用户Query. 

```
type_keywords = {
	"basic_chunk" = ["姓名",“年龄”,“年纪”,”出生年月“,“大学”,“学院”...],
	“skills_chunk” = [“python”,“C++”,“iOS”,“And”,“Java”,“PHP”...],
	“introduction_chunk” = [“爱学习”,“抗压”,“高效率”,“敏捷开发”,...],
	“certification_chunk” = [“计算机二级”,“大学英语四级”,“大学英语六级”,“雅思”,“AWS”,...],
	“project_chunk” = [“项目经验”,“担当”,“时期”,...]
}
```
例如: 

Query: 我想查找一个姓名叫Louis的人的个人信息.

Query分类(或者称用户意图分析): 在向量数据库中查找Chunk的类型为:Basic,关键词为:Louis

例如:  

Query: 我想查找一个拥有英语6级证书,并且Python开发经验有3年的人.
	
Query分类(或者称用户意图分析): 在向量数据库中查找Chunk的类型为:certification,关键词为:英语6级证书;Chunk的类型为:Skills,关键词为:Python 3年;

#### 提示: 向量数据库中的每个Chunk都包含数据来源: 
```
“source”:{
	"file": "简历(Louis).pdf",
	...
}
```
通过{"file": "简历(Louis).pdf"} 可以再次在向量数据库中找到这个人的完整信息:basic,skills, introduction等等

将这些信息形成结构化的上下文就可以交给LLM来处理了.


### 3 Query ReWriting的提示词怎么设计? 
这些所有的chunks都统称为FAQ. 在一个完整的AI Agent中,对Query ReWriting的要求就会更加全面,那么对Query进行完整分类(或者称意图分析)如下:

```

FAQ
|-- basic
|-- skills
|-- introduction
|-- certification
|-- project
|-- other
Tools
|- sendMails
|- getCurrentWeather
|- ......

```
那么设计Query ReWriting的提示词就需要通用可复用.
这里用到了Plan-and-Execute框架思路:[https://strictfrog.com/2026/02/15/Plan-and-Execute%E6%A1%86%E6%9E%B6%E8%AF%A6%E8%A7%A3%E4%B8%8E%E6%80%9D%E8%80%83/](https://strictfrog.com/2026/02/15/Plan-and-Execute%E6%A1%86%E6%9E%B6%E8%AF%A6%E8%A7%A3%E4%B8%8E%E6%80%9D%E8%80%83/)

#### Planner Prompt:
```
prompt = ‘
	You are a task planner in a multi-step AI system.
	User query: {Query}
	Your job:
	1, Break down the user query into executable steps.
	2, Each step must be atomic and tool-executable.
	3, Steps should be ordered logically.
	4, Do NOT execute anything.
	5, Only return JSON.
’
```

**例如:**

Query: 我想查找一个拥有英语6级证书,并且Python开发经验有3年的人.然后将这个人的个人信息发送到luochuanad@gmail.com

通过LLM(**Planner Prompt**)得出以下结果:

```
json
[
 {
	"step: 1,
	"action": "Search Database"
	"description": "Query the database to find individuals with an English CET-6 certificate 					and 3 years of Python development experience.",
	"parameters": {
		"certificate": "CET-6",
		"experience": {
			"Language": "Python",
			"years": 3
		}
	}
 },
 {
	"step": 2,
	"action": "Retrieve Personal Information",
	"description": "Extract the personal information of the individual(s) found in the 					previous step.",
	"parameters": {
		"fields": ["name", "email", "phone", "address"]
	}
 },
 {
	"step": 3,
	"action": "Format Email",
	"description": "Prepare an email containing the personal information of the individual(s) 					found.",
	"parameters": {
		"recipient": "luochuanad@gmail.com",
		"subject": "Candidate Information",
		"body": Include the personal information retrieved in step 2."
 	}
 },
 {
 	"step": 4,
	"action": "Send Email"
	"description": "Send the formatted email to the specified email address.",
	"parameters": {
		"recipient": "xxx@gmail.com",
		"content": "Use the email content prepared in step 3."
 	}
 }


]

```

#### 提示: {step: 1, "action": "Search Database"}

在"step: 1,"action"为 "Search Database"的时候,根据关键词certificate, experience,分别查找向量数据库中的certification_chunk和skills_chunk.










