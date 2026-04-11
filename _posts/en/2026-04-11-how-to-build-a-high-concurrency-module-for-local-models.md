---
layout:     post
title: How to Build a High-Concurrency Module for Local Models?
subtitle: AI Infra Layer
date:       2026-04-11
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- asyncio
- AI-Infra-Layer

---

## Background

> In a privatized large model system, I have deployed the Embedding model, Reranker model, and LLM locally, and I also need to support concurrency. Here, I have built a dedicated high-concurrency module.

## Approach

Queue + Batch Processing

### Architecture

```
Request1 \
Request2  \
Request3   →  Queue → Batch → one model.encode()
Request4  /
Request5 /
```

### Project Structure (High-Concurrency Batch Request Processing Module)

```
├── inference_engine/          High-concurrency batch request processing module
│   ├── queue_manager.py       # Queue management
│   ├── batch_scheduler.py     # Batch scheduling
│   ├── worker.py              # Worker executor
│   ├── task.py                # Request encapsulation
│   └── config.py              # Configuration (batch size, etc.)
```

## Details

### task.py

Unified abstraction: Task

```
class InferenceTask:
    def __init__(self, data, future):
        self.data = data
        self.future = future "future": ...
}
```
### queue_manager.py 

Queue management

```
import asyncio

class QueueManager:
    def __init__(self):
        self.queue = asyncio.Queue()

    async def put(self, task):
        await self.queue.put(task)

    async def get_batch(self, max_batch_size, timeout=0.01):
        .....
        return batch
```
### batch_scheduler.py

Batch scheduling

```
import asyncio

class BatchScheduler:
    def __init__(self, queue_manager, process_fn, batch_size=32):
        self.queue_manager = queue_manager
        self.process_fn = process_fn
        self.batch_size = batch_size

    async def start(self):
        while True:
            batch = await self.queue_manager.get_batch(self.batch_size)

            if not batch:
                continue

            inputs = [task.data for task in batch]

            # 👉 Core: unified processing
            results = self.process_fn(inputs)

            for task, result in zip(batch, results):
                task.future.set_result(result)
```
### worker.py

Worker executor

```
import asyncio

class InferenceWorker:
    def __init__(self, scheduler):
        self.scheduler = scheduler

    def start(self):
        asyncio.create_task(self.scheduler.start())
```
### config.py

Configuration

```
BATCH_SIZE = 32
TIMEOUT = 0.01
```


## Summary

In localized model deployments—whether Embedding models, reranker models, or local LLMs—the project structure is consistent. Therefore, the high-concurrency batch request processing module is extracted separately so that each localized microservice can uniformly integrate the `inference_engine` module.

## Future Work

The next step is resource scheduling policies, which I will explain in upcoming articles.