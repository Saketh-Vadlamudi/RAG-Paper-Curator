# RAG Paper Curator
## An end-to-end arXiv research assistant

<div align="center">
  <h3>Search, explore, and ask questions across arXiv papers</h3>
  <p>A personal RAG project built around automated ingestion, hybrid retrieval, local generation, caching, and observability.</p>
</div>

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.12+-blue.svg" alt="Python Version">
  <img src="https://img.shields.io/badge/FastAPI-0.115+-green.svg" alt="FastAPI">
  <img src="https://img.shields.io/badge/OpenSearch-2.19-orange.svg" alt="OpenSearch">
  <img src="https://img.shields.io/badge/Docker-Compose-blue.svg" alt="Docker">
  <img src="https://img.shields.io/badge/Status-Active-brightgreen.svg" alt="Status">
</p>

<br>

<p align="center">
  <a href="#-about-this-project">
    <img src="static/mother_of_ai_project_rag_architecture.gif" alt="RAG Paper Curator architecture" width="700">
  </a>
</p>

## 📖 About This Project

RAG Paper Curator is my end-to-end research assistant for arXiv papers. It collects paper metadata and PDFs, extracts structured content, indexes the resulting text, retrieves relevant passages, and generates source-backed answers with a local language model.

The project uses BM25 and vector retrieval together instead of relying on a single search method. It also includes the infrastructure needed to run the workflow locally: scheduled ingestion with Airflow, PostgreSQL storage, OpenSearch indexing, Ollama generation, Redis caching, and Langfuse tracing.

### **What It Does**

- Fetches papers from the arXiv API with rate limiting and retries
- Parses scientific PDFs with Docling
- Stores paper metadata and processed content in PostgreSQL
- Splits papers into section-aware, overlapping chunks
- Supports BM25 and hybrid search through OpenSearch
- Generates grounded answers with a local Ollama model
- Streams responses to a Gradio interface or API client
- Caches repeated requests in Redis
- Traces RAG operations with a self-hosted Langfuse instance

---

## 🚀 Quick Start

### **📋 Prerequisites**

