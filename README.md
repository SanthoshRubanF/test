# MAGI RAG System - FastAPI Backend

A production-ready Retrieval-Augmented Generation (RAG) system built with FastAPI, LlamaIndex, PostgreSQL (pgvector), and Google Gemini AI. Upload documents, process them automatically, and query using natural language with real-time streaming responses.

## 🚀 Features

### Core Capabilities

- **📄 Document Management**: Upload, process, and delete documents across multiple storage locations
- **🔍 Intelligent RAG**: Hybrid search combining BM25 + vector similarity with cross-encoder re-ranking
- **💬 Conversational AI**: Context-aware conversations with streaming responses via SSE
- **📊 Real-time Status Tracking**: Monitor document ingestion status (pending → processing → completed/failed)
- **🗄️ Multi-Storage Architecture**: PostgreSQL (metadata + vectors), GCS (files), in-memory caching
- **🔄 Background Processing**: Asynchronous document ingestion with status updates

### Advanced Features

- **Query Expansion**: Gemini-powered query variations for better retrieval
- **Structured Extraction**: Automatic metadata extraction (title, author, summary, entities)
- **Hybrid Search**: 70% vector similarity + 30% BM25 keyword matching
- **Cross-Encoder Re-ranking**: Improves relevance scoring using sentence-transformers
- **Conversation Persistence**: Full conversation history stored and retrievable
- **Orphaned Table Cleanup**: Utility to clean up orphaned vector tables

## 📋 Supported File Types

- **Documents**: PDF, DOCX, DOC, TXT, RTF, HTML, HTM, MD
- **Data**: CSV
- **Images**: JPEG, PNG, GIF, BMP, TIFF (OCR via Gemini Vision)
- **Email**: EML, MSG, MBOX
- **Archives**: ZIP (extracts and processes contents)

## 🛠️ Technology Stack

### Core Framework

- **FastAPI**: High-performance async web framework
- **Python 3.12+**: Modern Python with full type hints
- **Uvicorn**: Lightning-fast ASGI server

### AI & ML

- **LlamaIndex 0.10.28**: RAG orchestration framework
- **Google Gemini 2.5 Flash**: LLM for generation and analysis
- **Sentence Transformers**: Cross-encoder re-ranking
- **BM25**: Keyword-based retrieval

### Storage & Database

- **PostgreSQL + pgvector**: Vector storage and similarity search
- **Google Cloud Storage**: Document blob storage
- **psycopg2**: PostgreSQL adapter

### Data Processing

- **python-docx**: Word document parsing
- **BeautifulSoup4**: HTML parsing
- **pandas**: CSV/Excel processing
- **python-magic**: File type detection

## 📦 Prerequisites

### Required Services

1. **PostgreSQL Database** (with pgvector extension)
   - Version: 12+
   - Extension: `pgvector` enabled

2. **Google Cloud Platform**
   - Cloud Storage bucket
   - Service account with Storage Admin role
   - Gemini API access

3. **API Keys**

   - Google AI API key (for Gemini models)

## 🔧 Installation

### 1. Clone Repository

```bash
git clone <repository-url>
cd magi_rag_poc_fastapi
```

### 2. Create Virtual Environment

```bash
python -m venv .venv

# Windows
.venv\Scripts\activate

# Linux/Mac
source .venv/bin/activate
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

### 4. Set Up Environment Variables

Create a `.env` file in the root directory:

```env
# Database Configuration
DB_HOST=localhost
DB_PORT=5432
DB_NAME=your_database_name
DB_USER=your_db_user
DB_PASSWORD=your_db_password

# Google Cloud Storage
GCS_BUCKET_NAME=your-gcs-bucket-name

# Google AI
GOOGLE_API_KEY=your_google_ai_api_key

# Server Configuration
APP_HOST=127.0.0.1
APP_PORT=8000
```

### 5. Set Up Google Cloud Credentials

Place your GCS service account JSON file as `gcs_credentials.json` in the root directory:

```json
{
  "type": "service_account",
  "project_id": "your-project-id",
  "private_key_id": "...",
  "private_key": "...",
  "client_email": "...",
  "client_id": "...",
  ...
}
```

### 6. Initialize Database

The application automatically creates required tables on startup:

- `documents`: Document metadata and ingestion status
- `conversations`: Conversation history with messages

Enable pgvector extension manually in PostgreSQL:

```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

## 🚦 Running the Application

### Start the Server

```bash
uvicorn app:app --host 127.0.0.1 --port 8000 --reload
```

The server will start on: **<http://127.0.0.1:8000>**

### API Documentation

- **Swagger UI**: <http://127.0.0.1:8000/docs>
- **ReDoc**: <http://127.0.0.1:8000/redoc>

## 📡 API Endpoints

### Document Management

#### Upload Documents

```http
POST /upload
Content-Type: multipart/form-data

files: <file1>, <file2>, ...
index_name: "default" (optional)
```

**Response:**

