---
layout:     post
title:      FastAPI+Uvicorn+Gradio
subtitle:   快速构建高质量Web界面
date:       2026-04-04
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- FastAPI
- unicorn
- gradio

---

## 背景

> 在之前做过的AIAgent中的Web页面,都是以Html+Css+Js搭建, 一个字: 丑.

## 选择 

> FastAPI + Gradio 快速构建好看的UI

**下面是RAG可视化的UI:**

![](https://raw.githubusercontent.com/LuochuanAD/BlogSourceImage/refs/heads/master/BlogSourceImage/BlogSourceImage2026/gradio_ui.png)

**代码很简单,如下:**

[https://github.com/LuochuanAD/Fine-tuning-Learn/tree/main/FastAPI%2BUvicorn%2BGradio_Demo](https://github.com/LuochuanAD/Fine-tuning-Learn/tree/main/FastAPI%2BUvicorn%2BGradio_Demo)

**下面是语音聊天可视化的UI:**

![](https://raw.githubusercontent.com/LuochuanAD/BlogSourceImage/refs/heads/master/BlogSourceImage/BlogSourceImage2026/voice_demo.PNG)


## 取舍

**用 Gradio 的场景**

- 做 AI Demo
- 做内部工具
- 快速验证想法（MVP）
- Agent 调试界面
- RAG 可视化

## 限制 (用 Gradio 的场景)

1. UI 定制能力有限
2. 前端交互复杂度有限
3. 不适合正式 ToC 产品

## 未来

> 未来我用Next.js + FastAPI来搭建**ToC**的AIAgent的前后端













