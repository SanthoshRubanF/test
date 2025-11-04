# Conversational RAG API - Architecture Overview

## System Architecture

```text
┌─────────────────────────────────────────────────────────────────┐
│                         CLIENT APPLICATION                       │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                    FASTAPI APPLICATION (app.py)                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │          CONVERSATIONAL ENDPOINTS (NEW)                  │  │
│  ├──────────────────────────────────────────────────────────┤  │
│  │  • POST /conversation/create                             │  │
│  │  • POST /conversation/query    ← Main Feature            │  │
│  │  • GET  /conversation/{id}                               │  │
│  │  • GET  /conversations                                   │  │
│  │  • DELETE /conversation/{id}                             │  │
│  │  • GET  /documents                                       │  │
│  │  • GET  /document/{id}                                   │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │          DOCUMENT ENDPOINTS                              │  │
│  ├──────────────────────────────────────────────────────────┤  │
│  │  • POST /upload                                          │  │
│  │  • POST /query (legacy stateless)                        │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                   │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                    CORE LOGIC (main.py)                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  CONVERSATIONAL QUERY ENGINE (NEW)                      │   │
│  ├─────────────────────────────────────────────────────────┤   │
│  │                                                          │   │
│  │  query_rag_conversational()                             │   │
│  │    ├─ Retrieve conversation history                     │   │
│  │    ├─ Build contextual prompt                           │   │
│  │    ├─ Add document filter (if specified)                │   │
│  │    ├─ Query vector store                                │   │
│  │    └─ Return contextual answer                          │   │
│  │                                                          │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  STORAGE MANAGERS (NEW)                                 │   │
│  ├─────────────────────────────────────────────────────────┤   │
│  │                                                          │   │
│  │  conversations_store: Dict[str, Dict[str, Any]]         │   │
│  │    └─ {conversation_id: {conversation, messages}}       │   │
│  │                                                          │   │
│  │  documents_store: Dict[str, DocumentMetadata]           │   │
│  │    └─ {document_id: metadata}                           │   │
│  │                                                          │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  EXISTING RAG FUNCTIONS                                 │   │
│  ├─────────────────────────────────────────────────────────┤   │
│  │  • ingest_files()                                       │   │
│  │  • query_rag() (stateless)                              │   │
│  │  • load_documents()                                     │   │
│  │  • extract_text_with_gemini()                           │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                   │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                     EXTERNAL SERVICES                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │ PostgreSQL   │  │ Google Cloud │  │ Google Gemini AI     │  │
│  │ (PGVector)   │  │ Storage (GCS)│  │ (LLM + Embeddings)   │  │
│  │              │  │              │  │                      │  │
│  │ • Vectors    │  │ • Documents  │  │ • Text Generation    │  │
│  │ • Indices    │  │ • Blobs      │  │ • Embeddings         │  │
│  └──────────────┘  └──────────────┘  └──────────────────────┘  │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```text

---

## Data Flow: Conversational Query

```text
1. USER REQUEST
   │
   └─► POST /conversation/query
       {
         "query": "What are the key findings?",
         "conversation_id": "conv-123",
         "document_id": "doc-456"
       }

2. API ENDPOINT (app.py)
   │
   ├─► Validate conversation_id exists
   ├─► Retrieve conversation data from conversations_store
   └─► Extract message history

3. BUILD CONTEXT (main.py)
   │
   └─► query_rag_conversational()
       │
       ├─► Get last 5 messages from history
       │   Example:
       │   User: "What is the main topic?"
       │   Assistant: "The main topic is machine learning..."
       │   User: "Can you elaborate?"
       │   Assistant: "Machine learning involves..."
       │
       ├─► Build contextual prompt:
       │   """
       │   Previous conversation:
       │   User: What is the main topic?
       │   Assistant: The main topic is machine learning...
       │   User: Can you elaborate?
       │   Assistant: Machine learning involves...
       │   
       │   Current question (Focus on document ID: doc-456): 
       │   What are the key findings?
       │   """
       │
       └─► Query vector store with full context

4. VECTOR STORE QUERY
   │
   ├─► Generate embedding for contextual query
   ├─► Search PGVector database
   ├─► Retrieve top K similar chunks
   └─► Filter by document_id if specified

5. LLM GENERATION
   │
   ├─► Send context + retrieved chunks to Gemini
   ├─► Generate contextual answer
   └─► Return response

6. UPDATE CONVERSATION
   │
   ├─► Add user message to conversation history
   ├─► Add assistant response to conversation history
   ├─► Update conversation timestamp
   └─► Store in conversations_store

7. RETURN TO CLIENT
   │
   └─► Response:
       {
         "conversation_id": "conv-123",
         "message": {...},
         "answer": "The key findings are..."
       }
```text

