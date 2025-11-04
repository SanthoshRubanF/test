# Conversational RAG API Guide

## Overview

This API has been refactored from a simple Response AI to a full **Conversational AI** system that maintains conversation context across multiple queries. The system now supports:

- **Document-specific conversations**: Query specific documents with context awareness
- **Multi-turn dialogue**: Maintain conversation history for contextual responses
- **Session management**: Create, track, and manage conversation sessions
- **Document tracking**: Associate conversations with specific documents

## Key Differences: Response AI vs Conversational AI

### Before (Response AI)

- Each query was independent
- No conversation history
- No context retention
- Single endpoint for all queries

### After (Conversational AI)

- Conversations maintain context across multiple turns
- Full conversation history tracking
- Document-specific query support
- Session-based interactions
- Multiple endpoints for different use cases

---

## API Endpoints

### 1. Upload Documents

**POST** `/upload`

Upload files and automatically ingest them into the vector database.

**Request:**

```bash
curl -X POST "http://localhost:8080/upload" \

  -F "files=@document.pdf" \
  -F "index_name=my_index"
```text

**Response:**

```json
{
  "message": "Successfully uploaded 1 files. Ingestion started in background.",
  "uploaded_files": ["uuid_document.pdf"],
  "document_ids": ["doc-uuid-123"],
  "status": "ingestion_in_progress"
}
```text

---

### 2. List Documents

**GET** `/documents?index_name=default`

Get all documents in a specific index.

**Response:**

```json
{
  "documents": [
    {
      "document_id": "doc-uuid-123",
      "filename": "document.pdf",
      "upload_timestamp": "2025-11-04T10:30:00Z",
      "mime_type": "application/pdf",
      "file_size": 102400
    }
  ],
  "total": 1,
  "index_name": "default"
}
```text

---

### 3. Get Document Details

**GET** `/document/{document_id}`

Get metadata for a specific document.

**Response:**

```json
{
  "document_id": "doc-uuid-123",
  "filename": "document.pdf",
  "upload_timestamp": "2025-11-04T10:30:00Z",
  "index_name": "default",
  "mime_type": "application/pdf",
  "file_size": 102400
}
```text

---

### 4. Create Conversation

**POST** `/conversation/create`

Create a new conversation session, optionally tied to a specific document.

**Request:**

```json
{
  "document_id": "doc-uuid-123",  // Optional: tie to specific document
  "index_name": "default",
  "user_id": "user-123"  // Optional
}
```text

**Response:**

```json
{
  "conversation_id": "conv-uuid-456",
  "document_id": "doc-uuid-123",
  "document_name": "document.pdf",
  "index_name": "default",
  "created_at": "2025-11-04T10:35:00Z",
  "message": "Conversation created successfully"
}
```text

---

### 5. Conversational Query (Main Feature)

**POST** `/conversation/query`

Query with conversation context. This is the core conversational AI endpoint.

**Request:**

```json
{
  "query": "What is the main topic of this document?",
  "conversation_id": "conv-uuid-456",  // Use existing conversation
  "document_id": "doc-uuid-123",  // Optional: focus on specific document
  "index_name": "default",
  "top_k": 5  // Optional: number of similar chunks to retrieve
}
```text

**Response:**

```json
{
  "conversation_id": "conv-uuid-456",
  "message": {
    "role": "user",
    "content": "What is the main topic of this document?",
    "timestamp": "2025-11-04T10:36:00Z",
    "document_id": "doc-uuid-123"
  },
  "answer": "Based on the document, the main topic is..."
}
```text

**Auto-create conversation:**
If you don't provide `conversation_id`, a new conversation will be created automatically:

```json
{
  "query": "Tell me about this document",
  "document_id": "doc-uuid-123",
  "index_name": "default"
}
```text

---

### 6. Get Conversation History

**GET** `/conversation/{conversation_id}`

Retrieve full conversation history.

**Response:**

```json
{
  "conversation": {
    "conversation_id": "conv-uuid-456",
    "document_id": "doc-uuid-123",
    "document_name": "document.pdf",
    "index_name": "default",
    "created_at": "2025-11-04T10:35:00Z",
    "updated_at": "2025-11-04T10:36:00Z"
  },
  "messages": [
    {
      "role": "user",
      "content": "What is the main topic?",
      "timestamp": "2025-11-04T10:36:00Z"
    },
    {
      "role": "assistant",
      "content": "The main topic is...",
      "timestamp": "2025-11-04T10:36:01Z"
    }
  ]
}
```text

---

### 7. List All Conversations

**GET** `/conversations`

Get all conversation sessions.

**Response:**

```json
{
  "conversations": [
    {
      "conversation_id": "conv-uuid-456",
      "document_id": "doc-uuid-123",
      "document_name": "document.pdf",
      "index_name": "default",
      "created_at": "2025-11-04T10:35:00Z",
      "updated_at": "2025-11-04T10:36:00Z"
    }
  ],
  "total": 1
}
```text

