# MAGI LlamaIndex RAG POC FastAPI

A Retrieval-Augmented Generation (RAG) system built with FastAPI, LlamaIndex, and Google Gemini AI. This system allows you to upload documents, automatically process them, and query for information using natural language.

## Features

- **Document Upload & Processing**: Upload various file types (PDF, DOCX, images, emails, etc.) to Google Cloud Storage
- **Automatic Text Extraction**: Uses Google Gemini AI to extract text from multiple file formats
- **Structured Data Extraction**: Automatically extracts metadata like title, author, summary, key points, and entities
- **Vector Storage**: Stores document embeddings in PostgreSQL with pgvector for efficient similarity search
- **Natural Language Querying**: Query the system using natural language to get relevant answers
- **RESTful API**: Clean FastAPI-based REST API for easy integration
- **Multiple Index Support**: Support for multiple document indexes/namespaces
- **Background Processing**: Asynchronous document ingestion for better performance
- **Query Caching**: In-memory caching for improved response times
- **Comprehensive File Support**: Handles 20+ file formats including documents, images, and emails

## Supported File Types

- **Documents**: PDF, DOCX, DOC, TXT, RTF, HTML, HTM, MD
- **Data Files**: CSV
- **Images**: JPEG, PNG, GIF, BMP, TIFF
- **Email**: EML, MSG, MBOX, MBOX
- **Archives**: ZIP (with document contents)

## Prerequisites

- **Python 3.8+**
- **Google Cloud Platform account** with:
  - Google Cloud Storage bucket
  - Service account with appropriate permissions
  - Google AI API key (for Gemini)
- **PostgreSQL database** with pgvector extension enabled
- **Google Cloud credentials JSON file**

## Installation

1. **Clone the repository:**

   ```bash
   git clone <repository-url>
   cd magi_rag_poc_fastapi
   ```

2. **Create virtual environment:**

   ```bash
   python -m venv .venv
   # On Windows:
   .venv\Scripts\activate
   # On macOS/Linux:
   source .venv/bin/activate
   ```

3. **Install dependencies:**

   ```bash
   pip install -r requirements.txt
   ```

4. **Set up environment variables** by creating a `.env` file:

   ```env
   # Google Cloud Configuration
   GOOGLE_API_KEY=your_google_ai_api_key_here
   GOOGLE_APPLICATION_CREDENTIALS=gcs_credentials.json
   GCS_BUCKET_NAME=your_gcs_bucket_name

   # Database Configuration
   DB_HOST=localhost
   DB_PORT=5434
   DB_NAME=RAG_OCR_POC_DB
   DB_USER=postgres
   DB_PASSWORD=your_database_password

   # Application Configuration
   APP_HOST=127.0.0.1
   APP_PORT=8000

   # Vector Database Configuration
   VECTOR_DIMENSION=768
   SIMILARITY_TOP_K=5
   ```

5. **Place your Google Cloud service account credentials** in `gcs_credentials.json`

6. **Ensure your PostgreSQL database has the pgvector extension:**

   ```sql
   CREATE EXTENSION IF NOT EXISTS vector;
   ```

## Usage

### Starting the Server

#### Option 1: Using uvicorn directly

```bash
python -m uvicorn app:app --host 127.0.0.1 --port 8000
```

#### Option 2: Running the app directly

```bash
python app.py
```

The API will be available at `http://127.0.0.1:8000`

### API Documentation

Once the server is running, visit:

- **Interactive API Docs**: `http://127.0.0.1:8000/docs` (Swagger UI)
- **Alternative Docs**: `http://127.0.0.1:8000/redoc` (ReDoc)

## API Endpoints

### 1. Health Check

```http
GET /
```

**Response:**

```json
{
  "message": "Conversational RAG System API",
  "features": [
    "Document upload and ingestion",
    "Conversational queries with context",
    "Document-specific conversations",
    "Multi-turn dialogue support",
    "Conversation history management"
  ]
}
```

### 2. Upload Files

```http
POST /upload
```

Upload files to be processed and ingested into the RAG system.

**Parameters:**

- `files`: List of files to upload (multipart/form-data)
- `index_name`: Name of the index to store documents in (default: "default")