---

## Conversation Storage Structure

```python
conversations_store = {
    "conv-uuid-123": {
        "conversation": {
            "conversation_id": "conv-uuid-123",
            "document_id": "doc-uuid-456",
            "document_name": "research_paper.pdf",
            "index_name": "default",
            "created_at": "2025-11-04T10:00:00Z",
            "updated_at": "2025-11-04T10:05:00Z",
            "user_id": "user-789"
        },
        "messages": [
            {
                "role": "user",
                "content": "What is the main topic?",
                "timestamp": "2025-11-04T10:00:15Z",
                "document_id": "doc-uuid-456"
            },
            {
                "role": "assistant",
                "content": "The main topic is...",
                "timestamp": "2025-11-04T10:00:18Z",
                "document_id": "doc-uuid-456"
            },
            {
                "role": "user",
                "content": "Can you elaborate?",
                "timestamp": "2025-11-04T10:01:00Z",
                "document_id": "doc-uuid-456"
            },
            {
                "role": "assistant",
                "content": "Certainly! The topic involves...",
                "timestamp": "2025-11-04T10:01:05Z",
                "document_id": "doc-uuid-456"
            }
        ]
    }
}
```text

---

## Document Storage Structure

```python
documents_store = {
    "doc-uuid-456": {
        "document_id": "doc-uuid-456",
        "filename": "research_paper.pdf",
        "source": "upload",
        "upload_timestamp": "2025-11-04T09:00:00Z",
        "user_id": "user-789",
        "index_name": "default",
        "mime_type": "application/pdf",
        "file_size": 1024000,
        "blob_name": "uuid_research_paper.pdf"
    }
}
```text

---

## Comparison: Before vs After

### BEFORE (Response AI)

```text
User Query → RAG System → Answer
              ↓
        (No Memory)
```text

**Characteristics:**

- ❌ No conversation state
- ❌ No context retention
- ❌ Each query is independent
- ❌ Cannot handle follow-up questions
- ✅ Simple and fast
- ✅ Stateless (easy to scale)

---

### AFTER (Conversational AI)

```text
User Query → Conversation Manager → Context Builder → RAG System → Answer
                    ↓                      ↑
              Conversation Store ──────────┘
              Document Store
```text

**Characteristics:**

- ✅ Maintains conversation state
- ✅ Retains context across queries
- ✅ Multi-turn dialogue support
- ✅ Handles follow-up questions naturally
- ✅ Document-specific conversations
- ✅ Conversation history tracking
- ⚠️  More complex architecture
- ⚠️  Requires state management

---

## Key Features Comparison

| Feature | Response AI (Before) | Conversational AI (After) |
|---------|---------------------|---------------------------|
| Context Retention | ❌ No | ✅ Yes (last 5 messages) |
| Follow-up Questions | ❌ No | ✅ Yes |
| Document Filtering | ❌ No | ✅ Yes |
| Conversation History | ❌ No | ✅ Yes |
| Session Management | ❌ No | ✅ Yes |
| Multi-turn Dialogue | ❌ No | ✅ Yes |
| Backward Compatible | N/A | ✅ Yes (/query still works) |

---

## Usage Pattern

### Pattern 1: Explicit Conversation Creation

```text
1. Create conversation
2. Query with conversation_id
3. Continue querying with same conversation_id
4. View conversation history
5. Delete when done
```text

### Pattern 2: Implicit Conversation Creation

```text
1. Query without conversation_id
2. System auto-creates conversation
3. Continue with returned conversation_id
```text

### Pattern 3: Document-Specific Conversation

```text
1. Upload document → get document_id
2. Create conversation with document_id
3. All queries focus on that document
```text

---

## Scalability Considerations

### Current (Development)

- In-memory storage
- Fast but limited
- Resets on server restart

### Production Recommendations

- PostgreSQL for conversations
- Redis for caching
- Conversation pagination
- Cleanup old conversations
- Rate limiting per user

---

## Future Enhancements

1. **Persistent Storage** - Database-backed conversations
2. **User Authentication** - Multi-tenant support
3. **Conversation Search** - Full-text search in history
4. **Export/Import** - Save conversations
5. **Analytics** - Usage metrics and insights
6. **Streaming Responses** - Real-time answer generation
7. **Conversation Sharing** - Share conversation links
8. **Voice Integration** - Speech-to-text/text-to-speech

---

This architecture transforms the system from a simple Q&A tool into a sophisticated conversational AI platform! 🚀