---

### 8. Delete Conversation

**DELETE** `/conversation/{conversation_id}`

Delete a conversation session.

**Response:**

```json
{
  "message": "Conversation conv-uuid-456 deleted successfully"
}
```text

---

### 9. Simple Query (Legacy)

**POST** `/query`

Stateless query without conversation history (original Response AI behavior).

**Request:**

```json
{
  "query": "What is machine learning?",
  "index_name": "default",
  "top_k": 5
}
```text

**Response:**

```json
{
  "answer": "Machine learning is..."
}
```text

---

## Usage Examples

### Example 1: Document-Specific Conversation

```python
import requests

base_url = "http://localhost:8080"

# 1. Upload a document
files = {'files': open('research_paper.pdf', 'rb')}
upload_response = requests.post(f"{base_url}/upload", files=files)
document_id = upload_response.json()['document_ids'][0]

# 2. Create a conversation for this document
create_conv = requests.post(f"{base_url}/conversation/create", json={
    "document_id": document_id,
    "index_name": "default"
})
conversation_id = create_conv.json()['conversation_id']

# 3. Ask first question
query1 = requests.post(f"{base_url}/conversation/query", json={
    "query": "What is the main finding of this research?",
    "conversation_id": conversation_id,
    "document_id": document_id
})
print("Answer 1:", query1.json()['answer'])

# 4. Ask follow-up question (context is maintained)
query2 = requests.post(f"{base_url}/conversation/query", json={
    "query": "Can you elaborate on that finding?",
    "conversation_id": conversation_id,
    "document_id": document_id
})
print("Answer 2:", query2.json()['answer'])

# 5. Ask another follow-up
query3 = requests.post(f"{base_url}/conversation/query", json={
    "query": "What are the implications?",
    "conversation_id": conversation_id,
    "document_id": document_id
})
print("Answer 3:", query3.json()['answer'])
```text

### Example 2: Quick Conversational Query

```python

# Auto-create conversation and query in one step
response = requests.post(f"{base_url}/conversation/query", json={
    "query": "Summarize this document",
    "document_id": "doc-uuid-123"
})
print("Answer:", response.json()['answer'])
print("Conversation ID:", response.json()['conversation_id'])
```text

### Example 3: Multi-Document Conversation

```python

# Create general conversation (not tied to specific document)
create_conv = requests.post(f"{base_url}/conversation/create", json={
    "index_name": "default"
})
conversation_id = create_conv.json()['conversation_id']

# Query across all documents
query = requests.post(f"{base_url}/conversation/query", json={
    "query": "Compare the findings from all research papers",
    "conversation_id": conversation_id
})
```text

---

## Key Features

### 1. **Context Retention**
The system maintains the last 5 messages in conversation history to provide context for each new query.

### 2. **Document Filtering**
When `document_id` is provided, the system focuses queries on that specific document.

### 3. **Flexible Workflow**

- Create conversation first, then query (explicit)
- Query directly and auto-create conversation (implicit)
- Mix document-specific and general queries

### 4. **Conversation Management**

- Track all conversations
- View conversation history
- Delete old conversations
- Associate conversations with documents

---

## Technical Implementation

### Conversation Storage
Conversations are stored in-memory with this structure:

```python
conversations_store[conversation_id] = {
    "conversation": {
        "conversation_id": str,
        "document_id": Optional[str],
        "document_name": Optional[str],
        "index_name": str,
        "created_at": datetime,
        "updated_at": datetime
    },
    "messages": [
        {"role": "user", "content": str, "timestamp": datetime},
        {"role": "assistant", "content": str, "timestamp": datetime}
    ]
}
```text

### Query Processing
The `query_rag_conversational` function:
1. Retrieves conversation history
2. Builds contextual prompt with last 5 messages
3. Adds document filter if specified
4. Queries the RAG system with full context
5. Returns contextual answer

---

## Migration Guide

### Old Code (Response AI)

```python
response = requests.post("http://localhost:8080/query", json={
    "query": "What is this about?"
})
```text

### New Code (Conversational AI)

```python

# Option 1: Use conversation endpoint
response = requests.post("http://localhost:8080/conversation/query", json={
    "query": "What is this about?",
    "document_id": "doc-123"
})

# Option 2: Keep using /query for stateless queries (backward compatible)
response = requests.post("http://localhost:8080/query", json={
    "query": "What is this about?"
})
```text

---

## Best Practices

1. **Create conversations explicitly** for better tracking
2. **Include document_id** for document-specific conversations
3. **Reuse conversation_id** for follow-up questions
4. **Clean up old conversations** periodically
5. **Use /query endpoint** for one-off questions without context

---

## Future Enhancements

- Persistent conversation storage (database)
- Conversation search and filtering
- Export conversation history
- Conversation analytics
- Multi-user support with authentication
- Conversation sharing

---

## Support

For issues or questions, please refer to the main README.md or contact the development team.
