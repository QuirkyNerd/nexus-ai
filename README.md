# NexusAI — Multi-Agent Clinical Intelligence & Personal Health Platform

> Production-ready, multi-agent AI healthcare platform providing evidence-grounded clinical consultation, lab report analysis, RAG-backed medical intelligence, and personal health operating workflows aligned with WHO, CDC, NHS, and NIH guidelines.

---

---

## 📌 Overview

NexusAI is an agentic AI medical platform designed to bridge the gap between complex medical literature and patient-facing healthcare management. It processes text queries, lab test PDFs, and diagnostic images to deliver instant, multi-stage clinical guidance.

The system is built on a **Grounding & Safety First** architecture: rather than relying on a single black-box LLM, NexusAI combines a zero-latency query router, vector-retrieval RAG over clinical medical literature, a live NCBI/PubMed fallback verification engine, structured lab report parsers, and a local-first offline health storage engine.

---

## 🏗️ System Architecture

<p align="center">
  <img src="report/system%20architecture.png" alt="NexusAI System Architecture" width="1000"/>
</p>

---

---


## ✨ Key Features

| Capability | Technical Description |
|---|---|
| **Multi-Agent Orchestration** | Task-specialized agent network (`RouterAgent`, `RAGAgent`, `ReportAgent`, `ImageAgent`, `ConfidenceAgent`) |
| **Evidence-Grounded RAG** | Vector similarity search over medical corpora via Supabase pgvector (`sentence-transformers/all-MiniLM-L6-v2`) |
| **Live NCBI PubMed Fallback** | Automatic live web-retrieval fallback via NCBI E-utilities when internal vector similarity drops below `0.45` |
| **Ultra-Fast LLM Inference** | Groq Cloud integration running `llama-3.3-70b-versatile` with automated failover to `llama-3.1-8b-instant` |
| **Lab Report & Image Analysis** | PDF parser for CBC reports, metabolic panels, and diagnostic imagery with structured parameter extractions |
| **Local-First Health OS** | Client-side `health-store.ts` managing vitals, medications, appointments, EHR records, and emergency contacts offline |
| **Multilingual Clinical Engine** | 20+ language support (`web/lib/i18n.ts`) localized to user country, language, and measurement unit system |
| **Secure PostgreSQL Auth** | JWT session authentication with `bcrypt` password hashing, role management, and SMTP OTP email recovery |
| **Containerized Deployment** | Docker & Docker Compose setup supported by a production Makefile using the modern `uv` Python package manager |

---
---

## ⚡ Quick Start

### Prerequisites
- **Node.js**: `>= 18.17.0`
- **Python**: `>= 3.9, < 3.13`
- **Docker & Docker Compose**: (Optional, for containerized run)
- **uv**: Python package manager (`pip install uv`)

### 1. Clone Repository
```bash
git clone https://github.com/ruslanmv/ai-medical-chatbot.git
cd medical-bot-main
```

### 2. Run via Docker Compose
```bash
docker-compose up --build
```

### 3. Manual Local Setup

#### Backend Setup
```bash
cd backend
uv pip install -r requirements.txt
python main.py
```
*Backend runs at: `http://localhost:8000`*

#### Frontend Setup
```bash
cd web
npm install
npm run dev
```
*Frontend runs at: `http://localhost:3000`*

---

## 💻 Technology Stack

```
┌────────────────────────────────────────────────────────────────────────┐
│                            FRONTEND LAYER                              │
│   Next.js 14 (App Router) │ React 18 │ TypeScript │ Tailwind CSS      │
│   Lucide Icons │ Framer Motion │ html2pdf.js │ Web Speech API            │
└────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                        SERVERLESS PROXY / GATEWAY                      │
│   Vercel Edge Functions │ Next.js API Routes (/api/agent, /api/chat)   │
└────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                             BACKEND ENGINE                             │
│   FastAPI │ Python 3.10+ │ Pydantic │ SQLAlchemy Core │ PyJWT │ bcrypt   │
└────────────────────────────────────────────────────────────────────────┘
                                    │
                  ┌─────────────────┴─────────────────┐
                  ▼                                   ▼
┌───────────────────────────────────┐ ┌───────────────────────────────────┐
│     PERSISTENCE & VECTOR STORE     │ │          AI & LLM INFRA           │
│ PostgreSQL / Neon Database        │ │ Groq (Llama-3.3-70b-versatile)    │
│ Supabase (pgvector - 384 dim)     │ │ HuggingFace (all-MiniLM-L6-v2)    │
│ Client LocalStorage (HealthStore) │ │ NCBI / PubMed E-utilities API     │
└───────────────────────────────────┘ └───────────────────────────────────┘
```

