# RAG Enterprise Framework

A production-ready Retrieval Augmented Generation (RAG) framework designed for enterprise deployment with vector database integration, batch document processing, security controls, and multi-format ingestion.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    RAG Enterprise Framework                    │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────┐    ┌──────────────┐    ┌──────────────────┐   │
│  │ Document │───>│  Ingestion   │───>│  Vector Database │   │
│  │  Upload  │    │  Pipeline    │    │   (ChromaDB)     │   │
│  └──────────┘    └──────────────┘    └────────┬─────────┘   │
│                                                │              │
│  ┌──────────┐    ┌──────────────┐    ┌────────v─────────┐   │
│  │  User    │───>│   Query      │───>│   Retrieval +    │   │
│  │  Query   │    │   Engine     │    │   LLM Generation │   │
│  └──────────┘    └──────────────┘    └──────────────────┘   │
│                                                               │
├─────────────────────────────────────────────────────────────┤
│  Security: JWT Auth | Batch Processing | Multi-format Input  │
└─────────────────────────────────────────────────────────────┘
```

---

## Features

- Multi-format document ingestion — PDF, DOCX, TXT, Markdown, HTML with metadata preservation
- Batch processing with configurable batch sizes, progress tracking, and callbacks
- ChromaDB vector database with persistent storage and collection management
- JWT-based authentication with collection-level access control
- Streaming LLM responses with token tracking
- Multi-modal RAG — OCR (Tesseract), table extraction (Camelot), image analysis
- Reference extraction with source tracking and document name parsing
- Conversation management with isolated sessions
- Production ready — health checks, error handling, retry logic, graceful degradation

---

## Quick Start

```bash
git clone https://github.com/mkomera/rag-enterprise-framework.git
cd rag-enterprise-framework

pip install -r requirements.txt

cp .env.example .env
# Edit .env with your LLM API keys and settings

uvicorn app.main:app --host 0.0.0.0 --port 8000
```

---

## Project Structure

```
rag-enterprise-framework/
├── app/
│   ├── main.py                 # FastAPI application entry point
│   ├── config.py               # Configuration management
│   ├── rag/
│   │   ├── service.py          # Core RAG service (ingest + query)
│   │   ├── embeddings.py       # Embedding generation
│   │   ├── retriever.py        # Vector similarity search
│   │   └── generator.py        # LLM response generation
│   ├── ingestion/
│   │   ├── pipeline.py         # Document processing pipeline
│   │   ├── pdf_processor.py    # PDF with OCR + table extraction
│   │   ├── chunker.py          # Intelligent text chunking
│   │   └── metadata.py         # Metadata extraction
│   ├── storage/
│   │   ├── chromadb_client.py  # ChromaDB vector store
│   │   └── collection_mgr.py  # Collection lifecycle management
│   ├── security/
│   │   ├── auth.py             # JWT authentication
│   │   └── access_control.py   # Collection-level permissions
│   └── llm/
│       ├── provider.py         # Multi-provider LLM interface
│       ├── streaming.py        # Chunked streaming responses
│       └── token_tracker.py    # Usage tracking
├── tests/
├── docs/
├── requirements.txt
├── Dockerfile
├── docker-compose.yml
└── README.md
```

---

## Usage

### Ingest Documents

```python
from app.rag.service import RAGService

rag = RAGService(collection_name="engineering-docs")

# Single document
rag.ingest_document("path/to/document.pdf", metadata={"source": "manual"})

# Batch ingestion with progress
rag.ingest_batch(
    documents=["doc1.pdf", "doc2.docx", "doc3.md"],
    batch_size=100,
    on_progress=lambda p: print(f"Progress: {p}%")
)
```

### Query with RAG

```python
response = rag.query("What are the design specifications for component X?")
print(response.answer)
print(response.sources)

# Streaming
async for chunk in rag.query_stream("Explain the architecture"):
    print(chunk, end="", flush=True)
```

---

## Configuration

```env
LLM_PROVIDER=bedrock          # bedrock | openai | azure
LLM_MODEL=claude-3-sonnet
EMBEDDING_MODEL=all-MiniLM-L6-v2
CHROMADB_PATH=./data/chromadb
CHUNK_SIZE=1000
CHUNK_OVERLAP=200
BATCH_SIZE=100
JWT_SECRET=your-secret-key
AUTH_ENABLED=true
```

---

## Deployment

```yaml
services:
  rag-api:
    build: .
    ports:
      - "8000:8000"
    environment:
      - LLM_PROVIDER=bedrock
      - AUTH_ENABLED=true
    volumes:
      - ./data:/app/data
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
```

---

## Performance

| Metric | Value |
|--------|-------|
| Ingestion Speed | ~100 docs/min (batch mode) |
| Query Latency | < 2s (including LLM generation) |
| Vector Search | < 100ms |
| Concurrent Users | 50+ simultaneous queries |
| Document Formats | PDF, DOCX, TXT, MD, HTML |

---

## License

MIT
