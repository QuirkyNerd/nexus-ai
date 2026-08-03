# 🏥 NexusAI — AI Medical chatbot & Personal Health Operating System

> Production-ready, multi-agent AI healthcare platform providing evidence-grounded clinical consultation, lab report analysis, RAG-backed medical intelligence, and personal health operating workflows aligned with WHO, CDC, NHS, and NIH guidelines.

---

## 🚀 Deployment & Live Endpoints

| Service | Access Endpoint / Location | Description |
|---|---|---|
| **Production Web UI** | `https://medical-bot-mu.vercel.app` | Next.js 14 PWA frontend deployed on Vercel |
| **FastAPI Backend API** | `http://localhost:8000` | Multi-agent Python API service |
| **API Documentation** | `http://localhost:8000/docs` | Interactive Swagger / OpenAPI documentation |
| **Container Service** | `docker-compose up --build` | Dockerized backend and data volumes |

---

## 📌 Overview

** NexusAI** is an agentic AI medical platform designed to bridge the gap between complex medical literature and patient-facing healthcare management. It processes text queries, lab test PDFs, and diagnostic images to deliver instant, multi-stage clinical guidance.

The system is built on a **Grounding & Safety First** architecture: rather than relying on a single black-box LLM, MedOS combines a zero-latency query router, vector-retrieval RAG over clinical medical literature, a live NCBI/PubMed fallback verification engine, structured lab report parsers, and a local-first offline health storage engine.

```
       WHO / CDC / NHS / NIH Grounded Medical Knowledge Scaffold
                                   │
                                   ▼
 [User Input] ──► [Router Agent] ──┬──► [RAG Vector Search (Supabase pgvector)]
                                   ├──► [Report Agent (Lab PDF Parser)]
                                   ├──► [Image Agent (Diagnostic Scan Guidance)]
                                   └──► [Confidence Agent (NCBI PubMed Fallback)]
                                   │
                                   ▼
                 [Groq Llama 3.3 70B Clinical Inference]
                                   │
                                   ▼
             [Structured Guidance + Confidence Grounding Badge]
```

---

## 🏗️ System Architecture

```mermaid
sequenceDiagram
    autonumber
    actor User as Patient / Clinician
    participant UI as Next.js 14 Frontend
    participant LocalStore as HealthStore (LocalStorage)
    participant Proxy as Vercel API Gateway
    participant API as FastAPI Backend (/api/agent/chat)
    participant Router as RouterAgent
    participant RAG as Supabase pgvector RAG
    participant NCBI as NCBI / PubMed API
    participant Groq as Groq LLM (Llama 3.3 70B)
    participant Conf as ConfidenceAgent
    participant DB as PostgreSQL DB

    User->>UI: Submit query / Upload lab PDF or image
    UI->>LocalStore: Synchronize health context & profile
    UI->>Proxy: POST /api/agent/chat (Bearer Token)
    Proxy->>API: Forward request payload
    API->>Router: classify(query, has_image, has_report)
    Router-->>API: Intent (MEDICAL_QUESTION / REPORT_ANALYSIS / IMAGE_DIAGNOSIS)
    
    alt Intent == MEDICAL_QUESTION
        API->>RAG: search(query_vector, top_k=5)
        RAG-->>API: Clinical snippets (pgvector similarity score)
        
        alt RAG Similarity Score < 0.45 (Low Confidence)
            API->>NCBI: Live PubMed E-utilities search query
            NCBI-->>API: Top 3 recent abstract snippets
        end
        
        API->>Groq: Generate grounded response (Llama 3.3 70B)
        Groq-->>API: Structured Markdown response
        API->>Conf: Evaluate response & assign confidence badge
        Conf-->>API: Badge (🟢 High / 🟡 Moderate / 🔴 External PubMed)
    else Intent == REPORT_ANALYSIS
        API->>Groq: Extract & analyze lab parameters
        Groq-->>API: Structured report breakdown & recommendations
    end

    API->>DB: Persist conversation & messages
    API-->>Proxy: Return final response payload + metadata
    Proxy-->>UI: Render response + confidence badge + follow-up chips
    UI-->>User: Display clinical guidance & update dashboard
```

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

## 👥 Role-Based Access Control

| Role | Responsibilities & Capabilities |
|---|---|
| **Patient / User** | Chat with AI, upload lab reports/images, track vitals, manage medication schedules, export health summary |
| **Medical Coder / Clinician** | Review AI clinical rationale, cross-reference RAG evidence snippets, inspect lab parameter extractions |
| **Administrator** | Manage registered users, inspect vector database status, configure environment keys, review system logs |

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

### 2. Environment Configuration
Create a `.env` file in `backend/`:
```env
PORT=8000
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/medai_db
SUPABASE_URL=https://your-supabase-project.supabase.co
SUPABASE_KEY=your-supabase-anon-key
HF_API_KEY=your_huggingface_api_key
GROQ_API_KEY=your_groq_api_key
JWT_SECRET=your_super_secret_jwt_key
SMTP_EMAIL=your_email@gmail.com
SMTP_PASSWORD=your_app_password
```

