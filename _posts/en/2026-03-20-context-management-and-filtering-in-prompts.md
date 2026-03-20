---
layout:     post
title: Context Management and Filtering in Prompts
subtitle: Technical Blog Title
date:       2026-03-20
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- Prompt

---

## Background

> When developing a ChatAgent for document analysis, in multi-turn conversations, users upload a file for analysis, and the LLM answers user questions based on the analysis results of that file. However, during the user’s 2nd, 3rd, 4th... questions, sometimes the questions are completely unrelated to the file. In that case, always including the file analysis results in the prompt context becomes undesirable because the input tokens become excessive, raising costs. Managing and filtering the prompt context is therefore essential.

## Analysis

> 1. The final outcome effect is to dynamically manage the structure and length of the prompt context based on each of the user’s queries.

> 2. To dynamically manage the prompt context’s structure and length based on each user query, it is necessary to perform intent analysis on each question to decide whether the file should be analyzed again.

> 3. If the intent analysis shows that the user’s question is unrelated to the file, then the prompt context can be dynamically composed without including the file analysis results.

> 4. If the intent analysis shows that the user’s question is similar to the initial file analysis query, the previous file analysis results can be reused to answer. This involves scoring the relevance between “question <=> result.”

> 5. If the intent analysis indicates the user’s question differs from the initial file query but must be answered based on the file content, then the file should be re-analyzed via LLM.

## Workflow Diagram

![https://raw.githubusercontent.com/LuochuanAD/BlogSourceImage/refs/heads/master/BlogSourceImage/BlogSourceImage2026/prompt_manager.png](https://raw.githubusercontent.com/LuochuanAD/BlogSourceImage/refs/heads/master/BlogSourceImage/BlogSourceImage2026/prompt_manager.png)