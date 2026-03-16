---
layout:     post
title: Incremental Vector Update Strategy for Embedding
subtitle: Avoid Duplicate Embeddings
date:       2026-03-14
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- Embedding

---

## Background

> Only embed "new or changed content" instead of reprocessing all documents each time.

## 1. Three Levels of Incremental Updates

| Level    | Granularity | Recommendation |
| -------- | ----------- | -------------- |
| File-level | PDF         | ⭐⭐⭐⭐           |
| Page-level | page        | ⭐⭐⭐            |
| Chunk-level | paragraph   | ⭐⭐⭐⭐⭐          |

Recommended workflow:

```
Private API
 ↓
Fetch file
 ↓
PDF hash
 ↓
Is it a new file?
 ├── No → Skip
 └── Yes
      ↓
text extraction
 ↓
chunk
 ↓
chunk hash
 ↓
Exist?
 ├── Yes → Skip
 └── No
      ↓
embedding
 ↓
vector db
```

### Approach 1: File Hash

```
import hashlib

def file_hash(file_bytes):
    return hashlib.md5(file_bytes).hexdigest()

```
Database storage:
```
file_hash
file_name
processed_at
```

### Approach 2: Page Hash

> Split PDF by page, hash every page

Database storage:

```
file_id
page_number
page_hash
```

### Approach 3: Chunk Hash (Enterprise-level)

```
def chunk_hash(text):
    return hashlib.sha1(text.encode()).hexdigest()

```
Database storage:

```
chunk_id
chunk_hash
vector
metadata
```

### Vector Database Metadata Design (Approaches 1 and 3)

**Recommended metadata:**

```
{
  file_id: "pdf123",
  file_hash: "...",
  chunk_hash: "...",
  page: 3,
  source: "Louis_pdf"
}
```
**Advantages:**

- Delete specific files
- Update specific files
- Filter by source

## 2. Deduplication Strategies (Avoid Duplicate Embeddings)

### Approach 1: Deduplicate Paragraphs

Many PDFs contain:

```
Disclaimer
Footer
Company introduction
```
### Approach 2: Semantic Deduplication

> Two chunks similarity > 0.95

### Approach 3: Text Approximate Deduplication

> Using: datasketch

> Suitable for: web pages, emails, FAQs

## 3. Embedding Cache (Highly Effective)

Build a text_hash → embedding mapping

```
chunk
 ↓
hash
 ↓
check cache
```
If cache exists, use embedding directly