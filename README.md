# Document Processing API

A **refactored and simplified** FastAPI-based document processing system that automatically uploads documents to Google Cloud Storage and processes them through dual-flow architecture using Gemini AI and ChromaDB.

> **Version 2.0** - Refactored from complex monitoring system to focus on core upload → process → search workflow

## 🚀 Features

- **Automatic Upload Flow**: Frontend → GCS Storage → Background Processing
- **Dual-Flow Processing**:
  - **Flow A**: Document summarization and keyword extraction
  - **Flow B**: Document chunking for detailed search
- **Vector Search**: Semantic search across summaries and chunks
- **Multi-Format Support**: PDF, DOCX, images, text files, and more
- **Background Processing**: Non-blocking document processing
- **Real-time Status**: Monitor processing statistics and collection counts

## 📋 Architecture

```text
Frontend Upload → GCS Bucket → Gemini AI Processing → ChromaDB Storage
                                      ↓
                              Flow A (Summaries) + Flow B (Chunks)
                                      ↓
                            Vector Embeddings + Keyword Extraction
                                      ↓
                              ChromaDB Collections Storage
```

### Simplified Refactored Design

This version focuses on **core functionality only**:

- **Removed**: Complex monitoring, pub/sub notifications, real-time processing, background orchestrators
- **Kept**: Essential upload → process → search workflow
- **Added**: Efficient lazy initialization, background task processing
- **Simplified**: From 2,711 lines to 914 lines (66% reduction)

## 🛠️ Installation

### 1. Prerequisites

- Python 3.8+
- Google Cloud Project with Storage API enabled
- Gemini AI API access
- ChromaDB Cloud instance

### 2. Clone and Setup

```bash
git clone <repository-url>
cd magi_ocr_poc_fastapi
```

### 3. Create Virtual Environment

```bash
python -m venv .venv

# Windows
.venv\Scripts\activate

# Linux/Mac
source .venv/bin/activate
```

### 4. Install Dependencies

```bash
pip install -r requirements.txt
```

### 5. Install spaCy Model

```bash
python -m spacy download en_core_web_sm
```

## ⚙️ Configuration

### Environment Variables

Create a `.env` file in the root directory:

```env
# Google Cloud & AI Services
GEMINI_API_KEY=your-gemini-api-key
GOOGLE_CLOUD_PROJECT=your-project-id
GCS_BUCKET_NAME=your-storage-bucket
GOOGLE_APPLICATION_CREDENTIALS=gcs_credentials.json

# ChromaDB Cloud
CHROMA_HOST=your-chroma-host.com
CHROMA_PORT=443
CHROMA_API_KEY=your-chroma-api-key
CHROMA_TENANT=your-tenant-id
CHROMA_DATABASE=vector_db
CHROMA_USE_SSL=true
```

### Google Cloud Authentication

1. Create a service account in your Google Cloud Console
2. Download the JSON key file as `gcs_credentials.json`
3. Set the environment variable:

```bash
# Windows
set GOOGLE_APPLICATION_CREDENTIALS=gcs_credentials.json

# Linux/Mac
export GOOGLE_APPLICATION_CREDENTIALS=gcs_credentials.json
```

## 🚀 Running the Application

### Quick Start (If Environment is Ready)

```bash
# Activate virtual environment
.venv\Scripts\activate  # Windows
source .venv/bin/activate  # Linux/Mac

# Start the server
uvicorn magi_ocr_poc-fastapi_backend:app --host 0.0.0.0 --port 8000 --reload
```

### Development Mode

```bash
python magi_ocr_poc-fastapi_backend.py
```

### Production Mode

```bash
uvicorn magi_ocr_poc-fastapi_backend:app --host 0.0.0.0 --port 8000
```

### With Auto-reload (Recommended for Development)

```bash
uvicorn magi_ocr_poc-fastapi_backend:app --host 0.0.0.0 --port 8000 --reload
```

## 📚 API Documentation

Once running, access the interactive API documentation:

- **Swagger UI**: <http://localhost:8000/docs>
- **ReDoc**: <http://localhost:8000/redoc>
- **API Root**: <http://localhost:8000/>

## 🔗 API Endpoints

### 1. Upload Document

```http
POST /upload
Content-Type: multipart/form-data

{
  "file": <binary_file>
}
```

**Response:**

```json
{
  "status": "success",
  "message": "File uploaded successfully. Processing started in background.",
  "filename": "document.pdf",
  "gcs_path": "gs://bucket/20251014_143022_document.pdf",
  "doc_id": "sha256hash",
  "flow_a_processed": false,
  "flow_b_processed": false,
  "chunks_created": 0
}
```

### 2. Search Summaries (Flow A)

```http
POST /search/summary
Content-Type: application/json

{
  "query": "machine learning algorithms",
  "top_k": 5
}
```

### 3. Search Chunks (Flow B)

```http
POST /search/chunks
Content-Type: application/json

{
  "query": "data preprocessing techniques",
  "top_k": 10
}
```

### 4. Get Status

```http
GET /status
```

**Response:**

```json
{
  "status": "active",
  "processing_stats": {
    "total_processed": 25,
    "total_failed": 2,
    "success_rate": 92.6
  },
  "collections": {
    "summary_collection": {
      "name": "Flow A - Document Summaries",
      "count": 25
    },
    "per_doc_collection": {
      "name": "Flow B - Document Chunks",
      "count": 150
    }
  }
}
```

