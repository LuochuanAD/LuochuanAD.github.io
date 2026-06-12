---
layout:     post
title: FastAPI+Uvicorn+Gradio
subtitle: 高速で高品質なWebインターフェースの構築
date:       2026-04-04
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- FastAPI
- unicorn
- gradio

---

## 背景

> 以前作ったAIAgentのWebページはすべてHtml+Css+Jsで構築していて、一言でいうと：ダサかった。

## 選択

> FastAPI + Gradioで見栄えの良いUIを素早く構築

**以下はRAGの可視化UIです:**

![](https://raw.githubusercontent.com/LuochuanAD/BlogSourceImage/refs/heads/master/BlogSourceImage/BlogSourceImage2026/gradio_ui.png)

**コードは非常にシンプルで、以下の通りです:**

[https://github.com/LuochuanAD/Fine-tuning-Learn/tree/main/FastAPI%2BUvicorn%2BGradio_Demo](https://github.com/LuochuanAD/Fine-tuning-Learn/tree/main/FastAPI%2BUvicorn%2BGradio_Demo)

**以下はVoiceの可視化UIです:**

![](https://raw.githubusercontent.com/LuochuanAD/BlogSourceImage/refs/heads/master/BlogSourceImage/BlogSourceImage2026/voice_demo.PNG)

## トレードオフ

**Gradioを使うケース**

- AIデモの作成
- 社内ツールの開発
- アイデアの迅速な検証（MVP）
- Agentのデバッグ画面
- RAGの可視化

## 制約 (Gradioを使うケース)

1. UIのカスタマイズ性能に限界がある
2. フロントエンドの複雑なインタラクションには不向き
3. 正式なToC製品には適さない

## 将来展望

> 将来的にはNext.js + FastAPIを使って**ToC**向けのAIAgentのフロント・バックエンドを構築する予定です