# RAG-based Document Chatbot

RAG backend, featuring multi-stage retrieval and advanced re-ranking architectures.

---

## Engineering Highlights

* **Inference Speed:** Leveraging **Cerebras** for ultra-low latency LLM generation.
* **Hybrid Search:** Combined Semantic (Vector) and Lexical (BM25-style) retrieval using **Reciprocal Rank Fusion (RRF)**.
* **Re-ranking:** Integrated **Voyage AI** re-ranker to combat "lost in the middle" phenomena and ensure top-tier context relevance.
* **Async Processing:** Background workers handle heavy ingestion and document processing to keep the API responsive.

---

## Tech Stack

| Layer | Technology | Rationale |
| :--- | :--- | :--- |
| **LLM** | Cerebras (Llama-3) | Industry-leading tokens-per-second for real-time UX. |
| **Vector DB** | PostgreSQL + pgvector | Relational data integrity with high-performance vector similarity. |
| **Embeddings** | Gemini-001 | High-dimensional semantic mapping. |
| **Re-ranker** | Voyage AI | Optimised for pinpointing relevance in large context windows. |
| **Framework** | Django Ninja | Django ninja Fasapai like validation schema and django orm.  |

---

## Architecture Diagram Flow

             DATA INGESTION LAYER(Parsing -> Chunking -> Embedding -> Vector DB storage is done by Redis and Celery (Background Workers))
                      │        
        ┌──────────────┬───────────┼
        │              │           │     
    ┌───▼───┐      ┌───▼───┐  ┌───▼───┐  
    │  PDF  │      │ DOCX  │  │  .txt │ 
    │       │      │       │  |       │  
    └───┬───┘      └───┬───┘  └───┬───┘  
        │              │           │
        └──────────────┼───────────┘
                       │
            ┌──────────▼──────────┐
            │ Text Extraction &   │
            │ Chunking Pipeline   │
            │ (metadata + overlap)│
            └──────────┬──────────┘
                       │
            ┌──────────▼──────────────┐
            │ Embedding Generation    │
            │ (vector embeddings)     │
            └──────────┬──────────────┘
                       │
        ┌──────────────/
        │                              
    ┌───▼──────────┐          ┌────────-────────┐
    │ Vector DB    │          │ SQL Database    │
    │ (embeddings) │          │ (chat history)  │
    └───┬──────────┘          └────────┬────────┘
        │                              │
        │     USER QUERY               │
        │           │                  │
        └───┬───────┼──────────────────┘
            │       │
    ┌───────▼───────▼──────────────────────────┐
    │  HYBRID RETRIEVAL                        │
    │  ┌──────────────┐  ┌──────────────────┐  │
    │  │  BM25        │  │  Semantic Search │  │
    │  │ (Lexical)    │  │  (Embeddings)    │  │
    │  └──────┬───────┘  └────────┬─────────┘  │
    │         │                   │            │
    │         └───────────┬───────┘            │
    │                     │                    │
    │         ┌───────────▼──────────┐         │
    │         │ Reciprocal Rank      │         │
    │         │ Fusion (RRF)         │         │
    │         │ (combine results)    │         │
    │         └───────────┬──────────┘         │
    └─────────────────────┼────────────────────┘
                          │
            ┌─────────────▼────────────┐
            │ Reranking                │
            │ (Voyage Rerank 2.5)      │
            │                          │
            └─────────────┬────────────┘
                          │
            ┌─────────────▼──────────────┐
            │ Context Assembly           │
            │ System Prompt +            │
            │ Chat History (6 msgs) +    │
            │ Reranked Chunks            │
            └─────────────┬──────────────┘
                          │
            ┌─────────────▼──────────────┐
            │ LLM Inference              │
            │ (Cerebras API)             │
            │ (Llama 3.1-8B)             │
            └─────────────┬──────────────┘
                          │
            ┌─────────────▼──────────────┐
            │ Response & Persistence     │
            │ Return to user +           │
            │ Save to database           │
            └────────────────────────────┘
