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
| `GEMINI_EXTRACTION_MODEL` | Model for text extraction | `gemini-1.5-flash` |
| `GEMINI_SUMMARIZATION_MODEL` | Model for summarization | `gemini-1.5-flash` |
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
- **AI Models**: Google Gemini 1.5 Flash
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

- Use `gemini-1.5-flash` for faster processing
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
└── README.md                        # This file
```

### Key Components

#### Document Processing (`DocumentProcessor` class)

- Validates file types and sizes
- Orchestrates Flow A and Flow B processing
- Tracks processing statistics

#### Text Extraction Functions

- `extract_text_with_gemini()`: AI-powered text extraction
- `_extract_text_from_email()`: Email format parsing

#### Chunking Strategies

- `chunk_text()`: Character-based chunking with word boundaries
- `chunk_text_by_headers()`: Structure-aware chunking
- `chunk_text_smart()`: Automatic strategy selection

#### Keyword Extraction

- `generate_summary_with_gemini()`: AI-powered summarization + keywords
- `extract_keywords_with_spacy()`: NLP-based keyword extraction

#### Vector Operations

- `embed_text()` / `embed_texts()`: Generate embeddings
- `store_summary_to_chroma()`: Store summaries
- `store_chunks_to_chroma_batch()`: Batch store chunks

### Testing

```bash
# Run with auto-reload for development
uvicorn magi_ocr_poc_fastapi_backend:app --reload

# Access interactive API documentation
# http://localhost:8000/docs
```

## 🔐 Security Considerations

- Store API keys and credentials in `.env` file (never commit to git)
- Use environment variables for sensitive configuration
- Implement rate limiting for production deployments
- Validate file types and sizes before processing
- Use HTTPS/SSL for ChromaDB connections in production
- Add `.env` and `gcs_credentials.json` to `.gitignore`

## 📄 License

[Your License Here]

## 🤝 Contributing

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
