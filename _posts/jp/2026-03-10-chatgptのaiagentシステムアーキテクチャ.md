---
layout:     post
title: ChatGPTのAIAgentシステムアーキテクチャ
subtitle: アーキテクチャ
date:       2026-03-10
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- AIAgent
- Architecture
---

## 背景

> 前回の記事ではChatGPTの限界について説明しました。ChatGPTだけでは対応できない部分を補うために、本記事を書きました。

## 完全なAIエージェントシステムアーキテクチャ

```
                User
                 │
                 ▼
        ┌─────────────────┐
        │   API Gateway   │
        └─────────────────┘
                 │
                 ▼
        ┌─────────────────┐
        │ Agent Controller│
        └─────────────────┘
                 │
   ┌─────────────┼─────────────┐
   ▼             ▼             ▼
*Planner     *Tool Router    *Memory
   │             │             │
   ▼             ▼             ▼
*Workflow    Tool System     Storage
Engine           │               │
   │             │               │
   ▼             ▼               ▼
 *Skills      MCP Tools      Vector DB
                 │
                 ▼
            External APIs
           
```

## モジュール1：Agent Controller（システムの中枢）

役割：

> AIタスクの全工程をコントロールする

責任：

1. ユーザーリクエストを受け取る  
1. Plannerを呼び出す  
1. Toolを実行する  
1. Memoryを更新する  
1. 結果を返す  

**ワークフロー：**

```
User Query
   │
   ▼
Controller
   │
   ▼
Plan → Execute → Observe → Loop

```
これは典型的なAgentループです：
```
while not done:
    think
    act
    observe
```
この考え方は以下に由来します：

ReAct Agent

## モジュール2：Planner（タスクプランナー）

Plannerの役割：

> 複雑なタスクを複数のステップに分解する

例：ユーザーが以下を質問した場合

> 最近のAI企業の直近5件の資金調達状況を分析し、レポートを書いてほしい。

Plannerは以下を生成します：

Plan:
1. AI資金調達ニュースの検索  
1. 企業名の抽出  
1. 資金調達額の取得  
1. データの集計  
1. レポートの生成  

出力：

```
{
 "steps":[
  "search_news",
  "extract_companies",
  "get_funding_data",
  "generate_report"
 ]
}
```
Plannerは以下を用いることができます：

> LLM（大規模言語モデル）

または：

> ステートマシン

## モジュール3：Tool Router（ツールルーター）

役割：

> どのツールを呼び出すかを決定する

例：

ユーザー質問：「東京の天気は？」

Router：

> weather_api

ユーザー質問：「ユーザーの注文を調べて」

Router：

> database_query

Routerの戦略：

方法1  
```
LLM Router

LLMがツールを判定
```
方法2  
```
Embedding Router

クエリのembeddingと
ツールのembeddingの類似度で判断
```
方法3  
```
ルールベースRouter

if "weather"
→ weather tool
```

**商用システムでは**通常：

> LLM + ルールベース

## モジュール4：Tool System（ツールシステム）

> ツールはAIシステムの最重要機能。

ツールの種類：

**1 APIツール**

例：
```
weather API
stock API
crypto API
```
**2 データベースツール**

例：
```
SQLクエリ
ベクトル検索
```
**3 システムツール**

例：
```
メール送信
カレンダー作成
ファイル読み込み
```
**4 計算ツール**

例：
```
Python
コードインタプリタ
```
代表的なToolスキーマ：

```
{
"name": "search_news",
"description": "search latest news",
"parameters": {
 "type":"object",
 "properties":{
   "query":{"type":"string"}
 }
}
}
```
## モジュール5：Skills（スキルシステム）

Skillsは：

> ツールの組み合わせ

例：

Skill：

> Research Report

内部ワークフロー：
```
search
extract
analyze
summarize
```

Skillは実質的に：

> ミニワークフロー

例：

> skill_generate_report()

Skillのメリット：

1. 再利用性の向上  
1. Agentの複雑さを軽減  

## モジュール6：Memory（メモリシステム）

> AIシステムには長期記憶が必須。

Memoryは3種類に分けられる：

**1 ショートタームメモリ**

現在の対話内容。

例：

> 会話履歴

**2 ロングタームメモリ**

> ユーザー情報：ユーザーの嗜好、背景、過去の行動

保存先：

1. Redis  
1. データベース  

**3 セマンティックメモリ**

知識記憶：

> embedding、ベクトルDB

例：

```
ドキュメント

ノート

ナレッジベース
```
よく使われる：

```
Pinecone

Qdrant

Weaviate
```

## モジュール7：RAG（知識検索）

メリット：ハルシネーションの軽減

> 詳細は過去記事参照

## モジュール8：Workflow Engine（ワークフローエンジン）

複雑なタスクでは：

> ワークフローエンジンが必要

例：

```
Step1
Step2
Step3
```
サポート機能：

1. リトライ  
1. タイムアウト  
1. 分岐  
1. 並列処理  

主なOSS：

```
Temporal

Apache Airflow

Prefect
```
## モジュール9：MCP（ツール標準化）

将来のトレンド：

> Model Context Protocol

役割：

> ツールのインターフェース統一

例：
```
filesystem
browser
database
```
すべてのツールがMCPで公開される。

Agentの呼び出し：

> MCPクライアント