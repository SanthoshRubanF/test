# RAG System Architecture

## 🏛️ System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                         CLIENT / USER                            │
│                    (Browser / API Client)                        │
└───────────────────────────┬─────────────────────────────────────┘
                            │ HTTP/HTTPS
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                      API LAYER (FastAPI)                         │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  app.py - Main FastAPI Application                       │  │
│  │  - /upload      - Upload documents                       │  │
│  │  - /query       - Query RAG system (streaming)           │  │
│  │  - /documents   - List/manage documents                  │  │
│  │  - /conversations - Conversation management              │  │
│  └──────────────────────────────────────────────────────────┘  │
└───────────────────────────┬─────────────────────────────────────┘
                            │
            ┌───────────────┼───────────────┐
            │               │               │
            ▼               ▼               ▼
┌─────────────────┐ ┌──────────────┐ ┌──────────────────┐
│  CONFIG LAYER   │ │ SERVICE LAYER│ │  UTILS LAYER     │
├─────────────────┤ ├──────────────┤ ├──────────────────┤
│ settings.py     │ │ database.py  │ │ embeddings.py    │
│ - Database cfg  │ │ - DB ops     │ │ - Gemini embed   │
│ - GCS config    │ │ - Connection │ │ - Query expand   │
│ - AI config     │ │   pooling    │ │                  │
│ - Server cfg    │ │ - CRUD ops   │ │ file_processing  │
│ - RAG config    │ │              │ │ - Text extract   │
│                 │ │ storage.py   │ │ - File validate  │
│ Pydantic        │ │ - GCS ops    │ │                  │
│ validation      │ │ - Upload     │ │ logging_config   │
│ Type-safe       │ │ - Download   │ │ - Setup logs     │
└─────────────────┘ │ - Delete     │ │ - Get logger     │
                    │              │ └──────────────────┘
                    │ rag_service* │
                    │ - Vector ops │
                    │ - Query proc │
                    │ - Ingest     │
                    └──────────────┘
                            │
            ┌───────────────┼───────────────┐
            │               │               │
            ▼               ▼               ▼
┌──────────────────┐ ┌─────────────┐ ┌──────────────────┐
│   DATA LAYER     │ │   AI LAYER  │ │  STORAGE LAYER   │
├──────────────────┤ ├─────────────┤ ├──────────────────┤
│  PostgreSQL      │ │ Google AI   │ │  Google Cloud    │
│  + pgvector      │ │             │ │  Storage (GCS)   │
│                  │ │ Gemini      │ │                  │
│ Tables:          │ │ - 2.5 Flash │ │ Bucket:          │
│ - documents      │ │ - Embedding │ │ - Raw files      │
│ - conversations  │ │             │ │ - Uploaded docs  │
│ - vector stores  │ │ LlamaIndex  │ │                  │
│   (per document) │ │ - Indexing  │ │                  │
│                  │ │ - Retrieval │ │                  │
└──────────────────┘ └─────────────┘ └──────────────────┘
```

* rag_service.py to be created in Phase 2

## 📊 Data Flow Diagrams

### Document Upload Flow

```
User → FastAPI /upload
  │
  ├─→ Validate file type (utils)
  │
  ├─→ Upload to GCS (storage service)
  │     └─→ Returns blob_name
  │
  ├─→ Create metadata (models)
  │
  ├─→ Save to DB (database service)
  │     └─→ documents table
  │
  └─→ Background: Ingest (RAG service*)
        ├─→ Download from GCS
        ├─→ Extract text (utils)
        ├─→ Create embeddings (AI layer)
        ├─→ Store in vector DB (pgvector)
        └─→ Update status in DB
```

### Query Flow (Conversational RAG)

```
User → FastAPI /query (SSE streaming)
  │
  ├─→ Load conversation (database service)
  │
  ├─→ Expand query (utils - Gemini)
  │
  ├─→ RAG Service*:
  │     ├─→ Retrieve relevant docs (vector search)
  │     ├─→ BM25 search (hybrid)
  │     ├─→ Cross-encoder re-ranking
  │     └─→ Generate answer (Gemini)
  │
  ├─→ Stream response (SSE)
  │
  └─→ Save conversation (database service)
```

### Configuration Loading Flow

```
Application Start
  │
  ├─→ Load .env file (dotenv)
  │
  ├─→ config/settings.py
  │     ├─→ Parse environment variables
  │     ├─→ Validate with Pydantic
  │     ├─→ Set defaults
  │     └─→ Create settings object
  │
  ├─→ Initialize Services
  │     ├─→ Database Service (connection pool)
  │     ├─→ Storage Service (GCS client)
  │     └─→ AI Services (Gemini)
  │
  └─→ Ready to serve requests
