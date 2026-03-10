# 🧠 Eva: Multi-Agent Cognitive Engine

> The hierarchical AI backend powering intelligent reflections, mood analysis, and proactive personalization for the Midnight Momentum application.

## 📖 Overview

**Eva** is a standalone, state-of-the-art **Hierarchical Multi-Agent System** built with [LangGraph](https://python.langchain.com/docs/langgraph) and hosted on **Google Cloud Vertex AI**. 

While traditional chatbots simply reply to prompts, Eva is designed as an integrated cognitive co-pilot. She acts as the "brain" for the client application (Midnight Momentum), actively analyzing incoming user states, extracting life aspirations, dynamically managing database records, and autonomously triggering external application events.

*Note: This repository outlines Eva's architectural design, multi-agent structure, and tool integration. The client application (Midnight Momentum) is maintained in a separate repository.*

---

## ⚡ Tech Stack

Eva’s architecture leverages a modern, high-performance stack optimized for scalable, complex AI agent workflows:

- **Environment & Orchestration:** Google Cloud Vertex AI & LangGraph (Python)
- **Large Language Models (LLMs):** 
  - **Gemini 2.5 Pro:** Powers the Leader agent for complex reasoning, state evaluation, and intelligent task routing.
  - **Gemini 2.5 Flash & Gemini 3 Flash:** Powers specialized sub-agents for lightning-fast, cost-effective tool execution, summarization, and data extraction.
- **State Management:** Redis (High-speed state connection and LangGraph checkpointer for conversation memory)
- **Persistent Memory:** PostgreSQL (Relational storage for Moments, Moods, and Future Self Personas)

---

## 🏗️ System Architecture

Eva is designed using a **Supervisor/Worker Hierarchy**. The Leader acts as the central router, taking in the user's context from the client app and delegating tasks to highly specialized sub-agents. 

<!-- Insert Mermaid Diagram Here -->

---

## 🧠 Core Cognitive Components

To maximize efficiency, lower latency, and maintain clean context windows, Eva avoids relying on a single "mega-prompt." Instead, cognitive load is distributed across specialized agents.

### 1. The Leader (Supervisor Agent)
*Powered by Gemini 2.5 Pro.* 
The orchestrator of the LangGraph state. Whenever a payload is received from the client app (e.g., a chat message or a mood log), the Leader evaluates the intent, parses the Redis graph state, and routes the task to the exact sub-agent required. It then merges the outputs to return a cohesive response to the client.

### 2. 💭 Reflection Agent
*Goal: Prevent surface-level journaling by prompting deep thought.*
* **Mechanism:** Uses internal tools to analyze incoming text. Instead of a standard conversational reply, it identifies missing context and actively prompts the client application to ask the user clarifying questions, helping them unpack their true feelings and motivations.

### 3. 🔮 Insight & Mood Agent
*Goal: Track emotional trajectories and map out ultimate goals.*
* **Mechanism:** Continuously monitors mood arrays and summarizes recent moments. It utilizes specialized data-extraction tools to pull out long-term aspirations, dynamically generating and updating a **"Future Self Persona"** in PostgreSQL. This persona acts as the foundational context for all of Eva's future logic.

### 4. 📝 Moment Agent
*Goal: Autonomously manage the user's data structure.*
* **Mechanism:** Equipped with database CRUD capabilities. If the user mentions a new plan, thought, or burst of motivation in natural language, this agent automatically maps it to the schema and generates a structured "Moment" in PostgreSQL. It can also autonomously fetch and edit existing records based on new conversational context.

### 5. 🔔 Notification Agent
*Goal: Deliver highly personalized, context-aware nudges.*
* **Mechanism:** Bridges the gap between Eva's backend and the client's frontend. It leverages the user's Future Self Persona and recent Moments to draft highly bespoke notification copy. It then calls the `ScheduleNudgeTool` to interface directly with the client app's native notification engine.

---

## 🔄 Bidirectional API Paradigm

Most AI API integrations are unidirectional (Client asks → AI answers). Eva is designed as a **bidirectional engine**:

1. **Client ➡️ Eva**: The client app feeds UI interactions, explicit mood tracking, and chat history into Eva's LangGraph state (managed securely in Redis).
2. **Eva ➡️ Client**: Eva acts *on* the client. Through secure tool calling via Vertex AI, she autonomously manages PostgreSQL database entries (Moments) and actively triggers features on the client's operating system (Push Notifications). 

---
*Disclaimer: This repository serves as a structural and architectural showcase. Proprietary agent prompts, internal schema logic, and specific LangGraph implementation scripts are kept private to protect core intellectual property.*
