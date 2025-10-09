graph TB
    GCS[Google Cloud Storage] --> |Documents| Pipeline{Dual-Flow Pipeline}
    
    Pipeline --> FlowA[Flow A: Summary Processing]
    Pipeline --> FlowB[Flow B: Chunk Processing]
    
    FlowA --> Gemini25[Gemini 2.5 Pro<br/>Text Extraction]
    Gemini25 --> Gemini20[Gemini 2.0 Flash<br/>Summarization]
    Gemini20 --> SpacyA[spaCy NLP<br/>Keyword Extraction]
    SpacyA --> EmbedA[Sentence Transformers<br/>Summary Embeddings]
    EmbedA --> ChromaA[ChromaDB<br/>Summary Collection]
    
    FlowB --> Gemini25B[Gemini 2.5 Pro<br/>Text Extraction]
    Gemini25B --> Chunking[Header-Based<br/>Text Chunking]
    Chunking --> SpacyB[spaCy NLP<br/>Per-Chunk Keywords]
    SpacyB --> EmbedB[Sentence Transformers<br/>Chunk Embeddings]
    EmbedB --> ChromaB[ChromaDB<br/>Document Collection]
