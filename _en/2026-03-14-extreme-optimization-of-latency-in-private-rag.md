---
layout:     post
title: Extreme Optimization of Latency in Private RAG
subtitle: Private RAG
date:       2026-03-14
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- RAG

---

## Background

> During the construction of a RAG system, I significantly improved the accuracy of knowledge retrieved by RAG through designing Query Rewrite, Rerank, Structural Chunking, Structural Prompting, etc. However, when using the Flask application integrated with this RAG, I noticed a lot of waiting time. Therefore, I wrote this article to optimize the wait time as much as possible.

## Use Case:

> By calling a private API, obtaining a large number of PDFs, extracting text from PDFs via OCR, chunking, embedding, and finally writing into a vector database, providing access to the RAG knowledge base via a web application.

## Time Consumption Analysis:

| Step               | Time Proportion |
|:-------------------|:----------------|
| Private API Call   | 10%             |
| PDF Download      | 15%             |
| PDF Parsing       | 35%             |
| Chunking          | 5%              |
| Embedding         | 20%             |
| Writing to Vector DB | 15%           |

## Extreme Optimization

### Complete Architecture

> Split the architecture into two parts: Users query through the web application, while the backend handles the entire pipeline from "Private API call => ... => writing to vector database."

```
                ┌───────────────┐
                │   Web Server  │
                │   (Flask)     │
                └───────┬───────┘
                        │
                        │Query
                        ▼
                ┌───────────────┐
                │  Vector DB    │
                └───────────────┘


                Document Processing Pipeline (Backend)
Private API
      │
      ▼
Document Queue
      │
      ▼
Parser Workers
      │
      ▼
Embedding Workers
      │
      ▼
Vector DB

```

### 1. Asyncio-Based Asynchronous Requests to Private API for File Download

> Note: For APIs like Gmail, each API call only fetches the latest emails. Emails are loaded in real-time.

```
import aiohttp
import asyncio

async def download_attachment(session, mail):

    url = mail["url"]
    filename = f"pdfs/{mail['id']}.pdf"

    async with session.get(url) as resp:
        data = await resp.read()

    with open(filename, "wb") as f:
        f.write(data)

    return filename


async def download_all(mails):

    async with aiohttp.ClientSession() as session:

        tasks = [
            download_attachment(session, mail)
            for mail in mails
        ]

        results = await asyncio.gather(*tasks)

    return results


asyncio.run(download_all(mails))

```

### 2. Cache API Downloads

```
Private API
↓
Download
↓
Amazon S3
↓
Parsing
```

### 3. Parallel PDF Parsing

| Parser Library | Speed |
|:---------------|:-------|
| PyMuPDF        | Fast  |
| pdfminer       | Medium |
| OCR            | Slow  |

> Note: Many PDFs contain actual text, so OCR parsing is not always necessary.

1. Use PyMuPDF to check if text length exceeds 60; if yes, skip OCR parsing

Pseudocode:
```
def has_text(page):
    text = page.get_text()
    return len(text.strip()) > 60

```
2. Use CPU * 2 threads to parse the files
```
def parse_page(page):
    return page.get_text()

def parse_pdf(file_path):
    with fitz.open(file_path) as doc:
        with ThreadPoolExecutor(max_workers=8) as executor:
            executor.map(pipeline, pdf_files)

...
for r in executor.map(...):
    process(r)         
```

### 4. GPU-Accelerated OCR (Using onnxruntime)

Examples:
- PaddleOCR
- EasyOCR

Model Export:
```
PyTorch/Paddle
    ↓
ONNX
    ↓
ONNX Runtime
```

GPU Inference:

> CUDAExecutionProvider

Speed improvement: CPU → GPU approximately 10x

### 5. Chunking

> Keep chunk size within a certain range, e.g., 400–800 tokens

However, here I used Structural Chunking, so fixed-size chunking isn’t necessary.

Article: “Designing Structural Chunks for RAG”  
[https://strictfrog.com/2026/01/31/RAG%E4%B9%8BStructural-Chunks%E8%AE%BE%E8%AE%A1%E4%B8%8E%E6%80%9D%E8%80%83/](https://strictfrog.com/2026/01/31/RAG%E4%B9%8BStructural-Chunks%E8%AE%BE%E8%AE%A1%E4%B8%8E%E6%80%9D%E8%80%83/)

Three common chunking methods:  
```
Rule-based chunk (fastest)
Semantic chunk (embedding)
Layout chunk (complex documents)
```

**If model inference is involved in chunking, onnxruntime acceleration can be used**

> For example, using a layout detection model to identify document structure can utilize onnxruntime acceleration

### 6. Embedding Optimization (Using onnxruntime Acceleration)

- 1. Batch embedding with batch_size setting  
- 2. Create a thread pool for embedding, size = CPU cores / 2  
- 3. Convert embedding model to ONNX, use onnxruntime for acceleration

Partial architecture diagram:

```
chunk generator
      │
      ▼
chunk queue
      │
      ▼
Worker Pool (2~4)
      │
      ▼
Batch Builder (64 chunks)
      │
      ▼
ONNX Runtime Inference
      │
      ▼
Vector DB Batch Insert
```

Pseudocode:

```
import onnxruntime as ort
from concurrent.futures import ProcessPoolExecutor

session = ort.InferenceSession(
    "embedding.onnx",
    providers=["CPUExecutionProvider"]
)

def embed_batch(texts):

    inputs = tokenizer(
        texts,
        padding=True,
        truncation=True,
        return_tensors="np"
    )

    outputs = session.run(None, dict(inputs))

    return outputs[0]

def process_chunks(chunk_batch):

    embeddings = embed_batch(chunk_batch)

    vector_db.insert(embeddings)

def run_pipeline(chunks):

    batches = list(batch(chunks,64))

    with ProcessPoolExecutor(max_workers=4) as executor:
        executor.map(process_chunks, batches)


```

Assuming AWS: 8 vCPU, 1 GPU  
Configuration: worker = 2, batch_size = 128, onnxruntime-gpu

### 7. Vector DB Batch Inserts

> For example, insert 1000 vectors in one batch

### 8. Caching Mechanism

> Hash PDF, page, and chunk files to avoid duplicate embedding

Refer to this article: “Incremental Vector Update Strategy for Embeddings”

[https://strictfrog.com/2026/03/14/Embedding%E4%B9%8B%E5%A2%9E%E9%87%8F%E5%90%91%E9%87%8F%E6%9B%B4%E6%96%B0%E7%AD%96%E7%95%A5/](https://strictfrog.com/2026/03/14/Embedding%E4%B9%8B%E5%A2%9E%E9%87%8F%E5%90%91%E9%87%8F%E6%9B%B4%E6%96%B0%E7%AD%96%E7%95%A5/)

### 9. Model Warm-up

When starting Flask:

```
load_embedding_model()
load_ocr_model()
```