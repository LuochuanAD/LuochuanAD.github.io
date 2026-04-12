---
layout:     post
title:      AI Infra Layer之资源调度策略?
subtitle:   AI Infra Layer
date:       2026-04-12
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- AI-Infra-Layer

---

## 背景

> 在私有化大模型系统中,我已经将Embedding模型,Reranker模型,LLM部署本地,并且实现了高并发批处理.

## 配置 (config.py)

我已经本地化部署了多个服务(Embedding模型,Reranker模型,LLM等等), 那么在用户请求这些微服务时, 我该如何通过config.py来实现资源调度策略?

**区分**

| 服务        | 特点      | 调参策略               |
| --------- | ------- | ------------------ |
| embedding | 高吞吐、轻计算 | 大 batch + 多 worker |
| reranker  | 中等计算    | 中 batch            |
| LLM       | 超重计算    | 小 batch + 限流       |


**配置参数**

下面以本地模型的配置参数为例:

```
LLM_SERVICE_CONFIG = {

    "llm_model_name": "xxx",
    
    "batch_size": 4,
    "batch_timeout": 0.05,

    "max_queue_size": 50,
    "worker_count": 1,

    "queue_timeout": 0.2,
    "inference_timeout": 2.0,
    "total_timeout": 3.0,

    "rate_limit": 10,
    "enable_cache": False,

}

```
## 企业级架构 (config.yaml)

**解耦设计（Model Layer / Service Layer 分离）**

```

models:
  embedding_v1:
    type: embedding
    model_name: "xxxxxemdeddingModelxxx"
    device: "cpu"

  reranker_v1:
    type: reranker
    model_name: "xxxxrerankerxxxx"

  llm_v1:
    type: llm
    model_name: "xxx"  

services:
  embedding_service:
    model: embedding_v1
    
    runtime:
      batch_size: 64
      batch_timeout: 0.01
      max_queue_size: 500
      worker_count: 4
      queue_timeout: 0.05
      inference_timeout: 0.3
      total_timeout: 0.5
      rate_limit: 100
      enable_cache: true

  reranker_service:
    model: reranker_v1
    
    runtime:
      batch_size: 16
      batch_timeout: 0.02
      max_queue_size: 200
      worker_count: 2
      queue_timeout: 0.1
      inference_timeout: 0.5
      total_timeout: 0.8
      rate_limit: 50
      enable_cache: false

  llm_service:
    model: llm_v1
    
    runtime:
      batch_size: 4
      batch_timeout: 0.05
      max_queue_size: 50
      worker_count: 1
      queue_timeout: 0.2
      inference_timeout: 2.0
      total_timeout: 3.0
      rate_limit: 10
      enable_cache: false



```






