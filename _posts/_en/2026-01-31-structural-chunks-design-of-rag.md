---
layout:     post
title: Structural Chunks Design of RAG
subtitle: Structured Cutting
date:       2026-01-31
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- RAG 
- AI
- Chunks

---

## Introduction

> To improve the accuracy of retrieval and generation results in a RAG system, the core lies in the design of structured chunks.

### 1. Conventional Chunking Methods

#### 1. Fixed-size Chunking

```
from langchain.text_splitter import CharacterTextSplitter

text = "This is a sample text for testing fixed-size chunking with LangChain. By setting chunk_size and chunk_overlap, you can effectively control the size and content of the chunks."
text_splitter = CharacterTextSplitter(
    separator="",      # No specific character split
    chunk_size=512,    # Number of characters per chunk
    chunk_overlap=3,   # Number of overlapping characters
    length_function=len,
)
chunks = text_splitter.split_text(text)
print(chunks)

```

#### 2. Chunking by Sentences, Paragraphs, or Specific Punctuation

```
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_community.document_loaders import TextLoader

loader = TextLoader("../../data/xxx.txt")
docs = loader.load()

text_splitter = RecursiveCharacterTextSplitter(
    separators=["\n\n", "\n", "。", "，", " ", ""],  # Separator priorities
    chunk_size=200,
    chunk_overlap=10,
)
chunks = text_splitter.split_text(docs)

```

#### 3. Semantic-based Chunking
[https://zhuanlan.zhihu.com/p/1924496550433919103](https://zhuanlan.zhihu.com/p/1924496550433919103)

### 2. Structured Chunking Methods (Recommended)

#### 1. For Already Structured Documents

For example: directly chunking by tags such as headings, introduction, chapter one, chapter two, conclusion, etc.

[https://zhuanlan.zhihu.com/p/1987591795891269696](https://zhuanlan.zhihu.com/p/1987591795891269696)

#### 2. For Unstructured or Semi-structured Documents

To improve the accuracy of retrieval and generation, my approach is to **convert all unstructured and semi-structured documents into structured chunks**. Different document structures require designing different structured chunks.

For example: Suppose I want to chunk and vectorize hundreds of resumes into a vector database. These resumes use different templates. Using conventional chunking methods leads to poor accuracy in retrieval and generation.

My approach:

> First, from a human perspective: when facing resumes with different templates, it's easy to identify which part contains personal information, which part includes project experience, and which part lists certifications. So how can this be judged programmatically?

Step 1: Physical paragraph splitting (the goal is not semantic parsing, just splitting into paragraphs)

```
blocks = re.split(r'\n\s*\n',text)
```

Step 2: Classify paragraphs by type.

Categories:
Basic information (name, gender, birthplace, education, etc.);
Skills (python, C++, etc.);
Self-evaluation;
Certifications;
Project experience (one chunk per project experience).

All resumes will have a structure like this:
```
resume_chunks = [
	basic_chunk,
	skills_chunk,
	introduction_chunk,
	certification_chunk,
	project_chunk_1,
	project_chunk_2,
	project_chunk_3,
	......
	other
]
```

(1) Use **keyword density algorithm** for classification.

```
type_keywords = {
	"basic_chunk": ["姓名","年龄","年纪","出生年月","大学","学院",...],
	"skills_chunk": ["python","C++","iOS","Android","Java","PHP",...],
	"introduction_chunk": ["love learning","stress resistance","high efficiency","agile development",...],
	"certification_chunk": ["computer level 2","CET-4","CET-6","IELTS","AWS",...],
	"project_chunk": ["project experience","role","period",...]
}

```
score = number of matched keywords / paragraph length

(2) Use **date pattern density** for identification
```
(19|20)\d{2}年\s*\d{1,2}月
```
If appears >= 2 times, the paragraph likely describes a project or job experience.

Step 3: Semantic merging of adjacent paragraphs (Soft Merge)

> Solves the problem of a single project split into 3 chunks

```
	cos_sim(block[i], block[i+1]) > 0.85
```

After these three steps, you get structured chunks. After cleaning the data, **semantically coherent paragraphs** can be vectorized and stored in the vector database.

**Advantages**: Maximizes semantic continuity without using LLMs, reducing costs; improves retrieval and generation accuracy by 10x.

Example JSON for project_chunk_1:
```
chunk = {
	"chunk_id": "resume_Louis_project_01",
	"resume_id": "resume_Louis",
	"section_type": "project",
	"title": "Emotional Companion AI Agent",
	"period":{
		"from": "2026-01",
		"to": "2026-02"
	},
	"role": ["design", "development", "testing"],
	"skills": ["python", "js"],
	"content": "Project details: Developing an emotional companion AI agent to serve the programming community,...",
	"source":{
		"file": "Resume(Louis).pdf",
		"page": [2,3]
	}
}
```

## References

[https://developer.jdcloud.com/article/4408](https://developer.jdcloud.com/article/4408)