---
layout:     post
title: 企業向けAIAgent：ルーター + ReAct + 関数呼び出し
subtitle: アーキテクチャ
date:       2026-02-18
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- Function Tools 
- AI Agent
- ReAct

---

## 背景
> ユーザーのクエリには単純な情報検索とツール呼び出しの両方が含まれるため、汎用かつ再利用可能な成熟したシステムアーキテクチャの設計が必要です。


## アーキテクチャ構造 Architecture
```
User Query
      ↓
1️⃣ Intent Router
      ↓
┌───────────────┬──────────────────┐
│ Simple QA     │ Tool Required     │
│ (LLM only)    │ (ReAct loop)      │
└───────────────┴──────────────────┘
                      ↓
            2️⃣ ReAct Tool Loop
                      ↓
            3️⃣ Structured Tool Layer
                      ↓
            4️⃣ Guardrails & Policies
                      ↓
                 Final Output
```

## 1, 第一層:Intent Router

> Routerはツール呼び出しの必要性を判断します

**Router Prompt:**

```
prompt = ‘

	You are a task classifier.
	User query: {query}
 
	Classify the user query into one of:
 
	1. simple_qa
	2. retrieval
	3. tool_call
	4. multi_step
 
	Return JSON:
	{
  		"type": "..."
	}
’
```
## 2, 第二層:ReAct Tool Core

```
1. simple_qa ==》LLMが直接回答

2. retrieval ==》ベクトルデータベース検索またはブラウザ検索を呼び出し

3. tool_call  ==》ReActループに入る
4. multi_step ==》ReActループに入る

```
### ReActループのアーキテクチャ:

```
LLMが次の行動を決定
↓
ツールをひとつ呼び出す
↓
ツールの結果を追記
↓
繰り返す

```
**サンプルコード:**

```
while True:
 
    response = llm(messages, tools=tool_schema)
 
    if response.tool_calls:
        result = execute_tool(...)
        messages.append(tool_result)
 
    else:
        break

```

## 3, 第三層:Structured Tool Layer

> ここではChatGPTのFunction Callingの考え方を取り入れ、「**Tools Schema**」を定義することでLLMとツール呼び出しを接続します。

企業レベルのAIAgent:ReAct＋Function calling:
[https://strictfrog.com/ja/2026-02-17-%E4%BC%81%E6%A5%AD%E5%90%91%E3%81%91aiagentreact-%E9%96%A2%E6%95%B0%E5%91%BC%E3%81%B3%E5%87%BA%E3%81%97/](https://strictfrog.com/ja/2026-02-17-%E4%BC%81%E6%A5%AD%E5%90%91%E3%81%91aiagentreact-%E9%96%A2%E6%95%B0%E5%91%BC%E3%81%B3%E5%87%BA%E3%81%97/)

## 4, 第四層:Guardrails & Policies

**最大ステップ数の制限:**

```
max_steps = 15
```
**機密メール送信の禁止:**
```
if tool = send_mail:
	require confirmation
```
## 評価:（Router + ReActフレームワーク + Function Calling 推奨度：🌟🌟🌟🌟🌟）
```
メリット:

1, 汎用性    非常に高い
2, 制御性    高い
3, 拡張性    非常に高い
4, 複雑タスク　非常に高い
5, 簡単なタスクの効率 非常に高い

```