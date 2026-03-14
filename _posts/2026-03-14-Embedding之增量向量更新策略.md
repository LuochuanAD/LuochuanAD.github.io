---
layout:     post
title:      Embedding之增量向量更新策略
subtitle:   避免重复embedding
date:       2026-03-14
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- Embedding

---

## 背景

> 只对“新增或变化的内容”做 embedding，而不是每次重新处理所有文档。

## 一, 增量更新的三个层级

| 层级     | 粒度        | 推荐程度  |
| ------ | --------- | ----- |
| 文件级    | PDF       | ⭐⭐⭐⭐  |
| 页面级    | page      | ⭐⭐⭐   |
| chunk级 | paragraph | ⭐⭐⭐⭐⭐ |

推荐流程:

```
私有API
 ↓
获取文件
 ↓
PDF hash
 ↓
是否新文件
 ├── 否 → 跳过
 └── 是
      ↓
text extraction
 ↓
chunk
 ↓
chunk hash
 ↓
是否存在
 ├── 是 → 跳过
 └── 否
      ↓
embedding
 ↓
vector db
```

### 方案一: 文件hash

```
import hashlib

def file_hash(file_bytes):
    return hashlib.md5(file_bytes).hexdigest()

```
数据库存储:
```
file_hash
file_name
processed_at
```

### 方案二: 拆页hash

> PDF每页拆开, 每页hash

数据库存储:

```
file_id
page_number
page_hash
```

### 方案三: Chunk Hash（企业级）

```
def chunk_hash(text):
    return hashlib.sha1(text.encode()).hexdigest()

```
数据库存储:

```
chunk_id
chunk_hash
vector
metadata
```

### 二, 向量数据库的Metadata设计(方案一和方案三)

**推荐的metadata:**

```
{
  file_id: "pdf123",
  file_hash: "...",
  chunk_hash: "...",
  page: 3,
  source: "Louis_pdf"
}
```
**优点:**

- 删除某个文件
- 更新某个文件
- 过滤来源

## 去重策略 (避免重复embedding)

### 方案一: 去重段落

很多PDF有:

```
免责声明
页脚
公司介绍
```
### 方案二: 语义去重

> 两个chunk 相似度 > 0.95

### 方案三: 文本近似去重

> 使用：datasketch

> 适合：网页, 邮件, FAQ

## 三, Embedding Cache（非常有效）

建立 text_hash → embedding

```
chunk
 ↓
hash
 ↓
查 cache
```
如果存在cache,直接使用 embedding


