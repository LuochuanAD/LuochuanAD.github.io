---
layout:     post
title: ChatAgentのメモリレイヤー設計
subtitle: メモリレイヤー
date:       2026-04-25
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- AIAgent
- Memory

---

## 背景

> 以前いくつかのChatAgentを作る中で、Memory Layerの設計に多くの課題に直面しました。この記事ではMemory Layerの設計方法について解説します。

## 基本概念の解説

> Memory layerは人間の思考に倣い、短期記憶と長期記憶に分かれます。例えば、「10分前に東京のあるカフェに到着し、コーヒーを飲みながらこの記事を書いている」というのが短期記憶です。「私は純粋な中国人で、東京の生活が好きです」というのが長期記憶の“事実”タイプ、「Louisは偉大な企業を作る夢を持っていて、それは10年間続いている」というのは長期記憶の“個人的な夢”タイプです。

**ChatAgentの観点から見ると：**

- 短期記憶：直近3～5ターンの会話内容
- 長期記憶：新しい会話セッションを開始しても、ChatAgentが前回の会話内容を覚えていること。1週間後に再利用しても、ユーザーの個人的な好み（例：ユーモアがある、簡潔、専門的な視点など）を覚えていること。

## 保存方法

> Memory layerを設計する際、どのように保存するかを考慮する必要があります。

- 短期記憶：メモリ／Redis
- 長期記憶：qdrant、PostgreSQL＋pgvector、Pineconeなどの各種ベクトルデータベース

## 保存タイミング

**短期記憶の保存タイミング**

- 1, ユーザーが質問し、LLMが回答したら保存する。
- 2, 会話が10ターンを超えるか最大トークン数の制限に達したら圧縮する。

**長期記憶の保存タイミング**

- 1, ユーザーの質問から長期記憶として保存に値する内容を抽出する。例：
  - user_query: 「簡潔かつユーモラスな口調で話してください」これをLLMが判断しユーザー好みの長期記憶として抽出。
- 2, 会話が10ターンを超えた後、LLMによる圧縮処理を経て、その圧縮内容から長期記憶を抽出。
- 3, その他

## 短期記憶

> ChatAgentは多数のユーザーが利用し、1人のユーザーも複数の会話セッションを作成します。

短期記憶のデータ構造:

```
{
	user_id_xxx:[
			{session_id_xxx1:[{
				"role": "user",
				"content": "hi, 私はLouisです。この文章を読んでくれてうれしいです"
				},{
				"role": "assistant",
				"content": "hi Louis, xxxxxxx"
				},
			...
			]},
			{session_id_xxx2:[{
				"role": "user",
				"content": "こんにちは、今日の東京の天気はどうですか？"
				},{
				"role": "assistant",
				"content": "東京は今日晴れていて、気温は約15度です。良い1日をお過ごしください。xxxx"
				},
			...
			]},
		...
	],
	user_id_yyy:[...],
	...
}
```
> この「role」が"user"と"assistant"であることがわかります。ユーザーの入力とLLMの回答をそれぞれ表します。session_id_xxx1のリストが10ターン以上、またはトークン数が一定の制限を超えた場合、LLMでリストを再要約・抽出し、session_id_xxx1の内容をクリアして、要約された内容をその短期記憶に追加します。

**要約と抽出後の短期記憶データ構造：**

```
{
	user_id_xxx:[
			{session_id_xxx1:[{
				"role": "system",
				"content": "ユーザーLouisはまず自己紹介をし、その後今日の東京の天気を尋ねました。LLMの回答は、東京の天気は快晴で気温は約15度でした。xxxx"
				}
			]}
	],
	user_id_yyy:[...],
	...
}
```
## 長期記憶

> 長期記憶の設計ではマルチテナント、マルチユーザー、記憶の劣化、記憶の境界、共有範囲、制御範囲などを考慮する必要があります。

**長期記憶のデータ構造：**