```

## 🔄 Component Interactions

### Service Dependencies

```
┌─────────────────────────────────────────────┐
│              FastAPI App                    │
└─────────┬───────────────────────┬───────────┘
          │                       │
          ▼                       ▼
┌─────────────────┐     ┌─────────────────────┐
│  Config Module  │────▶│  Service Modules    │
│  - settings     │     │  - db_service       │
└─────────────────┘     │  - gcs_service      │
                        │  - rag_service*     │
                        └──────────┬──────────┘
                                   │
                                   ▼
                        ┌──────────────────────┐
                        │   Utils Modules      │
                        │   - embeddings       │
                        │   - file_processing  │
                        │   - logging          │
                        └──────────────────────┘
                                   │
                                   ▼
                        ┌──────────────────────┐
                        │   Models Module      │
                        │   - DocumentMetadata │
                        │   - Conversation     │
                        └──────────────────────┘
```

### Database Service Internal Architecture

```
┌───────────────────────────────────────────────┐
│          DatabaseService Class                │
├───────────────────────────────────────────────┤
│                                               │
│  Connection Management:                       │
│  ├─ _connection_pool (SimpleConnectionPool)  │
│  ├─ _initialize_pool()                       │
│  ├─ refresh_pool()                           │
│  ├─ _get_connection_with_retry()             │
│  └─ get_connection() [Context Manager]       │
│                                               │
│  Document Operations:                         │
│  ├─ save_document()                          │
│  ├─ get_document()                           │
│  ├─ get_all_documents()                      │
│  ├─ delete_document()                        │
│  └─ update_document_status()                 │
│                                               │
│  Conversation Operations:                     │
│  ├─ save_conversation()                      │
│  ├─ get_conversation()                       │
│  ├─ get_all_conversations()                  │
│  ├─ delete_conversation()                    │
│  └─ delete_conversations_by_document()       │
│                                               │
│  Vector Store Operations:                     │
│  ├─ drop_vector_table()                      │
│  ├─ get_orphaned_vector_tables()             │
│  └─ build_connection_url()                   │
│                                               │
│  Health & Maintenance:                        │
│  ├─ test_connection()                        │
│  └─ _create_tables()                         │
│                                               │
└───────────────────────────────────────────────┘
```

### Storage Service Internal Architecture

```
┌───────────────────────────────────────────────┐
│           GCSService Class                    │
├───────────────────────────────────────────────┤
│                                               │
│  Initialization:                              │
│  ├─ client (storage.Client)                  │
│  ├─ bucket (storage.Bucket)                  │
│  └─ bucket_name (str)                        │
│                                               │
│  File Operations:                             │
│  ├─ upload_file(content, filename)           │
│  │   └─→ Returns unique blob_name            │
│  ├─ download_file(blob_name, [local_path])   │
│  │   └─→ Returns local_path                  │
│  ├─ delete_file(blob_name)                   │
│  │   └─→ Returns success boolean             │
│  └─ list_files([prefix])                     │
│      └─→ Returns list of blob names          │
│                                               │
│  Metadata Operations:                         │
│  ├─ file_exists(blob_name)                   │
│  │   └─→ Returns boolean                     │
│  └─ get_file_metadata(blob_name)             │
│      └─→ Returns dict with metadata          │
│                                               │
└───────────────────────────────────────────────┘
```

## 🧩 Module Relationships

### Import Graph (Simplified)

```
app.py
  │
  ├─→ config.settings
  ├─→ services.db_service
  ├─→ services.gcs_service
  ├─→ services.rag_service*
  ├─→ models
  └─→ utils.logging_config

services/database.py
  │
  ├─→ config.settings
  ├─→ models.DocumentMetadata
  └─→ (external: psycopg2, sqlalchemy)

services/storage.py
  │
  ├─→ config.settings
  └─→ (external: google.cloud.storage)

services/rag_service.py*
  │
  ├─→ config.settings
  ├─→ services.database
  ├─→ services.storage
  ├─→ utils.embeddings
  ├─→ utils.file_processing
  ├─→ models
  └─→ (external: llama_index)

utils/embeddings.py
  │
  ├─→ config.settings
  └─→ (external: google.generativeai, llama_index)

utils/file_processing.py
  │
  ├─→ config.settings
  └─→ (external: docx, pandas, beautifulsoup4)

