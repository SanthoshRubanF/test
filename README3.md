# MAGI POC - RAG System with OCR Processing

A comprehensive Retrieval-Augmented Generation (RAG) system built with FastAPI, LlamaIndex, Google Gemini 2.5 Flash AI, and PostgreSQL with pgvector. This system enables document upload, automatic OCR/text extraction, structured metadata extraction, and natural language querying capabilities.

## 🚀 Features

### Core Functionality

- **Multi-format Document Processing**: Supports 20+ file types including PDF, DOCX, images, emails, and more
- **Advanced OCR & Text Extraction**: Uses Google Gemini 2.5 Flash AI for accurate text extraction from various formats
- **Structured Data Extraction**: Automatically extracts metadata (title, author, summary, key points, entities, categories, language, word count)
- **Vector Embeddings**: Custom Gemini text-embedding-004 integration for high-quality document embeddings
- **Natural Language Querying**: Ask questions in plain English and get contextual answers
- **Multiple Index Support**: Organize documents into separate namespaces/indexes
- **Background Processing**: Asynchronous document ingestion for optimal performance
- **Query Caching**: 5-minute in-memory caching for improved response times

### File Format Support

- **Documents**: PDF, DOCX, DOC, TXT, RTF, HTML, HTM, MD
- **Data Files**: CSV
- **Images**: JPEG, JPG, PNG, GIF, BMP, TIFF
- **Email Formats**: EML, MSG, MBOX, EMLX, MBX
- **Fallback Processing**: Robust text extraction for unsupported formats

### API Features

- **RESTful Design**: Clean FastAPI-based API with automatic documentation
- **File Upload**: Multipart form-data support for batch uploads
- **Error Handling**: Comprehensive error responses with detailed messages
- **Validation**: Input validation for file types, sizes, and parameters

## 🏗️ Architecture

### System Components

1. **API Layer** (`app.py`)
   - FastAPI application with REST endpoints
   - File upload handling with validation
   - Background task management
   - Request/response models using Pydantic

2. **RAG Engine** (`main.py`)
   - Document processing pipeline
   - Text extraction using Gemini 2.5 Flash AI
   - Structured data extraction and validation
   - Vector embeddings generation and storage
   - Query processing with similarity search
   - Multi-index support with PostgreSQL + pgvector

3. **Data Models** (`models.py`)
   - `DocumentMetadata`: File metadata and tracking
   - `StructuredDocumentData`: Extracted document properties
   - API request/response schemas

### Data Processing Pipeline

```text
Upload → GCS Storage → Background Processing → Text Extraction → Structured Data → Chunking → Embeddings → Vector DB → Query Ready
```

1. **Document Upload**: Files validated and stored in Google Cloud Storage
2. **Text Extraction**: Gemini 2.5 Flash AI extracts text from multiple formats
3. **Metadata Extraction**: Structured information automatically extracted
4. **Document Chunking**: Content split into 512-character chunks with 50-character overlap
5. **Vector Generation**: Custom Gemini embeddings created for each chunk
6. **Storage**: Embeddings stored in PostgreSQL with pgvector extension
7. **Query Processing**: Natural language queries converted to embeddings and matched against stored vectors

## 📋 Prerequisites

- **Python 3.8+**
- **Google Cloud Platform Account** with:
  - Cloud Storage bucket
  - Service account with Storage Admin and AI Platform Developer roles
  - Google AI API key (Gemini access)
- **PostgreSQL 12+** with pgvector extension
- **Google Cloud Credentials JSON file**

## 🛠️ Installation

### 1. Clone Repository

```bash
git clone <repository-url>
cd magi-poc-rag-ocr-processing
```

### 2. Create Virtual Environment

```bash
# Windows
python -m venv .venv
.venv\Scripts\activate

# macOS/Linux
python -m venv .venv
source .venv/bin/activate
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

### 4. Environment Configuration

Create a `.env` file in the project root:

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
APP_PORT=8080

# Vector Database Configuration
VECTOR_DIMENSION=768
SIMILARITY_TOP_K=5
```

### 5. Google Cloud Setup

1. **Create GCP Project** and enable APIs:
   - Cloud Storage API
   - Generative AI API

2. **Create Service Account** with required roles:
   - `roles/storage.admin` (Cloud Storage)
   - `roles/aiplatform.user` (Gemini API)

3. **Download Credentials**: Save service account key as `gcs_credentials.json`

### 6. Database Setup

```sql
-- Create database
CREATE DATABASE RAG_OCR_POC_DB;

-- Enable pgvector extension
\c RAG_OCR_POC_DB;
CREATE EXTENSION IF NOT EXISTS vector;
```

## 🚀 Usage

### Starting the Server

```bash
# Option 1: Direct execution
python app.py

# Option 2: Using uvicorn
uvicorn app:app --host 127.0.0.1 --port 8080
```

**API Documentation**: Visit `http://127.0.0.1:8080/docs` for interactive Swagger UI.

## 📡 API Endpoints

### Health Check

```http
GET /
```

**Response**: `{"message": "RAG System API"}`

### File Upload

```http
POST /upload
```

**Parameters**:

- `files`: List of files (multipart/form-data)
- `index_name`: Target index name (default: "default")

