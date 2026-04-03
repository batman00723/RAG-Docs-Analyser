# STATEFUL SEQUENTIAL AGENTIC RAG BOT WITH MEMORY

A production-ready RAG (Retrieval-Augmented Generation) agent built with **LangGraph**, **Django-ninja**. This agent features persistent conversation memory, autonomous query reformulation, and a self-correcting routing logic that switches between local Vector DB and Web Search.

## Architecture Flow

```mermaid
graph TD
    A[User Query] --> B[Reformulate Query Node]
    B --> C[Retrieve Local DB]
    C --> D[Generate Node]
    D --> E{Grader Logic (Gemini + Custom hardcoded)}
    E -- "Insufficient" --> F[Web Search Node]
    F --> D
    E -- "Sufficient" --> G[Final Response]
    G --> H[Save State to Postgres]
```



## Features

* **Persistent Memory:** Uses `PostgresSaver` and a custom `ConnectionPool` to maintain chat history across API calls.
* **Hybrid Search:** Intelligent routing between a local Vector Database (ChromaDB/PGVector) and real-time Web Search.
* **Self-Correction:** Integrated **Gemini 3.1 flash** grader that assesses answer sufficiency before returning a response.
* **Query Reformulation:** Automatically rewrites user queries (e.g., "his age" -> "LeBron James age") using conversation history to ensure search accuracy.

---

## Tech Stack

* **Core Logic:** LangGraph, LangChain
* **Backend:** Django, Django Ninja 
* **LLMs:** Cerebras (Inference), Gemini 3.1 Flash Lite (Grading/Routing)
* **Database:** PostgresSaver (Checkpointer/Memory), Postgres (Vector Store)
* **Search:** Tavily

---

<p align="center">
  <img src="Basic%20RAG%20Agent%20Sequential.png" width="800" alt="RAG Architecture">
</p>
