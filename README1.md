# MAGI OCR POC - FastAPI Document Processing System

A production-ready FastAPI application for intelligent document processing with AI-powered text extraction, summarization, and vector search capabilities using Google Gemini AI and ChromaDB.

## 🚀 Features

### Core Capabilities

- **Multi-format Document Support**: Process PDFs, Word documents, images, emails, and more
- **AI-Powered Text Extraction**: Leverage Google Gemini API for accurate OCR and text extraction
- **Dual Processing Flows**:
  - **Flow A**: Generate comprehensive document summaries with keywords
  - **Flow B**: Create searchable text chunks with intelligent document structure awareness
- **Vector Search**: Semantic search across summaries and document chunks using ChromaDB
- **Cloud Storage Integration**: Automatic upload to Google Cloud Storage (GCS)
- **Background Processing**: Asynchronous document processing with status tracking

### Advanced Features

- **Smart Chunking Strategies**:
  - Header-based chunking (Markdown/HTML structure-aware)
  - Character-based chunking with word boundary detection
  - Automatic fallback between strategies
- **Keyword Extraction**: Combined Gemini AI + spaCy NLP for enhanced accuracy
- **Batch Processing**: Optimized embedding generation and ChromaDB storage
- **Email Parsing**: Native support for EML, MSG, MBOX formats
- **Comprehensive Logging**: Detailed processing logs with timing metrics

## 📋 Supported File Types

### Documents

- PDF (`.pdf`)
- Microsoft Word (`.docx`, `.doc`)
- Text files (`.txt`, `.md`, `.rtf`)
- HTML (`.html`, `.htm`)
- CSV (`.csv`)

### Images (OCR)

- JPEG (`.jpg`, `.jpeg`)
- PNG (`.png`)
- BMP (`.bmp`)
- TIFF (`.tiff`)
- GIF (`.gif`)

### Email Formats

- EML (`.eml`)
- Outlook MSG (`.msg`)
- MBOX (`.mbox`)
- EMLX (`.emlx`)
- MBX (`.mbx`)

## 🛠️ Installation

### Prerequisites

- Python 3.10+ (tested with 3.13)
- Google Cloud Project with GCS bucket
- Google Gemini API key
- ChromaDB instance (cloud or self-hosted)

### Setup Instructions

1. **Clone the repository**

```bash
git clone <repository-url>
cd magi_ocr_poc_fastapi
```

1. **Install dependencies**

```bash
pip install -r requirements.txt
```

1. **Download spaCy language model**

```bash
python -m spacy download en_core_web_sm
```

1. **Configure environment variables**

Create a `.env` file in the project root:

```env
# Google Cloud & AI
GOOGLE_CLOUD_PROJECT=your-project-id
GCS_BUCKET_NAME=your-bucket-name
GEMINI_API_KEY=your-gemini-api-key
GEMINI_EXTRACTION_MODEL=gemini-2.5-pro
GEMINI_SUMMARIZATION_MODEL=gemini-2.5-flash

# ChromaDB Configuration
CHROMA_HOST=your-chroma-host
CHROMA_PORT=443
CHROMA_API_KEY=your-chroma-api-key
CHROMA_TENANT=default_tenant
CHROMA_DATABASE=vector_db
CHROMA_USE_SSL=true

# Processing Configuration
MAX_FILES_PER_UPLOAD=10
PREFER_HEADER_CHUNKING=true
```

1. **Set up GCS credentials**

Place your `gcs_credentials.json` file in the project root and set:

```bash
# Linux/Mac
export GOOGLE_APPLICATION_CREDENTIALS="./gcs_credentials.json"

# Windows PowerShell
$env:GOOGLE_APPLICATION_CREDENTIALS=".\gcs_credentials.json"
```

## 🚦 Running the Application

### Development Mode

```bash
uvicorn magi_ocr_poc_fastapi_backend:app --host 0.0.0.0 --port 8000 --reload
```

### Production Mode

```bash
python magi_ocr_poc_fastapi_backend.py
```

The API will be available at:

