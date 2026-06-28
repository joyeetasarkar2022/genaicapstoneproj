# RAG Capstone Agent

This capstone implements a FastAPI user interface and API for document ingestion, semantic search, and grounded RAG answers using LlamaIndex, Chroma, and a single ReAct-style agent.

## What is included

- FastAPI web UI with three screens:
  - Query screen: question textbox, prompt file upload, multiple-question file upload, and document upload.
  - Health screen: API, doc folder, RAG vector count, Chroma collection, and agent status.
  - Upload screen: upload documents to `doc/` and ingest into Chroma.
- Document ingestion for PDF, TXT, Markdown, CSV, Excel, JSON, YAML, and DOCX.
- Chunking with about 200 tokens, overlap, and sentence/paragraph boundaries.
- Chroma vector database under `data/vector_db`.
- LlamaIndex pipeline and optional LlamaIndex ReActAgent when `OPENAI_API_KEY` is configured.
- Guardrails for file type, file size, prompt injection patterns, path traversal, and grounded output fallback.
- Render deployment files: `render.yaml` and `Dockerfile`.

## Quick start

```bash
python -m venv .venv
. .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
cp .env.example .env
# Optional: edit .env and set OPENAI_API_KEY for LLM-based answers and the ReActAgent.
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

Open:

- UI: http://127.0.0.1:8000/
- Upload: http://127.0.0.1:8000/upload-ui
- Health: http://127.0.0.1:8000/health-ui
- OpenAPI: http://127.0.0.1:8000/docs

## Ingest documents from the doc folder

```bash
python scripts/ingest_folder.py --folder doc
```

## API examples

Upload and index:

```bash
curl -X POST http://127.0.0.1:8000/api/v1/upload \
  -F "ingest=true" \
  -F "files=@doc/project_requirement.docx"
```

Query with JSON:

```bash
curl -X POST http://127.0.0.1:8000/api/v1/query-json \
  -H "Content-Type: application/json" \
  -d '{"question":"What formats are supported for ingestion?","top_k":5}'
```

## Script placement map

| File | Place at folder | Purpose |
| --- | --- | --- |
| `app/main.py` | `app/` | FastAPI app factory, middleware, static files, startup, exception handler. |
| `app/api/routes.py` | `app/api/` | API endpoints and three UI screens. |
| `app/core/config.py` | `app/core/` | Environment settings and directory initialization. |
| `app/core/security.py` | `app/core/` | File validation, size limits, path-safe filenames, prompt guardrails. |
| `app/core/logging_config.py` | `app/core/` | Console and rotating file logging. |
| `app/models/schemas.py` | `app/models/` | Pydantic request and response schemas. |
| `app/services/document_loader.py` | `app/services/` | Extracts text from pdf/txt/csv/excel/json/yaml/docx. |
| `app/services/chunker.py` | `app/services/` | Sentence and overlap chunking for semantic search. |
| `app/services/vector_store.py` | `app/services/` | Chroma PersistentClient and LlamaIndex vector index. |
| `app/services/rag_pipeline.py` | `app/services/` | Ingestion, retrieval, grounded answer generation. |
| `app/services/agent.py` | `app/services/` | Single LlamaIndex ReActAgent wrapper with fallback. |
| `app/services/health.py` | `app/services/` | Health response for API/UI. |
| `app/templates/*.html` | `app/templates/` | Jinja2 UI screens. |
| `app/static/css/styles.css` | `app/static/css/` | UI styling. |
| `scripts/ingest_folder.py` | `scripts/` | CLI folder ingestion. |
| `scripts/run_local.sh` | `scripts/` | Local venv install and app startup. |
| `requirements.txt` | project root | Python dependencies for `pip install -r requirements.txt`. |
| `render.yaml` | project root | Render Blueprint deployment. |
| `Dockerfile` | project root | Optional container deployment. |
| `.env.example` | project root | Environment template. |
| `docs/*.md` | `docs/` | Architecture, implementation, API, security, limitations, deployment documentation. |
| Runtime user documents | `doc/` | User-uploaded documents and source files for ingestion. |

## Notes

The app works without an OpenAI key by returning extractive answers from retrieved chunks. Set `OPENAI_API_KEY` to enable LLM synthesis and the LlamaIndex ReActAgent.
