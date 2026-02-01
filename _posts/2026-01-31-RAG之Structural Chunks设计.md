---
layout:     post
title:      RAG之Structural Chunks设计
subtitle:   结构化切割
date:       2026-01-31
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- RAG 
- AI
- Chunks

---

## 前言

>针对RAG系统的构建，要想提高检索和生成结果的准确性，核心就是结构化Chunks的设计。

### 一, 常规的chunks切割方法 

#### 1，固定大小chunk分割 （Fixed-size Chunking）

```
from langchain.text_splitter import CharacterTextSplitter

text = "这是一个用LangChain测试固定大小分块的示例文本。通过设定chunk_size和chunk_overlap，可以很好地控制块的大小和内容。"
text_splitter = CharacterTextSplitter(
    separator="",      # 不按特定字符分割
    chunk_size=512,      # 每个块的字符数
    chunk_overlap=3,    # 重叠字符数
    length_function=len,
)
chunks = text_splitter.split_text(text)
print(chunks)

```

#### 2，按照句子，段落，某个标点符号等等分割。

```
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_community.document_loaders import TextLoader

loader = TextLoader("../../data/xxx.txt")
docs = loader.load()

text_splitter = RecursiveCharacterTextSplitter(
    separators=["\n\n", "\n", "。", "，", " ", ""],  # 分隔符优先级
    chunk_size=200,
    chunk_overlap=10,
)
chunks = text_splitter.split_text(docs)

```

#### 3，按照语义分割。
[https://zhuanlan.zhihu.com/p/1924496550433919103](https://zhuanlan.zhihu.com/p/1924496550433919103)

### 二, 结构化chunks切割方法。(推荐)

#### 1，针对已经是结构化的文档

例如: 标题,绪论,第一章,第二章,结论等等这样的标签直接切割.

[https://zhuanlan.zhihu.com/p/1987591795891269696](https://zhuanlan.zhihu.com/p/1987591795891269696)

#### 2, 针对非结构化、半结构化文档.

为了提高检索和生成结果的准确性,我的做法就是**将所有非结构化、半结构化文档都要转化为结构化chunks**. 那么针对不同结构的文档,需要设计不同的结构化chunks. 

例如: 我想将几百份简历分割,向量化存入向量数据库中.这几百份的简历使用了不同的模版. 如果使用常规的chunks切割方法,那么检索和生成的准确性都比较差. 

我的做法:

> 先思考一下: 从一个人类的角度,面对不同模版的简历,都能很轻松的判断出这块是个人信息,那块是项目经验,这块是个人的资格证书等等.  那么如何通过代码来判断呢?

第一步: 物理段落切分.(目的是不谈语义,只切割为一段落)

```
blocks = re.split(r'\n\s*\n',text)
```


第二步:段落按照类型判断分类.

分为:
基本信息(姓名,性别,出生地,学历...等等); 
skills(python,C++,...等等);
自我评价;
资格证书; 
项目经验(一个项目经验归位一个chunk).

所有简历的结构都将如下所示:
```
resume_chunks = [
	basic_chunk,
	skills_chunk,
	introduction_chunk
	certification_chunk,
	project_chunk_1,
	project_chunk_2,
	project_chunk_3,
......
	other
]
```

(1)使用**关键词密度算法**来判断.

```
type_keywords = {
	"basic_chunk" = ["姓名",“年龄”,“年纪”,”出生年月“,“大学”,“学院”...],
	“skills_chunk” = [“python”,“C++”,“iOS”,“And”,“Java”,“PHP”...],
	“introduction_chunk” = [“爱学习”,“抗压”,“高效率”,“敏捷开发”,...],
	“certification_chunk” = [“计算机二级”,“大学英语四级”,“大学英语六级”,“雅思”,“AWS”,...],
	“project_chunk” = [“项目经验”,“担当”,“时期”,...]
}

```
score = 命中关键词数 / 段落长度

(2)**时间模式密度**来判断
```
(19|20)\d{2}年\s*\d{1,2}月
```
出现>=2次 是项目或者职务经历

第三步: 相邻段落语义合并(Soft Merge)

> 解决一个项目被拆成3段问题

```
	cos_sim(block[i], block[i+1]) > 0.85
```


这三步做完后,就得到了结构化后的chunk,再次清洗数据后, 就可以将**语义完好的段落**向量化存入向量数据库了.

 **优点**:最大程度保持语义的连贯性,并且不使用LLM,降低了成本;提升检索和生成的准确性10倍.


project_chunk_1的json样例:
```
chunk = {
	"chunk_id": "resume_Louis_project_01",
	"resume_id": "resume_Louis",
	"section_type": "project",
	"title": "情感陪伴AI Agent",
	"period":{
		"from": "2026-01",
		"to": "2026-02"
	},
	"role": ["设计", “开发”, “测试”],
	"skills": ["python", "js"],
	"content": "项目内容:开发一个情感陪伴AI Agent,服务于广大程序员,...",
	“source”:{
		"file": "简历(Louis).pdf",
		"page": [2,3]
	}
}
```


## 参考

[https://developer.jdcloud.com/article/4408](https://developer.jdcloud.com/article/4408)









