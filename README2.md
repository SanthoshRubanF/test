# 🚀 Quick Start Guide - Conversational RAG API

## Overview

Your RAG system has been upgraded to a **Conversational AI** platform! You can now have natural, multi-turn conversations with your documents.

---

## What's New? 🎉

### Before (Response AI)

```python

# Each question was independent
response = query("What is machine learning?")

# ❌ System has no memory of this question

response = query("Tell me more")  

# ❌ "More about what?" - No context!

```text

### After (Conversational AI)

```python

# Start a conversation
conv_id = create_conversation()

# First question
response = query("What is machine learning?", conv_id)

# ✅ System remembers this

# Follow-up question
response = query("Tell me more", conv_id)

# ✅ System knows you mean machine learning!

# Another follow-up
response = query("What are the applications?", conv_id)

# ✅ System maintains full context!

```text

---

## Quick Start in 5 Steps

### Step 1: Start the Server

```bash
uvicorn app:app --reload --port 8080
```text

### Step 2: Upload a Document

```bash
curl -X POST "http://localhost:8080/upload" \

  -F "files=@your_document.pdf" \
  -F "index_name=default"
```text

**Response:**

```json
{
  "document_ids": ["doc-abc-123"],
  "message": "Successfully uploaded 1 files"
}
```text

### Step 3: Start a Conversation

```bash
curl -X POST "http://localhost:8080/conversation/create" \

  -H "Content-Type: application/json" \
  -d '{
    "document_id": "doc-abc-123",
    "index_name": "default"
  }'
```text

**Response:**

```json
{
  "conversation_id": "conv-xyz-789",
  "document_name": "your_document.pdf"
}
```text

### Step 4: Ask Questions (With Context!)

```bash

# First question
curl -X POST "http://localhost:8080/conversation/query" \

  -H "Content-Type: application/json" \
  -d '{
    "query": "What is the main topic?",
    "conversation_id": "conv-xyz-789",
    "document_id": "doc-abc-123"
  }'

# Follow-up question (maintains context)
curl -X POST "http://localhost:8080/conversation/query" \

  -H "Content-Type: application/json" \
  -d '{
    "query": "Can you elaborate on that?",
    "conversation_id": "conv-xyz-789"
  }'

# Another follow-up
curl -X POST "http://localhost:8080/conversation/query" \

  -H "Content-Type: application/json" \
  -d '{
    "query": "What are the key findings?",
    "conversation_id": "conv-xyz-789"
  }'
```text

### Step 5: View Conversation History

```bash
curl "http://localhost:8080/conversation/conv-xyz-789"
```text

**Response:**

```json
{
  "conversation": {
    "conversation_id": "conv-xyz-789",
    "document_name": "your_document.pdf"
  },
  "messages": [
    {"role": "user", "content": "What is the main topic?"},
    {"role": "assistant", "content": "The main topic is..."},
    {"role": "user", "content": "Can you elaborate on that?"},
    {"role": "assistant", "content": "Certainly! ..."}
  ]
}
```text

---

## Python Example

```python
import requests

BASE_URL = "http://localhost:8080"

# 1. Upload document
with open('document.pdf', 'rb') as f:
    response = requests.post(
        f"{BASE_URL}/upload",
        files={'files': f},
        data={'index_name': 'default'}
    )
    doc_id = response.json()['document_ids'][0]
    print(f"Uploaded: {doc_id}")

# 2. Create conversation
response = requests.post(
    f"{BASE_URL}/conversation/create",
    json={
        "document_id": doc_id,
        "index_name": "default"
    }
)
conv_id = response.json()['conversation_id']
print(f"Conversation: {conv_id}")

# 3. Start chatting!
questions = [
    "What is this document about?",
    "Can you summarize the key points?",
    "What are the main findings?",
    "Are there any limitations mentioned?"
]

for question in questions:
    response = requests.post(
        f"{BASE_URL}/conversation/query",
        json={
            "query": question,
            "conversation_id": conv_id,
            "document_id": doc_id
        }
    )
    answer = response.json()['answer']
    print(f"\nQ: {question}")
    print(f"A: {answer[:200]}...")

# 4. View full conversation
response = requests.get(f"{BASE_URL}/conversation/{conv_id}")
history = response.json()
print(f"\nTotal messages: {len(history['messages'])}")
```text

---

## Key Features

### 1. **Auto-Create Conversations**
Don't want to create a conversation first? Just query!

```python

# System auto-creates conversation
response = requests.post(
    f"{BASE_URL}/conversation/query",
    json={
        "query": "Summarize this document",
        "document_id": "doc-123"
    }
)

# Returns conversation_id you can use for follow-ups
conv_id = response.json()['conversation_id']
```text

