---
layout:     post
title: Workflow+limitedPlanningの設計思想
subtitle: ワークフロー＋限定プランニング
date:       2026-03-07
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- workflows
- structured


---

## 背景

> 実際の AI エージェントシステムでは、多くのチームが妥協案として次のアーキテクチャを採用しています：
Workflow + Limited Planning、つまり LLM を使って軽量な計画を行い、その後にあらかじめ定義されたワークフローを実行します。

```
実世界の多くの AI プロダクトがこのアーキテクチャを使っています
多くのシステムは基本的に Workflow + Limited Planning です。

例：

LangGraph（ステートマシンワークフロー）
Notion AI
Perplexity AI

これらのシステムは：

完全な自律型エージェントではなく
構造化されたワークフローです
```

## Workflow + Limited Planning の設計コンセプト

```
User Input
      ↓
Intent Detection  ユーザーの意図解析
      ↓
Task Planner
      ↓
Workflow Selection
      ↓
Workflow Execution
      ↓
Result
```

## 具体的な実行フロー（完全な例）

ユーザーが入力したと仮定：

> AI業界のレポートを書いてほしい

システムの流れ：

**Step 1 Intent Detection**

LLM がまずユーザーの意図を判断：

> intent = research_report


例えば出力：

```
{
 "intent": "research_report"
}
```

**Step 2 Planner（軽量な計画）**

Planner が：

> どの種類のワークフローが必要かを判断


例：

> research_workflow

**Step 3 Workflow Selection**

システムはあらかじめ定義されたプロセスを選択：

> Research Workflow

構造：

```
Search Data
↓
Extract Key Info
↓
Summarize
↓
Write Report
```

**Step 4 Workflow Execution**

各ステップで LLM やツールを呼び出す：

```
Search → Web API
Extract → LLM
Summarize → LLM
Write → LLM
```
## Limited Planning の実装パターン

異なるシステムはそれぞれ異なる実装を行いますが、主に3つのパターンがあります。

**パターン1 Router Agent（最も一般的）**

Planner はただ：

> どのエージェントを選択するかだけを担当

構造：
```
User Input
      ↓
Router
  ├ Research Agent
  ├ Coding Agent
  └ Writing Agent
```
例えば：

ユーザー入力：

> Pythonでクローラーを書いて

Router：

> coding_agent


**パターン2 ツール選択**

Planner はツールだけを選択：

```
Search
Database
Code Interpreter
```

例：

> User: AI企業の資金調達を調べて

Planner：

> use web_search

**パターン3 ワークフローのルーティング**

Planner はワークフローを選択：

```
content_workflow
analysis_workflow
coding_workflow
```

その後、決まったプロセスを実行。

## コードレベルの実装例（簡略版）

典型的な Python 構造：

```
def router(user_input):

    intent = llm_intent_detection(user_input)

    if intent == "research":
        return research_workflow(user_input)

    elif intent == "coding":
        return coding_workflow(user_input)

    elif intent == "summary":
        return summary_workflow(user_input)
```

workflow：
```
def research_workflow(query):

    data = search(query)

    info = extract(data)

    summary = summarize(info)

    report = write_report(summary)

    return report
```
ここでの：

> planning = router

完全なエージェントではありません。

## 参考

ChatGPT