## 📁 Supported File Types

- **Documents**: PDF, DOCX, DOC, TXT, RTF, HTML, HTM, MD
- **Images**: JPG, JPEG, PNG, BMP, TIFF, GIF (OCR extraction)
- **Data**: CSV

Maximum file size: **100MB** (configurable in code)
Chunk size: **1000 characters** with **100 character overlap**

## 🔄 Processing Workflow

1. **Upload**: Document uploaded to `/upload` endpoint
2. **GCS Storage**: File stored in Google Cloud Storage with timestamp
3. **Background Processing**: Automatic dual-flow processing starts
4. **Text Extraction**: Gemini AI extracts text from document
5. **Flow A**:
   - Generate summary using Gemini
   - Extract keywords with spaCy + Gemini
   - Create embeddings
   - Store in `summary_collection`
6. **Flow B**:
   - Split text into overlapping chunks
   - Extract keywords per chunk
   - Create embeddings per chunk
   - Store in `per_doc_collection`
7. **Search Ready**: Documents searchable via both flows

## 🔍 Search Capabilities

### Semantic Search

- **Vector Embeddings**: Uses `sentence-transformers/all-MiniLM-L6-v2` (384 dimensions)
- **Similarity Scoring**: Cosine similarity with normalized scores (1 - distance)
- **Dual Collections**:
  - `summary_collection` - Document summaries and metadata
  - `per_doc_collection` - Document chunks with parent references
- **Keyword Enhancement**: spaCy NLP (`en_core_web_sm`) + Gemini keyword extraction

### Search Types

- **Summary Search** (`/search/summary`): High-level document concepts and themes
- **Chunk Search** (`/search/chunks`): Detailed content search within document sections

### Processing Configuration

- **Chunk Size**: 1000 characters with 100 character overlap
- **Max File Size**: 100MB per upload
- **Embedding Model**: all-MiniLM-L6-v2 (384-dimensional vectors)
- **NLP Models**: Gemini 2.0 Flash Exp + spaCy en_core_web_sm

## 🗂️ Project Structure

```text
magi_ocr_poc_fastapi/
├── magi_ocr_poc-fastapi_backend.py    # Main FastAPI application (914 lines)
├── requirements.txt                    # Python dependencies (streamlined)
├── gcs_credentials.json               # Google Cloud service account key
├── .env                               # Environment variables (simplified)
├── .env.template                      # Template for deployment
├── pipeline.log                       # Application logs
└── README.md                          # Documentation
```

### Key Code Components

- **DocumentProcessor Class**: Handles dual-flow processing (Flow A + Flow B)
- **Lazy Initialization**: Efficient loading of models and clients
- **Background Processing**: Non-blocking document processing via FastAPI BackgroundTasks
- **ChromaDB Integration**: Two collections (`summary_collection`, `per_doc_collection`)
- **Gemini AI Integration**: Text extraction and summarization
- **Vector Search**: Semantic search with sentence-transformers embeddings

## 🛡️ Security Considerations

- **Environment Variables**: Never commit `.env` or credentials to version control
- **API Keys**: Rotate API keys regularly
- **CORS**: Configure appropriate CORS settings for production
- **File Validation**: Built-in file type and size validation
- **Input Sanitization**: Text cleaning and validation

## 📊 Monitoring

### Logs

- Application logs: `pipeline.log`
- Real-time console output
- Processing statistics via `/status` endpoint

### Health Checks

- `/status` endpoint provides system health
- Collection counts and processing statistics
- Dependency availability checks

## 🐛 Troubleshooting

### Common Issues

1. **Import Errors**: Ensure all dependencies installed

   ```bash
   pip install -r requirements.txt
   python -m spacy download en_core_web_sm
   ```

2. **Authentication Errors**: Check `GOOGLE_APPLICATION_CREDENTIALS`

   ```bash
   echo $GOOGLE_APPLICATION_CREDENTIALS  # Should point to gcs_credentials.json
   ```

3. **Environment Variables**: Verify `.env` file configuration

   ```bash
   python -c "from dotenv import load_dotenv; load_dotenv(); import os; print(os.getenv('GEMINI_API_KEY'))"
   ```

4. **ChromaDB Connection**: Check ChromaDB Cloud credentials and network access

5. **File Size Limits**: Default 100MB limit, check file size before upload

6. **Supported Formats**: Verify your file extension is in the supported list

### Debug Mode

Enable debug logging by modifying the logging level in the code:

```python
logging.basicConfig(level=logging.DEBUG)
```

### Performance Notes

- **Lazy Loading**: Models and clients initialize only when needed
- **Background Processing**: File uploads return immediately, processing happens asynchronously
- **Memory Efficient**: Chunking strategy prevents memory overload on large documents

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests
5. Submit a pull request

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🙏 Acknowledgments

- **FastAPI**: Modern, fast web framework for building APIs
- **Google Gemini 2.0**: Advanced AI for text extraction and summarization  
- **ChromaDB Cloud**: Vector database for semantic search
- **spaCy**: Industrial-strength NLP (`en_core_web_sm`)
- **sentence-transformers**: State-of-the-art embeddings (`all-MiniLM-L6-v2`)
- **Google Cloud Storage**: Reliable file storage and management

## 📞 Support

For issues and questions:

1. Check the logs in `pipeline.log`
2. Verify environment configuration
3. Test individual components via `/status`
4. Create an issue in the repository

---

Built with ❤️ for efficient document processing and semantic search