### 2. **Document-Specific Conversations**
Focus on specific documents:

```python

# Conversation tied to specific document
requests.post(f"{BASE_URL}/conversation/create", json={
    "document_id": "doc-123"  # All queries focus here
})
```text

### 3. **Multi-Document Conversations**
Query across all documents:

```python

# General conversation (no document_id)
requests.post(f"{BASE_URL}/conversation/create", json={
    "index_name": "default"  # Queries all documents
})
```text

### 4. **Conversation Management**

```python

# List all conversations
requests.get(f"{BASE_URL}/conversations")

# Get specific conversation
requests.get(f"{BASE_URL}/conversation/{conv_id}")

# Delete conversation
requests.delete(f"{BASE_URL}/conversation/{conv_id}")
```text

### 5. **Document Management**

```python

# List all documents
requests.get(f"{BASE_URL}/documents?index_name=default")

# Get specific document
requests.get(f"{BASE_URL}/document/{doc_id}")
```text

---

## API Endpoints Cheat Sheet

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/upload` | POST | Upload documents |
| `/conversation/create` | POST | Create conversation |
| `/conversation/query` | POST | **Ask questions (main!)** |
| `/conversation/{id}` | GET | View history |
| `/conversations` | GET | List all |
| `/conversation/{id}` | DELETE | Delete |
| `/documents` | GET | List documents |
| `/document/{id}` | GET | Get document info |
| `/query` | POST | Legacy stateless query |

---

## Common Use Cases

### Use Case 1: Research Paper Q&A

```python

# Upload research paper
upload_pdf("research_paper.pdf")

# Create conversation
conv_id = create_conversation(doc_id)

# Natural conversation
query("What is the research question?", conv_id)
query("What methodology did they use?", conv_id)
query("What were the results?", conv_id)
query("What are the limitations?", conv_id)
```text

### Use Case 2: Document Comparison

```python

# Upload multiple documents
doc1 = upload("paper1.pdf")
doc2 = upload("paper2.pdf")

# General conversation
conv_id = create_conversation()  # No specific document

# Compare across documents
query("Compare the findings from all papers", conv_id)
query("Which paper has stronger evidence?", conv_id)
```text

### Use Case 3: Customer Support Bot

```python

# Upload product manuals
manual_id = upload("product_manual.pdf")

# Create customer conversation
conv_id = create_conversation(manual_id)

# Customer questions
query("How do I set up the device?", conv_id)
query("What if that doesn't work?", conv_id)
query("Where can I get support?", conv_id)
```text

---

## Testing

Run the test script:

```bash
python test_conversational_api.py
```text

This will test all conversational features!

---

## Troubleshooting

### Issue: "Conversation not found"

**Solution:** Make sure you're using a valid conversation_id from create_conversation

### Issue: "No index found"

**Solution:** Upload documents first with /upload endpoint

### Issue: Context not maintained

**Solution:** Use the same conversation_id for all follow-up questions

### Issue: Server not responding

**Solution:** Check server is running: `uvicorn app:app --reload --port 8080`

---

## Best Practices

1. **Create explicit conversations** for better tracking
2. **Include document_id** for document-specific queries
3. **Reuse conversation_id** for follow-ups
4. **View history** to see what the system remembers
5. **Delete old conversations** to keep things clean

---

## What's Different from Before?

| Feature | Before | After |
|---------|--------|-------|
| Memory | ❌ None | ✅ Last 5 turns |
| Follow-ups | ❌ Can't handle | ✅ Natural |
| Document focus | ❌ No | ✅ Yes |
| History | ❌ No | ✅ Full history |
| Sessions | ❌ No | ✅ Yes |

---

## Next Steps

1. **Read** [CONVERSATIONAL_API_GUIDE.md](CONVERSATIONAL_API_GUIDE.md) for detailed API docs
2. **Review** [ARCHITECTURE.md](ARCHITECTURE.md) for system architecture
3. **Check** [REFACTORING_SUMMARY.md](REFACTORING_SUMMARY.md) for what changed
4. **Run** `test_conversational_api.py` to see it in action

---

## Need Help?

- 📖 Full API docs: `CONVERSATIONAL_API_GUIDE.md`
- 🏗️ Architecture: `ARCHITECTURE.md`
- 🔄 What changed: `REFACTORING_SUMMARY.md`
- 🧪 Test suite: `test_conversational_api.py`

---

**Enjoy your new conversational AI system! 🎉**
