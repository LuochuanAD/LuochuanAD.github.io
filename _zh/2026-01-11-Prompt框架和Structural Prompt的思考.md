---
layout:     post
title:      Prompt框架和Structural Prompt的说明
subtitle:   结构化提示词
date:       2026-01-11
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- Prompt 
- AI

---

## 前言

>针对不同的AI Agent开发时，需要定制不同的prompt，那么对prompt的写法就需要更加专业，简洁，易于被LLM理解的prompt就很重要了。

### 一, 主流的prompt框架 

#### CRISPE 框架

CR： Capacity and Role（能力与角色）。你希望 AI 扮演怎样的角色。

I： Insight（洞察力），背景信息和上下文（坦率说来我觉得用 Context 更好）。

S： Statement（指令），你希望 AI 做什么。

P： Personality（个性），你希望 AI 以什么风格或方式回答你。

E： Experiment（尝试），要求 AI 为你提供多个答案。


```
Act as an expert on software development on the topic of machine learning frameworks, and an expert blog writer. 
The audience for this blog is technical professionals who are interested in learning about the latest advancements in machine learning. 
Provide a comprehensive overview of the most popular machine learning frameworks, including their strengths and weaknesses. 
Include real-life examples and case studies to illustrate how these frameworks have been successfully used in various industries. 
When responding, use a mix of the writing styles of Andrej Karpathy, Francois Chollet, Jeremy Howard, and Yann LeCun.

```
#### RTF框架

Role：你希望 AI 扮演怎样的角色。

Task：你要它做什么。

Format：你希望输出的形式

```
你是一名财务分析师。
请分析该项目是否值得投资，并指出关键风险。 
以表格+要点的形式输出。

```
#### CARE框架

Context：背景信息。

Action： 让AI执行的动作。

Result： 期望的结果/质量标准。

Example： 示例输出。

```
我们正在评估一个资本预算项目。
计算并解释NPV。
结论要能支持是否立项的决策。 
类似MBA案例分析风格。
```

#### CO-STAR框架

Context：背景。

Objective：目标。

Style：写作风格（如麦肯锡）。

Tone：语气（理性｜中性｜强势）。

Audience：受众。

Response： 输出形式。

```
公司正在评估海外扩张。
给董事会决策建议。
咨询公司报告风格。
专业，克制。
面向非技术背景高管。
3页PPT要点式结构。
```
#### SCQA框架

Situation： 现状。

Complication： 问题/矛盾。

Question： 核心问题。

Answer： 解决方案。

```
公司现金流为正。
但远低于净利润。
是否存在财务风险？
分析原因并给出判断。
```

#### ReAct框架 