```
{
  "memory_id": "user_id_xxx_mem_002",
  "tenant_id": "app", 
  "user_id": "user_id_xxx",
  "agent_id": "worker_agent",

  "type": "preference",
  "scope": "private",

  "content": {
    "raw": "簡潔かつユーモラスな口調で話してください。",
    "summary": "User prefers concise and humorous tone in conversations",
    "structured": {
      "tone": ["concise", "humorous"],
      "communication_style": "casual",
      "priority": "high"
    }
  },

  "embedding": {
    "model": "e5-large",
    "vector": [0.123, 0.456, ...]
  },

  "metadata": {
    "importance": 0.92,
    "confidence": 0.96,
    "tags": ["preference", "tone", "style"],
    "source": "explicit_user_input",
    "version": 1
  },

  "time": {
    "created_at": "2026-04-20T10:00:00Z",
    "updated_at": "2026-04-25T08:00:00Z",
    "last_accessed": "2026-04-25T08:00:00Z",
    "decay_score": 0.95
  },

  "relations": [
    {
      "type": "related_to",
      "target_memory_id": "user_id_xxx_mem_001"
    }
  ],

  "access_control": {
    "visibility": "private",
    "allowed_users": [user_id_xxx],
    "allowed_roles": ["admin", "editor"]
  }
}
```
### 長期記憶各フィールドの解説

```
# ユーザーの10～20ターンの会話から抽出した長期記憶、mem_002とラベル付け
"memory_id": "user_id_xxx_mem_002",

# ユーザーが使うiOSアプリ
"tenant_id": "app_iOS", 

# ユーザーID
"user_id": "user_id_xxx",

# ユーザーが利用するChatAgentの名前は“worker_agent”
"agent_id": "worker_agent",

# 長期記憶の種類は「ユーザーの好み」。episodic | semantic | procedural | preference | profile
"type": "preference",

# 長期記憶はプライベート。private | share | public
"scope": "private",
```

```
"content": {

	# ユーザークエリの元の内容
    "raw": "簡潔かつユーモラスな口調で話してください。",

	# LLMによって要約抽出された長期記憶
    "summary": "User prefers concise and humorous tone in conversations",

	# LLMが要約抽出した長期記憶の構造化json
    "structured": {
      "tone": ["concise", "humorous"],
      "communication_style": "casual",
      "priority": "high"
    }
  }
```

```
"embedding": {
	# 長期記憶の埋め込みにMicrosoftの"e5-large"モデルを使用
    "model": "e5-large",
	
	# ベクトル化された長期記憶
    "vector": [0.123, 0.456, ...]
  },
```

```
"metadata": {
	# 長期記憶の重要度
    "importance": 0.92,

	# 長期記憶の信頼度
    "confidence": 0.96,
	
	# 長期記憶のタグ
    "tags": ["preference", "tone", "style"],

	# 長期記憶の出典: explicit_user_input | implicit_inference | system_generated | external_data | corrected_memory      

    "source": "explicit_user_input",

	# 長期記憶のバージョン番号
    "version": 1
  },
```

```
"time": {
	# 長期記憶の作成日時
    "created_at": "2026-04-20T10:00:00Z",
	
	# 長期記憶の更新日時
    "updated_at": "2026-04-25T08:00:00Z",

	# 長期記憶の最終アクセス日時
    "last_accessed": "2026-04-25T08:00:00Z",

	# 長期記憶の劣化スコア0～1。0.95はほぼ消去されず長期間保存に値することを意味
    "decay_score": 0.95
  },
```

```
"relations": [
    {

	# この長期記憶はユーザーの10～20ターンの会話から抽出されており、mem_001は0～10ターンの長期記憶を表す
      "type": "related_to",
      "target_memory_id": "user_id_xxx_mem_001"
    }
  ],
```

```
# アクセス制御
"access_control": {
	# この長期記憶はプライベート
    "visibility": "private",

	# user_id_xxxだけがアクセス可能
    "allowed_users": [user_id_xxx],

	# "admin"、"editor"の役割もアクセス可能
    "allowed_roles": ["admin", "editor"]
  }
```
## 実装のポイント

> コードは公開できないため、実装の要点だけ解説します。

- 1, 非同期スレッドで長期記憶の挿入・読み込みを行い、データベースのクラッシュ防止措置を実装する
- 2, 長期記憶の上書き、マージ、削除
- 3, 長期記憶をデータベースに挿入する前にフィルタリングと重複削除を行う（類似度、重要度、信頼度の観点から）
- 4, 長期記憶を読み出す際には検索結果のソートや再ランキングを行う
- 5, decay_scoreが0.2未満の場合、自動的に長期記憶を削除する