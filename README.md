# NOZ-Chatbot

# Technical Concept: Frag die NOZ

**Role:** AI Engineer / Data Scientist  
**Date:** December 4, 2025  
**Scope:** Data Strategy, Retrieval Algorithms, and System Architecture

---

## 1. Executive Summary

"Frag die NOZ" is not a standard chatbot; it is a **Domain-Specific Retrieval-Augmented Generation (RAG) system**. Unlike generic models (ChatGPT), it must answer based only on the NOZ archive, respecting the temporality of news (news ages quickly) and the regional context.

This concept proposes a **Hybrid Search Architecture** with a custom **Time-Decay Ranking Algorithm** to ensure users receive factually accurate and up-to-date answers.

---

## 2. The Core Challenge: "News is not Static"

Standard RAG approaches fail in the news domain due to three specific factors. Our architecture addresses these directly:

1. **Temporality**: A query like "Wer ist der VfL Trainer?" has different answers in 2021, 2023, and 2025. Vector similarity alone might retrieve the 2021 article if the semantic match is stronger.
   - **Solution:** Time-Decay Scoring.

2. **Specificity vs. Concept**: Users search for specific entities ("Bawinkel", "Timo Schultz") and broad concepts ("Housing market trends").
   - **Solution:** Hybrid Search (Keyword + Vector).

3. **Hallucination Risk**: A newspaper cannot afford to invent facts.
   - **Solution:** Strict Grounding & Citation Mapping.

---

## 3. Data Strategy & Ingestion Pipeline

We are dealing with **~1 Million articles**. Raw text ingestion is insufficient. We need an ETL pipeline that enriches data before indexing.

### 3.1. Chunking Strategy: Parent-Child Indexing

Splitting articles blindly breaks context. We use a **Parent-Child approach** to decouple searching from reading.

- **Child Chunk (The Search Target)**: Small segments (250–300 tokens).
  - *Why:* High density of meaning. Better for vector matching.

- **Parent Chunk (The LLM Context)**: The full article (or large window).
  - *Why:* When a child chunk matches, we retrieve the Parent to give the LLM the full story (e.g., the date, the author, the conclusion).

### 3.2. Metadata Enrichment (The Schema)

We extract structured data during ingestion to enable precise filtering.

**Data Schema (JSON Structure):**
```json
{
  "article_id": "12345",
  "title": "L67 in Bawinkel: Parents fight for safety",
  "text_content": "...",
  "vectors": {
    "dense": [0.12, -0.45, ...],  // Semantic meaning
    "sparse": {"bawinkel": 0.8, "l67": 0.9} // Keyword weights (BM25)
  },
  "metadata": {
    "publish_date": "2025-07-15T10:00:00Z",
    "location": ["Bawinkel", "Emsland"],
    "category": "Local",
    "entities": ["Thomas März", "Kita St. Marien"]
  }
}
```

---

## 4. Search & Retrieval Architecture

This is the "Brain" of the system. We do not rely on a single algorithm.

### 4.1. The Hybrid Search Logic

We use **Reciprocal Rank Fusion (RRF)** to combine two search methods:

1. **Dense Retrieval (Vector)**: Captures intent (e.g., "Traffic safety issues").
   - **Model:** `text-embedding-3-large` (OpenAI) or `intfloat/multilingual-e5-large` (Open Source).

2. **Sparse Retrieval (BM25)**: Captures exact keywords (e.g., "L67", "Bawinkel").

### 4.2. The "News Bias" Algorithm (Time Decay)

To solve the "Old News" problem, we apply a mathematical decay function to the score of retrieved documents.

**Logic:**
```
Score_final = Score_search × (1 / (1 + λ · (Time_now - Time_published)))
```

- **Effect:** An article from yesterday with a 90% match beats an article from 3 years ago with a 95% match.
- **Implementation:** This is implemented as a custom scoring profile in the Vector Database (Weaviate/Qdrant).

### 4.3. Re-Ranking (Precision Layer)

After retrieving the top 50 candidates, we pass them through a **Cross-Encoder** (e.g., `bge-reranker-v2-m3`).

- **Function:** It reads the query and the document pair deeply.
- **Result:** It discards irrelevant "false positives" that vector search found. We send only the top 5 to the LLM.

---

## 5. Answer Generation (LLM)

### 5.1. Model Selection

- **Primary Choice:** GPT-4o (via Azure OpenAI).
  - **Reasoning:** Best instruction following for German language and complex citation formatting. Azure ensures GDPR compliance (servers in Europe).

- **Alternative (Cost/Privacy):** Llama 3 (70B) self-hosted.
  - **Trade-off:** Higher maintenance effort, but data never leaves NOZ infrastructure.

### 5.2. System Prompting & Citations

The prompt is engineered to force **Evidence-Based Answers**.

**System Prompt:**

> "You are an assistant for the Neue Osnabrücker Zeitung. Answer the user's question using only the provided context.
> 
> - If the answer is not in the context, state 'I do not know'.
> - Cite every claim with the index of the source article, e.g., [1].
> - Prioritize the most recent articles in your synthesis."

---

## 6. Evaluation & Monitoring

How do we measure success?

### 6.1. Offline Evaluation (RAGAS Framework)

Before deploying updates, we run a test set of 50 curated questions against these metrics:

- **Faithfulness:** Is the answer derived only from the context?
- **Context Precision:** Did the search engine find the relevant article?

### 6.2. Online Monitoring

- **"No Answer" Rate:** If this spikes, our retrieval is failing (or we lack content).
- **User Feedback:** Thumbs up/down on specific answers.

---

## 7. Architectural Decisions & Trade-offs

| Decision        | Choice           | Alternative    | Rationale                                                                                   |
|-----------------|------------------|----------------|---------------------------------------------------------------------------------------------|
| Database        | Weaviate         | Pinecone       | Weaviate supports hybrid search and custom scoring (Time Decay) natively.                   |
| Search          | Hybrid           | Pure Vector    | Pure vector search fails on specific local names (e.g., small village names).               |
| Infrastructure  | Cloud (Azure)    | On-Premise     | Speed to MVP. We can move to On-Premise later for cost optimization.                        |

---

## 8. Future Outlook

- **Multimodal Search:** Using CLIP embeddings to allow users to search for images (e.g., "Show me photos of the stadium construction").
- **Personalization:** Boosting articles based on the user's reading history (e.g., "More sports news").

---
