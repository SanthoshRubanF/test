# Quick Start Guide - Refactored RAG System

## 🚀 Getting Started

### Prerequisites

- Python 3.12+
- PostgreSQL with pgvector extension
- Google Cloud Storage account
- Google AI API key

### Installation

1. **Clone and Navigate**
   ```bash
   cd "e:\VEGAS\AI Agents\Backup RAG\9 Final in Cloud DB"
   ```

2. **Create Virtual Environment**
   ```powershell
   python -m venv .venv
   .\.venv\Scripts\activate
   ```

3. **Install Dependencies**
   ```powershell
   pip install -r requirements-refactored.txt
   ```

4. **Configure Environment**
   ```powershell
   # Copy example env file
   Copy-Item .env.example .env
   
   # Edit .env with your values
   notepad .env
   ```

5. **Test Configuration**
   ```python
   # test_config.py
   from config import settings
   print(f"Database: {settings.database.host}")
   print(f"Bucket: {settings.gcs.bucket_name}")
   ```

## 📚 Usage Examples

### Using Services Directly

#### Database Service
```python
from services import db_service
from models import DocumentMetadata
from datetime import datetime, timezone

# Test connection
if db_service.test_connection():
    print("✅ Database connected!")

# Save a document
metadata = DocumentMetadata(
    document_id="test-123",
    filename="example.pdf",
    source="upload",
    upload_timestamp=datetime.now(timezone.utc),
    index_name="default",
    mime_type="application/pdf",
    file_size=1024,
    blob_name="unique-blob-name",
    ingestion_status="pending"
)
db_service.save_document(metadata)

# Get all documents
documents = db_service.get_all_documents(index_name="default")
for doc in documents:
    print(f"Document: {doc.filename}, Status: {doc.ingestion_status}")

# Update status
db_service.update_document_status("test-123", "completed")
```

#### Storage Service
```python
from services import gcs_service

# Upload file
with open("example.pdf", "rb") as f:
    content = f.read()
    blob_name = gcs_service.upload_file(content, "example.pdf")
    print(f"Uploaded to: {blob_name}")

# Download file
local_path = gcs_service.download_file(blob_name)
print(f"Downloaded to: {local_path}")

# Check if exists
if gcs_service.file_exists(blob_name):
    print("File exists in GCS")

# Get metadata
metadata = gcs_service.get_file_metadata(blob_name)
print(f"File size: {metadata['size']} bytes")

# Delete file
gcs_service.delete_file(blob_name)
```

### Using Utilities

#### Custom Embeddings
```python
from utils import CustomGeminiEmbedding

# Create embedding model
embed_model = CustomGeminiEmbedding()

# Get embedding for text
text = "This is a sample document"
embedding = embed_model._get_text_embedding(text)
print(f"Embedding dimension: {len(embedding)}")

# Batch embeddings
texts = ["Text 1", "Text 2", "Text 3"]
embeddings = embed_model._get_text_embeddings(texts)
print(f"Got {len(embeddings)} embeddings")
```

#### File Processing
```python
from utils import validate_file_extension, extract_text_with_gemini

# Validate file
if validate_file_extension("document.pdf"):
    print("✅ File type supported")

# Extract text
text = extract_text_with_gemini("document.pdf")
print(f"Extracted {len(text)} characters")
```

#### Query Expansion
```python
from utils import expand_query_with_gemini

query = "How do I configure the database?"
expanded = expand_query_with_gemini(query, max_expansions=3)

print("Original:", query)
print("Variations:", expanded[1:])
```

## 🔧 Configuration Reference

### Database Settings
```python
from config import settings

# Access database config
settings.database.host
settings.database.port
settings.database.name
settings.database.ssl_mode
settings.database.min_connections
settings.database.max_connections
```

### AI Settings
```python
# Access AI config
settings.ai.google_api_key
settings.ai.llm_model              # "models/gemini-2.5-flash"
settings.ai.embedding_model        # "models/text-embedding-004"
settings.ai.vector_dimension       # 768
settings.ai.similarity_top_k       # 5
```

