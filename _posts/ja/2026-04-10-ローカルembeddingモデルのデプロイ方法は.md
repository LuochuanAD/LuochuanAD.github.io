---
layout:     post
title: ローカルEmbeddingモデルのデプロイ方法は？
subtitle: AIインフラ層
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

> プライベート化された大規模モデルシステムにおいて、外部依存を避け、データ漏洩リスクを低減し制御性を向上させるために、Embeddingモデルをローカルにデプロイします。また、標準化されたインターフェースサービスを実現し、外部システムがHTTP経由でEmbedding機能をリクエストできるようにし、フォールトトレランス、拡張性、並行処理の対応も備えています。

## 選定

> FastAPI + Uvicorn + EmbeddingModel(e5-large)

### プロジェクト構成（ローカルEmbeddingモデルをFastAPIでラップ）

```
embedding-service/
│
├── app/
│   ├── main.py          # FastAPIエントリーポイント
│   ├── model.py         # モデルの読み込み
│   ├── schemas.py       # 入出力スキーマ定義
│   ├── router.py        # モデル用ルーティング
│   ├── services.py      # リクエスト処理サービス
│   └── config.py        # 設定
│
├── requirements.txt
└── run.sh

```

## 詳細解説

### config.py

主にEmbeddingマイクロサービスの設定項目（embedding_name、deviceなど）

```
MODEL_NAME = "intfloat/e5-large"
DEVICE = "cpu"   # GPUがある場合は "cuda" に変更
```

### router.py

モデルのルーティング設定：/embed、/embed_batch

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

ローカルEmbeddingモデルのAPIエンドポイント。OpenAI APIと同様に「api/v1/embeddings」として設定可能。

```
from fastapi import FastAPI
from .router import router

app = FastAPI(title="E5 Embedding Service")
app.include_router(router, prefix="/api/v1/embeddings")
```

### model.py

Embeddingモデルの実装。

```

from .config import MODEL_NAME, DEVICE
class EmbeddingModel:
    def __init__(self):
        self.model = SentenceTransformer(MODEL_NAME, device=DEVICE)

    def embed_query(self, text: str):
        # ⚠️ E5はプレフィックスが必要
        text = "query: " + text
        return self.model.encode(text, normalize_embeddings=True).tolist()

    def embed_passage(self, text: str):
        text = "passage: " + text
        return self.model.encode(text, normalize_embeddings=True).tolist()

    def embed_batch(self, texts: list[str], is_query=False):
        ...
        ).tolist()


# シングルトン（重複読み込み回避用）
embedding_model = EmbeddingModel()
```

### schemas.py

Embeddingモデルの入力・出力フォーマットの定義

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

Embeddingモデルのリクエスト処理メソッド

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

uvicornを使った高速起動コマンド:

uvicorn app.main:app --reload


## まとめ

ローカルにモデルをデプロイする場合、Embeddingモデル、rerankerモデル、ローカルLLMモデル問わず、プロジェクト構造はほぼ同じです。ここでは、それぞれの構造の違いを中心に説明しました。

## 今後

次回は高並行処理モジュールについて解説予定です。随時、記事をアップしていきます。