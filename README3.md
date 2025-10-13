graph LR
  subgraph GCS & Notifs
    GCS[Google Cloud Storage Bucket]
    PUBSUB[Pub/Sub Subscription]
  end

  subgraph API
    FASTAPI[FastAPI App / Endpoints<br/>(/pipeline, /monitoring, /search, /documents)]
  end

  subgraph Orchestrator
    RT[RealTimePipelineOrchestrator]
    POLL[Polling Monitor]
    PUB_LISTENER[Pub/Sub Listener]
  end

  subgraph DualFlow
    DFP[DualFlowProcessor]
    subgraph FlowA["Flow A — Summary"]
      OCR25[Gemini 2.5 Pro OCR (extract_text_with_gemini)]
      SUM20[Gemini 2.0 Flash summarization<br/>(generate_summary_with_gemini / generate_summary_gemini_20_flash)]
      NLP_A[spaCy keywords (extract_keywords_with_spacy)]
      EMB_A[SentenceTransformer embed_text/embed_texts]
      CHROMA_SUM[ChromaDB summary_collection (store_summary_to_chroma)]
    end
    subgraph FlowB["Flow B — Chunking"]
      OCR25b[Gemini 2.5 Pro OCR (same extractor)]
      CHUNK[Header-based chunking (header_based_chunking / chunk_text)]
      NLP_B[spaCy per-chunk keywords]
      EMB_B[SentenceTransformer per-chunk embeddings]
      CHROMA_DOC[ChromaDB per_doc_collection (store_chunk_to_chroma)]
    end
  end

  GCS -->|file upload / event| RT
  PUBSUB -->|notification| PUB_LISTENER --> RT
  RT --> POLL
  RT --> DFP
  DFP --> FlowA
  DFP --> FlowB
  FlowA --> CHROMA_SUM
  FlowB --> CHROMA_DOC
  FASTAPI -->|control / queries| RT
  FASTAPI -->|search requests| CHROMA_SUM
  FASTAPI -->|search requests| CHROMA_DOC
  CHROMA_SUM -->|store & query| CHROMA_DB[(ChromaDB Cloud)]
  CHROMA_DOC --> CHROMA_DB
  EMB_A --> CHROMA_DB
  EMB_B --> CHROMA_DB
