---
layout:     post
title: How to Deploy a Local Embedding Model?
subtitle: AI Infra Layer
date:       2026-04-10
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- FastAPI
- unicorn
- EmbeddingModel
- AI-Infra-Layer

---

## Background

> In a privatized large model system, to avoid external dependencies, reduce the risk of data leakage, and enhance controllability, the Embedding model is deployed locally. Additionally, a standardized interface service is implemented to support external systems in invoking the Embedding functionality via HTTP requests, with fault tolerance, scalability, and concurrency support.

## Tech Stack Choice

> FastAPI + Uvicorn + EmbeddingModel (e5-large)

### Project Structure (Local Embedding Model Wrapped with FastAPI)

```
embedding-service/
│
├── app/
│   ├── main.py          # FastAPI entry point
│   ├── model.py         # Model loading
│   ├── schemas.py       # Input/output schemas
│   ├── router.py        # Model routing
│   ├── services.py      # Request services
│   └── config.py        # Configuration
│
├── requirements.txt
└── run.sh
```

## Detailed Explanation

### config.py

Contains configuration settings for the embedding microservice, such as embedding_name, device, etc.

```
MODEL_NAME = "intfloat/e5-large"
DEVICE = "cpu"   # Change to "cuda" if you have a GPU
```

### router.py

Defines API routes for the model: `/embed`, `/embed_batch`

```
router = APIRouter()

@router.get("/")
def root():
    return {"message": "Welcome to the E5 Embedding Service!"}  

@router.post("/embed", response_model=EmbeddingResponse)
def embed(request: EmbeddingRequest):
    vector = embed_query(request.text)
    return {"vector": vector}

@router.post("/embed_batch", response_model=BatchEmbeddingResponse)
def batch_embed(request: BatchEmbeddingRequest):
    vectors = embed_batch(request.texts, is_query=request.is_query)
    return {"vectors": vectors}

```

### main.py

The local embedding model API interface, following the OpenAI API style at `api/v1/embeddings`

```
from fastapi import FastAPI
from .router import router

app = FastAPI(title="E5 Embedding Service")
app.include_router(router, prefix="api/v1/embeddings")
```

### model.py

Embedding model structure.

```
from .config import MODEL_NAME, DEVICE
class EmbeddingModel:
    def __init__(self):
        self.model = SentenceTransformer(MODEL_NAME, device=DEVICE)

    def embed_query(self, text: str):
        # ⚠️ E5 requires adding a prefix
        text = "query: " + text
        return self.model.encode(text, normalize_embeddings=True).tolist()

    def embed_passage(self, text: str):
        text = "passage: " + text
        return self.model.encode(text, normalize_embeddings=True).tolist()

    def embed_batch(self, texts: list[str], is_query=False):
        ...
        ).tolist()


# Singleton (to avoid reloading multiple times)
embedding_model = EmbeddingModel()
```

### schemas.py

Defines the input/output schemas for the Embedding model.

```
from pydantic import BaseModel
from typing import List

class EmbeddingRequest(BaseModel):
    text: str

class BatchEmbeddingRequest(BaseModel):
    texts: List[str]
    is_query: bool = False

class EmbeddingResponse(BaseModel):
    vector: List[float]

class BatchEmbeddingResponse(BaseModel):
    vectors: List[List[float]]
```

### services.py

Defines embedding model request methods.

```
from .model import embedding_model

def embed_query(text: str):
    return embedding_model.embed_query(text)

def embed_passage(text: str):
    return embedding_model.embed_passage(text)

def embed_batch(texts: list[str], is_query: bool = True):
    return embedding_model.embed_batch(texts, is_query)
```

### run.sh

Command to start the project with Uvicorn for faster startup:

```
uvicorn app.main:app --reload
```

## Summary

In local model deployment, whether it's the Embedding model, reranker model, or local LLM model, the project structure remains consistent. This article focuses on explaining the differences in each component.

## Future Work

The next step will cover high concurrency modules, which I will discuss in future articles.