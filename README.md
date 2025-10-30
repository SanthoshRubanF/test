# MAGI RAG POC FastAPI

A Retrieval-Augmented Generation (RAG) system built with FastAPI, LlamaIndex, and Google Gemini AI. This system allows you to upload documents, automatically process them, and query for information using natural language.

## Features

- **Document Upload & Processing**: Upload various file types (PDF, DOCX, images, emails, etc.) to Google Cloud Storage
- **Automatic Text Extraction**: Uses Google Gemini AI to extract text from multiple file formats
- **Structured Data Extraction**: Automatically extracts metadata like title, author, summary, key points, and entities
- **Vector Storage**: Stores document embeddings in PostgreSQL with pgvector for efficient similarity search
- **Natural Language Querying**: Query the system using natural language to get relevant answers
- **RESTful API**: Clean FastAPI-based REST API for easy integration
- **Multiple Index Support**: Support for multiple document indexes/namespaces

## Supported File Types

- PDF documents
- Microsoft Word documents (.docx, .doc)
- Text files (.txt)
- Rich Text Format (.rtf)
- HTML files
- Markdown files
- CSV files
- Images (JPEG, PNG, GIF, BMP, TIFF)
- Email files (.eml, .msg, .mbox)

## Prerequisites

- Python 3.8+
- Google Cloud Platform account with:
  - Google Cloud Storage bucket
  - Service account with appropriate permissions
  - Google AI API key (for Gemini)
- PostgreSQL database with pgvector extension
- Google Cloud credentials JSON file

## Installation

1. Clone the repository:

   ```bash
   git clone <repository-url>
   cd magi_rag_poc_fastapi
   ```

2. Install dependencies:

   ```bash
   pip install -r requirements.txt
   ```

3. Set up environment variables by creating a `.env` file:

   ```env
   # Google Cloud Configuration
   GOOGLE_API_KEY=your_google_ai_api_key
   GOOGLE_APPLICATION_CREDENTIALS=gcs_credentials.json
   GCS_BUCKET_NAME=your_gcs_bucket_name

   # Database Configuration
   DB_HOST=your_postgres_host
   DB_PORT=5432
   DB_NAME=your_database_name
   DB_USER=your_database_user
   DB_PASSWORD=your_database_password

   # Application Configuration
   APP_HOST=0.0.0.0
   APP_PORT=8000
   VECTOR_DIMENSION=768
   SIMILARITY_TOP_K=5
   ```

4. Place your Google Cloud service account credentials in `gcs_credentials.json`

5. Ensure your PostgreSQL database has the pgvector extension enabled:

   ```sql
   CREATE EXTENSION IF NOT EXISTS vector;
   ```

## Usage

### Starting the Server

Run the FastAPI server:

```bash
python -m uvicorn app:app --host 127.0.0.1 --port 8000
```

Or run directly:

```bash
python app.py
```

The API will be available at `http://127.0.0.1:8000`

### API Endpoints

#### Upload Files

```http
POST /upload
```

Upload files to be processed and ingested into the RAG system.

**Parameters:**

- `files`: List of files to upload (multipart/form-data)
- `index_name`: Name of the index to store documents in (default: "default")

**Example using curl:**

```bash
curl -X POST "http://127.0.0.1:8000/upload" \
     -F "files=@document.pdf" \
     -F "files=@document.docx" \
     -F "index_name=my_index"
```

**Response:**

```json
{
  "message": "Successfully uploaded and ingested 2 files",
  "uploaded_files": ["uuid1_document.pdf", "uuid2_document.docx"],
  "document_ids": ["doc-id-1", "doc-id-2"],
  "ingest_result": "Successfully ingested 2 files"
}
```

#### Query the RAG System

```http
POST /query
```

Query the system for information based on ingested documents.

**Request Body:**

```json
{
  "query": "What are the main points discussed in the documents?",
  "index_name": "default",
  "top_k": 5
}
```

**Example using curl:**

```bash
curl -X POST "http://127.0.0.1:8000/query" \
     -H "Content-Type: application/json" \
     -d '{
       "query": "Summarize the key findings",
       "index_name": "default",
       "top_k": 3
     }'
```

**Response:**

```json
{
  "answer": "Based on the documents, the key findings include..."
}
```

#### Health Check

```http
GET /
```

Returns a simple health check response.

## Architecture

### Components

1. **FastAPI Application** (`app.py`): REST API endpoints for file upload and querying
2. **RAG Engine** (`main.py`): Core RAG functionality including:
   - Document processing and text extraction
   - Structured data extraction using Gemini AI
   - Vector embeddings generation
   - Query processing and response generation
3. **Data Models** (`models.py`): Pydantic models for API requests/responses and document metadata
4. **Google Cloud Storage**: File storage for uploaded documents
5. **PostgreSQL + pgvector**: Vector database for document embeddings
6. **Google Gemini AI**: LLM for text extraction, structured data parsing, and query responses

### Data Flow

1. **Upload**: Files are uploaded via API → stored in GCS → processed for text extraction → structured data extracted → embeddings generated → stored in vector database
2. **Query**: Natural language query → converted to embedding → similarity search in vector database → relevant documents retrieved → LLM generates response

## Configuration

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `GOOGLE_API_KEY` | Google AI API key for Gemini | Required |
| `GCS_BUCKET_NAME` | Google Cloud Storage bucket name | Required |
| `DB_HOST` | PostgreSQL host | Required |
| `DB_PORT` | PostgreSQL port | 5432 |
| `DB_NAME` | PostgreSQL database name | Required |
| `DB_USER` | PostgreSQL username | Required |
| `DB_PASSWORD` | PostgreSQL password | Required |
| `APP_HOST` | FastAPI host | 0.0.0.0 |
| `APP_PORT` | FastAPI port | 8000 |
| `VECTOR_DIMENSION` | Embedding vector dimension | 768 |
| `SIMILARITY_TOP_K` | Number of similar documents to retrieve | 5 |

## Development

### Project Structure

```text
.
├── app.py              # FastAPI application and endpoints
├── main.py             # Core RAG functionality
├── models.py           # Data models and schemas
├── requirements.txt    # Python dependencies
├── gcs_credentials.json # Google Cloud credentials (not in repo)
└── README.md          # This file
```

### Running Tests

```bash
# Install test dependencies if any
pip install pytest

# Run tests
pytest
```

## API Documentation

Once the server is running, visit `http://127.0.0.1:8000/docs` for interactive API documentation powered by Swagger UI.

## Troubleshooting

### Common Issues

1. **Google Cloud Authentication Errors**
   - Ensure `gcs_credentials.json` is present and contains valid service account credentials
   - Verify the service account has appropriate permissions for GCS and AI APIs

2. **Database Connection Issues**
   - Ensure PostgreSQL is running and accessible
   - Verify pgvector extension is installed: `CREATE EXTENSION IF NOT EXISTS vector;`
   - Check database credentials in `.env` file

3. **File Processing Errors**
   - Verify uploaded files are in supported formats
   - Check file size limits and API quotas for Gemini

4. **Query Returns No Results**
   - Ensure documents have been successfully ingested
   - Check that the correct `index_name` is being used
   - Verify vector database contains data

### Logs

Check the console output for detailed error messages and processing information.

## License

[Add your license information here]

## Contributing

[Add contribution guidelines here]
