---
layout:     post
title: FastAPI+Uvicorn+Gradio
subtitle: Rapidly Building High-Quality Web Interfaces
date:       2026-04-04
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- FastAPI
- unicorn
- gradio

---

## Background

> Previously, the web pages in AIAgent projects were built using Html+Css+Js, in one word: ugly.

## Choice

> FastAPI + Gradio for quickly building attractive UIs

**Below is the UI for RAG visualization:**

![](https://raw.githubusercontent.com/LuochuanAD/BlogSourceImage/refs/heads/master/BlogSourceImage/BlogSourceImage2026/gradio_ui.png)

**The code is very simple, as follows:**

[https://github.com/LuochuanAD/Fine-tuning-Learn/tree/main/FastAPI%2BUvicorn%2BGradio_Demo](https://github.com/LuochuanAD/Fine-tuning-Learn/tree/main/FastAPI%2BUvicorn%2BGradio_Demo)


## Trade-offs

**Use cases for Gradio**

- Creating AI demos  
- Building internal tools  
- Rapid prototyping (MVP)  
- Agent debugging interfaces  
- RAG visualization  


## Limitations (when using Gradio)

1. Limited UI customization capabilities  
2. Limited complexity for frontend interactions  
3. Not suitable for production-level ToC products  


## Future

> In the future, I plan to use Next.js + FastAPI to build the frontend and backend of a **ToC** AIAgent.