- **API**: <http://localhost:8000>
- **Interactive Docs**: <http://localhost:8000/docs>
- **ReDoc**: <http://localhost:8000/redoc>

## 📚 API Endpoints

### 1. Upload Documents

**POST** `/upload`

Upload one or multiple documents for processing.

**Request:**

```bash
curl -X POST "http://localhost:8000/upload" \
  -F "files=@document1.pdf" \
  -F "files=@document2.docx"
```

**Response:**

```json
{
  "status": "success",
  "message": "Successfully started processing for 2 files.",
  "processed_files": [
    {
      "filename": "document1.pdf",
      "gcs_path": "gs://your-bucket/20251017_120000_document1.pdf",
      "doc_id": "abc123..."
    }
  ]
}
```

### 2. Search Summaries (Flow A)

**POST** `/search/summary`

Search across document summaries.

**Request:**

```json
{
  "query": "machine learning algorithms",
  "top_k": 5
}
```

**Response:**

```json
{
  "query": "machine learning algorithms",
  "collection": "summary_collection",
  "results": [
    {
      "id": "abc123...",
      "content": "This document discusses various ML algorithms...",
      "metadata": {
        "filename": "ml_guide.pdf",
        "flow": "A"
      },
      "similarity_score": 0.892,
      "flow": "A - Summary"
    }
  ],
  "total_results": 5
}
```

### 3. Search Chunks (Flow B)

**POST** `/search/chunks`

Search across document chunks for precise content retrieval.

**Request:**

```json
{
  "query": "neural network architecture",
  "top_k": 10
}
```

**Response:**

```json
{
  "query": "neural network architecture",
  "collection": "per_doc_collection",
  "results": [
    {
      "id": "abc123_chunk_5",
      "content": "Neural network architectures consist of...",
      "metadata": {
        "filename": "ml_guide.pdf",
        "chunk_index": 5,
        "parent_doc_id": "abc123...",
        "flow": "B",
        "chunk_length": 987
      },
      "similarity_score": 0.915,
      "flow": "B - Chunks"
    }
  ],
  "total_results": 10
}
```

### 4. System Status

**GET** `/status`

Get system health and processing statistics.

**Response:**

```json
{
  "status": "active",
  "processing_stats": {
    "total_processed": 42,
    "total_failed": 2,
    "success_rate": 95.5
  },
  "collections": {
    "summary_collection": {
      "name": "Flow A - Document Summaries",
      "count": 42
    },
    "per_doc_collection": {
      "name": "Flow B - Document Chunks",
      "count": 538
    }
  },
  "capabilities": {
    "gemini_available": true,
    "gcs_available": true,
    "chromadb_available": true,
    "nlp_available": true
  }
}
```

### 5. ChromaDB Connection Test

**GET** `/conection/chromadb`

Test ChromaDB connection and configuration.

## 🔧 Configuration Options

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `GEMINI_API_KEY` | Google Gemini API key | Required |
| `GOOGLE_CLOUD_PROJECT` | GCP project ID | Required |
| `GCS_BUCKET_NAME` | GCS bucket name | Required |
| `CHROMA_HOST` | ChromaDB host | Required |
| `CHROMA_PORT` | ChromaDB port | `443` |
| `CHROMA_API_KEY` | ChromaDB API key | Required |
| `CHROMA_TENANT` | ChromaDB tenant | `default_tenant` |
| `CHROMA_DATABASE` | ChromaDB database | `vector_db` |
| `CHROMA_USE_SSL` | Use SSL for ChromaDB | `true` |
| `GEMINI_EXTRACTION_MODEL` | Model for text extraction | `gemini-2.5-pro` |
| `GEMINI_SUMMARIZATION_MODEL` | Model for summarization | `gemini-2.5-flash` |
| `MAX_FILES_PER_UPLOAD` | Max files per upload | `10` |
| `PREFER_HEADER_CHUNKING` | Use header-based chunking | `true` |

### Processing Limits

- **Max file size**: 100 MB per file
- **Chunk size**: 1000 characters (configurable)
- **Chunk overlap**: 100 characters (configurable)
- **Max keywords**: 15 per document

## 🏗️ Architecture

### Processing Flows

