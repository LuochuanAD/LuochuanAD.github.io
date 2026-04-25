---
layout:     post
title: Design of the Memory Layer in ChatAgent
subtitle: Memory Layer
date:       2026-04-25
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- AIAgent
- Memory

---

## Background

> In several previous ChatAgent projects, I encountered many challenges when designing the Memory Layer. In this article, I will explain how to design the Memory Layer effectively.

## Core Concepts

> The Memory Layer mimics human cognition by dividing memory into short-term and long-term categories. For example: "I just arrived at a cafe in Tokyo 10 minutes ago and am currently sitting down, sipping coffee while writing this article." This represents short-term memory; "I am fully Chinese and enjoy living in Tokyo." This is a "fact" type of long-term memory; "Louis dreams of building a great company, a dream that has lasted 10 years." This is a "personal dream" type of long-term memory.

**From the perspective of ChatAgent:**

- Short-term memory: the last 3-5 turns of conversation.
- Long-term memory: when starting a new chat session, the ChatAgent still remembers the content from the previous session. Even after one week, the ChatAgent remembers the user’s preferences (e.g., to converse in a humorous, concise, or professional manner).

## Storage Methods

> When designing the memory layer, it is critical to consider how data will be stored.

- Short-term memory: in-memory / Redis.
- Long-term memory: various vector databases like qdrant, PostgreSQL + pgvector, Pinecone, etc.

## Storage Timing

**Short-term memory storage timing**

- 1. Store whenever the user asks a question and the LLM responds.
- 2. When the conversation exceeds 10 turns or the token limit, compress the data.

**Long-term memory storage timing**

- 1. Extract content worth storing as long-term memory from the user’s queries. For example:
- user_query: "Please chat with me in a concise, humorous tone." The LLM judges and extracts this user preference as long-term memory.
- 2. After more than 10 conversation turns, compress via LLM, then extract long-term memory from the summary.
- 3. And so on.

## Short-term Memory

> ChatAgent serves many users, each potentially creating multiple chat sessions.

Short-term memory data structure:

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
> It's clear that the `role` fields "user" and "assistant" represent user input and LLM responses respectively. When the message list under `session_id_xxx1` exceeds 10 turns or reaches a token limit, we use the LLM to summarize and extract key contents, then clear the original list and replace it with the summarized short-term memory content under the same session ID.

**Summary and extracted short-term memory data structure:**

```
{
	user_id_xxx:[
			{session_id_xxx1:[{
				"role": "system",
				"content": "User Louis first introduced himself, then asked about today's weather in Tokyo. LLM replied that Tokyo’s weather is sunny with a temperature of around 15°C."
				}
			]}
	],
	user_id_yyy:[...],
	...
}
```
## Long-term Memory

> Designing long-term memory requires handling multi-tenancy, multiple users, memory decay, boundaries, propagation ranges, and access controls.

**Long-term memory data structure:**

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
### Explanation of Long-term Memory Fields

```
# Long-term memory extracted from the user's 10-20 conversation turns, labeled mem_002
"memory_id": "user_id_xxx_mem_002",

# The tenant's ID, here the user is using the iOS app
"tenant_id": "app_iOS", 

# User's ID
"user_id": "user_id_xxx",

# ChatAgent name used by the user, "worker_agent"
"agent_id": "worker_agent",

# Long-term memory type: user preference. Possible types include episodic, semantic, procedural, preference, profile
"type": "preference",

# Scope of long-term memory: private, shared, or public
"scope": "private",
```

```
"content": {

	# Raw user query content
    "raw": "请用简洁,幽默的语气来和我聊天。",

	# Long-term memory summary extracted by LLM
    "summary": "User prefers concise and humorous tone in conversations",

	# Structured JSON of the long-term memory after LLM extraction
    "structured": {
      "tone": ["concise", "humorous"],
      "communication_style": "casual",
      "priority": "high"
    }
  }
```

```
"embedding": {
	# Long-term memory embedding model used is Microsoft's "e5-large"
    "model": "e5-large",
	
	# Vector representation of the long-term memory
    "vector": [0.123, 0.456, ...]
  },
```
```
"metadata": {
	# The importance score of this long-term memory
    "importance": 0.92,

	# The confidence score of this long-term memory
    "confidence": 0.96,
	
	# Tags associated with this long-term memory
    "tags": ["preference", "tone", "style"],

	# Source of this memory: explicit_user_input, implicit_inference, system_generated, external_data, corrected_memory      
    "source": "explicit_user_input",

	# Version number of this long-term memory
    "version": 1
  },
```

```
"time": {
	# Creation timestamp of this long-term memory
    "created_at": "2026-04-20T10:00:00Z",
	
	# Update timestamp of this long-term memory
    "updated_at": "2026-04-25T08:00:00Z",

	# Last access timestamp
    "last_accessed": "2026-04-25T08:00:00Z",

	# Decay score from 0 to 1; 0.95 indicates this memory is worth long-term retention and is unlikely to be deleted.
    "decay_score": 0.95
  },
```

```
"relations": [
    {

	# This long-term memory relates to the user’s 0-10 turn memory mem_001; here mem_002 refers to 10-20 turns.
      "type": "related_to",
      "target_memory_id": "user_id_xxx_mem_001"
    }
  ],
```

```
# Access control settings
"access_control": {
	# This long-term memory is private
    "visibility": "private",

	# Access allowed only to user_id_xxx
    "allowed_users": [user_id_xxx],

	# Roles allowed to access: "admin", "editor"
    "allowed_roles": ["admin", "editor"]
  }
```
## Key Implementation Points

> Since I cannot share the source code publicly, I will explain the key points of the implementation.

- 1. Use asynchronous threads to handle insertion and retrieval of long-term memory, with protection against database crashes.
- 2. Support for overwriting, merging, and deleting long-term memory.
- 3. Filter and deduplicate long-term memory before inserting into the database (consider similarity, importance, confidence).
- 4. Use retrieval ranking or reranking when reading long-term memory.
- 5. Automatically delete long-term memory when decay_score < 0.2.