config/settings.py
  │
  ├─→ models (Pydantic BaseModel)
  └─→ (external: dotenv, pydantic)
```

## 🔐 Security Architecture

```
┌─────────────────────────────────────────────┐
│           Environment Variables              │
│  (.env file - not in git)                   │
│  - DB credentials                            │
│  - API keys                                  │
│  - Bucket names                              │
└────────────┬────────────────────────────────┘
             │ (Loaded at startup)
             ▼
┌─────────────────────────────────────────────┐
│         Config Layer (settings.py)           │
│  - Validates all settings                    │
│  - Type checking                             │
│  - Default values                            │
│  - No credentials in code                    │
└────────────┬────────────────────────────────┘
             │ (Provides to)
             ▼
┌─────────────────────────────────────────────┐
│            Service Layer                     │
│  - Uses validated config                     │
│  - Parameterized SQL queries                 │
│  - Connection string sanitization            │
│  - Automatic cleanup (context managers)      │
└────────────┬────────────────────────────────┘
             │ (Connects to)
             ▼
┌─────────────────────────────────────────────┐
│         External Services                    │
│  - PostgreSQL (SSL/TLS)                     │
│  - Google Cloud Storage (HTTPS)              │
│  - Google AI API (HTTPS)                     │
└─────────────────────────────────────────────┘
```

## 📈 Scalability Considerations

### Current Architecture Supports:

1. **Horizontal Scaling**
   - Stateless API layer (can run multiple instances)
   - Connection pooling handles concurrent requests
   - Database handles locking/transactions

2. **Vertical Scaling**
   - Configurable pool sizes
   - Adjustable chunk sizes
   - Tunable batch sizes

3. **Future Enhancements** (Easy to add)
   - Redis caching layer (between services & DB)
   - Message queue for async processing (RabbitMQ/Celery)
   - Load balancer for API instances
   - Read replicas for database

### Scalability Architecture (Future)

```
             ┌─────────────┐
             │Load Balancer│
             └──────┬──────┘
                    │
        ┌───────────┼───────────┐
        │           │           │
        ▼           ▼           ▼
   ┌────────┐ ┌────────┐ ┌────────┐
   │ API #1 │ │ API #2 │ │ API #3 │
   └───┬────┘ └───┬────┘ └───┬────┘
       │          │          │
       └──────────┼──────────┘
                  │
         ┌────────┼────────┐
         │                 │
         ▼                 ▼
   ┌──────────┐      ┌──────────┐
   │  Redis   │      │  RabbitMQ│
   │  Cache   │      │  Queue   │
   └────┬─────┘      └────┬─────┘
        │                 │
        └────────┬────────┘
                 │
         ┌───────┼────────┐
         │                │
         ▼                ▼
   ┌──────────┐     ┌──────────┐
   │PostgreSQL│     │   GCS    │
   │  Primary │     │  Bucket  │
   └────┬─────┘     └──────────┘
        │
        ▼
   ┌──────────┐
   │PostgreSQL│
   │  Replica │
   └──────────┘
```

## 🧪 Testing Architecture

### Test Pyramid (To be implemented)

```
        /\
       /  \
      / UI \      ← E2E Tests (Few)
     /Tests \       - Full API workflows
    /────────\      - Real database
   /          \
  / Integration\   ← Integration Tests (Some)
 /    Tests     \    - Service interactions
/────────────────\   - Mocked externals
                 \
  Unit Tests      \ ← Unit Tests (Many)
    (Most)          - Individual functions
                    - Mocked dependencies
```

### Test Coverage Plan

```
config/
  └─ settings_test.py
     - Valid configuration
     - Invalid configuration
     - Missing required vars

services/
  ├─ database_test.py
  │  - Connection pooling
  │  - CRUD operations
  │  - Error handling
  │
  ├─ storage_test.py
  │  - Upload/download
  │  - File operations
  │  - Error handling
  │
  └─ rag_service_test.py*
     - Vector operations
     - Query processing
     - Conversation management

utils/
  ├─ embeddings_test.py
  ├─ file_processing_test.py
  └─ logging_test.py

api/
  └─ routes_test.py
     - All endpoints
     - Error cases
     - Auth (if added)
```

---

## 📚 Legend

- `*` = To be created in Phase 2
- `─→` = Data/control flow
- `├─` = Component/method
- `▼` = Directional flow
- `□` = Module/service boundary

---

**Status**: Architecture documented for Phase 1 (Complete) ✅

Next: Implement Phase 2 components following this architecture.
