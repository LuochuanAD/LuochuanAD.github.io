---
layout:     post
title:      ChatAgent之Memory Layer设计
subtitle:   Memory Layer
date:       2026-04-25
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- AIAgent
- Memory

---

## 背景

> 在之前做过的几个ChatAgent中,我在设计Memory Layer时,遇到了很多问题, 通过这篇文章我将讲解如何设计Memory Layer.

## 基础概念讲解

> Memory layer仿照人的思维方式,分为短期记忆和长期记忆. 比如说:”我10分钟前刚到东京的某家咖啡馆,刚刚才坐下来边喝咖啡边在写这篇文章.“ 这就是短期记忆; “我是一个纯中国人,喜欢东京这边的生活.” 这是长期记忆中“事实”类型; “Louis梦想创建一家伟大的企业,这一个梦想持续了10年” 这是长期记忆中的“个人梦想”类型.

**那么从ChatAgent的角度来看:**

- 短期记忆: 前3-5轮的会话内容.
- 长期记忆: 当我重新建立一个新的会话界面时,ChatAgent仍然能记得我上个会话界面的内容. 当我一周后再使用ChatAgent时,它仍然记得我的个人偏好(如: 从幽默,简洁,专业的角度和我聊天).


## 存储方式

> 要设计memory layer,那么需要考虑通过什么方式来存储. 

- 短期记忆: 内存/ Redis.
- 长期记忆: 各种向量数据库如: qdrant,PostgreSQL＋pgvector,Pinecone等等.

## 存储的时机

**短期记忆的存储时机**

- 1, 用户提问了并且LLM回答了,那就存储. 
- 2, 会话超过10轮或者一定限制的最大tokens时,就压缩.

**长期记忆的存储时机**

- 1, 从用户的提问中提取值得存为长期记忆记忆的内容. 如:
- user_query:“请用简洁,幽默的语气来和我聊天.”通过LLM判断提取为用户偏好的长期记忆.
- 2, 用户的会话超过10轮后,经过LLM压缩后.这时可以对这个压缩的内容进行长期记忆提取.
- 3, 等等


## 短期记忆

> ChatAgent是要给大量用户使用,并且用户在使用这个ChatAgent时也会创建多个会话.


短期记忆的数据结构:

```
{
	user_id_xxx:[
			{session_id_xxx1:[{
				"role": "user",
				"content": "hi, 我是Louis.很高兴你能看到这篇文章"
				},{
				"role": "assistant",
				"content": "hi Louis, xxxxxxx"
				},
			...
			]},
			{session_id_xxx2:[{
				"role": "user",
				"content": "你好,今天的东京天气怎么样?"
				},{
				"role": "assistant",
				"content": "东京今天的天气阳光明媚,气温15度左右,祝你度过美好的今天.xxxx"
				},
			...
			]},
		...
	],
	user_id_yyy:[...],
	...
}
```
> 很明显看到这个“role”为"user"和“assistant”,分别代表用户的输入和LLM的回答.当session_id_xxx1里面的列表超过10轮,或者列表的tokens超过了一定的限制, 这时我们需要将这个列表通过LLM重新总结和提取,同时清空session_id_xxx1里面的内容,并且将总结和提取后的内容加入到session_id_xxx1这个短期记忆中.


**总结和提取后的短期记忆数据结构:**

```
{
	user_id_xxx:[
			{session_id_xxx1:[{
				"role": "system",
				"content": "用户Louis先是自我介绍了,然后询问了东京今天的天气情况,LLM回复为:东京今天的天气阳光明媚,气温15度左右.xxxx"
				}
			]}
	],
	user_id_yyy:[...],
	...
}
```
## 长期记忆

> 长期记忆的设计需要考虑多租户,多用户,记忆的衰减,记忆的边界,传播的范围,控制范围等等.

**长期记忆的数据结构:**

```
{
  "memory_id": "user_id_xxx_mem_002",
  "tenant_id": "app", 
  "user_id": "user_id_xxx",
  "agent_id": "worker_agent",

  "type": "preference",
  "scope": "private",

  "content": {
    "raw": "请用简洁,幽默的语气来和我聊天。",
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
### 长期记忆各个字段解读

```
# 用户的10-20轮会话提取出来的长期记忆,标记为mem_002
"memory_id": "user_id_xxx_mem_002",

# 用户用的是iOS app
"tenant_id": "app_iOS", 

# 用户的ID
"user_id": "user_id_xxx",

# 用户用的ChatAgent名字是“worker_agent”
"agent_id": "worker_agent",

# 长期记忆的类型是 用户偏好. episodic | semantic | procedural | preference | profile
"type": "preference",

# 长期记忆是私有的. private | share | public
"scope": "private",

```

```
"content": {

	# 用户query的原始内容
    "raw": "请用简洁,幽默的语气来和我聊天。",

	# LLM总结提取后的长期记忆
    "summary": "User prefers concise and humorous tone in conversations",

	# LLM总结提取后的长期记忆的结构化json
    "structured": {
      "tone": ["concise", "humorous"],
      "communication_style": "casual",
      "priority": "high"
    }
  }

```

```
"embedding": {
	# 长期记忆用了微软的"e5-large"模型来embedding.
    "model": "e5-large",
	
	# 向量化后的长期记忆
    "vector": [0.123, 0.456, ...]
  },
```
```
"metadata": {
	# 这条长期记忆的重要程度
    "importance": 0.92,

	# 这条长期记忆的可信度
    "confidence": 0.96,
	
	# 这条长期记忆的标签
    "tags": ["preference", "tone", "style"],

	# 这条长期记忆的来源: explicit_user_input | implicit_inference | system_generated | external_data | corrected_memory      

    "source": "explicit_user_input",

	# 这条长期记忆的版本号
    "version": 1
  },
```

```
"time": {
	# 这条长期记忆的创建日期
    "created_at": "2026-04-20T10:00:00Z",
	
	# 这条长期记忆的更新日期
    "updated_at": "2026-04-25T08:00:00Z",

	# 这条长期记忆的最新使用时间
    "last_accessed": "2026-04-25T08:00:00Z",

	# 这条长期记忆的衰减分数0-1. 0.95代表着这条长期记忆值得长时间保存,几乎不会被删除.
    "decay_score": 0.95
  },
```

```
"relations": [
    {

	# 这条长期记忆是用户的10-20轮会话提取出来的, 上一条记忆mem_001,代表用户0-10轮的会话的长期记忆.
      "type": "related_to",
      "target_memory_id": "user_id_xxx_mem_001"
    }
  ],
```

```
# 访问控制
"access_control": {
	# 这条长期记忆是私有的 
    "visibility": "private",

	# 这条长期记忆只允许user_id_xxx访问
    "allowed_users": [user_id_xxx],

	# 这条长期记忆允许"admin", "editor"访问
    "allowed_roles": ["admin", "editor"]
  }

```
## 代码实现要点

> 因为我不能将代码公开,所以我只讲解代码实现中的要点

- 1, 通过异步线程来调用长期记忆的插入和读取,并且要有防止数据库崩溃的机制
- 2, 长期记忆的覆盖,合并, 删除
- 3, 长期记忆在插入到数据库前,要进行过滤去重(相似度,重要程度,可信度).
- 4, 长期记忆在读取时检索排序或者rerank排序
- 5, 当decay_score < 0.2时自动删除长期记忆.