```json
{
  "message": "Files uploaded successfully",
  "documents": [
    {
      "document_id": "uuid",
      "filename": "document.pdf",
      "blob_name": "uuid_document.pdf",
      "ingestion_status": "pending"
    }
  ]
}
```

#### Get All Documents

```http
GET /documents?index_name=default
```

**Response:**

```json
{
  "documents": [
    {
      "document_id": "uuid",
      "filename": "document.pdf",
      "source": "upload",
      "upload_timestamp": "2025-11-11T10:00:00",
      "index_name": "default",
      "mime_type": "application/pdf",
      "file_size": 1024000,
      "blob_name": "uuid_document.pdf",
      "ingestion_status": "completed"
    }
  ]
}
```

#### Check Ingestion Status

```http
GET /ingestion-status?document_ids=uuid1,uuid2
```

**Response:**

```json
{
  "uuid1": "completed",
  "uuid2": "processing"
}
```

#### Delete Document

```http
DELETE /document/{document_id}?bucket_name=your-bucket
```

**Response:**

```json
{
  "message": "Document deleted successfully",
  "document_id": "uuid",
  "deleted_from": [
    "documents_table",
    "vector_table",
    "conversations_table",
    "in_memory_stores",
    "gcs_bucket"
  ],
  "errors": []
}
```

### Conversation Management

#### Query with Streaming Response

```http
POST /query
Content-Type: application/json

{
  "query": "What is the main topic?",
  "filename": "document.pdf",
  "index_name": "default",
  "top_k": 5
}
```

**Response:** Server-Sent Events (SSE) stream

```text
data: {"type": "token", "content": "The "}
data: {"type": "token", "content": "main "}
data: {"type": "token", "content": "topic..."}
data: {"type": "complete", "full_response": "The main topic...", "sources": [...]}
```

#### Get All Conversations

```http
GET /conversations?index_name=default
```

**Response:**

```json
{
  "conversations": [
    {
      "filename": "document.pdf",
      "document_id": "uuid",
      "index_name": "default",
      "created_at": "2025-11-11T10:00:00",
      "updated_at": "2025-11-11T10:05:00",
      "messages": [
        {
          "role": "user",
          "content": "Question",
          "timestamp": "2025-11-11T10:00:00"
        },
        {
          "role": "assistant",
          "content": "Answer",
          "timestamp": "2025-11-11T10:00:01"
        }
      ]
    }
  ]
}
```

#### Delete Conversation

```http
DELETE /conversation/{filename}
```

### Utility Endpoints

#### Cleanup Orphaned Vector Tables

```http
POST /cleanup-orphaned-tables
```

**Response:**

```json
{
  "message": "Cleanup completed",
  "dropped_tables": ["table1", "table2"],
  "errors": []
}
```

#### Health Check

```http
GET /
```

## 🏗️ Architecture

### Data Flow

```text
Upload → GCS Storage → Background Ingestion → Vector Index → Query → Response
                              ↓
                    Status Updates (pending/processing/completed)
```

### Storage Layers

1. **PostgreSQL**
   - `documents` table: Metadata + ingestion status
   - `conversations` table: Chat history
   - `data_data_vectors_{uuid}` tables: Vector embeddings per document

2. **Google Cloud Storage**
   - Raw document blobs
   - Unique naming: `{uuid}_{filename}`

3. **In-Memory Cache**
   - `documents_store`: Quick metadata access
   - `conversations_store`: Active conversation cache

### Document Processing Pipeline

```text
1. Upload → GCS
2. Store metadata (status: pending)
3. Background task starts (status: processing)
4. Extract text (Gemini Vision for images)
5. Generate embeddings (Gemini text-embedding-004)
6. Create vector index in PostgreSQL
7. Extract structured metadata
8. Update status → completed/failed
9. Frontend polls status → shows notification
```

### Query Processing

```text
1. User query
2. Query expansion (Gemini generates variations)
3. Hybrid retrieval:
   - Vector similarity search (70%)
   - BM25 keyword search (30%)
4. Cross-encoder re-ranking
5. Context building
6. Gemini 2.5 Flash generation
7. Stream response via SSE
8. Save conversation history
```

## 🗄️ Database Schema

### documents Table

```sql
CREATE TABLE documents (
    document_id VARCHAR(255) PRIMARY KEY,
    filename VARCHAR(255) NOT NULL,
    source VARCHAR(50) DEFAULT 'upload',
    upload_timestamp TIMESTAMP NOT NULL,
    index_name VARCHAR(100) DEFAULT 'default',
    mime_type VARCHAR(100),
    file_size BIGINT,
    blob_name VARCHAR(500),
    ingestion_status VARCHAR(50) DEFAULT 'pending'
);
```

### conversations Table

```sql
CREATE TABLE conversations (
    filename VARCHAR(255) PRIMARY KEY,
    document_id VARCHAR(255),
    index_name VARCHAR(100) DEFAULT 'default',
    created_at TIMESTAMP NOT NULL,
    updated_at TIMESTAMP NOT NULL,
    user_id VARCHAR(100),
    messages JSONB,
    state JSONB
);
```

