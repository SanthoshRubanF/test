# MAGi RAG POC - Document Processing with OCR

A Proof of Concept (POC) implementation of a Retrieval-Augmented Generation (RAG) system for document processing with OCR capabilities. This system leverages Google's Gemini AI models for text extraction, document analysis, and natural language querying.

## Features

- **Multi-format Document Processing**: Supports PDF, DOCX, images (JPEG, PNG, TIFF), emails (EML, MBOX), HTML, CSV, and plain text files
- **OCR Integration**: Uses Google Gemini for optical character recognition on images and scanned documents
- **Structured Data Extraction**: Automatically extracts titles, authors, summaries, key points, entities, and categories from documents
- **Vector Search**: Powered by PostgreSQL with pgvector for efficient similarity search
- **RESTful API**: FastAPI-based endpoints for file upload and querying
- **Google Cloud Storage**: Secure file storage with automatic ingestion
- **Caching**: Query result caching for improved performance
- **Background Processing**: Asynchronous document ingestion to prevent blocking

## Architecture

The system consists of three main components:

1. **FastAPI Server** (`app.py`): Handles HTTP requests, file uploads, and API endpoints
2. **RAG Engine** (`main.py`): Core document processing, vectorization, and querying logic
3. **Data Models** (`models.py`): Pydantic models for structured data handling

### Data Flow

1. Documents are uploaded via API and stored in Google Cloud Storage
2. Background processing extracts text using Gemini (with OCR for images)
3. Structured information is extracted and validated
4. Documents are chunked and embedded using Gemini embeddings
5. Vectors are stored in PostgreSQL with pgvector
6. Natural language queries retrieve relevant document chunks
7. Gemini generates contextual answers based on retrieved content

## Prerequisites

- Python 3.8+
- PostgreSQL database with pgvector extension
- Google Cloud Platform account with:
  - Storage bucket
  - Service account credentials
  - Gemini API access
- Required Python packages (see `requirements.txt`)

## Installation

1. Clone the repository:

   ```bash
   git clone https://github.com/ConnITLabs/magi-poc-rag-ocr-processing.git
   cd magi-poc-rag-ocr-processing
   ```

2. Install dependencies:

   ```bash
   pip install -r requirements.txt
   ```

3. Set up PostgreSQL with pgvector:

   ```sql
   CREATE EXTENSION IF NOT EXISTS vector;
   ```

## Configuration

Create a `.env` file in the project root with the following variables:

```env
# Database Configuration
DB_HOST=your-postgres-host
DB_PORT=5432
DB_NAME=your-database-name
DB_USER=your-username
DB_PASSWORD=your-password

# Google Cloud Configuration
GOOGLE_API_KEY=your-gemini-api-key
GCS_BUCKET_NAME=your-storage-bucket-name

# Application Settings
APP_HOST=127.0.0.1
APP_PORT=8080
VECTOR_DIMENSION=768
SIMILARITY_TOP_K=5
```

### Google Cloud Setup

1. Create a Google Cloud Storage bucket
2. Create a service account with Storage Admin permissions
3. Download the service account key as `gcs_credentials.json`
4. Enable the Gemini API in your Google Cloud project

## Usage

### Starting the Server

```bash
python app.py
```

The API will be available at `http://127.0.0.1:8080`

### API Endpoints

#### Upload Documents

```http
POST /upload
Content-Type: multipart/form-data

files: [file1.pdf, file2.docx, ...]
index_name: default
```

**Response:**

```json
{
  "message": "Successfully uploaded 2 files. Ingestion started in background.",
  "uploaded_files": ["uuid1_file1.pdf", "uuid2_file2.docx"],
  "document_ids": ["doc-id-1", "doc-id-2"],
  "status": "ingestion_in_progress"
}
```

#### Query Documents

```http
POST /query
Content-Type: application/json

{
  "query": "What are the main points discussed in the documents?",
  "index_name": "default",
  "top_k": 5
}
```

**Response:**

```json
{
  "answer": "Based on the uploaded documents, the main points are..."
}
```

### Supported File Types

- **Documents**: PDF, DOCX, DOC, TXT, RTF, HTML, MD
- **Images**: JPEG, PNG, BMP, TIFF, GIF (processed with OCR)
- **Emails**: EML, MBOX, MBX
- **Data**: CSV
- **Other**: Various text-based formats

## Document Processing

When documents are uploaded, the system:

1. **Validates** file types and sizes
2. **Extracts text** using appropriate methods:
   - Gemini for supported formats (PDF, images, DOCX)
   - Fallback parsers for other formats
3. **Analyzes content** to extract:
   - Document title and author
   - Summary and key points
   - Named entities and categories
   - Language and word count
4. **Chunks text** into manageable pieces
5. **Generates embeddings** using Gemini
6. **Stores vectors** in PostgreSQL

## Querying

The RAG system supports natural language queries that:

- Search through document content semantically
- Retrieve the most relevant text chunks
- Generate contextual answers using Gemini
- Cache results for 5 minutes to improve performance

## Development

### Project Structure

```text
magi-poc-rag-ocr-processing/
├── app.py              # FastAPI application
├── main.py             # RAG engine and document processing
├── models.py           # Pydantic data models
├── requirements.txt    # Python dependencies
├── gcs_credentials.json # Google Cloud credentials (not in repo)
└── .env               # Environment variables (not in repo)
```

### Key Components

- **CustomGeminiEmbedding**: Custom embedding class using Gemini
- **Document Processing Pipeline**: Text extraction, structuring, chunking
- **Vector Store Integration**: PostgreSQL with pgvector
- **Query Engine**: Semantic search with LLM-powered responses

## Limitations

This is a POC implementation with the following limitations:

- Single index per upload (no multi-index support in UI)
- Basic error handling for document processing
- No user authentication or authorization
- Limited scalability testing
- No monitoring or logging infrastructure

## Contributing

This is a proof of concept project. For production use, consider:

- Adding comprehensive error handling
- Implementing user management and authentication
- Adding monitoring and logging
- Performance optimization for large document sets
- UI for document management and querying

## License

[Add appropriate license information]

## Contact

For questions or support, please contact the development team.
