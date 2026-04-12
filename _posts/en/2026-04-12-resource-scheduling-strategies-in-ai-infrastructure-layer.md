---
layout:     post
title: Resource Scheduling Strategies in AI Infrastructure Layer?
subtitle: AI Infra Layer
date:       2026-04-12
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- AI-Infra-Layer

---

## Background

> In a privatized large model system, I have deployed Embedding models, Reranker models, and LLMs locally, and implemented high-concurrency batch processing.

## Configuration (config.py)

I have locally deployed multiple services (Embedding models, Reranker models, LLMs, etc.). So how can I implement resource scheduling strategies through `config.py` when users request these microservices?

**Distinctions**

| Service   | Characteristics     | Tuning Strategy        |
| --------- | ------------------- | --------------------- |
| embedding | High throughput, low compute | Large batch + multiple workers |
| reranker  | Moderate compute     | Medium batch          |
| LLM       | Heavy compute        | Small batch + rate limiting |

**Configuration Parameters**

Below is an example configuration for a local model:

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

## Enterprise Architecture (config.yaml)

**Decoupled Design (Model Layer / Service Layer Separation)**

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