**Supported File Types**: PDF, DOCX, DOC, TXT, RTF, HTML, HTM, MD, CSV, JPG, JPEG, PNG, GIF, BMP, TIFF, EML, MSG, MBOX, EMLX, MBX

**Example**:

```bash
curl -X POST "http://127.0.0.1:8080/upload" \
     -F "files=@document.pdf" \
     -F "files=@report.docx" \
     -F "index_name=my_documents"
```

**Response**:

```json
{
  "message": "Successfully uploaded 2 files. Ingestion started in background.",
  "uploaded_files": ["uuid1_document.pdf", "uuid2_report.docx"],
  "document_ids": ["doc-id-1", "doc-id-2"],
  "status": "ingestion_in_progress"
}
```

### Query System

```http
POST /query
```

**Request Body**:

```json
{
  "query": "What are the main findings in the uploaded documents?",
  "index_name": "default",
  "top_k": 5
}
```

**Example**:

```bash
curl -X POST "http://127.0.0.1:8080/query" \
     -H "Content-Type: application/json" \
     -d '{
       "query": "Summarize the key points from the documents",
       "index_name": "default",
       "top_k": 3
     }'
```

**Response**:

```json
{
  "answer": "Based on the uploaded documents, the key findings include: 1) Market analysis shows 15% growth, 2) Customer satisfaction improved by 23%, 3) New product launch scheduled for Q2..."
}
```

## ⚙️ Configuration

### Environment Variables

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `GOOGLE_API_KEY` | Gemini API key | - | Yes |
| `GCS_BUCKET_NAME` | Cloud Storage bucket | - | Yes |
| `DB_HOST` | PostgreSQL host | - | Yes |
| `DB_PORT` | PostgreSQL port | 5434 | No |
| `DB_NAME` | Database name | - | Yes |
| `DB_USER` | Database user | - | Yes |
| `DB_PASSWORD` | Database password | - | Yes |
| `APP_HOST` | API server host | 127.0.0.1 | No |
| `APP_PORT` | API server port | 8080 | No |
| `VECTOR_DIMENSION` | Embedding dimension | 768 | No |
| `SIMILARITY_TOP_K` | Query retrieval count | 5 | No |

## 📁 Project Structure

```text
magi-poc-rag-ocr-processing/
├── app.py                    # FastAPI application & API endpoints
├── main.py                   # RAG engine & document processing
├── models.py                 # Pydantic data models & schemas
├── requirements.txt          # Python dependencies
├── README.md                 # Project documentation
├── .env                      # Environment variables (create)
├── gcs_credentials.json      # GCP credentials (create)
└── __pycache__/             # Python cache (auto-generated)
```

## 🛠️ Dependencies

### Core Framework

- `fastapi` - Modern web framework
- `uvicorn` - ASGI server
- `python-multipart` - Multipart form handling

### AI & ML

- `llama-index==0.10.28` - RAG framework
- `llama-index-vector-stores-postgres==0.1.14` - Vector DB integration
- `llama-index-llms-gemini==0.2.0` - Gemini LLM integration
- `google-generativeai` - Gemini API client

### Database

- `psycopg2-binary` - PostgreSQL driver
- `pgvector==0.2.5` - Vector extension

### File Processing

- `python-docx` - Word document parsing
- `python-magic-bin` - File type detection
- `beautifulsoup4` - HTML parsing
- `pandas` - CSV processing
- `lxml` - XML/HTML parsing

### Cloud & Utilities

- `google-cloud-storage` - GCS integration
- `python-dotenv` - Environment management

## 🔧 Development

### Key Implementation Details

- **Custom Embedding Model**: Extends LlamaIndex's `BaseEmbedding` with Gemini text-embedding-004
- **Text Extraction**: Primary Gemini 2.5 Flash AI with fallback parsers for unsupported formats
- **Document Chunking**: 512-character chunks with 50-character overlap for optimal retrieval
- **Query Caching**: 5-minute TTL in-memory cache for performance
- **Background Tasks**: FastAPI background tasks for non-blocking document processing

### Error Handling

- Comprehensive validation for file types and sizes
- Detailed error messages with context
- Graceful fallbacks for processing failures
- Logging for debugging and monitoring

## 🐛 Troubleshooting

### Common Issues

1. **"RAG system not properly initialized"**
   - Verify all environment variables are set
   - Check database connectivity and pgvector extension
   - Validate Google Cloud credentials

2. **File Upload Failures**
   - Confirm supported file formats
   - Check file size limits
   - Verify GCS permissions

3. **Empty Query Results**
   - Ensure documents were ingested successfully
   - Confirm correct `index_name` usage
   - Check vector database contents

4. **Authentication Errors**
   - Validate `gcs_credentials.json` file
   - Check service account permissions
   - Verify `GOOGLE_API_KEY` correctness

### Performance Optimization

- **Caching**: Query responses cached for 5 minutes
- **Async Processing**: Background ingestion prevents blocking
- **Chunking Strategy**: Optimized chunk size for retrieval quality
- **Similarity Search**: Configurable retrieval parameters

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'Add amazing feature'`)
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 📞 Support

For issues and questions:

1. Check the troubleshooting section
2. Review API documentation at `/docs`
3. Examine console logs for detailed errors
4. Ensure all prerequisites are configured

---

**Built with**: FastAPI, LlamaIndex, Google Gemini 2.5 Flash AI, PostgreSQL, pgvector