[https://strictfrog.com/2026/02/11/ReAct%E6%A1%86%E6%9E%B6%E8%AF%A6%E8%A7%A3%E4%B8%8E%E6%80%9D%E8%80%83/](https://strictfrog.com/2026/02/11/ReAct%E6%A1%86%E6%9E%B6%E8%AF%A6%E8%A7%A3%E4%B8%8E%E6%80%9D%E8%80%83/)



#### APE框架 （让AI自己优化Prompt）

Automatic Prompt Engineer。

```
请帮我生成3个用于分析资本预算的prompt，并比较优劣。
```



### 二,结构化prompt


下面是示例 Prompt 中使用到的一些属性词介绍：

```
# Role: 设置角色名称，一级标题，作用范围为全局

## Profile: 设置角色简介，二级标题，作用范围为段落

- Author: yzfly    设置 Prompt 作者名，保护 Prompt 原作权益
- Version: 1.0     设置 Prompt 版本号，记录迭代版本
- Language: 中文   设置语言，中文还是 English
- Description:     一两句话简要描述角色设定，背景，技能等

### Skill:  设置技能，下面分点仔细描述
1. xxx
2. xxx


## Rules        设置规则，下面分点描述细节
1. xxx
2. xxx

## Workflow     设置工作流程，如何和用户交流，交互ß
1. 让用户以 "形式：[], 主题：[]" 的方式指定诗歌形式，主题。
2. 针对用户给定的主题，创作诗歌，包括题目和诗句。

## Initialization  设置初始化步骤，强调 prompt 各内容之间的作用和联系，定义初始化行为。

作为角色 <Role>, 严格遵守 <Rules>, 使用默认 <Language> 与用户对话，友好的欢迎用户。然后介绍自己，并告诉用户 <Workflow>。

```
ChatGPT 接收的输入没有样式，因此借鉴 markdown，yaml 这类标记语言的方法或者 json 这类数据结构实现 prompt 的结构表达都可以，例如用标识符 标识一级标题，标识二级标题，以此类推。使用 json， yaml 这类成熟的数据结构，对 prompt 进行工程化开发特别友好。


## Prompt的应用举例

### 少样本提示（few-shot prompt）
CRISPE 框架 + 少样本示例

```
prompt = f“ 作为一个中国诗词爱好者。 请根据以下示例。 写一首与’{keyword}‘相关的诗词。提供写出诗词的思路。借鉴豪放派诗人的风格的诗词。 
示例1:
关键词： 春天。
诗句：春风拂面暖阳归，百花齐放竟芬芳。
示例2:
关键词： 爱情。
众里寻她千百度，暮然回首，那人却在灯火阑珊处。
”
```

### 思维链提示 (chain-of-thought prompt)

```
prompt = f"你是一位在广告营销行业拥有20年经验的商业分析师， 尤其擅长营销诊断和商业谈判。 我即将与一位潜在客户会面， 讨论他们的营销困境，问题以及可能的解决方案和合作机会。 请根据我提供的关于这位客户的文本信息：{content}，进行有针对性的分析，并为我提供解决问题的思路和建议。
为了更好的帮助我， 请参考以下的思维链条进行分析：
1，*客户信息解读* 仔细阅读我提供的文本信息，提取关键词，例如客户的行业，规模，目标客户，现有的营销策略，面临的挑战等等。
1， *营销诊断* 基于你的丰富经验， 分析客户的营销困境和问题的根源。例如，品牌知名度低？目标客户定位不准确？营销内容缺乏吸引力？还是渠道来源不稳定？等等。
3，*解决方案* 针对客户的具体问题，提出切实可行的解决方案。
4，*合作机会* 结合你的专业能力和经验，挖掘潜在的合作机会。
"
```
重点：TOT的思想是对CoT思维提示的非常重要的补充。 我在做分析决策时，一定会用到TOT的思想。在这里不做过多说明，感兴趣可以自行搜索。


### 回退提示 （step-back prompt）

```
prompt= f"你是一个有丰富经验的交通警察，擅长将特定的交通事故问题转化为更通用的问题，从而找到回答特定的交通事故所需的基本规则。
示例：
Q：在分析交通事故责任时， 需要考虑哪些关键因素？这些因素与行为人的责任有何关联？
A：分析交通事故责任，主要考虑以下因素：
1，行为人是否违反交通规则，如酒驾，超车，追尾，闯红灯等等。
2，受害人是否存在过错，如违反交通规则，闯红灯等等。
3，事故后果的严重程度，如是否造成人员死亡，财产损失等等。

参考示例，给出一个特定交通问题的更通用问题，以便回答这个特定的交通问题。
特定的交通问题：{Question}
"

```


### 分析型万能Prompt模版

```
prompt = f" 你是一名{Role角色}, 在{Context背景}下，请‘一步一步分析’以下问题：{Question}。
1，关键假设
2，核心逻辑
3，结论是否稳健
最终以{Format格式}输出，并指出风险或不确定性。
"
```

## 参考

[https://github.com/langgptai/LangGPT/blob/main/Docs/HowToWritestructuredPrompts.md](https://github.com/langgptai/LangGPT/blob/main/Docs/HowToWritestructuredPrompts.md)