## 🔐 Security Considerations

1. **Environment Variables**: Never commit `.env` or `gcs_credentials.json`
2. **Database Access**: Use read-only user for query operations
3. **API Rate Limiting**: Implement rate limiting in production
4. **Input Validation**: All inputs are validated via Pydantic models
5. **File Upload Limits**: Configure max file size in production

## 🐛 Troubleshooting

### Common Issues

#### 1. Import Errors / Module Not Found

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

#### 2. PostgreSQL Connection Failed

- Verify DB credentials in `.env`
- Ensure PostgreSQL is running
- Check pgvector extension: `CREATE EXTENSION vector;`

#### 3. GCS Authentication Error

- Verify `gcs_credentials.json` path
- Check service account permissions
- Ensure `GOOGLE_APPLICATION_CREDENTIALS` is set

#### 4. Gemini API Errors

- Verify `GOOGLE_API_KEY` in `.env`
- Check API quota limits
- Ensure Gemini API is enabled in GCP

#### 5. Vector Table Not Deleted

- Tables use hyphens: `data_data_vectors_{uuid-with-hyphens}`
- Deletion uses quoted identifiers: `DROP TABLE "table_name"`

#### 6. Python 3.12 Compatibility Issues

- Use versions from requirements.txt
- LlamaIndex 0.10.28 is compatible with Python 3.12
- Ensure Pydantic v1.10+ for compatibility

## 📊 Performance Optimization

### Recommended Settings

#### PostgreSQL

```sql
-- Optimize vector search
SET max_parallel_workers_per_gather = 4;
SET effective_cache_size = '4GB';
```

#### Indexing

```sql
-- Create indexes for faster queries
CREATE INDEX idx_documents_index_name ON documents(index_name);
CREATE INDEX idx_documents_status ON documents(ingestion_status);
CREATE INDEX idx_conversations_index ON conversations(index_name);
```

### Scaling Considerations

- Use connection pooling for PostgreSQL
- Implement Redis for distributed caching
- Consider horizontal scaling with load balancer
- Use async workers for background tasks

## 🧪 Development

### Code Structure

```text
magi_rag_poc_fastapi/
├── app.py                  # FastAPI application & endpoints
├── main.py                 # Core RAG logic & document processing
├── models.py               # Pydantic data models
├── requirements.txt        # Python dependencies
├── .env                    # Environment variables (not in repo)
├── gcs_credentials.json    # GCS service account (not in repo)
└── README.md              # This file
```

### Key Functions

#### main.py

- `ingest_files()`: Background document processing
- `query_rag_conversational()`: Main RAG query handler
- `delete_document()`: Comprehensive document deletion
- `cleanup_orphaned_vector_tables()`: Database maintenance
- `update_document_ingestion_status()`: Status tracking

#### app.py

- `POST /upload`: Document upload handler
- `POST /query`: Streaming query endpoint
- `DELETE /document/{id}`: Document deletion
- `GET /ingestion-status`: Status polling
- `POST /cleanup-orphaned-tables`: Orphaned table cleanup

#### models.py

- `DocumentMetadata`: Document information model
- `ConversationMessage`: Message structure
- `Conversation`: Conversation history model
- `StructuredDocumentData`: Extracted metadata

## 📝 Environment Variables Reference

| Variable | Description | Example | Required |
|----------|-------------|---------|----------|
| `DB_HOST` | PostgreSQL host address | `localhost` | ✅ |
| `DB_PORT` | PostgreSQL port | `5432` | ✅ |
| `DB_NAME` | Database name | `rag_database` | ✅ |
| `DB_USER` | Database username | `postgres` | ✅ |
| `DB_PASSWORD` | Database password | `your_password` | ✅ |
| `GCS_BUCKET_NAME` | GCS bucket name | `my-documents-bucket` | ✅ |
| `GOOGLE_API_KEY` | Gemini API key | `AIza...` | ✅ |
| `APP_HOST` | Server host | `127.0.0.1` | ❌ |
| `APP_PORT` | Server port | `8000` | ❌ |

## 🔄 API Response Status Codes

| Code | Meaning | Description |
|------|---------|-------------|
| 200 | OK | Request successful |
| 201 | Created | Resource created successfully |
| 400 | Bad Request | Invalid request parameters |
| 404 | Not Found | Resource not found |
| 500 | Internal Server Error | Server error occurred |

## 📚 Additional Resources

### LlamaIndex Documentation

- <https://docs.llamaindex.ai/>

### FastAPI Documentation

- <https://fastapi.tiangolo.com/>

### Google Gemini API

- <https://ai.google.dev/>

### pgvector Documentation

- <https://github.com/pgvector/pgvector>

## 📧 Support

For issues and questions:

1. Check the troubleshooting section
2. Review API documentation at `/docs`
3. Check console logs for detailed errors
4. Open a GitHub issue

---

Built with ❤️ using FastAPI, LlamaIndex, and Google Gemini AI