---

## 🧠 Multi-Agent & RAG Pipeline Mechanics

### 1. Zero-Latency Intent Classification (`RouterAgent`)
The `RouterAgent` in [`backend/agents/router_agent.py`](file:///d:/Downloads/medical-bot-main/medical-bot-main/backend/agents/router_agent.py) evaluates query string signals using optimized keyword signal sets:
- **`_PATIENT_SIGNALS`**: Words like `my report`, `lab result`, `my hemoglobin`, `blood test`.
- **`_IMAGE_SIGNALS`**: Terms like `x-ray`, `mri`, `ct scan`, `radiology`.
- **`_RESEARCH_SIGNALS`**: Terms like `mechanism`, `clinical trial`, `meta-analysis`, `biomarker`.
- **`_FOUNDATIONAL_SIGNALS`**: Terms like `what is`, `physiology`, `anatomy`, `symptoms of`.

### 2. Hybrid RAG Retrieval Engine (`RAGEngine`)
Implemented in [`backend/core/rag_engine.py`](file:///d:/Downloads/medical-bot-main/medical-bot-main/backend/core/rag_engine.py):
- **Chunking**: Text is chunked into 400-word segments with 50-word overlaps.
- **Embedding Generation**: Calls HuggingFace's inference pipeline for `sentence-transformers/all-MiniLM-L6-v2` returning 384-dimensional dense vectors.
- **Similarity Search**: Executes Supabase PostgreSQL vector RPC function `match_documents` to pull top candidates.

### 3. Dynamic Live PubMed Fallback (`ConfidenceAgent`)
Implemented in [`backend/agents/confidence_agent.py`](file:///d:/Downloads/medical-bot-main/medical-bot-main/backend/agents/confidence_agent.py):
- When internal RAG similarity score is **`< 0.45`**, the system queries the **NCBI PubMed E-utilities API** (`esearch.fcgi` + `esummary.fcgi`) to pull the top 3 live peer-reviewed abstracts, guaranteeing up-to-date evidence grounding.

---

---

## 🌐 Multilingual & Grounding Knowledge Scaffold

NexusAI injects a structured clinical knowledge scaffold into every system prompt (`lib/medical-knowledge.ts`), grounding answers in standard medical bodies:
- **WHO** (World Health Organization)
- **CDC** (Centers for Disease Control and Prevention)
- **NHS** (National Health Service)
- **NIH** (National Institutes of Health)
- **ICD-11 & BNF** (British National Formulary)

The frontend localization engine [`web/lib/i18n.ts`](file:///d:/Downloads/medical-bot-main/medical-bot-main/web/lib/i18n.ts) translates interfaces and outputs into **20+ global languages**, including English, Spanish, French, German, Mandarin, Hindi, Arabic, Japanese, Portuguese, and Russian.

---

## 🔌 API Architecture & Endpoints Summary

All backend API routes are prefixed with `/api/`.

| Module | Route Prefix | Primary Handler File | Purpose |
|---|---|---|---|
| **Agent Router** | `/api/agent` | [`backend/api/routes.py`](file:///d:/Downloads/medical-bot-main/medical-bot-main/backend/api/routes.py) | Main multi-agent execution endpoint (`/chat`) |
| **Medical Query** | `/api/medical-query` | [`backend/api/medical_query_router.py`](file:///d:/Downloads/medical-bot-main/medical-bot-main/backend/api/medical_query_router.py) | Direct query routing and triage |
| **Authentication** | `/api/auth` | [`backend/api/auth_router.py`](file:///d:/Downloads/medical-bot-main/medical-bot-main/backend/api/auth_router.py) | Login, signup, JWT validation, OTP password reset |
| **Conversations** | `/api/conversations`| [`backend/api/conversations_router.py`](file:///d:/Downloads/medical-bot-main/medical-bot-main/backend/api/conversations_router.py) | User chat session CRUD and message retrieval |
| **Schedules** | `/api/schedule` | [`backend/api/schedule_router.py`](file:///d:/Downloads/medical-bot-main/medical-bot-main/backend/api/schedule_router.py) | Medication schedules and reminder tracking |
| **Health OS** | `/api/health` | [`backend/api/health_router.py`](file:///d:/Downloads/medical-bot-main/medical-bot-main/backend/api/health_router.py) | Vitals, medicine inventory, emergency contacts, EHR |
| **RAG System** | `/api/rag` | [`backend/api/rag_router.py`](file:///d:/Downloads/medical-bot-main/medical-bot-main/backend/api/rag_router.py) | Document ingestion, status probes, search test |
| **Report Export** | `/api/export-report`| [`backend/api/export_router.py`](file:///d:/Downloads/medical-bot-main/medical-bot-main/backend/api/export_router.py) | PDF generation and health export handling |

---

## 📁 Key File Reference

| File Path | Description & Primary Responsibility |
|---|---|
| [`backend/main.py`](file:///d:/Downloads/medical-bot-main/medical-bot-main/backend/main.py) | FastAPI app entry point, CORS whitelist, middleware logger, global exception handler |
| [`backend/database.py`](file:///d:/Downloads/medical-bot-main/medical-bot-main/backend/database.py) | PostgreSQL engine initialization, connection pooling, and raw SQL schema definitions |
| [`backend/core/orchestrator.py`](file:///d:/Downloads/medical-bot-main/medical-bot-main/backend/core/orchestrator.py) | Multi-agent orchestrator managing query execution, agent traces, and response delivery |
| [`backend/core/rag_engine.py`](file:///d:/Downloads/medical-bot-main/medical-bot-main/backend/core/rag_engine.py) | Supabase pgvector semantic search engine & HuggingFace embedding integration |
| [`backend/agents/router_agent.py`](file:///d:/Downloads/medical-bot-main/medical-bot-main/backend/agents/router_agent.py) | Signal-based intent classifier mapping queries to canonical medical intents |
| [`backend/agents/confidence_agent.py`](file:///d:/Downloads/medical-bot-main/medical-bot-main/backend/agents/confidence_agent.py) | Confidence scoring agent & live NCBI/PubMed web retrieval fallback handler |
| [`backend/api/groq_client.py`](file:///d:/Downloads/medical-bot-main/medical-bot-main/backend/api/groq_client.py) | Groq Cloud API wrapper for `llama-3.3-70b-versatile` with automatic model fallback |
| [`web/components/NexusAIApp.tsx`](file:///d:/Downloads/medical-bot-main/medical-bot-main/web/components/NexusAIApp.tsx) | Main React client application shell with tab-based view switching |
| [`web/lib/health-store.ts`](file:///d:/Downloads/medical-bot-main/medical-bot-main/web/lib/health-store.ts) | Local-first offline health storage engine managing vitals, records, and JSON exports |
| [`web/lib/api-client.ts`](file:///d:/Downloads/medical-bot-main/medical-bot-main/web/lib/api-client.ts) | Shared API client with authorization header injection and normalized error handling |
| [`Makefile`](file:///d:/Downloads/medical-bot-main/medical-bot-main/Makefile) | Enterprise Makefile providing `install`, `test`, `lint`, `format`, `check`, and `docker` targets |
| [`docker-compose.yml`](file:///d:/Downloads/medical-bot-main/medical-bot-main/docker-compose.yml) | Multi-container Docker configuration for backend server and PostgreSQL data volumes |

---
---

## ⚠️ Medical Disclaimer

**NexusAI is designed for informational, educational, and workflow support purposes only.** 

It is not a substitute for professional medical advice, diagnosis, or treatment. Always seek the advice of a qualified healthcare provider with any questions regarding a medical condition. In case of a medical emergency, call your local emergency service (e.g., 911) or visit the nearest emergency department immediately.

---

## 📄 License

This project is developed for academic and research demonstration purposes under the MIT License.

© 2026 NexusAI

---