- Docker Desktop with Docker Compose
- Python 3.12
- [uv](https://docs.astral.sh/uv/getting-started/installation/)
- At least 8 GB of available memory and 20 GB of disk space
- A Jina AI API key for vector embeddings and hybrid search

### **⚡ Get Started**

```bash
# Clone the repository
git clone https://github.com/Saketh-Vadlamudi/RAG-Paper-Curator.git
cd RAG-Paper-Curator

# Create the local configuration
cp .env.example .env
# Set JINA_API_KEY in .env to enable hybrid search

# Install Python dependencies
uv sync

# Start the application stack
docker compose up --build -d

# Download the default local model
docker exec rag-ollama ollama pull llama3.2:1b

# Verify the API and its dependencies
curl http://localhost:8000/api/v1/health
```

Start the Gradio interface separately after the Docker services are healthy:

```bash
uv run python gradio_launcher.py
```

### **📚 Implementation Stages**

| Stage | Focus | Main Result |
|------|-------|-------------|
| **1** | Infrastructure | Containerized API, database, search, orchestration, and local LLM services |
| **2** | Data ingestion | Automated arXiv metadata and PDF processing pipeline |
| **3** | Keyword retrieval | BM25 search with filters and relevance scoring |
| **4** | Hybrid retrieval | Section-aware chunking, embeddings, and reciprocal rank fusion |
| **5** | RAG application | Local answer generation, streaming API, and Gradio UI |
| **6** | Operations | Langfuse tracing and Redis response caching |

The notebooks under `notebooks/week1` through `notebooks/week6` document the implementation and provide focused experiments for each stage.

### **📊 Local Services**

| Service | URL | Purpose |
|---------|-----|---------|
| **API documentation** | http://localhost:8000/docs | Interactive FastAPI reference |
| **Gradio interface** | http://localhost:7861 | Browser-based RAG client |
| **Langfuse** | http://localhost:3000 | RAG traces and latency inspection |
| **Airflow** | http://localhost:8080 | Ingestion workflow management |
| **OpenSearch Dashboards** | http://localhost:5601 | Search index inspection |

The local Airflow container creates the development login `admin` / `admin` during initialization.

---

## 📚 Stage 1: Infrastructure Foundation ✅

The first stage establishes the local service stack and connects the FastAPI application to its supporting systems.

### **🏗️ Architecture Overview**

<p align="center">
  <img src="static/week1_infra_setup.png" alt="Infrastructure setup" width="800">
</p>

**Core components:**

- FastAPI for the HTTP API and service health checks
- PostgreSQL 16 for paper metadata and content
- OpenSearch 2.19 for keyword and vector retrieval
- Apache Airflow 2.10.3 for scheduled ingestion
- Ollama for local language-model inference
- Docker Compose for service orchestration

### **📓 Reference Notebook**

```bash
uv run jupyter notebook notebooks/week1/week1_setup.ipynb
```

---

## 📚 Stage 2: Data Ingestion Pipeline ✅

This stage connects the arXiv API to the storage and indexing workflow. Papers are fetched with retry and rate-limit handling, PDFs are parsed with Docling, and the processed records are persisted for retrieval.

### **🏗️ Architecture Overview**

<p align="center">
  <img src="static/week2_data_ingestion_flow.png" alt="Data ingestion architecture" width="800">
</p>

**Pipeline components:**

- `ArxivClient` for metadata discovery and PDF downloads
- `PDFParserService` for structured scientific-document extraction
- `MetadataFetcher` for coordinating fetch, parse, and persistence operations
- Airflow DAGs for repeatable ingestion runs
- PostgreSQL and OpenSearch for storage and retrieval

### **📓 Reference Notebook**

```bash
uv run jupyter notebook notebooks/week2/week2_arxiv_integration.ipynb
```

---

## 📚 Stage 3: BM25 Keyword Search ✅

BM25 provides the exact-term retrieval layer for paper titles, abstracts, authors, and technical vocabulary. The search service includes category filters, pagination, relevance sorting, and date sorting.

### **🏗️ Architecture Overview**

<p align="center">
  <img src="static/week3_opensearch_flow.png" alt="OpenSearch BM25 architecture" width="800">
</p>

**Main implementation:**

- `src/services/opensearch/client.py` manages indices and search requests
- `src/services/opensearch/query_builder.py` builds BM25 queries
- `src/routers/search.py` contains the standalone keyword-search router
- `notebooks/week3` contains indexing and retrieval experiments

### **📓 Reference Notebook**

```bash
uv run jupyter notebook notebooks/week3/week3_opensearch.ipynb
```

---

## 📚 Stage 4: Chunking and Hybrid Search ✅

Hybrid retrieval combines BM25 matches with vector similarity. Papers are divided into section-aware chunks, embedded with Jina AI, and ranked through reciprocal rank fusion in OpenSearch.

### **🏗️ Architecture Overview**

<p align="center">
  <img src="static/week4_hybrid_opensearch.png" alt="Hybrid retrieval architecture" width="800">
</p>

**Main implementation:**

- `src/services/indexing/text_chunker.py` creates overlapping, section-aware chunks
- `src/services/indexing/hybrid_indexer.py` prepares documents for the hybrid index
- `src/services/embeddings` integrates Jina embeddings
- `src/routers/hybrid_search.py` exposes BM25 and hybrid modes through one endpoint

### **📓 Reference Notebook**

```bash
uv run jupyter notebook notebooks/week4/week4_hybrid_search.ipynb
```

### **Search Request Example**

```bash
curl -X POST http://localhost:8000/api/v1/hybrid-search/ \
  -H "Content-Type: application/json" \
  -d '{
    "query": "efficient transformer attention",
    "use_hybrid": true,
    "size": 5,
    "categories": ["cs.AI"]
  }'
```

Setting `use_hybrid` to `false` uses BM25 without generating a query embedding.

---

## 📚 Stage 5: RAG Pipeline and Interface ✅

The RAG layer retrieves relevant chunks, constructs a focused prompt, and sends it to an Ollama model. Both complete JSON responses and server-sent streaming responses are available.

### **🏗️ Architecture Overview**

<p align="center">
  <img src="static/week5_complete_rag.png" alt="Complete RAG architecture" width="900">
</p>

**Main implementation:**

- `src/routers/ask.py` provides standard and streaming RAG endpoints
- `src/services/ollama` manages local generation and prompt construction
- `src/gradio_app.py` provides the interactive research interface
- Source URLs are returned with generated answers

### **📓 Reference Notebook**

```bash
uv run jupyter notebook notebooks/week5/week5_complete_rag_system.ipynb
```

### **Question Example**

```bash
curl -X POST http://localhost:8000/api/v1/ask \
  -H "Content-Type: application/json" \
  -d '{
    "query": "What are the main approaches to efficient attention?",
    "top_k": 3,
    "use_hybrid": true,
    "model": "llama3.2:1b",
    "categories": ["cs.AI"]
  }'
```

The streaming equivalent is available at `POST /api/v1/stream` and returns server-sent events as answer tokens are generated.

---

## 📚 Stage 6: Monitoring and Caching ✅

The final stage adds operational visibility and avoids repeated generation work. Langfuse records the retrieval and generation path, while Redis stores exact-match responses for a configurable period.

### **🏗️ Architecture Overview**

<p align="center">
  <img src="static/week6_monitoring_and_caching.png" alt="Monitoring and caching architecture" width="900">
</p>

**Main implementation:**

- `src/services/langfuse` records request, embedding, search, prompt, and generation spans
- `src/services/cache` stores and retrieves exact-match RAG responses
- Cache failures degrade gracefully to the normal retrieval and generation path
- The local Langfuse deployment uses dedicated PostgreSQL and ClickHouse services

### **📓 Reference Notebook**

```bash
uv run jupyter notebook notebooks/week6/week6_cache_testing.ipynb
```

---

## ⚙️ Configuration Management

### **Environment Configuration**

Configuration is loaded from a single `.env` file. The checked-in `.env.example` contains the complete set of application defaults and service connection values.

```bash
cp .env.example .env
```

### **Key Configuration Groups**

| Prefix or variable | Purpose |
|--------------------|---------|
| `ARXIV__*` | Query limits, categories, retries, and download concurrency |
| `PDF_PARSER__*` | Page limits, file-size limits, OCR, and table extraction |
| `OPENSEARCH__*` | Index names, vector dimensions, and hybrid-search settings |
| `CHUNKING__*` | Chunk size, overlap, and section-aware splitting |
| `JINA_API_KEY` | Embedding access for vector and hybrid retrieval |
| `OLLAMA_HOST`, `OLLAMA_MODEL` | Local generation service and default model |
| `LANGFUSE__*` | Self-hosted tracing connection and credentials |
| `REDIS__*` | Cache connection and response TTL |

The Docker Compose file overrides service hosts with container names. Local Python commands use the host-facing values from `.env`.

---

## 🔧 Reference and Development Guide

### **🛠️ Technology Stack**

| Component | Role |
|-----------|------|
| **FastAPI** | API and dependency lifecycle |
| **PostgreSQL 16** | Paper metadata and parsed content |
| **OpenSearch 2.19** | BM25, vector, and hybrid retrieval |
| **Apache Airflow 2.10.3** | Scheduled ingestion workflows |
| **Docling** | Scientific PDF parsing |
| **Jina AI** | Text and query embeddings |
| **Ollama** | Local LLM inference |
| **Gradio** | Interactive browser interface |
| **Redis** | Exact-match response cache |
| **Langfuse** | RAG tracing and observability |

Development tooling includes uv, Ruff, MyPy, Pytest, and Docker Compose.

### **🏗️ Project Structure**

```text
RAG-Paper-Curator/
├── src/
│   ├── main.py                 # FastAPI application and service lifecycle
│   ├── routers/                # Health, search, and RAG endpoints
│   ├── services/               # Ingestion, retrieval, generation, cache, and tracing
│   ├── repositories/           # Database access layer
│   ├── schemas/                # API and service data models
│   ├── db/                     # Database setup
│   └── gradio_app.py           # Interactive client
├── airflow/
│   └── dags/                   # arXiv ingestion workflow
├── notebooks/                  # Implementation notes and experiments by stage
├── tests/                      # Automated test suite
├── static/                     # Architecture diagrams
├── compose.yml                 # Local service stack
├── Makefile                    # Common development commands
└── pyproject.toml              # Python project configuration
```

### **📡 API Endpoints**

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/v1/health` | GET | Application and dependency health |
| `/api/v1/hybrid-search/` | POST | BM25 or hybrid paper search |
| `/api/v1/ask` | POST | Source-backed RAG response |
| `/api/v1/stream` | POST | Streaming RAG response |
| `/docs` | GET | Interactive OpenAPI documentation |

### **🔧 Essential Commands**

```bash
make start       # Build and start the service stack
make status      # Show container status
make logs        # Follow container logs
make test        # Run the test suite
make lint        # Run Ruff and MyPy
make format      # Format Python code
make stop        # Stop the service stack

curl http://localhost:8000/api/v1/health  # Check application health
```

Direct equivalents remain available through `docker compose`, `uv run pytest`, and `uv run ruff`.

---

## 🛠️ Troubleshooting

- **API unavailable:** Check `docker compose ps` and `docker compose logs api`.
- **Hybrid search falls back to BM25:** Confirm that `JINA_API_KEY` is set and restart the API container.
- **Ollama model missing:** Run `docker exec rag-ollama ollama pull llama3.2:1b`.
- **OpenSearch unavailable:** Allow the container to finish its startup checks, then inspect `docker compose logs opensearch`.
- **Airflow login unavailable:** Use the local `admin` / `admin` credentials and inspect `docker compose logs airflow`.
- **Port conflict:** Stop the local process using ports `3000`, `5432`, `5601`, `6379`, `7861`, `8000`, `8080`, `9200`, or `11434`.
- **Resource pressure:** Increase the memory assigned to Docker Desktop or stop optional services.

A complete local reset removes persisted container data:

```bash
docker compose down --volumes
docker compose up --build -d
```

---

## 💰 Running Costs

The API, database, search engine, orchestration, LLM, cache, and monitoring stack all run locally. Hybrid search uses Jina AI embeddings and may depend on the limits of the configured Jina account. No paid hosted LLM is required by the default setup.

---

## 📄 License

This project is available under the [MIT License](LICENSE).