Create a `.env.local` file in `web/`:
```env
NEXT_PUBLIC_API_URL=http://localhost:8000
GROQ_API_KEY=your_groq_api_key
```

### 3. Run via Docker Compose (Recommended)
```bash
docker-compose up --build
```

### 4. Manual Local Setup

#### Backend Setup
```bash
cd backend
uv pip install -r requirements.txt
python main.py
```
*Backend runs at: `http://localhost:8000` (API Docs at `http://localhost:8000/docs`)*

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

## 🗄️ Database Architecture

The backend utilizes PostgreSQL with SQLAlchemy Core in [`backend/database.py`](file:///d:/Downloads/medical-bot-main/medical-bot-main/backend/database.py).

```sql
-- Core User & Auth Tables
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT,
    email TEXT UNIQUE NOT NULL,
    password_hash TEXT NOT NULL,
    role TEXT NOT NULL DEFAULT 'user',
    is_admin BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE password_reset_tokens (
    id SERIAL PRIMARY KEY,
    email TEXT NOT NULL,
    otp_code TEXT NOT NULL,
    expires_at TIMESTAMP NOT NULL,
    is_used BOOLEAN DEFAULT FALSE
);

-- Conversation History Tables
CREATE TABLE conversations (
    id TEXT PRIMARY KEY,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    title TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE messages (
    id SERIAL PRIMARY KEY,
    conversation_id TEXT REFERENCES conversations(id) ON DELETE CASCADE,
    role TEXT CHECK(role IN ('user','assistant')),
    content TEXT NOT NULL,
    metadata TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Health OS Subsystem Tables
CREATE TABLE medication_schedules ( id TEXT PRIMARY KEY, user_id INTEGER REFERENCES users(id), medication_name TEXT NOT NULL, dosage TEXT NOT NULL, time TEXT NOT NULL, status TEXT DEFAULT 'pending' );
CREATE TABLE health_vitals ( id TEXT PRIMARY KEY, user_id INTEGER REFERENCES users(id), type TEXT NOT NULL, value TEXT NOT NULL, unit TEXT NOT NULL, timestamp TIMESTAMP NOT NULL );
CREATE TABLE health_records ( id TEXT PRIMARY KEY, user_id INTEGER REFERENCES users(id), title TEXT NOT NULL, type TEXT NOT NULL, date TIMESTAMP NOT NULL, provider TEXT );
CREATE TABLE health_medicines ( id TEXT PRIMARY KEY, user_id INTEGER REFERENCES users(id), name TEXT NOT NULL, dose TEXT NOT NULL, form TEXT NOT NULL, quantity INTEGER NOT NULL );
CREATE TABLE health_contacts ( id TEXT PRIMARY KEY, user_id INTEGER REFERENCES users(id), name TEXT NOT NULL, role TEXT NOT NULL, phone TEXT, email TEXT );
CREATE TABLE ehr_profiles ( id TEXT PRIMARY KEY, user_id INTEGER REFERENCES users(id), data TEXT NOT NULL );
```

---

## 🌐 Multilingual & Grounding Knowledge Scaffold

MedOS injects a structured clinical knowledge scaffold into every system prompt (`lib/medical-knowledge.ts`), grounding answers in standard medical bodies:
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
| [`web/components/MedOSApp.tsx`](file:///d:/Downloads/medical-bot-main/medical-bot-main/web/components/MedOSApp.tsx) | Main React client application shell with tab-based view switching |
| [`web/lib/health-store.ts`](file:///d:/Downloads/medical-bot-main/medical-bot-main/web/lib/health-store.ts) | Local-first offline health storage engine managing vitals, records, and JSON exports |
| [`web/lib/api-client.ts`](file:///d:/Downloads/medical-bot-main/medical-bot-main/web/lib/api-client.ts) | Shared API client with authorization header injection and normalized error handling |
| [`Makefile`](file:///d:/Downloads/medical-bot-main/medical-bot-main/Makefile) | Enterprise Makefile providing `install`, `test`, `lint`, `format`, `check`, and `docker` targets |
| [`docker-compose.yml`](file:///d:/Downloads/medical-bot-main/medical-bot-main/docker-compose.yml) | Multi-container Docker configuration for backend server and PostgreSQL data volumes |

---

## 🛠️ DevOps & Development Commands

This repository includes a production-grade `Makefile` for streamlined development workflows:

```bash
make install       # Install production dependencies using uv
make install-dev   # Install development and testing dependencies
make format        # Format Python code using black and isort
make lint          # Run static code analysis with flake8 and pylint
make test          # Run complete Pytest suite
make test-unit     # Run unit tests only
make test-cov      # Generate HTML code coverage report
make check         # Run format, lint, and type checks in sequence
```

---

## ⚠️ Medical Disclaimer

**NexusAI is designed for informational, educational, and workflow support purposes only.** 

It is not a substitute for professional medical advice, diagnosis, or treatment. Always seek the advice of a qualified healthcare provider with any questions regarding a medical condition. In case of a medical emergency, call your local emergency service (e.g., 911) or visit the nearest emergency department immediately.

---

## 📄 License

Distributed under the **Apache-2.0 License**. See `LICENSE` for more information.
