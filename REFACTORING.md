# Refactored RAG System

## 🎯 Architecture Overview

This is a **production-ready, refactored** RAG (Retrieval-Augmented Generation) system with clean separation of concerns.

## 📁 New Project Structure

```
.
├── config/                     # Configuration management
│   ├── __init__.py
│   └── settings.py            # Centralized settings with validation
│
├── models/                     # Data models (Pydantic)
│   ├── __init__.py
│   └── models.py              # DocumentMetadata, Conversation, etc.
│
├── services/                   # Business logic layer
│   ├── __init__.py
│   ├── database.py            # Database operations
│   ├── storage.py             # GCS operations
│   └── rag_service.py         # RAG engine (to be created)
│
├── utils/                      # Utilities
│   ├── __init__.py
│   ├── embeddings.py          # Custom Gemini embeddings
│   ├── file_processing.py     # File extraction utilities
│   └── logging_config.py      # Logging setup
│
├── api/                        # API layer (FastAPI routes)
│   ├── __init__.py
│   └── routes.py              # API endpoints
│
├── logs/                       # Application logs
│
├── app.py                      # Main FastAPI application
├── main.py                     # Legacy (to be deprecated)
├── models.py                   # Legacy models (kept for compatibility)
├── requirements.txt            # Python dependencies
├── .env                        # Environment variables
└── README.md                   # This file
```

## 🔧 Key Improvements

### 1. **Configuration Management**
- ✅ Centralized configuration in `config/settings.py`
- ✅ Pydantic validation for all settings
- ✅ Environment variable parsing with defaults
- ✅ Type-safe configuration access

### 2. **Service Layer Pattern**
- ✅ **DatabaseService**: All DB operations isolated
- ✅ **GCSService**: Cloud storage operations
- ✅ **RAGService**: Core RAG functionality (to be migrated)

### 3. **Separation of Concerns**
- ✅ API layer separate from business logic
- ✅ Database layer with connection pooling
- ✅ Storage layer for file operations
- ✅ Utilities for cross-cutting concerns

### 4. **Error Handling**
- ✅ Consistent error handling patterns
- ✅ Retry logic for database connections
- ✅ Graceful degradation
- ✅ Comprehensive logging

### 5. **Code Quality**
- ✅ Type hints throughout
- ✅ Docstrings for all functions
- ✅ Single Responsibility Principle
- ✅ DRY (Don't Repeat Yourself)

## 🚀 Migration Path

### Phase 1: Foundation ✅ COMPLETED
- [x] Configuration management
- [x] Database service
- [x] Storage service
- [x] Utility modules

### Phase 2: Core RAG (Next Steps)
- [ ] Extract RAG logic to `services/rag_service.py`
- [ ] Document ingestion service
- [ ] Query processing service
- [ ] Conversation management service

### Phase 3: API Layer
- [ ] Refactor API routes to use new services
- [ ] Update `app.py` to use refactored services
- [ ] Remove dependencies on `main.py`

### Phase 4: Cleanup
- [ ] Deprecate `main.py`
- [ ] Update documentation
- [ ] Add unit tests
- [ ] Performance optimization

## 📝 Usage Examples

### Using the New Configuration

```python
from config import settings

# Access typed configuration
print(settings.database.host)
print(settings.ai.vector_dimension)
print(settings.rag.chunk_size)
```

### Using Database Service

```python
from services import db_service
from models import DocumentMetadata

# Save document
metadata = DocumentMetadata(...)
db_service.save_document(metadata)

# Get all documents
documents = db_service.get_all_documents(index_name="default")

# Test connection
if db_service.test_connection():
    print("Database connected!")
```

### Using Storage Service

```python
from services import gcs_service

# Upload file
blob_name = gcs_service.upload_file(file_content, "example.pdf")

# Download file
local_path = gcs_service.download_file(blob_name)

# Delete file
gcs_service.delete_file(blob_name)
```

## 🔑 Benefits of Refactoring

1. **Maintainability**: Clear structure, easy to find code
2. **Testability**: Services can be unit tested independently
3. **Scalability**: Easy to add new features
4. **Reusability**: Services can be used across different endpoints
5. **Type Safety**: Full type hints reduce bugs
6. **Documentation**: Self-documenting code with clear structure

## 📊 Comparison: Before vs After

### Before (Monolithic)
```
app.py (592 lines) + main.py (2392 lines) = 2984 lines in 2 files
- Configuration scattered everywhere
- DB code duplicated
- No clear separation of concerns
- Hard to test
```

### After (Modular)
```
config/ (120 lines)
services/ (500+ lines, well-organized)
utils/ (300+ lines)
api/ (to be created)
- Centralized configuration
- Reusable services
- Clear separation of concerns
- Easy to test
```

## 🛠️ Next Steps

To complete the refactoring:

1. **Create RAG Service** (`services/rag_service.py`)
   - Extract vector store logic
   - Extract query processing
   - Extract conversation management

2. **Create API Routes** (`api/routes.py`)
   - Move FastAPI routes from `app.py`
   - Use new services
   - Clean endpoint logic

3. **Update Main App** (`app.py`)
   - Import from new structure
   - Remove old dependencies
   - Use dependency injection

4. **Add Tests**
   - Unit tests for services
   - Integration tests for API
   - Test fixtures

## 📚 Documentation

Each module includes:
- Module docstring explaining purpose
- Function docstrings with Args/Returns
- Type hints for all parameters
- Inline comments for complex logic

## 🎓 Learning Resources

- **Pydantic**: https://docs.pydantic.dev/
- **FastAPI**: https://fastapi.tiangolo.com/
- **Service Layer Pattern**: https://martinfowler.com/eaaCatalog/serviceLayer.html
- **Clean Architecture**: https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html

---

**Status**: Phase 1 Complete ✅ | Phase 2 In Progress 🚧