#### Flow A: Summary Processing

1. Extract text from document using Gemini AI
2. Clean and normalize text
3. Generate comprehensive summary (200-300 words)
4. Extract keywords using Gemini + spaCy
5. Generate embeddings using Sentence Transformers
6. Store in `summary_collection` in ChromaDB

#### Flow B: Chunk Processing

1. Extract and clean text
2. **Smart Chunking**:
   - Detect document structure (Markdown/HTML headers)
   - Apply header-based chunking if structure found
   - Fallback to character-based chunking
3. Generate embeddings for all chunks in batches
4. Extract keywords from full document (optimization)
5. Store chunks with metadata in `per_doc_collection`

### Technology Stack

- **Web Framework**: FastAPI 0.110.0
- **AI Models**: Google Gemini 2.5 pro & Flash
- **Embeddings**: Sentence Transformers (all-MiniLM-L6-v2)
- **NLP**: spaCy (en_core_web_sm)
- **Vector Database**: ChromaDB 1.1.1
- **Cloud Storage**: Google Cloud Storage
- **Server**: Uvicorn (ASGI)

## 📊 Performance Optimizations

### Implemented Optimizations

1. **Batch Embedding Generation**: Process 32 chunks at a time
2. **Batch ChromaDB Storage**: Store 50 documents per batch
3. **Global Keyword Extraction**: Extract once per document, reuse for all chunks
4. **Async Background Processing**: Non-blocking document processing
5. **Smart Chunking**: Preserve document structure for better context
6. **Progress Logging**: Detailed timing metrics for performance monitoring

### Typical Processing Times

- **Text Extraction (PDF)**: 30-60 seconds via Gemini API
- **Summarization**: 5-10 seconds per document
- **Embedding Generation**: ~2-5 seconds per 100 chunks
- **ChromaDB Storage**: ~1-3 seconds per 100 chunks

## 🐛 Troubleshooting

### Common Issues

#### 1. spaCy Model Not Found

```bash
# Solution: Download the model
python -m spacy download en_core_web_sm
```

#### 2. GCS Authentication Error

```bash
# Ensure credentials file exists and environment variable is set
export GOOGLE_APPLICATION_CREDENTIALS="./gcs_credentials.json"
```

#### 3. ChromaDB Connection Failed

- Verify `CHROMA_HOST`, `CHROMA_PORT`, and `CHROMA_API_KEY`
- Check SSL settings (`CHROMA_USE_SSL`)
- Test with `/conection/chromadb` endpoint

#### 4. Gemini API Rate Limits

- Use `gemini-2.5-pro` for faster processing
- Implement retry logic for production deployments
- Monitor API usage in Google Cloud Console

### Logs

Check `pipeline.log` for detailed processing logs and error messages.

## 📝 Development Notes

### Dependencies

- Requires Python 3.10+ (tested with 3.13)
- Uses Python 3.13-compatible versions of SciPy and PyTorch
- All dependencies specified in `requirements.txt`

### Code Structure

