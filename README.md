# Document Processing API

A simplified FastAPI-based document processing system with **dual-flow architecture** using **Gemini AI 2.0 Flash** and **ChromaDB Cloud** for intelligent document analysis and semantic search.

## 🚀 Key Features

- **Streamlined Upload Flow**: Upload → GCS → Automatic Background Processing
- **Dual-Flow Processing Architecture**:
  - **Flow A**: Document summarization with Gemini AI + keyword extraction  
  - **Flow B**: Intelligent text chunking with overlapping segments
- **Semantic Vector Search**: Search both summaries and detailed chunks
- **Multi-Format Support**: PDF, DOCX, images (OCR), text files, and more
- **Background Processing**: Non-blocking document processing with FastAPI BackgroundTasks
- **Real-time Monitoring**: Live processing statistics and collection health checks

## 📋 System Architecture

```text
Frontend Upload → GCS Bucket → Gemini 2.0 Flash Processing → ChromaDB Cloud
                                         ↓
                               Flow A: Summary Pipeline
                               ├─ Gemini summarization
                               ├─ spaCy keyword extraction  
                               ├─ Sentence-transformer embeddings
                               └─ Store in summary_collection
                                         ↓
                               Flow B: Chunk Pipeline  
                               ├─ Text chunking (1000 chars, 100 overlap)
                               ├─ Per-chunk keyword extraction
                               ├─ Per-chunk embeddings
                               └─ Store in per_doc_collection
```

## 🛠️ Installation

### 1. Prerequisites

- Python 3.8+
- Google Cloud Project with Storage API enabled
- Gemini AI API access
- ChromaDB Cloud instance

### 2. Clone and Setup

```bash
git clone <repository-url>
cd magi_ocr_poc_fastapi_backend
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

### Required Environment Variables

Create a `.env` file with these **essential** variables:

```env
# Google Cloud & AI Services (REQUIRED)
GOOGLE_CLOUD_PROJECT=your-project-id
GCS_BUCKET_NAME=your-storage-bucket
GEMINI_API_KEY=your-gemini-api-key
GOOGLE_APPLICATION_CREDENTIALS=gcs_credentials.json

# ChromaDB Cloud (REQUIRED)  
CHROMA_HOST=your-chroma-host.com
CHROMA_PORT=443
CHROMA_API_KEY=your-chroma-api-key
CHROMA_TENANT=your-tenant-id
CHROMA_DATABASE=vector_db
CHROMA_USE_SSL=true
```

### Built-in Configuration

The application includes these **pre-configured settings**:

- **AI Models**: `gemini-2.0-flash-exp` (extraction & summarization)
- **Embeddings**: `sentence-transformers/all-MiniLM-L6-v2` (384 dimensions)
- **Chunking**: 1000 characters with 100 character overlap
- **File Limit**: 100MB maximum upload size
- **Collections**: `summary_collection` (Flow A) + `per_doc_collection` (Flow B)

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

### Development Mode

```bash
python magi_ocr_poc_fastapi_backend.py
```

### Production Mode

```bash
uvicorn magi_ocr_poc_fastapi_backend:app --host 0.0.0.0 --port 8000
```

### With Auto-reload

```bash
uvicorn magi_ocr_poc_fastapi_backend:app --host 0.0.0.0 --port 8000 --reload
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

The system supports these file formats with automatic format detection:

- **Documents**: `.pdf`, `.docx`, `.doc`, `.txt`, `.rtf`, `.html`, `.htm`, `.md`
- **Images**: `.jpg`, `.jpeg`, `.png`, `.bmp`, `.tiff`, `.gif` (Gemini Vision OCR)
- **Data**: `.csv`

**Limits**: 100MB maximum file size (configurable via `CONFIG.max_file_size_mb`)

## 🔄 Complete Processing Workflow

### Phase 1: Upload & Storage

1. **Upload Request**: Document sent to `/upload` endpoint
2. **Validation**: File type and size validation
3. **GCS Upload**: File stored with timestamp: `YYYYMMDD_HHMMSS_filename.ext`
4. **Background Task**: Processing starts asynchronously via `BackgroundTasks`

### Phase 2: Text Extraction

1. **Gemini Vision**: `gemini-2.0-flash-exp` extracts text from any format
2. **Text Cleaning**: Normalize whitespace and remove special characters

### Phase 3: Dual-Flow Processing

1. **Flow A - Summary Pipeline**:
   - Gemini generates 200-300 word summary
   - Gemini extracts 10-15 keywords
   - spaCy NLP extracts additional keywords (entities, noun phrases)
   - `sentence-transformers` creates 384-dim embeddings
   - Store in ChromaDB `summary_collection`

2. **Flow B - Chunk Pipeline**:
   - Split text into 1000-char chunks (100-char overlap)
   - Extract keywords per chunk with spaCy
   - Generate embeddings per chunk
   - Store in ChromaDB `per_doc_collection`