### Server Settings
```python
# Access server config
settings.server.host
settings.server.port
settings.server.debug
settings.server.log_level
```

### RAG Settings
```python
# Access RAG config
settings.rag.chunk_size            # 512
settings.rag.chunk_overlap         # 128
settings.rag.hybrid_search_weight  # 0.7
settings.rag.use_cross_encoder     # True
```

## 🧪 Testing

### Test Database Connection
```python
from services import db_service

# Simple test
success = db_service.test_connection()
print("Database OK" if success else "Database Failed")

# Detailed test
try:
    with db_service.get_connection() as conn:
        cursor = conn.cursor()
        cursor.execute("SELECT version()")
        version = cursor.fetchone()
        print(f"PostgreSQL version: {version[0]}")
        cursor.close()
except Exception as e:
    print(f"Error: {e}")
```

### Test GCS Connection
```python
from services import gcs_service

# List files
files = gcs_service.list_files()
print(f"Found {len(files)} files in bucket")

# Test upload/download/delete cycle
test_content = b"Hello, World!"
blob_name = gcs_service.upload_file(test_content, "test.txt")
local = gcs_service.download_file(blob_name)

with open(local, "rb") as f:
    assert f.read() == test_content

gcs_service.delete_file(blob_name)
print("✅ GCS test passed")
```

## 📝 Common Tasks

### Adding a New Service

1. Create file in `services/` directory
2. Define service class
3. Export in `services/__init__.py`

Example:
```python
# services/my_service.py
import logging
from config import settings

logger = logging.getLogger(__name__)

class MyService:
    def __init__(self):
        logger.info("MyService initialized")
    
    def my_method(self):
        # Implementation
        pass

my_service = MyService()
```

### Adding a New Utility

1. Create file in `utils/` directory
2. Define utility functions
3. Export in `utils/__init__.py`

Example:
```python
# utils/my_util.py
def my_utility_function(param: str) -> str:
    """
    Utility function description.
    
    Args:
        param: Parameter description
        
    Returns:
        Return value description
    """
    return param.upper()
```

### Logging

```python
from utils import get_logger

logger = get_logger(__name__)

logger.debug("Debug message")
logger.info("Info message")
logger.warning("Warning message")
logger.error("Error message")
logger.critical("Critical message")
```

## 🐛 Troubleshooting

### Database Connection Issues

```python
from services import db_service

# Refresh connection pool
db_service.refresh_pool()

# Test after refresh
db_service.test_connection()
```

### Import Errors

If you get import errors:
```powershell
# Make sure you're in the project directory
cd "e:\VEGAS\AI Agents\Backup RAG\9 Final in Cloud DB"

# Make sure virtual environment is activated
.\.venv\Scripts\activate

# Reinstall dependencies
pip install -r requirements-refactored.txt
```

### Configuration Errors

```python
# Validate configuration
from config import settings

try:
    print("Configuration loaded successfully!")
    print(f"Database: {settings.database.host}")
    print(f"GCS Bucket: {settings.gcs.bucket_name}")
except Exception as e:
    print(f"Configuration error: {e}")
    print("Check your .env file")
```

## 📊 Monitoring

### Check Service Health

```python
from services import db_service, gcs_service

# Database health
db_healthy = db_service.test_connection()

# GCS health
gcs_healthy = len(gcs_service.list_files()) >= 0

print(f"Database: {'✅' if db_healthy else '❌'}")
print(f"GCS: {'✅' if gcs_healthy else '❌'}")
```

## 🎓 Next Steps

1. **Explore the refactored code**: Check `services/`, `utils/`, `config/`
2. **Read REFACTORING.md**: Understand the architecture
3. **Review models.py**: Understand data structures
4. **Test services**: Run the examples above
5. **Integration**: Wait for Phase 2 completion for full RAG integration

---

**Need Help?** Check `REFACTORING.md` for detailed architecture documentation.
