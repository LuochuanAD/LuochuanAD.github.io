---
layout:     post
title:      如何部署本地Embedding模型?
subtitle:   AI Infra Layer
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

## 背景

> 在私有化大模型系统中, 为了避免外部依赖, 降低数据泄露的风险并提升可控性, 现将Embedding模型部署本地, 还要实现标准化接口服务,支持外部系统通过HTTP服务请求调用Embedding功能, 具备容错机制, 可扩展性和并发支持.

## 选择 

> FastAPI + Uvicorn + EmbeddingModel(e5-large)

### 项目结构 (本地Embedding模型用FastAPI封装)

```
embedding-service/
│
├── app/
│   ├── main.py          # FastAPI入口
│   ├── model.py         # 模型加载
│   ├── schemas.py       # 输入输出结构
│   ├── router.py        # 模型路由
│   ├── services.py      # 请求服务
│   └── config.py        # 配置
│
├── requirements.txt
└── run.sh

```

## 详解

### config.py

主要是一些embeddin微服务的配置,如: embeddin_name, device等等

```
MODEL_NAME = "intfloat/e5-large"
DEVICE = "cpu"   # 如果你有GPU改成 "cuda"
```
### router.py
设置模型的路由: /embed, /embed_batch

```
outer = APIRouter()

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

是本地embeding模型的借口,可以参照openAI API设置为“api/vi/embedding”

```
from fastapi import FastAPI
from .router import router

app = FastAPI(title="E5 Embedding Service")
app.include_router(router,prefix="api/v1/embeddings")
```

### model.py

Embedding模型的结构.

```

from .config import MODEL_NAME, DEVICE
class EmbeddingModel:
    def __init__(self):
        self.model = SentenceTransformer(MODEL_NAME, device=DEVICE)

    def embed_query(self, text: str):
        # ⚠️ E5 必须加 prefix
        text = "query: " + text
        return self.model.encode(text, normalize_embeddings=True).tolist()

    def embed_passage(self, text: str):
        text = "passage: " + text
        return self.model.encode(text, normalize_embeddings=True).tolist()

    def embed_batch(self, texts: list[str], is_query=False):
        ...
        ).tolist()


# 单例（避免重复加载）
embedding_model = EmbeddingModel()
```

### schemas.py

定义 Embedding模型的输入,输出格式

```
from pydantic import BaseModel
from typing import List

class EmbedRequest(BaseModel):
    text: str

class BatchEmbedRequest(BaseModel):
    texts: List[str]
    is_query: bool = False

class EmbedResponse(BaseModel):
    vector: List[float]

class BatchEmbedResponse(BaseModel):
    vectors: List[List[float]]

```

### servives.py

定义 Embedding模型请求方法

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

用uvicorn加速启动项目命令:

uvicorn app.main:app --reload


## 总结

在本地化部署模型中, 不管是Embedding模型,reranker模型,本地LLM模型. 项目的结构都是一样的.所以这里主要讲解每个结构的不同.

## 未来

下一步是高并发模块的内容, 我会在后续的文章中讲解




