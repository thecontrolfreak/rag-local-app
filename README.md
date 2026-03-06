<div align="center">

# 🔍 Local RAG

### Privacy-First Retrieval-Augmented Generation System

[![Python](https://img.shields.io/badge/Python-3.11%2B-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-Async-009688?style=for-the-badge&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![React](https://img.shields.io/badge/React-18-61DAFB?style=for-the-badge&logo=react&logoColor=black)](https://react.dev/)
[![pgvector](https://img.shields.io/badge/pgvector-PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)](https://github.com/pgvector/pgvector)
[![Ollama](https://img.shields.io/badge/Ollama-Local_LLM-000000?style=for-the-badge)](https://ollama.ai/)
[![License: MIT](https://img.shields.io/badge/License-MIT-22c55e?style=for-the-badge)](LICENSE)

<br/>

> A fully local, privacy-first document intelligence system — ingest PDFs and documents, generate dense vector embeddings via Ollama, and query them with a local LLM. Zero cloud calls. Zero data leakage.

</div>

---

## Table of Contents

- [Overview](#overview)
- [RAG Pipeline](#rag-pipeline)
- [System Architecture](#system-architecture)
- [Tech Stack](#tech-stack)
- [Database Schema](#database-schema)
- [API Reference](#api-reference)
- [Getting Started](#getting-started)
- [Project Structure](#project-structure)
- [Architectural Decisions](#architectural-decisions)
- [Author](#author)

---

## Overview

Local RAG is an end-to-end **Retrieval-Augmented Generation** application that runs entirely on your own hardware. It implements the full RAG loop — document ingestion, chunking, embedding, vector search, context assembly, LLM generation, and citation extraction — without any external API calls. All models run locally via Ollama or a compatible llama.cpp server.

| Dimension | Value |
|-----------|-------|
| Embedding model | `nomic-embed-text` (768-dim vectors, via Ollama) |
| Vector storage | PostgreSQL + `pgvector` (cosine similarity) |
| Fast LLM | `llama3.1:8b` |
| Accurate LLM | `llama3.3:70b` |
| Similarity threshold | 0.30 (configurable) |
| Top-K retrieval | 6 chunks (configurable) |
| Supported formats | PDF, TXT, MD, CSV |
| Auth | JWT HS256, 24-hour expiry |

---

## RAG Pipeline

```
User uploads document
        │
        ▼
┌──────────────────────────────────────────────────────────────────┐
│  INGESTION PIPELINE                                               │
│                                                                    │
│  1. Text extraction   (PyPDF2 per page / UTF-8 for text files)   │
│  2. Chunking          (~900 tokens, overlapping windows)         │
│  3. Embedding         (nomic-embed-text via Ollama, batched)     │
│  4. Storage           (chunks + vector(768) → PostgreSQL)        │
└───────────────────────────────┬──────────────────────────────────┘
                                │  Document status: INDEXED
                                ▼
User asks a question
        │
        ▼
┌──────────────────────────────────────────────────────────────────┐
│  QUERY PIPELINE                                                    │
│                                                                    │
│  1. Embed question    (same nomic-embed-text model)              │
│  2. Vector search     (pgvector cosine similarity, top-K=6)      │
│  3. Similarity gate   (threshold=0.30 — below → NOT_FOUND)       │
│  4. Context assembly  (ranked chunks → source block)             │
│  5. LLM generation    (llama3.1:8b / llama3.3:70b via Ollama)   │
│  6. Citation extract  (chunk id, page, match %, snippet)         │
│  7. Persist           (message + citations_json → PostgreSQL)    │
└───────────────────────────────┬──────────────────────────────────┘
                                │
                                ▼
           Answer with inline citations returned to UI
```

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    Browser (React 18)                            │
│                                                                   │
│  React Router v7 · AuthContext (JWT) · UploadDropzone           │
│  DocumentList · ChatPage (model toggle, temp slider, citations)  │
└────────────────────┬────────────────────────────────────────────┘
                     │  /api/* (Vite dev proxy → :8000)
                     ▼
┌─────────────────────────────────────────────────────────────────┐
│                    FastAPI (Uvicorn, async)                       │
│                                                                   │
│  Routers: auth · documents · indexing · rag · chats · health    │
│                                                                   │
│  Application layer (use cases):                                  │
│    IndexDocumentUseCase   ← ingestion orchestration              │
│    RAGQueryUseCase        ← query orchestration                  │
│                                                                   │
│  Infrastructure:                                                  │
│    OllamaEmbeddingClient  ·  OllamaLLMClient                    │
│    LlamaCppLLMClient      ·  PgvectorRetriever                   │
│    ChunkingService        ·  TextExtractor  ·  FileStorage       │
└────────────────────┬────────────────────────────────────────────┘
                     │  asyncpg (SQLAlchemy async)
                     ▼
┌─────────────────────────────────────────────────────────────────┐
│                    PostgreSQL + pgvector                          │
│                                                                   │
│  users · documents · chunks · chunk_embeddings (vector(768))    │
│  chats · messages (citations_json JSONB)                         │
└─────────────────────────────────────────────────────────────────┘
                     │  httpx (async)
                     ▼
┌─────────────────────────────────────────────────────────────────┐
│   Ollama (localhost:11434)                                        │
│   • nomic-embed-text  — embedding generation                     │
│   • llama3.1:8b       — fast inference                          │
│   • llama3.3:70b      — accurate inference                      │
└─────────────────────────────────────────────────────────────────┘
```

---

## Tech Stack

### Backend

| Component | Technology |
|-----------|-----------|
| Web framework | FastAPI + Uvicorn (async) |
| ORM | SQLAlchemy 2.0 (async engine via `asyncpg`) |
| Migrations | Alembic |
| Database | PostgreSQL + `pgvector` extension |
| Embeddings | Ollama `nomic-embed-text` (768-dim, batched) |
| LLM | Ollama (`llama3.1:8b` / `llama3.3:70b`) or llama.cpp |
| PDF parsing | PyPDF2 |
| Auth | bcrypt + python-jose (JWT HS256) |
| HTTP client | httpx (async) |
| Settings | pydantic-settings + `.env` |

### Frontend

| Component | Technology |
|-----------|-----------|
| Framework | React 18 |
| Routing | React Router DOM v7 |
| Build tool | Vite 6 |
| State | React Context API (AuthContext) |
| API | Typed fetch wrapper with JWT injection |

---

## Database Schema

```
users
  └── documents  (status: UPLOADED → INDEXING → INDEXED | FAILED)
        └── chunks
              └── chunk_embeddings  (vector(768), pgvector)

users
  └── chats
        └── messages  (role, content, citations_json JSONB, answer_status)
```

| Table | Key Columns |
|-------|-------------|
| `users` | `id` (UUID), `email`, `password_hash` |
| `documents` | `id`, `user_id`, `original_filename`, `stored_path`, `mime_type`, `status`, `enabled` |
| `chunks` | `id`, `doc_id`, `chunk_index`, `page_number`, `text` |
| `chunk_embeddings` | `id`, `chunk_id`, `embedding` (vector(768)), `model_name` |
| `chats` | `id`, `user_id`, `title`, `created_at`, `updated_at` |
| `messages` | `id`, `chat_id`, `role`, `content`, `citations_json` (JSONB), `answer_status`, `selected_doc_ids_json` |

**3 Alembic migrations** bootstrap the full schema including pgvector extension enablement.

---

## API Reference

| Method | Route | Auth | Description |
|--------|-------|------|-------------|
| `POST` | `/api/auth/signup` | Public | Create user account |
| `POST` | `/api/auth/login` | Public | Authenticate, return JWT |
| `POST` | `/api/documents/upload` | JWT | Upload document (multipart/form-data) |
| `GET` | `/api/documents` | JWT | List user's documents with status |
| `PATCH` | `/api/documents/:id/toggle` | JWT | Enable/disable document from search scope |
| `DELETE` | `/api/documents/:id` | JWT | Delete document and all associated chunks |
| `POST` | `/api/indexing/:doc_id` | JWT | Trigger ingestion pipeline (re-index deletes old chunks) |
| `POST` | `/api/rag/query` | JWT | Execute RAG query, returns answer + citations |
| `GET` | `/api/chats` | JWT | List chat sessions |
| `POST` | `/api/chats` | JWT | Create new chat |
| `GET` | `/api/chats/:id/messages` | JWT | Retrieve message history for a chat |
| `GET` | `/api/health` | Public | Service health check |

### RAG Query Request

```json
{
  "chat_id": "uuid",
  "question": "What is the conclusion of the paper?",
  "scope": { "mode": "all" },
  "model_mode": "fast",
  "temperature": 0.7
}
```

### RAG Query Response

```json
{
  "answer": "The paper concludes that...",
  "answer_status": "ANSWERED",
  "citations": [
    {
      "chunk_id": "uuid",
      "document_name": "paper.pdf",
      "page_number": 12,
      "similarity_score": 0.87,
      "snippet": "In conclusion, the results demonstrate..."
    }
  ]
}
```

---

## Getting Started

### Prerequisites

- Python 3.11+
- PostgreSQL with `pgvector` extension
- Ollama with `nomic-embed-text` and `llama3.1:8b` pulled
- Node.js 20+

### 1. Pull Required Ollama Models

```bash
ollama pull nomic-embed-text
ollama pull llama3.1:8b          # fast mode
ollama pull llama3.3:70b         # accurate mode (optional, requires ~40GB RAM)
```

### 2. Clone & Set Up Backend

```bash
git clone https://github.com/SriRammSS/rag-local-app.git
cd rag-local-app/apps/api

python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### 3. Configure Environment

Create `apps/api/.env`:

```ini
APP_NAME=RAG Local API
APP_ENV=development
DATABASE_URL=postgresql+asyncpg://postgres:<password>@localhost:5432/rag_local
JWT_SECRET_KEY=<generate-with: openssl rand -hex 32>
JWT_ALGORITHM=HS256
JWT_ACCESS_TOKEN_EXPIRE_MINUTES=1440

OLLAMA_BASE_URL=http://localhost:11434
OLLAMA_EMBED_MODEL=nomic-embed-text

LLM_RUNTIME=ollama          # "ollama" | "llamacpp"
LLM_BASE_URL=http://localhost:11434
LLM_FAST_MODEL=llama3.1:8b
LLM_ACCURATE_MODEL=llama3.3:70b

RAG_TOP_K=6
RAG_SIM_THRESHOLD=0.30
CORS_ORIGINS=["http://localhost:5173"]
```

### 4. Initialize Database

```bash
# Create the database and enable pgvector
createdb rag_local

cd apps/api
alembic upgrade head
```

### 5. Start Backend & Frontend

```bash
# Terminal 1 — Backend
cd apps/api && uvicorn main:app --reload --port 8000

# Terminal 2 — Frontend
cd apps/web && npm install && npm run dev
```

Frontend at `http://localhost:5173` · API at `http://localhost:8000`

---

## Project Structure

```
rag-local-app/
├── apps/
│   ├── api/                                    # FastAPI backend (clean architecture)
│   │   ├── main.py                             # App factory — router registration
│   │   ├── core/                               # Config, DB session, JWT helpers
│   │   ├── domain/models/                      # SQLAlchemy ORM models
│   │   ├── application/
│   │   │   ├── interfaces/                     # Abstract EmbeddingClient, LLMClient
│   │   │   └── use_cases/                      # IndexDocumentUseCase, RAGQueryUseCase,
│   │   │                                       # AuthUseCase, DocumentUseCases, ChatUseCases
│   │   ├── infrastructure/
│   │   │   ├── embedding/ollama_client.py      # Batched /api/embed calls
│   │   │   ├── llm/                            # OllamaLLMClient · LlamaCppLLMClient
│   │   │   ├── pgvector_retriever.py           # Cosine similarity search
│   │   │   ├── chunking_service.py             # Overlapping word-based chunker
│   │   │   ├── text_extractor.py               # PyPDF2 + plain text
│   │   │   └── repositories/                   # Async SQLAlchemy repos
│   │   └── interfaces/api/                     # FastAPI routers + Pydantic schemas
│   │       └── auth · documents · indexing · rag · chats · health
│   │
│   └── web/                                    # React 18 SPA
│       └── src/
│           ├── pages/                          # LoginPage, SignupPage, DashboardPage,
│           │                                   # ChatLayout, ChatPage
│           ├── components/                     # Navbar, DocumentList, UploadDropzone,
│           │                                   # ChatSidebar, ProtectedRoute
│           ├── contexts/AuthContext.jsx         # JWT token state
│           └── lib/api.js                      # Fetch wrapper with auth injection
│
└── storage/uploads/                            # Local file storage (gitignored)
    └── <user_id>/<doc_id>/<filename>
```

---

## Architectural Decisions

| Decision | Rationale |
|----------|-----------|
| **Clean layered architecture** | Strict separation of domain, application, infrastructure, and interface layers — use cases are testable without HTTP or database dependencies |
| **Abstract LLM interface** | `LLMClient` ABC decouples use cases from Ollama or llama.cpp specifics; swapping inference backends requires no changes to the RAG pipeline |
| **pgvector over a dedicated vector DB** | Keeps the stack to a single database engine; cosine similarity on 768-dim vectors at this scale does not require a dedicated ANN index service |
| **Similarity threshold gate** | Prevents the LLM from hallucinating answers when no relevant context exists — returns a clean NOT_FOUND rather than generating low-confidence responses |
| **Document-scoped queries** | `scope.mode: "doc"` restricts vector search to specific document IDs, enabling precise targeted questions without cross-document noise |
| **Re-index deletes old chunks** | Triggering indexing on an already-indexed document atomically deletes previous embeddings before re-generating — prevents stale vector contamination |

---

## Author

**Sri Ramm Sekar Sasirekha**

[![GitHub](https://img.shields.io/badge/GitHub-SriRammSS-181717?style=flat&logo=github)](https://github.com/SriRammSS)

---

<div align="center">
<sub>Built to demonstrate production-grade RAG system design with clean architecture, local LLM inference, and pgvector-powered semantic search — entirely on-premise.</sub>
</div>
