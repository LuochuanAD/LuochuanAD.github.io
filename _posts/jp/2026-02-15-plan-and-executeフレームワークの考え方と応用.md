---
layout:     post
title: Plan-and-Executeフレームワークの考え方と応用
subtitle: 計画と解決
date:       2026-02-15
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- Prompt 
- AI

---

## 背景

> LangChainのPlan-and-Executeフレームワークは、Plan-and-Solveに関する論文に触発されています。Plan-and-Executeは、複雑な長期計画に非常に適しており、複雑な問題を複数のサブタスクに分解して一つずつ解決します。これにより大規模モデルの呼び出しは頻繁に行われますが、ReActエージェントのループ中に発生するプロンプトの長さ超過問題を回避できます。

## Plan-and-Solve戦略

![](https://raw.githubusercontent.com/LuochuanAD/BlogSourceImage/refs/heads/master/BlogSourceImage/BlogSourceImage2026/COT%E5%92%8CPlan-and-Solve%E5%AF%B9%E6%AF%94.png)

上図は、サンプルなしChain-of-Thought（COT）とPlan-and-Solveの比較です。Plan-and-Solveは本質的にCOTを具体的な小タスクに細分化し、それらのタスクを一つずつ処理していきます。Plan-and-ExecuteフレームワークはこのPlan-and-Solveの考え方と同様ですが、実行面ではよりAIの実務的な操作に寄っています。

## Plan-and-Executeフレームワークの適用例

> Question ReWriting設計や複数のエージェント調整作業などに適しています。

シナリオ: 完全なRAGシステムを構築したAIエージェントシステム内

**例:**

**Question**: Louisと刘亦菲の個人情報を検索し、それらの個人情報に基づいて結婚招待状を作成し、その内容をworldXXTest@gmail.comに送信してください。

Plan-and-ExecuteはタスクをPlanとExecuteに分割します。

LLMを使い、Questionを約5つの**Plan**に分解します:
```
|-1- データベースでLouisの個人情報を検索する
|-2- ウェブで刘亦菲の個人情報を検索する
|-3- ウェブで結婚招待状のテンプレートを検索する
|-4- Louisと刘亦菲の結婚招待状の内容を生成する
|-5- 内容をworldXXTest@gmail.comに送信する
```
ReActフレームワークのLLMを使い、この5つのPlanを順に**Execute**します:
```
|-1- ”Louisの個人情報：iOSおよびAIエンジニア、男性、性格良好、温和、親切、ユーモアがある...“
|-2- ToolツールBrowserを呼び出し、得た情報：”刘亦菲の個人情報：茜茜、女性、映画・ドラマスター、美貌と実力を兼ね備え、自由を愛する...“
|-3- 結婚招待状のテンプレート：”私たちは結婚します。xxx様へ、ご招待申し上げます。2026年xx月xx日に...にて開催します...“
|-4- Louisと刘亦菲の結婚招待状の内容：”私たちは結婚します。xxx様へ、2026年2月31日に月の北極宮殿で太陽暦の大結婚式を執り行います。当日は星間船LL001号が迎えに参ります。2月30日の夜12時に上海の東方明珠タワー屋上で乗船してください。“
|-5- 工具sendMailを使い、結婚招待状の内容を送信する
```
**まとめ: LLMを1回呼び出して5つのPlanを生成し、ReActエージェントを5回呼び出して各Planを処理。**

#### Plannerプロンプト:
```
prompt = ‘
	You are a task planner in a multi-step AI system.
	User query: {Query}
	Your job:
	1, Break down the user query into executable steps.
	2, Each step must be atomic and tool-executable.
	3, Steps should be ordered logically.
	4, Do NOT execute anything.
	5, Only return JSON.
’
```

**例:**

Query: 英語6級の資格を持ち、Python開発経験が3年ある人物を探し、その人物の個人情報をluochuanad@gmail.comに送信したい。

LLM（**Planner Prompt**）を使って、以下の結果を得ました:

```
json
[
 {
	"step: 1,
	"action": "Search Database"
	"description": "英語CET-6資格および3年間のPython開発経験を持つ人物をデータベースで検索する。",
	"parameters": {
		"certificate": "CET-6",
		"experience": {
			"Language": "Python",
			"years": 3
		}
	}
 },
 {
	"step": 2,
	"action": "Retrieve Personal Information",
	"description": "前ステップで見つかった人物の個人情報を抽出する。",
	"parameters": {
		"fields": ["name", "email", "phone", "address"]
	}
 },
 {
	"step": 3,
	"action": "Format Email",
	"description": "見つかった人物の個人情報を含むメールを準備する。",
	"parameters": {
		"recipient": "luochuanad@gmail.com",
		"subject": "Candidate Information",
		"body": "ステップ2で取得した個人情報を含める。"
 	}
 },
 {
 	"step": 4,
	"action": "Send Email"
	"description": "整形済みメールを指定のメールアドレスに送信する。",
	"parameters": {
		"recipient": "xxx@gmail.com",
		"content": "ステップ3で準備したメール内容を利用する。"
 	}
 }
]
```

## 参考

Lei Wangらの論文「Plan-and-Solve Prompting: Improving Zero-Shot Chain-of-Thought Reasoning by Large Language Models」[https://aclanthology.org/2023.acl-long.147.pdf](https://aclanthology.org/2023.acl-long.147.pdf)