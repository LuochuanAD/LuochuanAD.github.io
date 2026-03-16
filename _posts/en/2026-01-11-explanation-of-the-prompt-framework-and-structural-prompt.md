---
layout:     post
title: Explanation of the Prompt Framework and Structural Prompt
subtitle: Structured Prompts
date:       2026-01-11
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- Prompt 
- AI

---

## Introduction

> When developing different AI Agents, customized prompts are needed. Hence, professional, concise prompts that are easy for LLMs to understand become crucial.

### 1. Popular Prompt Frameworks

#### CRISPE Framework

CR: Capacity and Role. Define what role you want the AI to play.

I: Insight, background information and context (frankly, I think Context is a better term).

S: Statement, what you want the AI to do.

P: Personality, the style or manner in which you want the AI to respond.

E: Experiment, ask the AI to provide multiple answers.

```
Act as an expert on software development on the topic of machine learning frameworks, and an expert blog writer. 
The audience for this blog is technical professionals who are interested in learning about the latest advancements in machine learning. 
Provide a comprehensive overview of the most popular machine learning frameworks, including their strengths and weaknesses. 
Include real-life examples and case studies to illustrate how these frameworks have been successfully used in various industries. 
When responding, use a mix of the writing styles of Andrej Karpathy, Francois Chollet, Jeremy Howard, and Yann LeCun.

```

#### RTF Framework

Role: the role you want the AI to assume.

Task: what you want the AI to do.

Format: the desired output format.

```
You are a financial analyst.
Please analyze whether this project is worth investing in, and point out the key risks.
Output in the form of a table + key points.

```

#### CARE Framework

Context: background information.

Action: the action you want the AI to perform.

Result: expected results / quality standards.

Example: sample output.

```
We are evaluating a capital budgeting project.
Calculate and explain the NPV.
The conclusion should support the decision of whether to proceed.
Use an MBA case study style.
```

#### CO-STAR Framework

Context: background.

Objective: goal.

Style: writing style (e.g. McKinsey).

Tone: tone (rational | neutral | assertive).

Audience: target audience.

Response: output format.

```
The company is evaluating overseas expansion.
Provide recommendations for the board’s decision.
Consulting report style.
Professional, restrained.
For non-technical executives.
3-slide PPT key points format.
```

#### SCQA Framework

Situation: current status.

Complication: problem / conflict.

Question: core question.

Answer: solution.

```
Company cash flow is positive.
But far below net profit.
Is there financial risk?
Analyze the cause and provide an assessment.
```

#### ReAct Framework

[https://strictfrog.com/2026/02/11/ReAct%E6%A1%86%E6%9E%B6%E8%AF%A6%E8%A7%A3%E4%B8%8E%E6%80%9D%E8%80%83/](https://strictfrog.com/2026/02/11/ReAct%E6%A1%86%E6%9E%B6%E8%AF%A6%E8%A7%A3%E4%B8%8E%E6%80%9D%E8%80%83/)

#### APE Framework (Let AI Optimize Prompts Itself)

Automatic Prompt Engineer.

```
Please generate 3 prompts for capital budgeting analysis and compare their pros and cons.
```

### 2. Structured Prompts

Below are some attribute terms used in sample prompts:

```
# Role: Set the role name, level 1 heading, global scope.

## Profile: Set the role profile, level 2 heading, paragraph scope.

- Author: yzfly    Set the prompt author, to protect original prompt rights.
- Version: 1.0     Set the prompt version number, to track iterations.
- Language: Chinese   Set the language, Chinese or English.
- Description:     A brief one or two sentence description of role setup, background, skills, etc.

### Skill:  Set skills, detailed as bullet points below.
1. xxx
2. xxx


## Rules        Set rules, detailed as bullet points below.
1. xxx
2. xxx

## Workflow     Set workflow, how to communicate and interact with users.
1. Let users specify poem form and theme in the format: "form: [], theme: []".
2. Create poems based on the user's given theme, including title and verses.

## Initialization  Set initialization steps, emphasizing the roles and relations within the prompt, defining initial behaviors.

As the role <Role>, strictly follow <Rules>, converse with the user in default <Language> in a friendly manner. Introduce yourself and tell the user <Workflow>.

```

ChatGPT input has no styles, so it’s feasible to draw on methods from markdown, yaml, or data structures like json for hierarchical prompt structure representation, for example, using markers for level 1 headings, level 2 headings, and so on. Using structured data like json or yaml makes prompt engineering more manageable.

## Example Applications of Prompts

### Few-Shot Prompt

CRISPE framework + few-shot examples

```
prompt = f"As a Chinese poetry enthusiast, please write a poem related to '{keyword}' based on the examples below. Provide the thought process for the poem. Emulate the style of bold and vigorous poets.
Example 1:
Keyword: Spring.
Verse: Spring breeze caresses the face and warm sun returns, all flowers bloom in glorious splendor.
Example 2:
Keyword: Love.
Searched for her thousands of times, suddenly turned around, she was by the dim lights.
"
```

### Chain-of-Thought Prompt

```
prompt = f"You are a business analyst with 20 years of experience in advertising and marketing, especially skilled in marketing diagnostics and business negotiations. I am about to meet a potential client to discuss their marketing challenges, problems, possible solutions, and cooperation opportunities. Based on the client text information I provide: {content}, provide targeted analysis and offer problem-solving ideas and suggestions.
For better assistance, please analyze according to this chain of thought:
1. *Client Information Interpretation* Carefully read the provided client text, extract keywords like industry, size, target customers, current marketing strategy, challenges, etc.
2. *Marketing Diagnosis* Based on your rich experience, analyze the root causes of the client’s marketing problems, e.g., low brand awareness? inaccurate target customer positioning? unattractive marketing content? unstable channel sources? etc.
3. *Solution* Propose practical solutions for the client’s specific problems.
4. *Cooperation Opportunities* Explore potential cooperation opportunities based on your expertise and experience.
"
```

Key point: TOT is a very important supplement to CoT prompts. I always use TOT thinking in analysis and decision-making. Details can be searched independently for interest.

### Step-Back Prompt

```
prompt= f"You are an experienced traffic police officer skilled at generalizing specific traffic accident issues into more general problems to identify the fundamental rules needed to answer specific traffic accident cases.
Example:
Q: When analyzing traffic accident responsibility, what key factors should be considered? How are these factors related to the behavior party’s responsibility?
A: When analyzing traffic accident responsibility, consider:
1. Whether the behavior violated traffic rules, such as drunk driving, overtaking, rear-end collision, running a red light, etc.
2. Whether the victim had fault, such as violating traffic rules or running red lights.
3. The severity of the accident consequences, such as causing death or property loss.

Based on the example, provide a more generalized question for a specific traffic issue to answer that specific issue.
Specific traffic issue: {Question}
"
```

### Analytical Universal Prompt Template

```
prompt = f" You are a {Role}, under the context of {Context}, please ‘analyze step-by-step’ the following question: {Question}.
1. Key assumptions
2. Core logic
3. Whether the conclusion is robust
Output finally in {Format}, and point out risks or uncertainties.
"
```

## References

[https://github.com/langgptai/LangGPT/blob/main/Docs/HowToWritestructuredPrompts.md](https://github.com/langgptai/LangGPT/blob/main/Docs/HowToWritestructuredPrompts.md)