**Supported file types:** PDF, DOCX, DOC, TXT, RTF, HTML, MD, CSV, JPG, PNG, GIF, BMP, TIFF, EML, MSG, MBOX

**Example using curl:**

```bash
curl -X POST "http://127.0.0.1:8000/upload" \
     -F "files=@document.pdf" \
     -F "files=@report.docx" \
     -F "index_name=my_documents"
```

**Response:**

```json
{
  "message": "Successfully uploaded 2 files. Ingestion started in background.",
  "uploaded_files": ["uuid1_document.pdf", "uuid2_report.docx"],
  "document_ids": ["doc-id-1", "doc-id-2"],
  "status": "ingestion_in_progress"
}
```

### 3. Query the RAG System (Conversational)

```http
POST /query
```

Query the system for information based on ingested documents with conversation context.

**Request Body:**

```json
{
  "query": "What are the main points discussed in the uploaded documents?",
  "filename": "document.pdf",
  "index_name": "default",
  "top_k": 5
}
```

**Parameters:**

- `query`: The natural language question to ask
- `filename`: The filename of the document to query (creates conversation if doesn't exist)
- `index_name`: Which document index to search (default: "default")
- `top_k`: Number of similar documents to consider (optional, default: 5)

**Example using curl:**

```bash
curl -X POST "http://127.0.0.1:8000/query" \
     -H "Content-Type: application/json" \
     -d '{
       "query": "Summarize the key findings from the documents",
       "filename": "report.pdf",
       "index_name": "default",
       "top_k": 3
     }'
```

**Response:**

```json
{
  "filename": "report.pdf",
  "message": {
    "role": "user",
    "content": "Summarize the key findings from the documents",
    "timestamp": "2024-01-15T10:30:00Z",
    "document_id": "doc-id-123"
  },
  "answer": "Based on the uploaded documents, the key findings include: 1) Market analysis shows 15% growth, 2) Customer satisfaction improved by 23%, 3) New product launch scheduled for Q2..."
}
```

### 4. Get Conversation

```http
GET /conversation/{filename}
```

Get the conversation history and details for a specific document.

**Parameters:**

- `filename`: The filename of the document

**Example using curl:**

```bash
curl -X GET "http://127.0.0.1:8000/conversation/document.pdf"
```

**Response:**

```json
{
  "conversation": {
    "filename": "document.pdf",
    "document_id": "doc-id-123",
    "index_name": "default",
    "messages": [],
    "created_at": "2024-01-15T10:00:00Z",
    "updated_at": "2024-01-15T10:30:00Z"
  },
  "messages": [
    {
      "role": "user",
      "content": "What is this document about?",
      "timestamp": "2024-01-15T10:15:00Z",
      "document_id": "doc-id-123"
    },
    {
      "role": "assistant",
      "content": "This document discusses market analysis and growth projections...",
      "timestamp": "2024-01-15T10:15:05Z",
      "document_id": "doc-id-123"
    }
  ]
}
```

### 5. List All Conversations

```http
GET /conversations
```

Get a list of all conversations stored in the database.

**Example using curl:**

```bash
curl -X GET "http://127.0.0.1:8000/conversations"
```

**Response:**

```json
{
  "conversations": [
    {
      "filename": "document1.pdf",
      "document_id": "doc-id-123",
      "index_name": "default",
      "created_at": "2024-01-15T10:00:00Z",
      "updated_at": "2024-01-15T10:30:00Z",
      "message_count": 4
    },
    {
      "filename": "document2.docx",
      "document_id": "doc-id-456",
      "index_name": "default",
      "created_at": "2024-01-15T11:00:00Z",
      "updated_at": "2024-01-15T11:15:00Z",
      "message_count": 2
    }
  ],
  "total": 2
}
```

### 6. Delete Conversation

```http
DELETE /conversation/{filename}
```

Delete a conversation and its history from both memory and database.

**Parameters:**

- `filename`: The filename of the document conversation to delete

**Example using curl:**

```bash
curl -X DELETE "http://127.0.0.1:8000/conversation/document.pdf"
```

**Response:**

```json
{
  "message": "Conversation for document.pdf deleted successfully"
}
```

### 7. List Documents

```http
GET /documents
```

List all documents in a specific index.

**Parameters:**

- `index_name`: Name of the index to list documents from (default: "default")

**Example using curl:**

```bash
curl -X GET "http://127.0.0.1:8000/documents?index_name=default"
```

**Response:**

```json
{
  "documents": [
    {
      "document_id": "doc-id-123",
      "filename": "document.pdf",
      "upload_timestamp": "2024-01-15T10:00:00Z",
      "index_name": "default",
      "mime_type": "application/pdf",
      "file_size": 1024000
    }
  ],
  "total": 1,
  "index_name": "default"
}
```

### 8. Get Document Metadata

```http
GET /document/{filename}
```

Get metadata for a specific document.

**Parameters:**

- `filename`: The filename of the document

**Example using curl:**

```bash
curl -X GET "http://127.0.0.1:8000/document/document.pdf"
```

**Response:**

```json
{
  "document_id": "doc-id-123",
  "filename": "document.pdf",
  "upload_timestamp": "2024-01-15T10:00:00Z",
  "index_name": "default",
  "mime_type": "application/pdf",
  "file_size": 1024000
}
```

## Conversational Features

This RAG system supports multi-turn conversations with document-specific context. Key features include:

### Document-Specific Conversations

- **Per-Document Context**: Each uploaded document gets its own conversation thread
- **Persistent History**: Conversation history is maintained across queries
- **Contextual Responses**: The system remembers previous questions and answers within each document's conversation

### Conversation Management

- **Automatic Creation**: Conversations are created automatically when you first query a document
- **History Retrieval**: Get complete conversation history for any document
- **Conversation Listing**: View all active conversations across all documents
- **Conversation Deletion**: Remove conversations when no longer needed

### Query Enhancement

- **Context-Aware Queries**: Each query considers the conversation history for more relevant responses
- **Follow-up Questions**: Ask follow-up questions that reference previous context
- **Document Grounding**: All responses are grounded in the specific document being queried

**Example Conversation Flow:**

1. Upload `annual_report.pdf`
2. Query: "What are the main revenue streams?"
3. Follow-up: "How has revenue changed compared to last year?"
4. Follow-up: "What are the biggest challenges mentioned?"

The system maintains context across all these queries, providing coherent and document-specific responses.

## Architecture

### Core Components

1. **FastAPI Application** (`app.py`)
   - REST API endpoints
   - File upload handling
   - Background task processing
   - Error handling and validation

2. **RAG Engine** (`main.py`)
   - Document processing and text extraction
   - Structured data extraction using Gemini AI
   - Vector embeddings generation and storage
   - Query processing with caching
   - Multiple index support

3. **Data Models** (`models.py`)
   - Pydantic models for API requests/responses
   - Document metadata structures
   - Structured document data schemas

4. **External Services**
   - **Google Cloud Storage**: File storage for uploaded documents
   - **PostgreSQL + pgvector**: Vector database for embeddings
   - **Google Gemini AI**: LLM for text extraction and query responses

### Data Processing Pipeline

1. **Upload Phase**:
   - Files uploaded via API → Validated for supported formats
   - Stored in Google Cloud Storage with unique identifiers
   - Document metadata created and tracked

2. **Ingestion Phase** (Background Processing):
   - Files downloaded from GCS
   - Text extracted using Gemini AI or fallback parsers
   - Structured data (title, summary, entities) extracted
   - Documents chunked into smaller segments
   - Embeddings generated and stored in vector database

3. **Query Phase**:
   - Natural language query received
   - Query converted to embedding vector
   - Similarity search performed in vector database
   - Relevant document chunks retrieved
   - LLM generates contextual response

### Key Features

- **Asynchronous Processing**: File ingestion happens in background tasks
- **Query Caching**: 5-minute in-memory cache for improved performance
- **Multiple Indexes**: Support for separate document namespaces
- **Comprehensive Text Extraction**: Handles 20+ file formats
- **Structured Metadata**: Automatic extraction of document properties
- **Error Handling**: Robust error handling with detailed logging

## Configuration

### Environment Variables

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `GOOGLE_API_KEY` | Google AI API key for Gemini | - | Yes |
| `GCS_BUCKET_NAME` | Google Cloud Storage bucket name | - | Yes |
| `DB_HOST` | PostgreSQL host | - | Yes |
| `DB_PORT` | PostgreSQL port | 5434 | No |
| `DB_NAME` | PostgreSQL database name | - | Yes |
| `DB_USER` | PostgreSQL username | - | Yes |
| `DB_PASSWORD` | PostgreSQL password | - | Yes |
| `APP_HOST` | FastAPI server host | 127.0.0.1 | No |
| `APP_PORT` | FastAPI server port | 8000 | No |
| `VECTOR_DIMENSION` | Embedding vector dimension | 768 | No |
| `SIMILARITY_TOP_K` | Documents to retrieve for queries | 5 | No |

### Google Cloud Setup

1. **Create a Google Cloud Project**
2. **Enable required APIs**:
   - Google Cloud Storage API
   - Google AI (Gemini) API
3. **Create a service account** with these roles:
   - Storage Admin (for GCS access)
   - AI Platform Developer (for Gemini API)
4. **Download the service account key** as JSON and save as `gcs_credentials.json`

### Database Setup

1. **Install PostgreSQL** with pgvector extension
2. **Create database**:

   ```sql
   CREATE DATABASE RAG_OCR_POC_DB;
   ```

3. **Enable pgvector**:

   ```sql
   \c RAG_OCR_POC_DB;
   CREATE EXTENSION IF NOT EXISTS vector;
   ```

## Development

### Project Structure

```text
magi_rag_poc_fastapi/
├── app.py                    # FastAPI application and API endpoints
├── main.py                   # Core RAG functionality and processing
├── models.py                 # Pydantic data models and schemas
├── requirements.txt          # Python dependencies
├── .env                      # Environment variables (create from template)
├── gcs_credentials.json      # Google Cloud credentials (not in repo)
├── README.md                 # This documentation
├── .gitignore               # Git ignore rules
└── .venv/                   # Virtual environment (not in repo)
```

### Dependencies

**Core Framework:**

- `fastapi` - Web framework
- `uvicorn` - ASGI server

**AI/ML:**

- `llama-index` - RAG framework
- `google-generativeai` - Gemini AI integration
- `llama-index-llms-gemini` - LlamaIndex Gemini integration

**Database:**

- `psycopg2-binary` - PostgreSQL driver
- `pgvector` - Vector extension support
- `llama-index-vector-stores-postgres` - Vector store integration

**File Processing:**

- `python-docx` - Word document processing
- `python-magic-bin` - File type detection
- `beautifulsoup4` - HTML parsing
- `pandas` - CSV processing
- `lxml` - XML/HTML parsing

**Cloud Services:**

- `google-cloud-storage` - GCS integration

**Utilities:**

- `python-dotenv` - Environment variable management
- `python-multipart` - Multipart form data handling

## Troubleshooting

### Common Issues

1. **"RAG system not properly initialized"**
   - Check that all environment variables are set correctly
   - Verify database connection and pgvector extension
   - Ensure Google Cloud credentials are valid

2. **File upload fails**
   - Check file size limits
   - Verify supported file formats
   - Ensure GCS bucket exists and credentials have write access

3. **Query returns no results**
   - Confirm documents were successfully ingested
   - Check correct `index_name` is being used
   - Verify vector database contains data

4. **Google Cloud authentication errors**
   - Ensure `gcs_credentials.json` exists and is valid
   - Check service account has required permissions
   - Verify `GOOGLE_API_KEY` is correct

5. **Database connection errors**
   - Confirm PostgreSQL is running
   - Check connection parameters in `.env`
   - Ensure pgvector extension is installed

### Performance Optimization

- **Query Caching**: Responses are cached for 5 minutes to improve performance
- **Background Processing**: File ingestion happens asynchronously
- **Chunking**: Documents are split into 512-character chunks with 50-character overlap
- **Similarity Search**: Configurable number of similar documents retrieved

### Monitoring

- Check console output for detailed processing logs
- API responses include processing status and document counts
- Background tasks provide ingestion progress updates

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Support

For questions or issues:

1. Check the troubleshooting section above
2. Review the API documentation at `/docs`
3. Check the console logs for detailed error messages
4. Ensure all prerequisites are properly configured