### Phase 4: Search Ready

1. **Instant Access**: Documents searchable via semantic similarity in both collections

## 🔍 Advanced Search Capabilities

### Semantic Vector Search

- **Embeddings Model**: `sentence-transformers/all-MiniLM-L6-v2` (384 dimensions)
- **Similarity Algorithm**: Cosine similarity with `1 - distance` scoring
- **Dual Collection Architecture**: Independent search across summary vs. chunk collections
- **Enhanced Keywords**: Combined Gemini AI + spaCy NLP keyword extraction

### Search Modes

- **Summary Search** (`/search/summary`): High-level document concepts, themes, and overviews
- **Chunk Search** (`/search/chunks`): Granular content search within specific document sections
- **Configurable Results**: 1-20 results per query (default: 5)

### ChromaDB Collections

- **`summary_collection`**: Stores document summaries with metadata
- **`per_doc_collection`**: Stores text chunks with parent document tracking

## 🗂️ Project Structure

```text
magi_ocr_poc_fastapi/
├── magi_ocr_poc_fastapi_backend.py    # Main FastAPI application (914 lines)
├── requirements.txt                    # Python dependencies  
├── gcs_credentials.json               # Google Cloud service account key
├── .env                               # Environment configuration
├── .env.template                      # Environment template (no secrets)
├── pipeline.log                       # Application logs
├── README.md                          # Documentation
└── __pycache__/                       # Python cache files
```

### Code Architecture

- **Lazy Initialization**: Dependencies loaded on-demand via `get_*_client()` functions
- **Background Processing**: Uses FastAPI `BackgroundTasks` for non-blocking processing
- **DocumentProcessor Class**: Handles dual-flow processing logic
- **ChromaDB Collections**: Two separate collections for different search granularities
- **Error Handling**: Comprehensive logging and HTTP exception handling

## 🛡️ Security Considerations

- **Environment Variables**: Never commit `.env` or credentials to version control
- **API Keys**: Rotate API keys regularly
- **CORS**: Configure appropriate CORS settings for production
- **File Validation**: Built-in file type and size validation
- **Input Sanitization**: Text cleaning and validation

## 📊 Monitoring & Health Checks

### Logging System

- **File Logging**: `pipeline.log` with timestamp, level, and detailed messages
- **Console Output**: Real-time logs via `StreamHandler`
- **Log Levels**: INFO level for normal operations, ERROR for failures

### Status Endpoint (`/status`)

Real-time system information including:

- **Processing Statistics**: Success/failure counts and success rate
- **Collection Health**: Document counts in both ChromaDB collections
- **Dependency Status**: Availability of Gemini, GCS, ChromaDB, spaCy, sentence-transformers
- **Configuration Info**: Bucket name, file size limits, supported extensions

### Built-in Health Monitoring

- **Environment Validation**: `validate_environment()` checks required variables and dependencies
- **Lazy Connection**: Clients initialize on first use with error handling
- **ChromaDB Heartbeat**: Connection verification during startup

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

### Debug Mode

Enable debug logging by modifying the logging level:

```python
logging.basicConfig(level=logging.DEBUG)
```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests
5. Submit a pull request

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🏗️ Technical Stack

### Core Technologies

- **FastAPI 0.104+**: Modern async web framework with automatic API documentation
- **Google Gemini 2.0 Flash**: Advanced AI for text extraction, OCR, and summarization
- **ChromaDB Cloud**: Vector database with HTTP client for embeddings storage
- **sentence-transformers**: `all-MiniLM-L6-v2` model for semantic embeddings
- **spaCy**: `en_core_web_sm` model for NLP and keyword extraction
- **Google Cloud Storage**: Scalable document storage with timestamps

### Python Dependencies

Key packages from `requirements.txt`:

- `fastapi` - Web framework
- `google-generativeai` - Gemini AI client
- `google-cloud-storage` - GCS integration  
- `chromadb` - Vector database client
- `sentence-transformers` - Embedding generation
- `spacy` - Natural language processing
- `numpy` - Numerical operations
- `python-dotenv` - Environment variable management

## 📞 Support & Troubleshooting

### Quick Diagnostics

1. **Check Application Logs**: View `pipeline.log` for detailed error messages
2. **Verify Environment**: Ensure all variables in `.env` are correctly set
3. **Test System Health**: Use `/status` endpoint to check component availability
4. **Validate Dependencies**: Run environment validation on startup

### Performance Notes

- **Background Processing**: Document processing runs asynchronously to avoid blocking uploads
- **Lazy Loading**: AI models and clients initialize on first use to reduce startup time  
- **Chunking Strategy**: 1000-character chunks with 100-character overlap for optimal search granularity
- **Memory Management**: Uses temporary files for Gemini processing with automatic cleanup

---

**Document Processing API v2.0.0** - Simplified FastAPI implementation focusing on core upload → process → search workflow