```text
magi_ocr_poc_fastapi/
├── magi_ocr_poc_fastapi_backend.py  # Main application code (1115 lines)
├── requirements.txt                  # Python dependencies
├── gcs_credentials.json             # GCS service account credentials
├── .env                             # Environment configuration
├── pipeline.log                      # Processing logs
└── # MAGi OCR POC Backend

A powerful FastAPI-based document processing system that leverages Google's Gemini AI for OCR, text extraction, summarization, and intelligent document chunking with vector database storage.

## 🌟 Features

- **Multi-Format Document Support**: Process PDFs, Word documents, images, emails, and text files
- **AI-Powered Text Extraction**: Uses Google Gemini AI for accurate OCR and text extraction
- **Dual Processing Flows**:
  - **Flow A**: Document summarization with keyword extraction
  - **Flow B**: Intelligent document chunking with structure-aware splitting
- **Vector Database Integration**: ChromaDB for semantic search and retrieval
- **Cloud Storage**: Google Cloud Storage (GCS) integration for file management
- **NLP Capabilities**: spaCy and SentenceTransformers for keyword extraction and embeddings
- **Smart Chunking**: Header-based chunking for structured documents with fallback to character-based chunking
- **RESTful API**: Comprehensive endpoints for upload, search, and status monitoring

## 📋 Supported File Types

- **Documents**: `.pdf`, `.docx`, `.doc`, `.txt`, `.rtf`, `.html`, `.htm`, `.md`, `.csv`
- **Images**: `.jpg`, `.jpeg`, `.png`, `.bmp`, `.tiff`, `.gif`
- **Emails**: `.eml`, `.msg`, `.mbox`, `.emlx`, `.mbx`

## 🏗️ Architecture

### Processing Flows

#### Flow A: Document Summarization
1. Extract text from document using Gemini AI
2. Generate comprehensive summary (200-300 words)
3. Extract keywords using Gemini + spaCy
4. Create embeddings using SentenceTransformers
5. Store in `summary_collection` in ChromaDB

#### Flow B: Document Chunking
1. Extract text from document
2. Apply intelligent chunking:
   - Header-based chunking for structured documents (Markdown/HTML)
   - Character-based chunking with word boundary detection
3. Generate embeddings for all chunks (batch processing)
4. Extract keywords from full document (optimized)
5. Store in `per_doc_collection` in ChromaDB

### Smart Chunking System

The system includes an intelligent chunking mechanism that:
- **Detects document structure**: Identifies Markdown headers (`#`, `##`, `###`) and HTML headers (`<h1>`, `<h2>`, etc.)
- **Preserves context**: Maintains header hierarchy for better semantic understanding
- **Respects boundaries**: Breaks at natural section breaks, sentence boundaries, or word boundaries
- **Prevents infinite loops**: Guaranteed forward progress in chunking algorithm
- **Configurable**: Adjustable chunk size and overlap parameters

## 🚀 Quick Start

### Prerequisites

- Python 3.13+
- Docker (optional)
- Google Cloud account with:
  - Gemini API access
  - Cloud Storage bucket
- ChromaDB instance (hosted or local)

### Environment Variables

Create a `.env` file in the project root:

```env
# Google Cloud Configuration
GOOGLE_CLOUD_PROJECT=your-project-id
GCS_BUCKET_NAME=your-bucket-name
GEMINI_API_KEY=your-gemini-api-key
GEMINI_EXTRACTION_MODEL=gemini-1.5-flash
GEMINI_SUMMARIZATION_MODEL=gemini-1.5-flash

# ChromaDB Configuration
CHROMA_HOST=your-chroma-host
CHROMA_PORT=443
CHROMA_API_KEY=your-chroma-api-key
CHROMA_TENANT=default_tenant
CHROMA_DATABASE=vector_db
CHROMA_USE_SSL=true

# Processing Configuration
MAX_FILES_PER_UPLOAD=10
PREFER_HEADER_CHUNKING=true
```

### Installation

#### Option 1: Local Installation

```bash
# Clone the repository
git clone https://github.com/ConnITLabs/magi-poc-ocr-processing-backend.git
cd magi-poc-ocr-processing-backend

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Download spaCy model
python -m spacy download en_core_web_sm

# Run the application
python magi_ocr_poc_fastapi_backend.py
```

#### Option 2: Docker

```bash
# Build the image
docker build -t magi-ocr-backend .

# Run the container
docker run -p 8000:8000 --env-file .env magi-ocr-backend
```

The API will be available at `http://localhost:8000`

## 📚 API Documentation

### Interactive Documentation

Once running, access the interactive API documentation:

- **Swagger UI**: <http://localhost:8000/docs>
- **ReDoc**: <http://localhost:8000/redoc>

### Endpoints

#### `GET /`

Root endpoint with API information

#### `POST /upload`

Upload one or more documents for processing

**Request**:

- Method: `POST`
- Content-Type: `multipart/form-data`
- Body: List of files

**Response**:

```json
{
  "status": "success",
  "message": "Successfully started processing for 2 files.",
  "processed_files": [
    {
      "filename": "document.pdf",
      "gcs_path": "gs://bucket/20241017_120000_document.pdf",
      "doc_id": "abc123..."
    }
  ]
}
```

