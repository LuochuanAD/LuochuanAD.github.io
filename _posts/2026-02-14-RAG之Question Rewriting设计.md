---
layout:     post
title:      RAG之Question ReWriting设计
subtitle:   Question ReWriting
date:       2026-02-14
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- RAG
- Question ReWriting
---

## 背景

> 在设计RAG系统的过程中,我通过设计了Structural Chunks和Structural Prompt后, 检索出来的结果是完全符合理想的准确性. 但我想更加准确和高效率的提升检索效果, 我想设计了一个结合Structural Chunks的Question ReWriting.

## 选择

> 我有哪些方案?

### 方案1: 
无脑的直接写提示词告诉LLM: 将用户的Question重新生成AI能够理解的提示词并适当的进行补充形成一个完整的结构化的Question.

### 方案2:
结合Structural Chunks的设计 [https://strictfrog.com/2026/01/31/RAG%E4%B9%8BStructural-Chunks%E8%AE%BE%E8%AE%A1%E4%B8%8E%E6%80%9D%E8%80%83/](https://strictfrog.com/2026/01/31/RAG%E4%B9%8BStructural-Chunks%E8%AE%BE%E8%AE%A1%E4%B8%8E%E6%80%9D%E8%80%83/), 对用户的Question进行分类.


## 取舍

> 我为什么选方案2, 不选方案1

方案1 只适合通用型的Question ReWriting, 有效果,但没有结合特定的RAG来设计

方案2 我将用户的Question进行分类(或者说意图分析),2个大类 FAQ和Tools的调用, **这篇文章详解FAQ, Tools的调用在另外一篇文章详解.
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
> 在Structural Chunks的设计这篇文章 [https://strictfrog.com/2026/01/31/RAG%E4%B9%8BStructural-Chunks%E8%AE%BE%E8%AE%A1%E4%B8%8E%E6%80%9D%E8%80%83/](https://strictfrog.com/2026/01/31/RAG%E4%B9%8BStructural-Chunks%E8%AE%BE%E8%AE%A1%E4%B8%8E%E6%80%9D%E8%80%83/),我们就是使用关键词密度来Chunks切割, 这里我们同样通过关键词密度算法来分类用户Question. 

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

Question: 我想查找一个姓名叫Louis的人的个人信息.

Question分类(或者称用户意图分析): 在向量数据库中查找Chunk的类型为:Basic,关键词为:Louis

例如:  

Question: 我想查找一个拥有英语6级证书,并且Python开发经验有3年的人.
	
Question分类(或者称用户意图分析): 在向量数据库中查找Chunk的类型为:certification,关键词为:英语6级证书;Chunk的类型为:Skills,关键词为:Python 3年;

#### 提示: 向量数据库中的每个Chunk都包含数据来源: 
```
“source”:{
	"file": "简历(Louis).pdf",
	...
}
```
通过{"file": "简历(Louis).pdf"} 可以再次在向量数据库中找到这个人的完整信息:basic,skills, introduction等等

将这些信息形成结构化的上下文就可以交给LLM来处理了.


### 3 Question ReWriting的提示词怎么设计? 
这些所有的chunks都统称为FAQ. 在一个完整的AI Agent中,对Question ReWriting的要求就会更加全面,那么对Question进行完整分类(或者称意图分析)如下:

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
那么针对FAQ设计的Question ReWriting的提示词 和针对整个AI Agent设计的Question ReWriting的提示词就要区分了.

#### FAQ:
```

prompt = ‘’

```

#### AIAgent:
```

prompt = ‘’

```














