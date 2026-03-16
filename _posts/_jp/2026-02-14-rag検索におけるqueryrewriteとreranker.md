---
layout:     post
title: RAG検索におけるQueryReWriteとReranker
subtitle: クエリの書き換え
date:       2026-02-14
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- RAG
- QueryReWriting
- Reranker

---

## 背景

> RAGシステムを設計する中で、Structural ChunksとStructural Promptを設計した後、検索結果は理想的な精度に達しました。しかし、さらに精度を向上させるには、Query RewriteとRerankerによる二段階検索が必要でした。

## Query Rewrite

### 方法1: 同義語拡張

**例:**

```
ユーザークエリ: python

書き換え: 
python言語
python開発
pythonバージョン
```
評価: 速い、コスト低、カバー範囲が限定的

### 方法2: LLM Rewrite（推奨）

> 一般的にはLLMを使って3つのサブクエリを生成し、それぞれで検索した結果を統合します。

**注意: ここでは説明のため中国語でpromptを書いていますが、実際は英語が望ましいです**

```
prompt = “ユーザーの検索クエリを複数の検索クエリに書き換え、3つの異なる検索クエリを返す“
```
プロセス:
```
User Query
    ↓
LLM Rewrite
    ↓
Query 1
Query 2
Query 3
    ↓
Vector Search
    ↓
結果マージ
```
### 方法3: HyDE（慎重に）

> LLMの知識を使ってユーザークエリから予想回答を生成し、その予想回答を元にベクトル検索を行います。

プロセス:

```
User Query
    ↓
LLM 予想回答生成
    ↓
Embedding
    ↓
Vector Search
```
**注意: LLM知識とRAGの知識には差が大きく、LLM自体の知識期限の制約もあるため、予想回答が誤っていたり古くなっている可能性があります。HyDEの使用は慎重に。**

### 方法4: 多言語翻訳（推奨）

> RAGナレッジベースが英語と日本語で構築されており、ユーザークエリが中国語の場合はどうするか？

**例:**

```
ユーザークエリ: 出力ログの保持期間はどのくらいですか？

まず英語と日本語に翻訳：

What is the retention period for the output logs?

出力ログの保存期間はどれくらいですか？

この3つのクエリでベクトル検索を行います。
```
プロセス:
```
User Query
    ↓
LLM 翻訳
    ↓
Query 1 (中国語)
Query 2 (英語)
Query 3 (日本語)
    ↓
Vector Search
    ↓
結果マージ
```
## Rerankerによる再順位付け

- “Vector Search”のベクトル検索はあくまで一次検索で、再現率（recall）を高めるためTopKは通常20〜50
- Rerankerが絞り込みで精度（precision）を高め、TopKは通常3〜5

### 方法1: LLMベースのReranker

> LLMを使って一次検索で得られた文書をスコア付けまたは選択

```
prompt = “
	Given the query and the following documents, rank with which document is most relevant.
	Query: {query}
	Documents:
	{chunk_1}
	{chunk_2}
	{chunk_3}
	...
	Return the IDs of the most relevant documents.

”
```
評価: トークン消費が多く遅い

### 方法2: 類似度微調整モデルベース（推奨）

> 一次検索で得られたChunkとユーザークエリをモデルに直接入力し、関連度スコアを算出

```
英語:
ms-marco-MiniLM-L-6-v2
ms-marco-MiniLM-L-12-v2
bge-reranker-base

日本語:
japanese-reranker-cross-encoder-xsmall-v1
japanese-reranker-cross-encoder-base-v1
japanese-reranker-cross-encoder-large-v1

中国語:
bge-reranker-large
bge-reranker-base
m3e-reranker

多言語Cross-Encoderモデル:

BAAI/bge-reranker-v2-m3
Alibaba-NLP/gte-multilingual-reranker-base
cross-encoder/ms-marco-MiniLM-L6-v2
```
評価: 精度が高い