**Example (curl)**:

```bash
curl -X POST "http://localhost:8000/upload" \
  -F "files=@document.pdf" \
  -F "files=@report.docx"
```

**Example (Python)**:

```python
import requests

files = [
    ('files', open('document.pdf', 'rb')),
    ('files', open('report.docx', 'rb'))
]
response = requests.post('http://localhost:8000/upload', files=files)
print(response.json())
```

#### `POST /search/summary`

Search document summaries (Flow A)

**Request**:

```json
{
  "query": "quarterly financial results",
  "top_k": 5
}
```

**Response**:

```json
{
  "query": "quarterly financial results",
  "collection": "summary_collection",
  "results": [
    {
      "id": "abc123...",
      "content": "Summary of the document...",
      "metadata": {
        "filename": "Q3_report.pdf",
        "flow": "A"
      },
      "similarity_score": 0.892,
      "flow": "A - Summary"
    }
  ],
  "total_results": 5
}
```

#### `POST /search/chunks`

Search document chunks (Flow B)

**Request**:

```json
{
  "query": "revenue breakdown by region",
  "top_k": 10
}
```

**Response**:

```json
{
  "query": "revenue breakdown by region",
  "collection": "per_doc_collection",
  "results": [
    {
      "id": "abc123_chunk_5",
      "content": "Regional revenue data...",
      "metadata": {
        "filename": "annual_report.pdf",
        "chunk_index": 5,
        "parent_doc_id": "abc123...",
        "flow": "B"
      },
      "similarity_score": 0.876,
      "flow": "B - Chunks"
    }
  ],
  "total_results": 10
}
```

#### `GET /status`

Get system status and statistics

**Response**:

```json
{
  "status": "active",
  "processing_stats": {
    "total_processed": 42,
    "total_failed": 3,
    "success_rate": 93.3
  },
  "collections": {
    "summary_collection": {
      "name": "Flow A - Document Summaries",
      "count": 42
    },
    "per_doc_collection": {
      "name": "Flow B - Document Chunks",
      "count": 358
    }
  },
  "capabilities": {
    "gemini_available": true,
    "gcs_available": true,
    "chromadb_available": true,
    "nlp_available": true
  },
  "config": {
    "bucket_name": "your-bucket",
    "max_file_size_mb": 100,
    "max_files_per_upload": 10,
    "supported_extensions": [".pdf", ".docx", ...]
  }
}
```

#### `GET /health`

Health check endpoint for monitoring

**Response** (200 OK):

```json
{
  "status": "healthy",
  "timestamp": "2024-10-17T12:00:00",
  "service": "MAGi OCR POC Backend",
  "components": {
    "gemini_api": "available",
    "gcs": "available",
    "chromadb": "available",
    "chromadb_connection": "connected",
    "nlp": "available"
  }
}
```

**Response** (503 Service Unavailable):

```json
{
  "status": "unhealthy",
  "timestamp": "2024-10-17T12:00:00",
  "service": "MAGi OCR POC Backend",
  "components": {...},
  "issues": ["ChromaDB connection failed"]
}
```

#### `GET /conection/chromadb`

Check ChromaDB connection details

## 🔧 Configuration

### Chunking Configuration

The system provides flexible chunking options:

```python
CONFIG.chunk_size = 1000  # Maximum characters per chunk
CONFIG.chunk_overlap = 100  # Overlap between chunks
CONFIG.prefer_header_chunking = True  # Use structure-aware chunking
```

### Embedding Configuration

```python
CONFIG.embedding_model = "sentence-transformers/all-MiniLM-L6-v2"
CONFIG.embedding_dimension = 384
```

### File Processing Limits

```python
CONFIG.max_file_size_mb = 100
CONFIG.max_files_per_upload = 10
```

## 📊 Performance Optimization

The system includes several optimizations:

1. **Batch Embedding Generation**: Processes 32 chunks at a time to reduce API calls
2. **Batch ChromaDB Storage**: Stores chunks in batches of 50 for efficiency
3. **Global Keyword Extraction**: Extracts keywords once from full document instead of per-chunk
4. **Smart Chunking Detection**: Automatically detects document structure to choose optimal chunking method
5. **Async Processing**: Background processing for uploaded documents

