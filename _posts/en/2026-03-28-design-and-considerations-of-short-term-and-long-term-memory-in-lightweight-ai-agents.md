---
layout:     post
title: Design and Considerations of Short-Term and Long-Term Memory in Lightweight AI Agents
subtitle: Long Short-Term Memory
date:       2026-03-28
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- Memory

---

## Background

> Scenario: A lightweight ChatAgent embedded in a ChatGPT-style conversational interface, where users can engage in dialogue, document analysis, as well as other document generation and formatting tasks.

**Note: This specific AIgent focuses on real-time conversation and analysis. This article does not cover RAG or long-term task planning.**

**Problems to solve:**

- 1. Design long- and short-term memory based on user intent analysis (intent router)
- 2. Short-term memory design for follow-up questions triggered by quoting a text segment
- 3. Address shortcomings of long-term memory: (Who the user is, historical behavior, preferences, long-term goals) | (system role personality, answer style)

## Design Approach

- 1. Design a conversation history database that saves every interaction with the ChatAgent as a complete data entry in real-time.
- 2. Implement a user intent analyzer (intent router) to decide whether the LLM can directly answer using function calling, if short-term memory is needed, if long-term memory is needed, if file analysis should be triggered, or if the system role personality and answer style should be adjusted.
- 3. Build an Agent that generates prompt tokens matching user demands for system role personality, answer style, etc., and writes or modifies `prompt_system.txt` accordingly. This allows the ChatAgent to dynamically adjust on each user reply.
- 4. Based on user intent analysis, save dialogues involving file analysis into long-term memory.
- 5. Under normal circumstances, save the user’s last 5 conversation turns as short-term memory, retrievable from the history database.
- 6. When users quote a segment for follow-up, search the history database for the quote’s location and save the 3 conversation turns around that position as short-term memory.

## Architecture Diagram

![](https://raw.githubusercontent.com/LuochuanAD/BlogSourceImage/refs/heads/master/BlogSourceImage/BlogSourceImage2026/ChatAgent.png)

## Architecture Details

### Intent Layer (LLM)

> Use an LLM to analyze user input intent, categorized into: QA question, file analysis needed, memory query, system role configuration.

Structured output JSON:

```
intent_layer = {
	type: "qa / file_analysis / memory_query / config_change",
	need_memory: true/false,
	need_file: true/false
}
```

### Control Layer

> Rule-based decision making to invoke functional modules

Pseudocode:

```
intent = analyze_intent(user_input)

if intent["need_memory"]:
    memory = retrieve()

if intent["need_file"]:
    file = analyze()
......
```

**QA questions:**

Control whether to call tools, decide which tool to invoke, then:

- 1. Use LLM + function calling to answer user queries.
- 2. Use Tool Registry + rule matching (tool intent and user input) to select tools.

**file_analysis**

In OpenAI, files and user input can be directly passed as parameters to the API.

**memory_query**

> Query long- and short-term memory modules.

- Typically, short-term memory covers the last 3-5 turns of conversation.
- When users quote text for follow-up, search the history database for the quote’s position → take 3 turns of context.
- The number of turns for short-term memory must observe token limits, e.g., tokens < 3000.

**config_change** System_Agent (LLM)

> This involves calling a system Agent to extract personality traits, response style, etc., based on user requests and dynamically update the System_prompt part of the prompt tokens. System_Prompt = base_prompt + dynamic_profile.

LLM structured output JSON:

```
dynamic_profile = {
  "tone": "formal",
  "style": "concise",
  "persona": "technical expert"
}
```

## Context Builder Layer

> Manages prompt context, dynamically adjusting prompt structure and size to help reduce noise and save tokens in LLM calls.

## Memory Update Layer

1. Save user queries and LLM responses into the conversation history database.
2. Save user summaries or file analysis results into the long-term memory database.
3. Short-term memory is dynamically and temporarily retrieved from the conversation history database.

### Improvements

1. Upgrade the conversation history database to a vector database for faster retrieval. This also enables scalability to RAG.
2. Add debug log outputs, for example:

```
【Intent】
Type: file_followup

【Memory Hit】
Hit: Record #23

【Context Used】
- Last 5 conversation turns
- File summary

【Final Answer Source】
Based on file content + historical questions
```