### Processing Times

Typical processing times (may vary based on document size and API latency):

- **Text Extraction**: 30-60 seconds for PDFs
- **Summarization**: 5-15 seconds per document
- **Embedding Generation**: ~5 seconds for 100 chunks
- **ChromaDB Storage**: ~2 seconds for 100 chunks

## 🔐 Security

- **Non-root Docker user**: Runs as `appuser` for enhanced security
- **API Key authentication**: Secure access to Gemini and ChromaDB
- **CORS middleware**: Configurable cross-origin resource sharing
- **File validation**: Strict file type and size validation
- **Content sanitization**: Text cleaning and validation

## �️ Debugging & Troubleshooting

### Frequently Asked Questions

#### spaCy Model Not Found

```bash
python -m spacy download en_core_web_sm
```

#### ChromaDB Connection Issues

- Verify `CHROMA_HOST`, `CHROMA_PORT`, and `CHROMA_API_KEY`
- Check SSL settings (`CHROMA_USE_SSL`)
- Ensure ChromaDB instance is running

#### Gemini API Errors

- Verify `GEMINI_API_KEY` is valid
- Check API quota and rate limits
- Ensure model names are correct

#### GCS Upload Failures

- Verify `GCS_BUCKET_NAME` exists
- Check Google Cloud credentials
- Ensure service account has write permissions

### Logging

Logs are written to:

- Console: `stdout`
- File: `pipeline.log`

Set log level in code:

```python
logging.basicConfig(level=logging.INFO)  # or DEBUG, WARNING, ERROR
```

## 📦 Dependencies

Core dependencies:

- **fastapi**: Web framework
- **uvicorn**: ASGI server
- **google-generativeai**: Gemini AI integration
- **google-cloud-storage**: GCS integration
- **chromadb**: Vector database
- **sentence-transformers**: Embedding generation
- **spacy**: NLP and keyword extraction
- **numpy**: Numerical operations
- **pydantic**: Data validation

See `requirements.txt` for complete list with versions.

## 🔄 Development

### Project Structure

```text
magi-poc-ocr-processing-backend/
├── magi_ocr_poc_fastapi_backend.py  # Main application
├── requirements.txt                  # Python dependencies
├── dockerfile                        # Docker configuration
├── gcs_credentials.json             # GCS service account key (gitignored)
├── .env                             # Environment variables (gitignored)
├── pipeline.log                     # Application logs
└── README.md                        # This file
```

### Running in Development Mode

```bash
# With auto-reload (not recommended for Python 3.13)
uvicorn magi_ocr_poc_fastapi_backend:app --reload --host 0.0.0.0 --port 8000

# Production mode
python magi_ocr_poc_fastapi_backend.py
```

### Testing

```bash
# Test health endpoint
curl http://localhost:8000/health

# Test upload
curl -X POST "http://localhost:8000/upload" -F "files=@test.pdf"

# Test search
curl -X POST "http://localhost:8000/search/summary" \
  -H "Content-Type: application/json" \
  -d '{"query": "test", "top_k": 5}'
```

## 🤝 Collaboration

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 License

This project is part of the MAGi OCR POC initiative by ConnIT Labs.

## 🙏 Acknowledgments

- Google Gemini AI for OCR and summarization
- ChromaDB for vector database capabilities
- Hugging Face for SentenceTransformers
- spaCy for NLP capabilities

## 📧 Support

For issues, questions, or contributions, please open an issue on the GitHub repository.

---

**Repository**: [ConnITLabs/magi-poc-ocr-processing-backend](https://github.com/ConnITLabs/magi-poc-ocr-processing-backend)

**Version**: 2.0.0

**Last Updated**: October 2024

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## 📞 Support

For issues and questions:

- Check `pipeline.log` for detailed error messages
- Review the `/status` endpoint for system health
- Consult the interactive API docs at `/docs`

---

**Built with FastAPI, Google Gemini AI, and ChromaDB** 🚀
