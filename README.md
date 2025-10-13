# GCS Document Ingestion Pipeline — Full line-by-line annotated source

This file contains the original source code followed immediately (line-by-line) by an explanation for every line: what it does and why it is present. It is the combined content of the three parts I prepared earlier.

Notes:
- The original source code lines are shown in sequence and each original line is followed by an "Explanation:" line describing the line.
- To convert to PDF: save this file as GCS_Document_Ingestion_Annotated_Full.md and run:
  - pandoc GCS_Document_Ingestion_Annotated_Full.md -o GCS_Document_Ingestion_Annotated_Full.pdf
  - or open in an editor that can export Markdown to PDF.

----- BEGIN ANNOTATED SOURCE (FULL) -----

# 1
import os
Explanation: Imports the `os` module from the Python standard library, used for interacting with the operating system (environment variables, file removal, path operations, etc.).

# 2
import re
Explanation: Imports the `re` module for regular expression matching and string pattern operations.

# 3
import json
Explanation: Imports `json` to encode/decode JSON strings and objects.

# 4
import time
Explanation: Imports `time` for time-related functions like sleep() and timestamps.

# 5
import logging
Explanation: Imports Python's logging library to record informational, warning, and error messages.

# 6
import datetime
Explanation: Imports the `datetime` module for working with date/time objects and ISO formatting.

# 7
import tempfile
Explanation: Imports `tempfile` to create temporary files (e.g., when writing a blob to disk before uploading to an API).

# 8
import threading
Explanation: Imports the `threading` module to run tasks concurrently using threads.

# 9
import hashlib
Explanation: Imports `hashlib` to compute cryptographic hashes (e.g., SHA-256) for file deduplication or identifiers.

# 10
import sys
Explanation: Imports the `sys` module; often used to access stdout/stderr, interpreter info, or to exit.

# 11
from concurrent.futures import ThreadPoolExecutor, as_completed
Explanation: Imports concurrency helpers: ThreadPoolExecutor to execute tasks in thread pool and as_completed to iterate futures as they finish.

# 12
from typing import List, Dict, Any, Tuple, Optional, Set
Explanation: Imports type hints for function signatures and dataclass fields.

# 13
from collections import Counter, deque
Explanation: Imports Counter for frequency counts and deque (double-ended queue) which is used for efficient append/pop from both ends (used later for quota tracking).

# 14
from dataclasses import dataclass, field
Explanation: Imports dataclass decorator and field factory to declare lightweight classes for configuration.

# 15
from pathlib import Path
Explanation: Imports Path from pathlib for path manipulation (platform independent).

# 16
import numpy as np
Explanation: Imports NumPy as np for numerical arrays and vector operations (used for embeddings).

# 17
from contextlib import asynccontextmanager
Explanation: Imports asynccontextmanager decorator to define FastAPI lifespan event handlers (async startup/shutdown context).

# 18
from fastapi import FastAPI, HTTPException, BackgroundTasks
Explanation: Imports FastAPI, HTTPException for raising HTTP errors, and BackgroundTasks to schedule background tasks.

# 19
from fastapi.responses import HTMLResponse
Explanation: Import HTMLResponse to return HTML on GET / root endpoint.

# 20
from fastapi.middleware.cors import CORSMiddleware
Explanation: Import CORS middleware to allow cross-origin requests for the API.

# 21
from pydantic import BaseModel, Field
Explanation: Imports Pydantic BaseModel and Field to define request/response schemas used by FastAPI endpoints.

# 22
from dotenv import load_dotenv
Explanation: Imports load_dotenv to load environment variables from a .env file into os.environ.

# 23
import uvicorn
Explanation: Imports uvicorn ASGI server used to run the FastAPI application when executed directly.

# 24
# Google Cloud & Gemini
Explanation: Comment marking the start of imports and checks for Google Generative AI (Gemini) and Google Cloud libraries.

# 25
try:
Explanation: Start try/except to attempt imports that may not exist in all environments (safe optional dependency imports).

# 26
    import google.generativeai as genai  # type: ignore
Explanation: Tries to import the Google Generative AI client library (aliased as genai). `# type: ignore` suppresses type-checker errors if types are unavailable.

# 27
    from google.generativeai import GenerativeModel  # type: ignore
Explanation: Imports GenerativeModel class that encapsulates an AI model for generating content.

# 28
    from google.generativeai.types import File as GeminiFile  # type: ignore
Explanation: Imports a File type from the gemini types if present, aliasing to GeminiFile for clarity.

# 29
    gemini_available = True
Explanation: Sets a flag indicating the gemini library imported successfully.

# 30
    
    # Try to import specific functions, but don't fail if they're not available
Explanation: Comment indicating the code will attempt to import helper functions that might not be present in older/newer gemini versions.

# 31
    try:
Explanation: Nested try for importing specific functions with fallback.

# 32
        from google.generativeai import configure, upload_file, get_file, delete_file  # type: ignore
Explanation: Attempt to import convenience helper functions from the gemini library (configure, upload_file, get_file, delete_file).

# 33
    except ImportError:
Explanation: If the specific functions are not present, fall back.

# 34
        configure = getattr(genai, 'configure', None)
Explanation: Fallback: get configure from the genai module by attribute lookup if present; otherwise None.

# 35
        upload_file = getattr(genai, 'upload_file', None)
Explanation: Fallback for upload_file attribute.

# 36
        get_file = getattr(genai, 'get_file', None)
Explanation: Fallback for get_file attribute.

# 37
        delete_file = getattr(genai, 'delete_file', None)
Explanation: Fallback for delete_file attribute.

# 38
        
except ImportError:
Explanation: If importing google.generativeai failed, handle it here.

# 39
    genai = None
Explanation: Set genai to None to indicate library is unavailable.

# 40
    GenerativeModel = None
Explanation: Nullify GenerativeModel to avoid NameError elsewhere.

# 41
    GeminiFile = None
Explanation: Nullify GeminiFile.

# 42
    configure = None
Explanation: Nullify configure function reference.

# 43
    upload_file = None
Explanation: Nullify upload_file function reference.

# 44
    get_file = None
Explanation: Nullify get_file function reference.

# 45
    delete_file = None
Explanation: Nullify delete_file function reference.

# 46
    gemini_available = False
Explanation: Mark gemini as not available.

# 47
try:
Explanation: Try to import Google Cloud Storage library.

# 48
    from google.cloud import storage  # type: ignore
Explanation: Import the google cloud storage module for interacting with GCS.

# 49
    from google.api_core import exceptions as gcs_exceptions  # type: ignore
Explanation: Import GCS-specific exception types for error handling.

# 50
    from google.cloud.storage import Blob  # type: ignore
Explanation: Import Blob class (type) used for typed hints on blob objects.

# 51
    gcs_available = True
Explanation: Mark that Google Cloud Storage is available.

# 52
    StorageClient = storage.Client
Explanation: Save the Storage client constructor to StorageClient for later initialization.

# 53
    BlobType = Blob
Explanation: Save Blob type to BlobType for typing.

# 54
except ImportError:
Explanation: If google-cloud-storage is not installed, handle gracefully.

# 55
    storage = None
Explanation: Nullify storage module.

# 56
    gcs_exceptions = None  
Explanation: Nullify exceptions reference.

# 57
    Blob = None
Explanation: Nullify Blob.

# 58
    gcs_available = False
Explanation: Mark GCS as unavailable.

# 59
    StorageClient = None  # type: ignore
Explanation: Nullify StorageClient.

# 60
    BlobType = None  # type: ignore
Explanation: Nullify BlobType.

# 61
# Google Cloud Pub/Sub for Real-time Notifications
Explanation: Comment about Pub/Sub optional import that follows.

# 62
try:
Explanation: Attempt to import Pub/Sub client.

# 63
    from google.cloud import pubsub_v1  # type: ignore
Explanation: Import the pubsub_v1 client used for subscribing to Pub/Sub topics.

# 64
    pubsub_available = True
Explanation: Indicate Pub/Sub available.

# 65
except ImportError:
Explanation: Handle missing pubsub dependency.

# 66
    pubsub_v1 = None
Explanation: Nullify pubsub_v1.

# 67
    pubsub_available = False
Explanation: Mark Pub/Sub as unavailable.

# 68
# NLP / Embeddings
Explanation: Comment to mark NLP and embedding library imports.

# 69
try:
Explanation: Try block to import spaCy and sentence-transformers.

# 70
    import spacy  # type: ignore
Explanation: Import spaCy, an NLP library for tokenization, parsing, named entity recognition.

# 71
    from spacy.lang.en import English  # type: ignore
Explanation: Import English language class (not used heavily later but common to initialize a tokenizer).

# 72
    from sentence_transformers import SentenceTransformer  # type: ignore
Explanation: Import SentenceTransformer model class used to create embedding models.

# 73
    nlp_available = True
Explanation: Mark NLP components available.

# 74
except ImportError:
Explanation: If those third-party libraries are missing.

# 75
    spacy = None
Explanation: Nullify spacy reference.

# 76
    SentenceTransformer = None
Explanation: Nullify SentenceTransformer.

# 77
    nlp_available = False
Explanation: Mark NLP as unavailable.

# 78
# ChromaDB
Explanation: Comment marking ChromaDB optional import.

# 79
try:
Explanation: Attempt to import chromadb library.

# 80
    import chromadb  # type: ignore
Explanation: Import chromadb, a vector database library.

# 81
    from chromadb.config import Settings  # type: ignore
Explanation: Import Settings (client config) from chromadb.

# 82
    chromadb_available = True
Explanation: Indicate chromadb is available.

# 83
except ImportError:
Explanation: Handle missing chromadb dependency.

# 84
    chromadb = None
Explanation: Nullify chromadb.

# 85
    Settings = None  # type: ignore
Explanation: Nullify Settings.

# 86
    chromadb_available = False
Explanation: Mark chromadb as unavailable.

# 87
# Additional imports
Explanation: Comment — uvicorn import already present earlier; redundant import appears again in the original file.

# 88
import uvicorn  # type: ignore
Explanation: Re-import uvicorn (duplicate). This is harmless but redundant; likely leftover from merging code — the same module was imported on line 23.

# 89
# Load environment variables
Explanation: Comment indicating the .env loader call next.

# 90
load_dotenv()
Explanation: Loads environment variables from a .env file into os.environ if present.

# 91
# ============ CONFIGURATION ============
Explanation: Section header comment.

# 92
@dataclass
Explanation: Decorator to declare the next class as a dataclass (automatically generates __init__, repr, etc.)

# 93
class Config:
Explanation: Defines a configuration dataclass called Config which will hold runtime settings.

# 94
    """Global configuration settings"""
Explanation: Docstring for the Config dataclass describing purpose.

# 95
    # Google Cloud Settings
Explanation: Inline comment grouping GCS settings.

# 96
    project_id: str = field(default_factory=lambda: os.getenv("GOOGLE_CLOUD_PROJECT", ""))
Explanation: Dataclass field for project_id; default is read from environment variable GOOGLE_CLOUD_PROJECT or empty string.

# 97
    gcs_bucket_name: str = field(default_factory=lambda: os.getenv("GCS_BUCKET_NAME", ""))
Explanation: Field for bucket name, default from env GCS_BUCKET_NAME.

# 98
    gcs_bucket_prefix: str = field(default_factory=lambda: os.getenv("GCS_BUCKET_PREFIX", ""))
Explanation: Field for prefix/filter inside bucket, default from env GCS_BUCKET_PREFIX.

# 99
    
    # Gemini Settings
Explanation: Comment grouping Gemini-specific settings.

# 100
    gemini_api_key: str = field(default_factory=lambda: os.getenv("GEMINI_API_KEY", ""))
Explanation: Field to hold Gemini API key from environment.

# 101
    extraction_model: str = "gemini-2.0-flash-exp"
Explanation: Default model name for extraction (a Gemini model identifier).

# 102
    summarization_model: str = "gemini-2.0-flash-exp"
Explanation: Default model name for summarization.

# 103
    
    # ChromaDB Cloud Settings
Explanation: Comment grouping ChromaDB cloud connection settings.

# 104
    chroma_host: str = field(default_factory=lambda: os.getenv("CHROMA_HOST", ""))
Explanation: Host for ChromaDB service from env CHROMA_HOST.

# 105
    chroma_port: int = field(default_factory=lambda: int(os.getenv("CHROMA_PORT", "443")))
Explanation: Port for ChromaDB service with default 443 (converted to int).

# 106
    chroma_api_key: str = field(default_factory=lambda: os.getenv("CHROMA_API_KEY", ""))
Explanation: API key for ChromaDB cloud.

# 107
    chroma_tenant: str = field(default_factory=lambda: os.getenv("CHROMA_TENANT", "default_tenant"))
Explanation: Tenant identifier for multi-tenant ChromaDB instances.

# 108
    chroma_database: str = field(default_factory=lambda: os.getenv("CHROMA_DATABASE", "vector_db"))
Explanation: Database name to use within ChromaDB.

# 109
    chroma_use_ssl: bool = field(default_factory=lambda: os.getenv("CHROMA_USE_SSL", "true").lower() == "true")
Explanation: Boolean flag to choose https vs http based on environment variable (defaults to true).

# 110
    
    # Processing Settings
Explanation: Grouping processing settings.

# 111
    max_workers: int = 1  # Reduced to prevent quota exhaustion
Explanation: Default thread pool worker count (1). Comment notes reduced concurrency to avoid API quota exhaustion.

# 112
    batch_size: int = 5   # Smaller batches to manage API calls
Explanation: Batch size for processing; small by default to manage API call counts.

# 113
    chunk_size: int = 1000
Explanation: Default character length for text chunks.

# 114
    chunk_overlap: int = 100
Explanation: Overlap size between adjacent chunks to preserve context.

# 115
    
    # Monitoring Settings
Explanation: Grouping monitoring-related settings.

# 116
    monitoring_interval: int = field(default_factory=lambda: int(os.getenv("MONITORING_INTERVAL", "1")))  # seconds - Real-time monitoring every second
Explanation: Interval between polling checks (default 1 second or from env MONITORING_INTERVAL).

# 117
    max_retries: int = 3
Explanation: Max retry attempts for API calls / transient errors.

# 118
    retry_delay: int = 5
Explanation: Base retry delay (seconds) used for non-quota retries.

# 119
    
    # Gemini API Quota Management
Explanation: Grouping Gemini quota management configuration.

# 120
    api_requests_per_minute: int = 10  # Conservative limit (under 15/min)
Explanation: Configured requests/min quota limit for Gemini to throttle calls.

# 121
    api_delay_between_calls: float = 6.5  # Seconds between API calls (60/10 + buffer)
Explanation: Minimum delay between consecutive API calls to spread requests across time.

# 122
    quota_retry_delay: int = 60  # Wait 1 minute on quota exceeded
Explanation: Base backoff delay when quota errors occur.

# 123
    
    # Real-time Pub/Sub Settings
Explanation: Grouping Pub/Sub related configuration.

# 124
    enable_pubsub: bool = field(default_factory=lambda: os.getenv("ENABLE_PUBSUB", "false").lower() == "true")
Explanation: Toggle to enable Pub/Sub integration from environment variable.

# 125
    pubsub_topic: str = field(default_factory=lambda: os.getenv("PUBSUB_TOPIC", "projects/magi-mvp-dev/topics/document-processing-topic"))
Explanation: Default pubsub topic string (project-specific) if not provided through env.

# 126
    pubsub_subscription: str = field(default_factory=lambda: os.getenv("PUBSUB_SUBSCRIPTION", "projects/magi-mvp-dev/subscriptions/document-processing-topic-sub"))
Explanation: Default subscription string for Pub/Sub if not provided.

# 127
    
    # Folder Filtering Settings
Explanation: Grouping folder filter settings.

# 128
    include_folders: Optional[List[str]] = None  # Process only these folders (None = all)
Explanation: Optional list to limit processing to specific folders within the bucket.

# 129
    exclude_folders: List[str] = field(default_factory=lambda: [])  # Exclude these folders
Explanation: List of folders to exclude.

# 130
    root_only: bool = field(default_factory=lambda: os.getenv("PROCESS_ROOT_ONLY", "false").lower() == "true")
Explanation: If true, only root-level files are processed (no subfolders).

# 131
    folder_depth_limit: int = field(default_factory=lambda: int(os.getenv("FOLDER_DEPTH_LIMIT", "10")))  # Max folder depth
Explanation: Max allowed folder nesting depth to be processed (env-configurable).

# 132
    
    # File Processing Settings
Explanation: Grouping supported file types and size limits.

# 133
    supported_extensions: Set[str] = field(default_factory=lambda: {
Explanation: Field providing the set of supported file extensions.

# 134
        '.pdf', '.docx', '.doc', '.txt', '.rtf', 
Explanation: Supported document file extensions (part of set).

# 135
        '.jpg', '.jpeg', '.png', '.bmp', '.tiff', '.gif',
Explanation: Supported image types; images can be OCR'd by Gemini.

# 136
        '.html', '.htm', '.md', '.csv'
Explanation: Supported web/markup and CSV formats.

# 137
    })
Explanation: End of set literal for supported extensions.

# 138
    max_file_size_mb: int = 100
Explanation: Maximum allowed file size in megabytes for processing.

# 139
    
    # Embedding Settings
Explanation: Grouping embedding model configuration.

# 140
    embedding_model: str = "sentence-transformers/all-MiniLM-L6-v2"
Explanation: Default SentenceTransformers model name for generating embeddings.

# 141
    embedding_dimension: int = 384
Explanation: Embedding vector dimension expected for the chosen model (384 for all-MiniLM-L6-v2).

# 142
CONFIG = Config()
Explanation: Instantiate the global CONFIG dataclass using defaults and environment variables.

# 143
# Global Constants
Explanation: Comment for constants derived from CONFIG.

# 144
GCS_BUCKET_NAME = CONFIG.gcs_bucket_name
Explanation: Shortcut constant for configured bucket name.

# 145
GEMINI_API_KEY = CONFIG.gemini_api_key
Explanation: Shortcut constant for Gemini API key.

# 146
SUPPORTED_EXTENSIONS = CONFIG.supported_extensions
Explanation: Alias to the set of supported extensions from the config.

# 147
# ============ LOGGING SETUP ============
Explanation: Section header for configuring logging behavior.

# 148
logging.basicConfig(
Explanation: Start of basic logging configuration call.

# 149
    level=logging.INFO,
Explanation: Set logging level to INFO (messages of level INFO and above will be emitted).

# 150
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
Explanation: Set the log message format to include timestamp, logger name, level, and message.

# 151
    handlers=[
Explanation: Begin list of handlers to send logs to multiple outputs.

# 152
        logging.FileHandler('pipeline.log'),
Explanation: Add a file handler to write logs to "pipeline.log".

# 153
        logging.StreamHandler(sys.stdout)
Explanation: Add stream handler to print logs to standard output (useful in containers).

# 154
    ]
Explanation: End of handlers list.

# 155
)
Explanation: End of logging.basicConfig call.

# 156
logger = logging.getLogger(__name__)
Explanation: Create or retrieve a module-level logger using this module's __name__.

# 157
# ============ INITIALIZATION & VALIDATION ============
Explanation: Section header.

# 158
def validate_environment():
Explanation: Define function to validate required environment variables and library availability.

# 159
    """Validate required environment variables and dependencies"""
Explanation: Docstring describing function purpose.

# 160
    missing_vars: List[str] = []
Explanation: Initialize list to collect missing environment variable names.

# 161
    missing_deps: List[str] = []
Explanation: Initialize list to collect missing dependency names.

# 162
    
    # Check environment variables
Explanation: Comment block for env var checks.

# 163
    if not CONFIG.gemini_api_key:
Explanation: Check if Gemini API key is set in CONFIG.

# 164
        missing_vars.append("GEMINI_API_KEY")
Explanation: If not, record it in missing_vars.

# 165
    
    # ChromaDB Cloud validation
Explanation: Comment for chroma-specific checks.

# 166
    if not CONFIG.chroma_host:
Explanation: Check chroma host presence.

# 167
        missing_vars.append("CHROMA_HOST (Required for ChromaDB Cloud)")
Explanation: Append descriptive message for missing CHROMA_HOST.

# 168
    if not CONFIG.chroma_api_key:
Explanation: Check chroma API key.

# 169
        missing_vars.append("CHROMA_API_KEY (Required for ChromaDB Cloud)")
Explanation: Append descriptive message for missing CHROMA_API_KEY.

# 170
    
    # Check dependencies
Explanation: Comment for dependency checks.

# 171
    if not gcs_available:
Explanation: If google-cloud-storage was not imported.

# 172
        missing_deps.append("google-cloud-storage")
Explanation: Add to missing dependencies list.

# 173
    if not gemini_available:
Explanation: If gemini library not available.

# 174
        missing_deps.append("google-generativeai")
Explanation: Add package name to missing_deps.

# 175
    if not nlp_available:
Explanation: If NLP libs not available.

# 176
        missing_deps.append("spacy, sentence-transformers")
Explanation: List missing NLP packages.

# 177
    if not chromadb_available:
Explanation: If chromadb is not available.

# 178
        missing_deps.append("chromadb")
Explanation: Add chromadb to missing dependencies.

# 179
    
    if missing_vars:
Explanation: If any environment variables are missing.

# 180
        logger.error(f"Missing environment variables: {missing_vars}")
Explanation: Log an error listing missing environment variables.

# 181
        
    if missing_deps:
Explanation: If any dependencies missing.

# 182
        logger.error(f"Missing dependencies: {missing_deps}")
Explanation: Log missing dependency packages.

# 183
        
    return len(missing_vars) == 0 and len(missing_deps) == 0
Explanation: Return True only when no missing environment variables and no missing dependencies.

# 184
# Initialize Gemini
Explanation: Section comment — attempt to configure Gemini if possible.

# 185
if CONFIG.gemini_api_key and gemini_available and configure is not None:
Explanation: If key present, gemini lib available, and configure function exists, try to configure.

# 186
    try:
Explanation: Try block for calling configure.

# 187
        configure(api_key=CONFIG.gemini_api_key)
Explanation: Call configure helper (if available) to set API key for gemini.

# 188
    except (AttributeError, TypeError):
Explanation: Catch attribute or type errors (compatibility issues across versions).

# 189
        # Some versions of the library may not have configure or different signature
Explanation: Comment explaining fallback.

# 190
        try:
Explanation: Nested try fallback using genai.configure.

# 191
            genai.configure(api_key=CONFIG.gemini_api_key)  # type: ignore
Explanation: Try calling configure via the module reference if direct configure import didn't exist.

# 192
        except:
Explanation: If fallback fails, silently pass (best-effort config).

# 193
            pass
Explanation: No-op in exception handler.

# 194
    logger.info("Gemini API configured successfully")
Explanation: Log success if configuration calls didn't throw exceptions.

# 195
elif not gemini_available:
Explanation: Else branch when gemini library is not installed.

# 196
    logger.warning("Gemini AI library not available - install google-generativeai")
Explanation: Warn the operator about missing gemini dependency.

# 197
else:
Explanation: Else case where gemini library present but API key missing or configure None.

# 198
    logger.warning("Gemini API key not found - text processing will be limited")
Explanation: Warn that the key is missing and functionality will be limited.

# 199
# Lazy initialization placeholders (loaded on first use)
Explanation: Comment noting that embedding and nlp models will be loaded lazily.

# 200
embedding_model = None
Explanation: Global placeholder for the embedding model instance.

# 201
nlp = None
Explanation: Global placeholder for spaCy NLP model instance.

# 202
def get_embedding_model():
Explanation: Define a function to lazy-load the embedding model.

# 203
    """Lazy load embedding model"""
Explanation: Docstring summarizing function purpose.

# 204
    global embedding_model
Explanation: Declare that we'll modify the global embedding_model variable.

# 205
    if embedding_model is None and nlp_available and SentenceTransformer is not None:
Explanation: Check whether the model hasn't been loaded and sentence-transformers is available.

# 206
        try:
Explanation: Try to create the model instance.

# 207
            embedding_model = SentenceTransformer(CONFIG.embedding_model)
Explanation: Instantiate the SentenceTransformer with the configured model name.

# 208
            logger.info(f"Embedding model loaded: {CONFIG.embedding_model}")
Explanation: Log the successful model load.

# 209
        except Exception as e:
Explanation: Catch any exception during model loading.

# 210
            logger.error(f"Failed to load embedding model: {e}")
Explanation: Log error encountered while loading model.

# 211    return embedding_model
Explanation: Return the loaded embedding model (or None). NOTE: This originally is on one line "return embedding_model"; kept as intended.

# 212
def get_nlp_model():
Explanation: Define function to lazy-load spaCy NLP model.

# 213
    """Lazy load spaCy model"""
Explanation: Docstring.

# 214
    global nlp
Explanation: Declare global nlp variable for assignment.

# 215
    if nlp is None and nlp_available and spacy is not None:
Explanation: Check conditions before attempting to load spaCy model.

# 216
        try:
Explanation: Try block for loading the spaCy model.

# 217
            nlp = spacy.load("en_core_web_sm")
Explanation: Load the small English model for spaCy.

# 218
            logger.info("spaCy model loaded successfully")
Explanation: Log success.

# 219
        except OSError:
Explanation: Catch OSError which often indicates the model package isn't installed.

# 220
            logger.warning("spaCy model not found. Run: python -m spacy download en_core_web_sm")
Explanation: Warn instructing how to download spaCy's language model.

# 221    return nlp
Explanation: Return the spaCy `nlp` model (or None).

# 222
# Lazy initialization for ChromaDB
Explanation: Comment indicating next section is for Chroma client lazy init.

# 223
chroma_client = None
Explanation: Global placeholder for chroma client.

# 224
def get_chroma_client():
Explanation: Define function to initialize and return a ChromaDB client (lazy).

# 225
    """Initialize ChromaDB Cloud client with vector_db database"""
Explanation: Docstring.

# 226
    global chroma_client
Explanation: Indicate we'll modify the global chroma_client variable.

# 227
    if chroma_client is None and chromadb_available and chromadb is not None:
Explanation: Only attempt to create the client if not already created and chromadb library is present.

# 228
        try:
Explanation: Try block for client creation.

# 229
            if not CONFIG.chroma_api_key or not CONFIG.chroma_host:
Explanation: Ensure necessary env vars for ChromaDB cloud connection are present.

# 230
                logger.error("ChromaDB Cloud requires CHROMA_API_KEY and CHROMA_HOST environment variables")
Explanation: Log error if required values missing.

# 231
                return None
Explanation: Early return if config is incomplete.

# 232
            
            # ChromaDB Cloud setup with SSL
Explanation: Comment about next steps (setup client with SSL when required).

# 233
            protocol = "https" if CONFIG.chroma_use_ssl else "http"
Explanation: Determine protocol string based on config.

# 234
            logger.info(f"Connecting to ChromaDB Cloud: {protocol}://{CONFIG.chroma_host}:{CONFIG.chroma_port}")
Explanation: Log connection attempt location.

# 235
            
            chroma_client = chromadb.HttpClient(
Explanation: Create an HttpClient instance with host, port, ssl, headers, tenant, database and settings.

# 236
                host=CONFIG.chroma_host,
Explanation: Pass host to HttpClient.

# 237
                port=CONFIG.chroma_port,
Explanation: Pass port.

# 238
                ssl=CONFIG.chroma_use_ssl,
Explanation: Enable SSL if configured.

# 239
                headers={"X-Chroma-Token": CONFIG.chroma_api_key},
Explanation: Provide API key header for authentication.

# 240
                tenant=CONFIG.chroma_tenant,
Explanation: Provide tenant value.

# 241
                database=CONFIG.chroma_database,
Explanation: Provide database name.

# 242
                settings=Settings(
Explanation: Start constructing Settings object for client configuration (only if Settings is not None).

# 243
                    chroma_api_impl="chromadb.api.fastapi.FastAPI",
Explanation: Set implementation type for chroma API.

# 244
                    chroma_server_host=CONFIG.chroma_host,
Explanation: Set server host in settings.

# 245
                    chroma_server_http_port=CONFIG.chroma_port,
Explanation: Set server http port in settings.

# 246
                    anonymized_telemetry=False
Explanation: Disable telemetry in the settings.

# 247
                ) if Settings else None
Explanation: Only pass Settings if the Settings symbol is available; otherwise pass None.

# 248
            )
Explanation: End of chromadb.HttpClient constructor call.

# 249
            
            # Test connection
Explanation: Comment indicating a connection test follows.

# 250
            heartbeat = chroma_client.heartbeat()
Explanation: Call heartbeat to test server availability and return a value indicating status.

# 251
            logger.info(f"ChromaDB Cloud connected successfully - Heartbeat: {heartbeat}")
Explanation: Log heartbeat result showing connection success.

# 252
            logger.info(f"Using tenant: {CONFIG.chroma_tenant}")
Explanation: Log selected tenant.

# 253
            logger.info(f"Using database: {CONFIG.chroma_database}")
Explanation: Log selected database.

# 254
            
            logger.info(f"Using ChromaDB Cloud database: {CONFIG.chroma_database}")
Explanation: Duplicate log emphasizing which database is used.

# 255
            
        except Exception as e:
Explanation: Catch exceptions during client creation.

# 256
            logger.error(f"Failed to connect to ChromaDB Cloud: {e}")
Explanation: Log error message with exception info.

# 257
            chroma_client = None
Explanation: Reset chroma_client to None on failure.

# 258    return chroma_client
Explanation: Return the chroma_client object (or None).

# 259
# ============ UTILITY FUNCTIONS ============
Explanation: Section header for small utility functions used throughout the pipeline.

# 260
def get_file_hash(content: bytes) -> str:
Explanation: Define function to compute SHA-256 hash of file bytes.

# 261
    """Generate SHA-256 hash of file content"""
Explanation: Docstring.

# 262
    return hashlib.sha256(content).hexdigest()
Explanation: Compute and return hex digest of SHA-256 for deduplication/ID purposes.

# 263
def is_supported_file(filename: str) -> bool:
Explanation: Define function to determine if a filename has a supported extension.

# 264
    """Check if file extension is supported"""
Explanation: Docstring.

# 265
    return Path(filename).suffix.lower() in SUPPORTED_EXTENSIONS
Explanation: Return True if file suffix (lowercased) is in SUPPORTED_EXTENSIONS set.

# 266
def get_file_size_mb(blob: Any) -> float:
Explanation: Define helper to compute file size in megabytes from a blob object.

# 267
    """Get file size in MB"""
Explanation: Docstring.

# 268
    return blob.size / (1024 * 1024) if blob.size else 0
Explanation: Convert blob.size (bytes) to MB; if missing/0 return 0.

# 269
def truncate_text(text: str, max_length: int = 1000) -> str:
Explanation: Define helper to truncate text to a max length with ellipsis.

# 270
    """Truncate text to specified length"""
Explanation: Docstring.

# 271
    return text[:max_length] + "..." if len(text) > max_length else text
Explanation: Truncate and append ellipsis only if text exceeded max_length.

# 272
def clean_text(text: str) -> str:
Explanation: Define text normalizer that strips excessive whitespace and removes uncommon special characters.

# 273
    """Clean and normalize text"""
Explanation: Docstring.

# 274
    if not text:
Explanation: If input is falsy (None or empty), return empty string.

# 275
        return ""
Explanation: Return empty string for empty input.

# 276
    
    # Remove excessive whitespace
Explanation: Comment describing the first cleaning step.

# 277
    text = re.sub(r'\s+', ' ', text)
Explanation: Replace any run of whitespace (tabs, newlines, multiple spaces) with a single space.

# 278
    
    # Remove special characters but keep basic punctuation
Explanation: Comment describing second cleaning step.

# 279
    text = re.sub(r'[^\w\s\.\,\!\?\;\:\-\(\)]', '', text)
Explanation: Remove characters that are not word characters, whitespace, or basic punctuation (period, comma, etc.).

# 280
    
    return text.strip()
Explanation: Strip leading/trailing whitespace and return cleaned text.

# 281
def chunk_text(text: str, chunk_size: Optional[int] = None, overlap: Optional[int] = None) -> List[str]:
Explanation: Define generic chunker that splits text into overlapping chunks using chunk_size and overlap defaults from CONFIG.

# 282
    """Split text into overlapping chunks"""
Explanation: Docstring.

# 283
    if not text:
Explanation: If empty text, return empty list.

# 284
        return []
Explanation: Return empty list for empty input.

# 285
    
    chunk_size = chunk_size or CONFIG.chunk_size
Explanation: Use provided chunk_size or fallback to CONFIG.chunk_size.

# 286
    overlap = overlap or CONFIG.chunk_overlap
Explanation: Use provided overlap or fallback to CONFIG.chunk_overlap.

# 287
    
    if len(text) <= chunk_size:
Explanation: If text is smaller than a single chunk, return it as single chunk.

# 288
        return [text]
Explanation: Return a single-element list containing the whole text.

# 289
    
    chunks: List[str] = []
Explanation: Initialize list that will hold chunks.

# 290
    start = 0
Explanation: Initialize start index for chunking.

# 291
    
    while start < len(text):
Explanation: Loop until the entire text is chunked.

# 292
        end = start + chunk_size
Explanation: Compute initial end index for the current chunk.

# 293
        
        # Try to break at word boundary
Explanation: Comment about trying to avoid splitting in the middle of words.

# 294
        if end < len(text):
Explanation: If the end index is not at the end of text, attempt to find a nearby word boundary.

# 295
            while end > start and text[end] not in ' \n\t':
Explanation: Walk backward until a whitespace is found, to avoid cutting in the middle of a word.

# 296
                end -= 1
Explanation: Decrement end until a whitespace boundary is found or we reach start.

# 297
            if end == start:  # No word boundary found
Explanation: If no whitespace found in the chunk range (e.g., extremely long word), fallback to fixed-size slice.

# 298
                end = start + chunk_size
Explanation: Force end to chunk_size ahead to avoid infinite loop.

# 299
        
        chunk = text[start:end].strip()
Explanation: Slice the text into chunk and strip surrounding whitespace.

# 300
        if chunk:
Explanation: If the chunk is non-empty, append it.

# 301
            chunks.append(chunk)
Explanation: Append current chunk to list of chunks.

# 302
        
        start = end - overlap
Explanation: Advance start to end minus overlap to create overlap between chunks.

# 303
        if start <= 0:
Explanation: If start became non-positive due to overlap adjustments, fix to avoid infinite loop.

# 304
            start = end
Explanation: Set start to end if overlap caused non-positive index.

# 305
    
    return chunks
Explanation: Return the final list of chunks.

# ============ EMBEDDING FUNCTIONS ============
Explanation: Section header introducing functions used to generate and handle embeddings.

def get_embedding_dimension() -> int:
Explanation: Define a function that returns the configured embedding vector dimension.

    """Get embedding dimension"""
Explanation: Docstring describing the function.

    return CONFIG.embedding_dimension
Explanation: Return the embedding dimension from global CONFIG (e.g., 384).

def embed_texts(texts: List[str]) -> List[np.ndarray]:
Explanation: Define function to compute embeddings for a list of texts, returning numpy arrays.

    """Generate embeddings for list of texts"""
Explanation: Docstring.

    model = get_embedding_model()
Explanation: Obtain (or lazily load) the embedding model instance.

    if not model or not texts:
Explanation: If model failed to load or texts list is empty, handle gracefully.

        return [np.zeros(get_embedding_dimension()) for _ in texts]
Explanation: Return zero vectors of configured dimension for each input text as a fallback.

    try:
Explanation: Try block around the model inference to catch runtime errors.

        embeddings = model.encode(texts, convert_to_numpy=True)  # type: ignore
Explanation: Call SentenceTransformer.encode to get embeddings as numpy arrays.

        return [emb for emb in embeddings]
Explanation: Convert the embeddings result into a list of numpy arrays and return it.

    except Exception as e:
Explanation: Catch any exception from embedding generation.

        logger.error(f"Embedding generation failed: {e}")
Explanation: Log an error describing what went wrong.

        return [np.zeros(get_embedding_dimension()) for _ in texts]
Explanation: Return fallback zero vectors to keep pipeline moving.

def embed_text(text: str) -> Optional[np.ndarray]:
Explanation: Define wrapper to embed a single string using embed_texts.

    """Generate embedding for single text"""
Explanation: Docstring.

    embeddings = embed_texts([text])
Explanation: Call embed_texts with a single-element list.

    return embeddings[0] if embeddings else None
Explanation: Return the first embedding if present, otherwise None.

# ============ CHROMADB FUNCTIONS ============
Explanation: Section header for functions interacting with ChromaDB.

def get_or_create_collection(collection_name: str):
Explanation: Define function to get or create a ChromaDB collection by name.

    """Get or create a ChromaDB collection"""
Explanation: Docstring.

    client = get_chroma_client()
Explanation: Get the global chroma client (lazy-init).

    if not client:
Explanation: If client initialization failed, handle gracefully.

        logger.error("ChromaDB client not initialized")
Explanation: Log an error.

        return None
Explanation: Return None to indicate failure.

    try:
Explanation: Try block for collection retrieval/creation.

        collection = client.get_or_create_collection(
Explanation: Call client's get_or_create_collection method.

            name=collection_name,
Explanation: Pass requested collection name.

            metadata={"hnsw:space": "cosine"}
Explanation: Set metadata (here specifying similarity space for HNSW indexing as cosine).

        )
Explanation: Close get_or_create_collection call.

        logger.info(f"Collection '{collection_name}' ready")
Explanation: Log that collection is ready to use.

        return collection
Explanation: Return the collection object.

    except Exception as e:
Explanation: Catch exceptions during collection creation.

        logger.error(f"Failed to get/create collection '{collection_name}': {e}")
Explanation: Log the failure with the exception message.

        return None
Explanation: Return None to indicate failure.

def store_summary_to_chroma(collection: Any, doc_id: str, summary: str, keywords: List[str], 
                          embedding: Optional[np.ndarray], metadata: Dict[str, Any]) -> bool:
Explanation: Define function to add a document summary to a ChromaDB collection.

    """Store document summary in ChromaDB"""
Explanation: Docstring.

    if not collection or embedding is None:
Explanation: Validate required inputs, returning False if missing.

        return False
Explanation: Early return on invalid inputs.

    try:
Explanation: Try block for adding the document.

        collection.add(
Explanation: Call collection.add to insert document, metadata, id, and embedding.

            documents=[summary],
Explanation: Provide the summary text as the document content.

            metadatas=[metadata],
Explanation: Provide metadata dict.

            ids=[doc_id],
Explanation: Provide unique id for the document (e.g., file hash).

            embeddings=[embedding.tolist()]
Explanation: Convert numpy array to list and pass as embedding.

        )
Explanation: End collection.add call.

        logger.debug(f"Summary stored for document: {doc_id}")
Explanation: Debug log indicating successful store.

        return True
Explanation: Return True on success.

    except Exception as e:
Explanation: Handle any exception encountered writing to ChromaDB.

        logger.error(f"Failed to store summary for {doc_id}: {e}")
Explanation: Log error.

        return False
Explanation: Return False on failure.

def store_chunk_to_chroma(collection: Any, chunk_id: str, content: str, keywords: List[str], 
                         embedding: Optional[np.ndarray], metadata: Dict[str, Any]) -> bool:
Explanation: Define function to store per-chunk data in ChromaDB for Flow B.

    """Store document chunk in ChromaDB for Flow B"""
Explanation: Docstring.

    if not collection or embedding is None:
Explanation: Validate inputs.

        return False
Explanation: Return False if inputs invalid.

    try:
Explanation: Try block for storing chunk.

        # Convert keywords list to comma-separated string for ChromaDB
Explanation: Comment noting the conversion of keywords to a string in metadata.

        keywords_str = ", ".join(keywords) if keywords else ""
Explanation: Join keywords into single string or empty string if none.

        collection.add(
Explanation: Use collection.add to insert the chunk.

            documents=[content],
Explanation: Chunk content stored as the document text.

            metadatas=[{**metadata, "keywords": keywords_str}],
Explanation: Merge provided metadata with keywords string.

            ids=[chunk_id],
Explanation: Use unique chunk id.

            embeddings=[embedding.tolist()]
Explanation: Convert embedding to list for storage.

        )
Explanation: End of collection.add call.

        logger.debug(f"Chunk stored: {chunk_id}")
Explanation: Debug log on success.

        return True
Explanation: Return True indicating success.

    except Exception as e:
Explanation: Handle storage exception.

        logger.error(f"Failed to store chunk {chunk_id}: {e}")
Explanation: Log error details.

        return False
Explanation: Return False on failure.

# ============ GEMINI API QUOTA MANAGEMENT ============
Explanation: Section header for code that throttles calls to the Gemini API to avoid quota errors.

# Global quota tracking
Explanation: Comment noting that global variables follow for quota tracking.

_api_call_times: deque[float] = deque()
Explanation: Initialize a deque to store timestamps of recent API calls for rate-limiting.

_api_lock = threading.Lock()
Explanation: Create a lock to synchronize access to quota tracking across threads.

_last_api_call = 0
Explanation: Store timestamp of the last API call for enforcing minimum delay between calls.

def _wait_for_quota():
Explanation: Define function that blocks until it's safe to make another Gemini API call.

    """Ensure we don't exceed API quota limits"""
Explanation: Docstring.

    global _api_call_times, _last_api_call
Explanation: Declare global variables to be used.

    with _api_lock:
Explanation: Acquire lock to safely examine and update shared quota state.

        current_time = time.time()
Explanation: Get current epoch time in seconds.

        # Remove calls older than 1 minute
Explanation: Comment describing cleanup of old timestamps.

        while _api_call_times and current_time - _api_call_times[0] > 60:
Explanation: While there are stored call times older than 60 seconds, pop from left.

            _api_call_times.popleft()
Explanation: Remove the oldest timestamp.

        # Check if we're near the limit
Explanation: Comment describing check against allowed calls per minute.

        if len(_api_call_times) >= CONFIG.api_requests_per_minute:
Explanation: If number of calls in past minute meets/exceeds configured per-minute limit.

            wait_time = 60 - (current_time - _api_call_times[0]) + 1
Explanation: Compute wait time until the earliest recorded call falls outside 60s window, with +1s buffer.

            logger.warning(f"Quota management: Waiting {wait_time:.1f}s to avoid API limits")
Explanation: Log a warning indicating the code will wait to avoid quota errors.

            time.sleep(wait_time)
Explanation: Sleep for computed wait_time to throttle.

        # Ensure minimum delay between calls
Explanation: Comment describing additional per-call spacing to avoid bursts.

        time_since_last = current_time - _last_api_call
Explanation: Compute time since last API call.

        if time_since_last < CONFIG.api_delay_between_calls:
Explanation: If it's been too little time since last call.

            sleep_time = CONFIG.api_delay_between_calls - time_since_last
Explanation: Compute remaining needed delay.

            logger.debug(f"API rate limiting: Waiting {sleep_time:.1f}s")
Explanation: Debug log about waiting.

            time.sleep(sleep_time)
Explanation: Sleep the remainder.

        # Record this API call
Explanation: Comment noting that current call will be recorded after waiting.

        _api_call_times.append(time.time())
Explanation: Append current timestamp to deque to record the call.

        _last_api_call = time.time()
Explanation: Update last call timestamp to now.

def _handle_quota_error(filename: str, attempt: int = 1) -> bool:
Explanation: Define helper to handle quota errors with exponential backoff.

    """Handle quota exceeded errors with exponential backoff"""
Explanation: Docstring.

    if attempt > CONFIG.max_retries:
Explanation: If we've already exceeded retry attempts, give up.

        logger.error(f"Max retries exceeded for {filename} due to quota limits")
Explanation: Log max-retry exceeded.

        return False
Explanation: Return False indicating no further retries.

    backoff_time = CONFIG.quota_retry_delay * (2 ** (attempt - 1))  # Exponential backoff
Explanation: Compute exponential backoff time (e.g., 60s, 120s, 240s for attempts 1..3).

    logger.warning(f"Quota exceeded for {filename}. Waiting {backoff_time}s (attempt {attempt}/{CONFIG.max_retries})")
Explanation: Log that the system will wait before retrying.

    time.sleep(backoff_time)
Explanation: Sleep for the computed backoff time.

    return True
Explanation: Return True to indicate a retry should be attempted after backoff.

# ============ GEMINI AI FUNCTIONS ============
Explanation: Section header for functions that use Gemini to extract text and generate summaries, with quota handling.

def extract_text_with_gemini(content: bytes, filename: str) -> Optional[str]:
Explanation: Define function to upload a file to Gemini Vision and request text extraction (OCR), handling quota and retries.

    """Extract text from file using Gemini Vision with quota management"""
Explanation: Docstring.

    for attempt in range(1, CONFIG.max_retries + 1):
Explanation: Loop over retry attempts from 1 to max_retries.

        try:
Explanation: Try block for full extraction flow.

            # Quota management
Explanation: Comment: call the quota-waiter before calling Gemini.

            _wait_for_quota()
Explanation: Wait until it's permissible to make an API call.

            # Create a temporary file
Explanation: Comment: write bytes to a temp file for upload.

            with tempfile.NamedTemporaryFile(delete=False, suffix=Path(filename).suffix) as tmp_file:
Explanation: Create a temporary file on disk with same file extension as filename; keep file after close.

                tmp_file.write(content)
Explanation: Write the blob bytes into the temporary file.

                tmp_path = tmp_file.name
Explanation: Save temporary file path for later access and cleanup.

            logger.info(f"Gemini API Call {attempt}/{CONFIG.max_retries}: Extracting text from {filename}")
Explanation: Log that an extraction attempt is being made.

            # Upload file to Gemini
Explanation: Comment: proceed to upload using either helper or genai API.

            if upload_file is None:
Explanation: If upload_file helper wasn't found in the imported module.

                raise Exception("Gemini AI library not available or upload_file not supported")
Explanation: Raise exception to be handled below.

            try:
Explanation: Try uploading using upload_file function reference.

                uploaded_file = upload_file(path=tmp_path, display_name=filename)
Explanation: Call upload_file helper passing path and display name; may return a metadata object.

            except (AttributeError, TypeError):
Explanation: If upload_file call signature isn't compatible.

                # Fallback to genai module access
Explanation: Comment noting fallback.

                try:
Explanation: Try the genai.upload_file path.

                    uploaded_file = genai.upload_file(path=tmp_path, display_name=filename)  # type: ignore
Explanation: Use genai module's upload_file function as a fallback.

                except:
Explanation: If fallback fails, raise a more descriptive exception.

                    raise Exception("upload_file not available in this version of google-generativeai")
Explanation: Raise exception indicating no upload function is available.

            # Wait for processing
Explanation: Comment: wait for uploaded file to finish processing on Gemini side.

            while uploaded_file.state.name == "PROCESSING":
Explanation: Poll the uploaded_file state until it's no longer PROCESSING.

                time.sleep(2)
Explanation: Sleep briefly to avoid tight-loop polling.

                try:
Explanation: Try to refresh uploaded_file status via get_file.

                    if get_file is not None:
Explanation: If a get_file helper is available.

                        uploaded_file = get_file(uploaded_file.name)
Explanation: Refresh uploaded_file object via helper.

                    else:
Explanation: Otherwise use genai.get_file fallback.

                        uploaded_file = genai.get_file(uploaded_file.name)  # type: ignore
Explanation: Refresh using genai.get_file.

                except (AttributeError, TypeError):
Explanation: If get_file not available or callable signature mismatch.

                    raise Exception("get_file not available in this version of google-generativeai")
Explanation: Raise an exception to be handled by outer except.

            if uploaded_file.state.name == "FAILED":
Explanation: If processing failed on provider side, raise error.

                raise Exception(f"File processing failed: {uploaded_file.state}")
Explanation: Raise descriptive exception.

            # Another API call - wait for quota
Explanation: Comment about ensuring quota before next Gemini call.

            _wait_for_quota()
Explanation: Call to ensure safe call rate for subsequent model call.

            # Generate text extraction
Explanation: Comment: instantiate a GenerativeModel and request content generation to extract text.

            if GenerativeModel is None:
Explanation: If the GenerativeModel class wasn't imported/available.

                raise Exception("GenerativeModel not available")
Explanation: Raise descriptive exception.

            model = GenerativeModel(CONFIG.extraction_model)
Explanation: Instantiate the configured extraction model (e.g., gemini-2.0-flash-exp).

            prompt = """
Explanation: Build a prompt to instruct the model to extract text and preserve structure.

            Extract all text content from this document. 
Explanation: Part of the prompt instructing to extract all text.

            Maintain the original structure and formatting as much as possible.
Explanation: Prompt instructing to keep structure.

            Include headers, paragraphs, lists, tables, and any other text elements.
Explanation: Prompt instructing to capture all text elements.

            If this is an image, perform OCR to extract any visible text.
Explanation: Prompt instructing OCR for images.

            """
Explanation: End prompt string.

            response = model.generate_content([uploaded_file, prompt])  # type: ignore
Explanation: Call model.generate_content with uploaded file and prompt to get a response object; signature varies by versions.

            # Clean up
Explanation: Comment describing attempt to delete uploaded file and temporary file.

            try:
Explanation: Try block for cleanup.

                if delete_file is not None:
Explanation: If delete_file helper available, call it.

                    delete_file(uploaded_file.name)
Explanation: Use delete_file helper to remove file from Gemini storage.

                elif genai is not None:
Explanation: Else if module genai exists, call its delete_file.

                    genai.delete_file(uploaded_file.name)  # type: ignore
Explanation: Call genai.delete_file as fallback.

            except (AttributeError, TypeError):
Explanation: If deletion helpers are not present in this version, ignore.

                # delete_file not available in this version
Explanation: Comment noting delete_file might not exist.

                pass
Explanation: No-op on cleanup error.

            os.unlink(tmp_path)
Explanation: Remove the temporary file from disk.

            logger.info(f"[SUCCESS] Gemini extraction successful for {filename}")
Explanation: Log success.

            return response.text if response.text else None
Explanation: Return the extracted text (or None if empty).

        except Exception as e:
Explanation: Outer exception handler for extraction loop attempt.

            error_msg = str(e).lower()
Explanation: Lowercase error message for keyword matching.

            # Check for quota/rate limit errors
Explanation: Check if error indicates quota / rate limiting.

            if any(quota_keyword in error_msg for quota_keyword in ['quota', 'rate limit', 'too many requests', '429']):
Explanation: If keywords indicate quota issues, attempt to back off and retry.

                logger.warning(f"Quota limit hit for {filename}: {e}")
Explanation: Log a warning describing quota hit.

                if _handle_quota_error(filename, attempt):
Explanation: Call backoff handler; if it returns True, we will retry.

                    continue  # Retry
Explanation: Continue to next attempt iteration.

                else:
Explanation: If backoff handler indicated no more retries, break.

                    break  # Max retries exceeded
Explanation: Break out of retry loop.

            else:
Explanation: Non-quota error path.

                logger.error(f"Gemini text extraction failed for {filename}: {e}")
Explanation: Log an error describing failure.

                break  # Non-quota error, don't retry
Explanation: Break to avoid retries on non-quota errors.

            # Clean up on error
Explanation: Cleanup temporary file if it exists when an exception happened.

            if 'tmp_path' in locals():
Explanation: Check whether tmp_path variable was created.

                try:
Explanation: Try removing tmp file.

                    os.unlink(tmp_path)
Explanation: Attempt to delete temporary file.

                except:
Explanation: Ignore cleanup errors.

                    pass
Explanation: No-op on cleanup exception.

    logger.error(f"Failed to extract text from {filename} after {CONFIG.max_retries} attempts")
Explanation: Log final failure after exhausting retry attempts.

    return None
Explanation: Return None to indicate extraction failure.

def generate_summary_with_gemini(text: str) -> Tuple[Optional[str], List[str]]:
Explanation: Define a function to generate a document summary and keywords using Gemini with retries and quota control.

    """Generate summary and keywords using Gemini with quota management"""
Explanation: Docstring.

    if not text or not text.strip():
Explanation: If input text is empty or whitespace, return early.

        return None, []
Explanation: Return None summary and empty keyword list for empty input.

    for attempt in range(1, CONFIG.max_retries + 1):
Explanation: Loop for retry attempts.

        try:
Explanation: Try block for summary generation.

            # Quota management
Explanation: Ensure rate limits before making model call.

            _wait_for_quota()
Explanation: Wait for quota allowances.

            logger.info(f"Gemini API Call {attempt}/{CONFIG.max_retries}: Generating summary")
Explanation: Log attempt to generate summary.

            if genai is None or GenerativeModel is None:
Explanation: Validate gemini availability.

                raise Exception("Gemini AI not available")
Explanation: Raise exception if Gemini not available.

            model = GenerativeModel(CONFIG.summarization_model)
Explanation: Instantiate the configured summarization model.

            prompt = f"""
Explanation: Build a prompt asking for a summary and keywords.

            Analyze the following document text and provide:

            1. A comprehensive summary (200-300 words) that captures the main themes, key points, and important details
Explanation: Prompt: request a 200-300 word summary.

            2. A list of 10-15 relevant keywords and key phrases that best represent the document's content

            Document text:
Explanation: Prompt includes a truncated document text to respect token limits.

            {text[:8000]}  # Limit input to avoid token limits

            Please format your response as:
            SUMMARY:
            [Your summary here]

            KEYWORDS:
            [keyword1, keyword2, keyword3, ...]
            """
Explanation: End prompt string.

            response = model.generate_content(prompt)  # type: ignore
Explanation: Call the model to generate content based on the prompt.

            if not response.text:
Explanation: If response text is empty, warn and return.

                logger.warning("Empty response from Gemini summarization")
Explanation: Log warning.

                return None, []
Explanation: Return empty results on empty response.

            # Parse response
Explanation: Prepare to parse the model's formatted output into summary and keywords.

            response_text = response.text
Explanation: Store response text.

            summary = None
Explanation: Initialize summary variable.

            keywords = []
Explanation: Initialize keywords list.

            if "SUMMARY:" in response_text and "KEYWORDS:" in response_text:
Explanation: If the response contains both markers, parse in structured way.

                parts = response_text.split("KEYWORDS:")
Explanation: Split at KEYWORDS: marker.

                summary_part = parts[0].replace("SUMMARY:", "").strip()
Explanation: Remove SUMMARY: mark and strip whitespace to get the summary text.

                keywords_part = parts[1].strip()
Explanation: Extract the keywords component.

                summary = summary_part
Explanation: Assign parsed summary.

                # Extract keywords
Explanation: Comment describing keyword extraction.

                keywords_text = keywords_part.replace("[", "").replace("]", "")
Explanation: Remove square brackets from keywords list.

                keywords = [kw.strip() for kw in keywords_text.split(",") if kw.strip()]
Explanation: Split by commas and strip whitespace to create keywords list.

            else:
Explanation: Fallback parsing if the model did not return the structured format.

                # Fallback parsing
Explanation: Comment describing fallback strategy.

                summary = response_text[:500] + "..." if len(response_text) > 500 else response_text
Explanation: Truncate long responses to 500 chars for summary fallback.

                keywords = []
Explanation: Default to empty keywords if parsing failed.

            logger.info(f"[SUCCESS] Gemini summarization successful")
Explanation: Log successful summarization.

            return summary, keywords
Explanation: Return parsed summary and keywords.

        except Exception as e:
Explanation: Catch exceptions during summarization.

            error_msg = str(e).lower()
Explanation: Lowercase for matching.

            # Check for quota/rate limit errors
Explanation: Check if error message indicates a quota problem.

            if any(quota_keyword in error_msg for quota_keyword in ['quota', 'rate limit', 'too many requests', '429']):
Explanation: If quota-like keyword present, back off.

                logger.warning(f"Quota limit hit during summarization: {e}")
Explanation: Log warning.

                if _handle_quota_error("summarization", attempt):
Explanation: Backoff and possibly retry.

                    continue  # Retry
Explanation: Continue next attempt.

                else:
Explanation: No more retries allowed.

                    break  # Max retries exceeded
Explanation: Break from retry loop.

            else:
Explanation: Non-quota error.

                logger.error(f"Gemini summarization failed: {e}")
Explanation: Log an error.

                break  # Non-quota error, don't retry
Explanation: Stop retrying on non-quota exceptions.

    logger.error(f"Failed to generate summary after {CONFIG.max_retries} attempts")
Explanation: Log overall failure after retry loop finishes.

    return None, []
Explanation: Return None summary and empty keywords to indicate failure.

# ============ NLP PROCESSING ============
Explanation: Section header for spaCy-based NLP processing utilities.

def extract_keywords_with_spacy(text: str, max_keywords: int = 15) -> List[str]:
Explanation: Define function to extract keywords using spaCy: entities, noun chunks, and important nouns/adjectives.

    """Extract keywords using spaCy NLP"""
Explanation: Docstring.

    model = get_nlp_model()
Explanation: Lazy-load spaCy model.

    if not model or not text:
Explanation: If no model or empty input, return empty.

        return []
Explanation: Return empty list.

    try:
Explanation: Try block for NLP operations.

        doc = model(text[:10000])  # Limit text length for processing
Explanation: Process first 10k characters of text with spaCy to avoid heavy CPU or memory usage.

        # Extract named entities
Explanation: Comment: extract entities of interest.

        entities = [ent.text.lower() for ent in doc.ents 
Explanation: Build list of entity texts in lowercase.

                   if ent.label_ in ['PERSON', 'ORG', 'GPE', 'PRODUCT', 'EVENT']]
Explanation: Filter entities by relevant labels.

        # Extract noun phrases
Explanation: Comment: extract short noun chunks.

        noun_phrases = [chunk.text.lower() for chunk in doc.noun_chunks 
Explanation: Collect noun chunks lowercased.

                       if len(chunk.text.split()) <= 3]
Explanation: Limit to noun chunks up to 3 words.

        # Extract important nouns and adjectives
Explanation: Comment: collect salient nouns/adjectives using POS and stopword filtering.

        important_words = [token.lemma_.lower() for token in doc 
Explanation: Lemmatize tokens and lowercase.

                          if token.pos_ in ['NOUN', 'ADJ'] and 
Explanation: Keep only nouns and adjectives.

                          len(token.text) > 3 and 
Explanation: Filter out very short tokens.

                          not token.is_stop and 
Explanation: Exclude stop words.

                          token.is_alpha]
Explanation: Keep only alphabetic tokens (no punctuation/numbers).

        # Combine and count
Explanation: Combine keyword candidates and count frequency.

        all_keywords = entities + noun_phrases + important_words
Explanation: Concatenate lists of keyword candidates.

        keyword_counts = Counter(all_keywords)
Explanation: Use Counter to count occurrences.

        # Get top keywords
Explanation: Select most common keywords up to max_keywords.

        top_keywords = [kw for kw, _ in keyword_counts.most_common(max_keywords)]
Explanation: Extract top keywords preserving order by frequency.

        return top_keywords
Explanation: Return the list of top keywords.

    except Exception as e:
Explanation: Handle any exceptions during NLP processing.

        logger.error(f"spaCy keyword extraction failed: {e}")
Explanation: Log the error.

        return []
Explanation: Return empty list on failure.

# ============ DUAL-FLOW DOCUMENT PROCESSING ============
Explanation: Section header for the main DualFlowProcessor class implementing Flow A (summary) and Flow B (chunks).

class DualFlowProcessor:
Explanation: Define the DualFlowProcessor class.

    """Handles dual-flow document processing: Summary + Chunk flows"""
Explanation: Class docstring.

    def __init__(self):
Explanation: Constructor method for initialization.

        self.processed_count = 0
Explanation: Initialize processed document count.

        self.failed_count = 0
Explanation: Initialize failed document count.

        self.start_time = time.time()
Explanation: Record start time for runtime stats.

        # Flow statistics
Explanation: Comment indicating flow-specific counters.

        self.flow_a_count = 0  # Summary processing
Explanation: Counter for Flow A (summaries).

        self.flow_b_count = 0  # Chunk processing
Explanation: Counter for Flow B (chunks).

    def process_document_dual_flow(self, blob: Any) -> Optional[Dict[str, Any]]:
Explanation: Define method to run both flows concurrently for a blob (GCS object).

        """Process document using dual-flow architecture"""
Explanation: Docstring.

        try:
Explanation: Try block to process a single document.

            logger.info(f"Processing document with dual-flow: {blob.name}")
Explanation: Log which document is being processed.

            # Common preprocessing
Explanation: Comment noting common checks before both flows.

            if not is_supported_file(blob.name):
Explanation: Skip unsupported file types.

                logger.warning(f"Unsupported file type: {blob.name}")
Explanation: Log a warning.

                return None
Explanation: Return None to indicate skipping.

            file_size_mb = get_file_size_mb(blob)
Explanation: Compute file size in MB.

            if file_size_mb > CONFIG.max_file_size_mb:
Explanation: Check against configured max file size.

                logger.warning(f"File too large ({file_size_mb:.1f}MB): {blob.name}")
Explanation: Log warning for oversized file.

                return None
Explanation: Skip large file.

            # Download and extract text with Gemini 2.5 Pro
Explanation: Comment: download blob content and use specialized Gemini 2.5 extraction.

            content = blob.download_as_bytes()
Explanation: Download blob content as bytes.

            file_hash = get_file_hash(content)
Explanation: Compute hash to use as unique doc ID.

            extracted_text = self.extract_text_gemini_25_pro(content, blob.name)
Explanation: Call helper to extract text using Gemini 2.5 Pro wrapper.

            if not extracted_text:
Explanation: If extraction failed, log and skip.

                logger.warning(f"No text extracted from: {blob.name}")
Explanation: Log warning.

                return None
Explanation: Return None to indicate failure.

            cleaned_text = clean_text(extracted_text)
Explanation: Clean the extracted text using clean_text helper.

            # Execute both flows concurrently
Explanation: Comment: use ThreadPoolExecutor to run summary and chunk flows in parallel.

            with ThreadPoolExecutor(max_workers=2) as executor:
Explanation: Create a thread pool with two workers.

                flow_a_future = executor.submit(self.flow_a_summary_processing, cleaned_text, file_hash, blob.name)
Explanation: Submit Flow A summary processing job.

                flow_b_future = executor.submit(self.flow_b_chunk_processing, cleaned_text, file_hash, blob.name)
Explanation: Submit Flow B chunk processing job.

                flow_a_result = flow_a_future.result()
Explanation: Wait for Flow A to finish and get result.

                flow_b_result = flow_b_future.result()
Explanation: Wait for Flow B to finish and get result.

            # Combine results
Explanation: Build metadata and combine outputs from both flows.

            metadata: Dict[str, Any] = {
Explanation: Create metadata dict to describe processing.

                "filename": blob.name,
Explanation: Include filename.

                "file_hash": file_hash,
Explanation: Include file hash as doc id.

                "file_size_mb": file_size_mb,
Explanation: Include file size.

                "processed_at": datetime.datetime.now().isoformat(),
Explanation: Timestamp when processed.

                "text_length": len(cleaned_text),
Explanation: Text length for diagnostics.

                "flow_a_processed": flow_a_result is not None,
Explanation: Boolean indicating if Flow A succeeded.

                "flow_b_processed": flow_b_result is not None,
Explanation: Boolean indicating if Flow B succeeded.

                "chunk_count": len(flow_b_result.get("chunks", [])) if flow_b_result else 0
Explanation: Number of chunks processed in Flow B (if available).

            }
Explanation: End metadata dict.

            return {
Explanation: Return a dict summarizing results from dual-flow processing.

                "doc_id": file_hash,
Explanation: Document ID.

                "filename": blob.name,
Explanation: Filename again.

                "extracted_text": cleaned_text,
Explanation: The cleaned text extracted.

                "flow_a_result": flow_a_result,
Explanation: Flow A result payload.

                "flow_b_result": flow_b_result,
Explanation: Flow B result payload.

                "metadata": metadata
Explanation: Attach metadata.

            }
Explanation: End return dict.

        except Exception as e:
Explanation: Catch any exception during processing.

            logger.error(f"Dual-flow processing failed for {blob.name}: {e}")
Explanation: Log failure.

            self.failed_count += 1
Explanation: Increment failed count.

            return None
Explanation: Return None on error.

    def extract_text_gemini_25_pro(self, content: bytes, filename: str) -> str:
Explanation: Wrapper method to call extract_text_with_gemini; named for clarity that uses Gemini 2.5 Pro.

        """Extract text using Gemini 2.5 Pro"""
Explanation: Docstring.

        result = extract_text_with_gemini(content, filename)
Explanation: Call external helper that handles upload and model invocation.

        return result if result is not None else ""
Explanation: Return result string or empty string on failure.

    def flow_a_summary_processing(self, text: str, doc_id: str, filename: str) -> Optional[Dict[str, Any]]:
Explanation: Define Flow A pipeline: summarization, keywords, embedding, and storing summary to ChromaDB.

        """Flow A: Summary Processing Pipeline"""
Explanation: Docstring.

        try:
Explanation: Try block for Flow A steps.

            logger.info(f"Flow A: Processing summary for {filename}")
Explanation: Log beginning of Flow A.

            # Step 1: Gemini 2.0 Flash Summarization
Explanation: Comment describing use of Gemini 2.0 Flash for summarization.

            summary = self.generate_summary_gemini_20_flash(text)
Explanation: Call helper that generates a summary using Gemini 2.0 Flash model.

            # Step 2: spaCy NLP Keyword Extraction
Explanation: Extract keywords, preferring summary if available, else first 2000 chars of text.

            keywords = extract_keywords_with_spacy(summary or text[:2000])
Explanation: Use spaCy to extract keywords from summary or early part of text.

            # Step 3: Sentence Transformers Summary Embeddings
Explanation: Generate embeddings for the summary (or truncated text fallback).

            summary_embedding = embed_text(summary or text[:1000])
Explanation: Call embed_text for summary embedding; may return numpy array or None.

            # Step 4: Store in ChromaDB Summary Collection
Explanation: Persist summary and embedding to a Chroma collection.

            collection = get_or_create_collection("summary_collection")
Explanation: Get or create summary collection.

            metadata: Dict[str, Any] = {"filename": filename, "flow": "A"}
Explanation: Build metadata to store alongside summary.

            store_summary_to_chroma(
Explanation: Call helper to store summary to ChromaDB.

                collection, doc_id, summary or text[:1000], keywords, 
Explanation: Provide collection, id, summary text (or truncated fallback), and keywords.

                summary_embedding, metadata
Explanation: Provide embedding and metadata.

            )
Explanation: End store_summary_to_chroma call.

            self.flow_a_count += 1
Explanation: Increment Flow A success counter.

            logger.info(f"Flow A completed for {filename}")
Explanation: Log completion.

            return {
Explanation: Return structured result for Flow A.

                "summary": summary,
Explanation: The generated summary text.

                "keywords": keywords,
Explanation: Keywords extracted via spaCy.

                "embedding": summary_embedding.tolist() if summary_embedding is not None else None,
Explanation: Convert embedding to list for serialization if present.

                "collection": "summary_collection"
Explanation: Indicate which collection was used.

            }
Explanation: End return dict.

        except Exception as e:
Explanation: Catch exceptions during Flow A.

            logger.error(f"Flow A processing failed for {filename}: {e}")
Explanation: Log error.

            return None
Explanation: Return None on failure.

    def flow_b_chunk_processing(self, text: str, doc_id: str, filename: str) -> Optional[Dict[str, Any]]:
Explanation: Define Flow B pipeline: header-based chunking, per-chunk keywords & embeddings, and storing into ChromaDB.

        """Flow B: Chunk Processing Pipeline"""
Explanation: Docstring.

        try:
Explanation: Try block for Flow B.

            logger.info(f"Flow B: Processing chunks for {filename}")
Explanation: Log start of Flow B.

            # Step 1: Header-Based Text Chunking
Explanation: Chunk the document using header patterns to preserve logical sections.

            chunks = self.header_based_chunking(text)
Explanation: Call header-based chunking helper to get list of chunk strings.

            chunk_results: List[Dict[str, Any]] = []
Explanation: Initialize list to hold per-chunk metadata results.

            collection = get_or_create_collection("per_doc_collection")
Explanation: Get or create collection for document chunks.

            for i, chunk in enumerate(chunks):
Explanation: Loop over each chunk with index.

                chunk_id = f"{doc_id}_chunk_{i}"
Explanation: Construct unique chunk id by combining doc id and index.

                # Step 2: spaCy NLP Per-Chunk Keywords
Explanation: Extract keywords from each chunk.

                chunk_keywords = extract_keywords_with_spacy(chunk)
Explanation: Call spaCy keyword extractor.

                # Step 3: Sentence Transformers Chunk Embeddings
Explanation: Compute embedding for the chunk.

                chunk_embedding = embed_text(chunk)
Explanation: Get embedding for chunk text.

                # Step 4: Store in ChromaDB Document Collection
Explanation: Prepare metadata and store chunk to ChromaDB.

                chunk_metadata: Dict[str, Any] = {
Explanation: Build chunk-level metadata.

                    "filename": filename, 
Explanation: Store filename.

                    "chunk_index": i,
Explanation: Store index.

                    "parent_doc_id": doc_id,
Explanation: Store parent document id.

                    "flow": "B"
Explanation: Indicate flow B.

                }
Explanation: End chunk_metadata dict.

                store_chunk_to_chroma(
Explanation: Call helper to persist chunk to ChromaDB.

                    collection, chunk_id, chunk, chunk_keywords,
Explanation: Provide collection, chunk id, chunk text, keywords.

                    chunk_embedding, chunk_metadata
Explanation: Provide embedding and metadata.

                )
Explanation: End store_chunk_to_chroma call.

                chunk_results.append({
Explanation: Append result summary for this chunk.

                    "chunk_id": chunk_id,
Explanation: Chunk id.

                    "text": chunk[:500] + "..." if len(chunk) > 500 else chunk,
Explanation: Truncate preview of chunk text if too long.

                    "keywords": chunk_keywords,
Explanation: List of keywords for chunk.

                    "embedding": chunk_embedding.tolist() if chunk_embedding is not None else None
Explanation: Convert embedding to list if present.

                })
Explanation: End appended dict.

            self.flow_b_count += 1
Explanation: Increment Flow B counter.

            logger.info(f"Flow B completed for {filename} with {len(chunks)} chunks")
Explanation: Log completion and chunk count.

            return {
Explanation: Return Flow B result payload.

                "chunks": chunk_results,
Explanation: Array of chunk results.

                "chunk_count": len(chunks),
Explanation: Number of chunks.

                "collection": "per_doc_collection"
Explanation: Which collection was used.

            }
Explanation: End return dict.

        except Exception as e:
Explanation: Catch exceptions during Flow B.

            logger.error(f"Flow B processing failed for {filename}: {e}")
Explanation: Log error.

            return None
Explanation: Return None on failure.

    def generate_summary_gemini_20_flash(self, text: str) -> str:
Explanation: Helper method to generate a summary using Gemini 2.0 Flash model with quotas.

        """Generate summary using Gemini 2.0 Flash with quota management"""
Explanation: Docstring.

        if not gemini_available or not CONFIG.gemini_api_key:
Explanation: If Gemini not available or API key missing, return empty string early.

            return ""
Explanation: Return empty summary fallback.

        for attempt in range(1, CONFIG.max_retries + 1):
Explanation: Retry loop.

            try:
Explanation: Try block for model call.

                # Quota management
Explanation: Ensure safe call rate.

                _wait_for_quota()
Explanation: Wait until safe.

                logger.info(f"Gemini 2.0 Flash API Call {attempt}/{CONFIG.max_retries}: Generating summary")
Explanation: Log attempt to use Gemini 2.0 Flash.

                # Use Gemini 2.0 Flash for faster summarization
Explanation: Comment.

                if genai is None or GenerativeModel is None:
Explanation: Validate availability.

                    raise Exception("Gemini AI not available")
Explanation: Raise if not available.

                model = GenerativeModel('gemini-2.0-flash-exp')
Explanation: Instantiate the 2.0 Flash model.

                prompt = f"""
Explanation: Build prompt for the model to produce a concise summary.

                Please provide a comprehensive summary of the following document.
                Focus on key points, main topics, and important insights.
                Keep the summary concise but informative (2-3 paragraphs).
                
                Document:
                {text[:8000]}  # Limit for API efficiency
                """
Explanation: End of prompt.

                response = model.generate_content(prompt)  # type: ignore
Explanation: Call model.generate_content to get summary.

                result = response.text.strip() if response.text else ""
Explanation: Extract text from response and trim whitespace.

                if result:
Explanation: If result non-empty, log success and return it.

                    logger.info(f"[SUCCESS] Gemini 2.0 Flash summary successful")
Explanation: Log success.

                    return result
Explanation: Return summary text.

                else:
Explanation: If empty result, warn and try again (if attempts left).

                    logger.warning(f"Empty response from Gemini 2.0 Flash (attempt {attempt})")
Explanation: Log warning.

            except Exception as e:
Explanation: Catch exceptions during summarization.

                error_msg = str(e).lower()
Explanation: Lowercase error string.

                # Check for quota/rate limit errors
Explanation: Check for quota-like keywords.

                if any(quota_keyword in error_msg for quota_keyword in ['quota', 'rate limit', 'too many requests', '429']):
Explanation: If quota issue, backoff and possibly retry.

                    logger.warning(f"Quota limit hit in Gemini 2.0 Flash: {e}")
Explanation: Log warning.

                    if _handle_quota_error("gemini-2.0-flash", attempt):
Explanation: Call backoff handler.

                        continue  # Retry
Explanation: Continue to next attempt.

                    else:
Explanation: No more retries allowed.

                        break  # Max retries exceeded
Explanation: Break retry loop.

                else:
Explanation: Non-quota error.

                    logger.error(f"Gemini 2.0 Flash summarization failed: {e}")
Explanation: Log error.

                    break  # Non-quota error, don't retry
Explanation: Stop retrying.

        logger.error(f"Gemini 2.0 Flash failed after {CONFIG.max_retries} attempts")
Explanation: Log final failure message for summary generation.

        return ""
Explanation: Return empty string on failure.

    def header_based_chunking(self, text: str) -> List[str]:
Explanation: Define a header-aware chunking function that splits the document at header-like patterns to create semantically meaningful chunks.

        """Perform header-based text chunking"""
Explanation: Docstring.

        try:
Explanation: Try block for chunking logic which uses regex patterns to split text.

            # Split by common header patterns
Explanation: Comment.

            header_patterns = [
Explanation: Define list of regex patterns representing common header separators.

                r'\n#+\s+',      # Markdown headers
Explanation: Markdown headers like "# Header" or "## Subheader".

                r'\n\d+\.\s+',   # Numbered sections
Explanation: Numbered lists "1. Section".

                r'\n[A-Z][A-Z\s]+:',  # ALL CAPS headers
Explanation: Lines with capitalized all-caps labels followed by colon.

                r'\n[A-Z][a-z]+:',    # Title case headers
Explanation: Lines with TitleCase followed by colon.

                r'\n\n[A-Z]',    # New paragraphs starting with capital
Explanation: Paragraph separation detection (two newlines then capital letter).

            ]
Explanation: End header_patterns list.

            chunks = [text]
Explanation: Start with the full text as the only chunk, then iteratively split by patterns.

            for pattern in header_patterns:
Explanation: Loop through header patterns to progressively split chunks.

                new_chunks: List[str] = []
Explanation: Prepare list to collect results of splitting this iteration.

                for chunk in chunks:
Explanation: Iterate existing chunks and split each by current pattern.

                    splits = re.split(pattern, chunk)
Explanation: Use re.split to split chunk by pattern; note this removes the delimiter.

                    new_chunks.extend([s.strip() for s in splits if s.strip()])
Explanation: Add non-empty stripped splits to new_chunks.

                chunks = new_chunks
Explanation: Replace chunks with newly split results for next pattern.

            # Filter and size chunks appropriately
Explanation: Post-processing to ensure chunks are of suitable sizes.

            final_chunks: List[str] = []
Explanation: Initialize final list for chunks after size adjustments.

            for chunk in chunks:
Explanation: Iterate over chunks produced by header splitting.

                if len(chunk) > CONFIG.chunk_size:
Explanation: If chunk exceeds configured chunk size, split further by sentence boundaries.

                    # Further split large chunks
Explanation: Comment.

                    sentences = re.split(r'[.!?]+', chunk)
Explanation: Split chunk into sentences by sentence-ending punctuation (note punctuation removed).

                    current_chunk = ""
Explanation: Buffer to accumulate sentences into a chunk up to chunk_size.

                    for sentence in sentences:
Explanation: Loop through sentences to accumulate into sized chunks.

                        if len(current_chunk + sentence) > CONFIG.chunk_size:
Explanation: If adding sentence would exceed chunk size.

                            if current_chunk:
Explanation: If the current buffer has content, finalize and append it to final_chunks.

                                final_chunks.append(current_chunk.strip())
Explanation: Append buffer stripped.

                            current_chunk = sentence
Explanation: Start a new buffer with the current sentence.

                        else:
Explanation: If buffer plus sentence remains within chunk size.

                            current_chunk += sentence + ". "
Explanation: Append sentence plus a period and space (we previously removed punctuation with split).

                    if current_chunk:
Explanation: After looping sentences, append any remaining buffered chunk.

                        final_chunks.append(current_chunk.strip())
Explanation: Append final buffer content.

                elif len(chunk) > 100:  # Minimum chunk size
Explanation: If chunk is smaller than chunk_size but larger than a minimum threshold, accept it.

                    final_chunks.append(chunk)
Explanation: Append chunk as-is to final_chunks.

            return final_chunks[:50]  # Limit number of chunks
Explanation: Return up to first 50 chunks to bound memory and storage.

        except Exception as e:
Explanation: On any exception during header-based chunking, log and fallback to simple fixed-size chunking.

            logger.error(f"Header-based chunking failed: {e}")
Explanation: Log the error.

            # Fallback to simple chunking
Explanation: Note fallback behavior.

            return [text[i:i+CONFIG.chunk_size] for i in range(0, len(text), CONFIG.chunk_size)]
Explanation: Return list of fixed-size slices of the text as fallback.

    def process_document(self, blob: Any) -> Optional[Dict[str, Any]]:
Explanation: Define an alternate (single-flow) document processing method used elsewhere in the codebase; similar to dual-flow but simpler.

        """Process a single document"""
Explanation: Docstring.

        try:
Explanation: Try block for a single-document process.

            logger.info(f"Processing document: {blob.name}")
Explanation: Log which document is being processed.

            # Validate file
Explanation: Basic file validation before processing.

            if not is_supported_file(blob.name):
Explanation: Skip unsupported file types.

                logger.warning(f"Unsupported file type: {blob.name}")
Explanation: Log warning.

                return None
Explanation: Return None to indicate skipping.

            file_size_mb = get_file_size_mb(blob)
Explanation: Compute file size in MB.

            if file_size_mb > CONFIG.max_file_size_mb:
Explanation: Skip files exceeding size limit.

                logger.warning(f"File too large ({file_size_mb:.1f}MB): {blob.name}")
Explanation: Log the oversized condition.

                return None
Explanation: Return None.

            # Download file content
Explanation: Download blob bytes.

            content = blob.download_as_bytes()
Explanation: Get bytes of the blob.

            file_hash = get_file_hash(content)
Explanation: Compute file hash for doc id.

            # Extract text
Explanation: Use extract_text_with_gemini helper (not the 2.5 wrapper) to extract text.

            extracted_text = extract_text_with_gemini(content, blob.name)
Explanation: Call extraction helper to perform OCR/extraction.

            if not extracted_text:
Explanation: If extraction failed, log and return.

                logger.warning(f"No text extracted from: {blob.name}")
Explanation: Log warning.

                return None
Explanation: Return None.

            # Clean and process text
Explanation: Clean text for NLP and summarization.

            cleaned_text = clean_text(extracted_text)
Explanation: Call clean_text helper.

            # Generate summary and keywords with Gemini
Explanation: Use Gemini summarization and then spaCy for additional keywords.

            summary, gemini_keywords = generate_summary_with_gemini(cleaned_text)
Explanation: Call generate_summary_with_gemini to get summary and keywords.

            # Extract additional keywords with spaCy
Explanation: Run spaCy on full cleaned text.

            spacy_keywords = extract_keywords_with_spacy(cleaned_text)
Explanation: Extract keywords via spaCy.

            # Combine keywords
Explanation: Combine Gemini-provided keywords with spaCy-extracted ones, deduplicate and limit count.

            all_keywords = list(set(gemini_keywords + spacy_keywords))[:20]
Explanation: Make a set to deduplicate then convert to list and keep first 20.

            # Generate embeddings
Explanation: Create an embedding for the summary (or fallback to first 1000 chars).

            summary_embedding = embed_text(summary or cleaned_text[:1000])
Explanation: Embed the summary or initial part of the text.

            # Create metadata
Explanation: Build metadata dict for this document.

            metadata: Dict[str, Any] = {
Explanation: Start metadata.

                "filename": blob.name,
Explanation: Include filename.

                "file_hash": file_hash,
Explanation: Include file hash.

                "file_size_mb": file_size_mb,
Explanation: Include size.

                "extraction_method": "gemini",
Explanation: Indicate extraction method used.

                "processed_at": datetime.datetime.now().isoformat(),
Explanation: Timestamp for processed time.

                "text_length": len(cleaned_text),
Explanation: Text length.

                "summary_length": len(summary) if summary else 0,
Explanation: Summary length if present.

                "keyword_count": len(all_keywords)
Explanation: Number of keywords.

            }
Explanation: End metadata dict.

            return {
Explanation: Return structured result.

                "doc_id": file_hash,
Explanation: Document ID.

                "filename": blob.name,
Explanation: Filename.

                "extracted_text": cleaned_text,
Explanation: Extracted and cleaned text.

                "summary": summary,
Explanation: Generated summary.

                "keywords": all_keywords,
Explanation: Combined keyword list.

                "summary_embedding": summary_embedding,
Explanation: Embedding (numpy array or None).

                "metadata": metadata
Explanation: Metadata dict.

            }
Explanation: End return dict.

        except Exception as e:
Explanation: Catch any error during processing.

            logger.error(f"Document processing failed for {blob.name}: {e}")
Explanation: Log error.

            return None
Explanation: Return None to indicate failure.

# ============ REAL-TIME MONITORING ORCHESTRATOR ============
Explanation: Section header introducing orchestrator class that monitors bucket and triggers processing (supports Pub/Sub + polling).

class RealTimePipelineOrchestrator:
Explanation: Define orchestrator class that manages monitoring, Pub/Sub subscription, and processing.

    """Enhanced orchestrator with real-time Pub/Sub notifications + polling"""
Explanation: Docstring describing capabilities.

    def __init__(self, bucket_name: str):
Explanation: Constructor that takes a bucket name and sets up GCS and internal state.

        self.bucket_name = bucket_name
Explanation: Store bucket name.

        self.processor = DualFlowProcessor()
Explanation: Create a DualFlowProcessor instance for document processing.

        self.processed_docs: Set[str] = set()
Explanation: Keep a set of processed document paths to avoid reprocessing.

        self.processing_errors: Dict[str, Exception] = {}
Explanation: Dictionary mapping filenames to exceptions encountered during processing.

        self.is_monitoring = False
Explanation: Flag indicating whether polling monitor is active.

        self.monitor_thread = None
Explanation: Thread reference for polling monitor.

        # Folder filtering attributes
Explanation: Attributes controlling which files/folders to process.

        self.include_folders: Optional[List[str]] = None
Explanation: Optional list of folders to include (populated by config).

        self.exclude_folders: List[str] = []
Explanation: List of folders to exclude.

        self.root_only: bool = False
Explanation: Whether to limit processing to root files only.

        self.folder_depth_limit: int = 10
Explanation: Maximum folder depth allowed.

        # Real-time Pub/Sub components
Explanation: Attributes related to Pub/Sub subscriber and state.

        self.pubsub_subscriber = None
Explanation: Placeholder for pubsub subscriber client.

        self.pubsub_listening = False
Explanation: Boolean to indicate if Pub/Sub listener active.

        self.subscription_path = None
Explanation: Subscription path string (project/subscription).

        self.streaming_pull_future = None
Explanation: StreamingPullFuture object reference for canceling subscription.

        # Processing stats
Explanation: Counters for real-time vs polling processed docs.

        self.realtime_processed = 0
Explanation: Count of documents processed via Pub/Sub real-time triggers.

        self.polling_processed = 0
Explanation: Count of documents processed via polling.

        self.last_detection_time = None
Explanation: Timestamp for the last detected document.

        # Initialize GCS client
Explanation: If google cloud storage client is available, initialize it and locate bucket.

        if gcs_available and StorageClient is not None:
Explanation: Check GCS availability.

            self.storage_client = StorageClient()
Explanation: Instantiate storage client.

            try:
Explanation: Try to access the bucket and verify existence.

                self.bucket = self.storage_client.bucket(bucket_name)  # type: ignore
Explanation: Get bucket object for bucket_name.

                if not self.bucket.exists():  # type: ignore
Explanation: Check whether bucket exists; note this may perform an API call.

                    logger.error(f"Bucket '{bucket_name}' not found")
Explanation: Log error if not found.

                    raise ValueError(f"Bucket '{bucket_name}' not found")
Explanation: Raise ValueError up the stack to indicate inability to proceed.

                logger.info(f"Connected to GCS bucket: {bucket_name}")
Explanation: Log successful connection to bucket.

            except Exception as e:
Explanation: Handle exceptions while connecting to GCS.

                logger.error(f"Failed to connect to bucket '{bucket_name}': {e}")
Explanation: Log the exception.

                raise
Explanation: Re-raise exception to fail initialization.

        else:
Explanation: If GCS client isn't available, raise ImportError.

            raise ImportError("Google Cloud Storage not available")
Explanation: Raise ImportError to indicate missing dependency.

        # Initialize Pub/Sub if enabled
Explanation: If configured to use Pub/Sub and pubsub library is available, initialize subscription.

        if CONFIG.enable_pubsub and pubsub_available:
Explanation: Check configuration flag and library availability.

            self._initialize_pubsub()
Explanation: Call private method to initialize Pub/Sub subscriber.

    def _initialize_pubsub(self):
Explanation: Private helper to set up Pub/Sub subscriber client and subscription path.

        """Initialize Pub/Sub subscriber for real-time notifications"""
Explanation: Docstring.

        try:
Explanation: Try to create Pub/Sub subscriber client and compute subscription path.

            if pubsub_v1 is None:
Explanation: Validate pubsub library exists.

                raise ImportError("Pub/Sub not available")
Explanation: Raise error if not available.

            self.pubsub_subscriber = pubsub_v1.SubscriberClient()
Explanation: Create SubscriberClient instance.

            self.subscription_path = self.pubsub_subscriber.subscription_path(
Explanation: Compute subscription path using project id and subscription name.

                CONFIG.project_id, CONFIG.pubsub_subscription
Explanation: Supply project and subscription names.

            )
Explanation: End subscription_path assignment.

            logger.info(f"[INIT] Pub/Sub initialized: {self.subscription_path}")
Explanation: Log initialization.

            logger.info(f"[READY] Real-time monitoring ready for bucket: {self.bucket_name}")
Explanation: Log readiness for real-time monitoring.

        except Exception as e:
Explanation: Handle any initialization exceptions.

            logger.error(f"Pub/Sub initialization failed: {e}")
Explanation: Log the failure.

            logger.info("Falling back to polling-only monitoring")
Explanation: Inform that system will fall back to polling.

            self.pubsub_subscriber = None
Explanation: Nullify pubsub subscriber to indicate it's not available.

    def start_realtime_monitoring(self):
Explanation: Public method to start real-time monitoring (Pub/Sub listener + polling fallback).

        """Start enhanced real-time monitoring with Pub/Sub + polling"""
Explanation: Docstring.

        # Start Pub/Sub listener if available
Explanation: Attempt to start Pub/Sub listener first if subscriber exists and not currently listening.

        if self.pubsub_subscriber and not self.pubsub_listening:
Explanation: Check conditions.

            self._start_pubsub_listener()
Explanation: Start the Pub/Sub listener in background.

        # Always start polling as fallback
Explanation: Ensure polling monitor starts regardless of Pub/Sub availability.

        self.start_monitoring()
Explanation: Start polling monitor.

        monitoring_type = "Real-time (Pub/Sub + Polling)" if self.pubsub_listening else "Polling Only"
Explanation: Determine monitoring type string for logging.

        logger.info(f"[START] Enhanced monitoring started: {monitoring_type}")
Explanation: Log startup information.

        logger.info(f"[CONFIG] Monitoring interval: {CONFIG.monitoring_interval} seconds")
Explanation: Log the interval being used for polling.

    def _start_pubsub_listener(self):
Explanation: Private helper to start Pub/Sub subscription listening in a background thread.

        """Start Pub/Sub listener for instant notifications"""
Explanation: Docstring.

        if not self.pubsub_subscriber:
Explanation: If subscriber not initialized, return early.

            return
Explanation: Early return.

        self.pubsub_listening = True
Explanation: Mark that Pub/Sub listening is active.

        def callback(message: Any):
Explanation: Define callback to handle messages delivered by subscription.

            try:
Explanation: Try block to parse and handle messages.

                # Parse the Pub/Sub message
Explanation: Decode payload and parse JSON content per GCS event format.

                data = json.loads(message.data.decode('utf-8'))
Explanation: Decode bytes to UTF-8 and parse JSON.

                # Check if it's a finalize event (file upload completed)
Explanation: Only react to OBJECT_FINALIZE events indicating upload completion.

                if data.get('eventType') == 'OBJECT_FINALIZE':
Explanation: Check event type.

                    bucket_name = data.get('bucketId')
Explanation: Extract bucket id from message.

                    file_name = data.get('objectId')
Explanation: Extract file path/name.

                    if bucket_name == self.bucket_name and file_name:
Explanation: Ensure event corresponds to configured bucket.

                        logger.info(f"[REAL-TIME] REAL-TIME DETECTION: New document via Pub/Sub: {file_name}")
Explanation: Log detection.

                        self._process_new_document_realtime(file_name, "pubsub")
Explanation: Trigger immediate processing for detected file.

                message.ack()
Explanation: Acknowledge message to Pub/Sub to mark as processed.

            except Exception as e:
Explanation: Catch exceptions in message handling.

                logger.error(f"Pub/Sub message processing error: {e}")
Explanation: Log the error.

                message.nack()
Explanation: Nack message so it can be retried.

        # Start listening in background thread
Explanation: Wrap subscription listening in a thread so it doesn't block the main process.

        def listen_for_messages():
Explanation: Function that subscribes and waits on streaming pull future.

            try:
Explanation: Try block for subscription and listening.

                if self.pubsub_subscriber is None or self.subscription_path is None:
Explanation: Validate subscriber and subscription path.

                    logger.error("Pub/Sub subscriber or subscription path not available")
Explanation: Log error.

                    return
Explanation: Exit listener function.

                self.streaming_pull_future = self.pubsub_subscriber.subscribe(  # type: ignore
Explanation: Call subscribe() to receive messages by invoking callback for each message.

                    self.subscription_path, callback=callback
Explanation: Provide subscription path and callback function.

                )
Explanation: End call to subscribe.

                logger.info(f"Listening for real-time notifications on {self.subscription_path}")
Explanation: Log that listener has started.

                with self.pubsub_subscriber:
Explanation: Enter context manager for subscriber to manage resources.

                    try:
Explanation: Wait on streaming pull future result which blocks until terminated or exception.

                        self.streaming_pull_future.result()
Explanation: Block awaiting messages; will throw on cancel.

                    except KeyboardInterrupt:
Explanation: If interrupted by keyboard, cancel the streaming pull.

                        self.streaming_pull_future.cancel()
Explanation: Cancel.

                        logger.info("Pub/Sub listener stopped by user")
Explanation: Log user stopping.

            except Exception as e:
Explanation: Top-level listener error handling.

                logger.error(f"Pub/Sub listener error: {e}")
Explanation: Log exception.

        pubsub_thread = threading.Thread(target=listen_for_messages, daemon=True)
Explanation: Create a daemon thread to run the listener.

        pubsub_thread.start()
Explanation: Start the thread.

    logger.info("Real-time Pub/Sub listener started")
Explanation: NOTE: This logger call appears at module indentation level in the original source (not inside method) — it logs that the listener started. It may execute during class definition time due to indentation error in original code (this is likely a bug). This line logs an informational message.

    def _process_new_document_realtime(self, file_name: str, trigger_type: str = "polling"):
Explanation: Define method to immediately process a newly-detected document, called by Pub/Sub callback or polling.

        """Process new document immediately upon detection"""
Explanation: Docstring.

        try:
Explanation: Try block for processing steps.

            if file_name in self.processed_docs:
Explanation: Skip documents already processed.

                logger.debug(f"Document already processed: {file_name}")
Explanation: Debug log.

                return
Explanation: Return early.

            # Check folder filtering first
Explanation: Apply folder filters before fetching blob.

            if not self._should_process_file(file_name):
Explanation: Use folder rules to decide whether to process this file.

                logger.debug(f"Document filtered out by folder rules: {file_name}")
Explanation: Debug log that file was filtered.

                return
Explanation: Return early if filtered.

            # Get the blob
Explanation: Fetch blob object from GCS bucket based on file path/name.

            blob = self.bucket.blob(file_name)  # type: ignore
Explanation: Get Blob object.

            if not blob.exists():  # type: ignore
Explanation: Check whether the blob exists in the bucket.

                logger.warning(f"Document not found in bucket: {file_name}")
Explanation: Warn if file does not exist.

                return
Explanation: Return early.

            # Check if supported file type
Explanation: Ensure file extension is supported.

            if not is_supported_file(file_name):
Explanation: Check extension.

                logger.debug(f"Unsupported file type, skipping: {file_name}")
Explanation: Debug log and skip.

                return
Explanation: Return early.

            # Process immediately
Explanation: Begin actual processing and timing.

            start_time = time.time()
Explanation: Record start time for processing.

            logger.info(f"[PROCESSING] INSTANT PROCESSING ({trigger_type.upper()}): {file_name}")
Explanation: Log an instant processing event with trigger type.

            result = self._process_single_document(blob)
Explanation: Process document using internal helper that delegates to DualFlowProcessor.

            processing_time = time.time() - start_time
Explanation: Compute processing duration.

            if result:
Explanation: If processing succeeded, update processed sets and emit events.

                self.processed_docs.add(file_name)
Explanation: Mark file as processed.

                if trigger_type == "pubsub":
Explanation: Increment approprate counter based on trigger type.

                    self.realtime_processed += 1
Explanation: Increment real-time counter.

                else:
Explanation: Else increment polling counter.

                    self.polling_processed += 1
Explanation: Increment polling counter.

                self.last_detection_time = datetime.datetime.now()
Explanation: Update last detection timestamp.

                logger.info(f"[SUCCESS] REAL-TIME SUCCESS: {file_name} processed in {processing_time:.2f}s via {trigger_type}")
Explanation: Log success.

                # Emit real-time event
Explanation: Call method to log/emit event for monitoring dashboards.

                self._emit_processing_event("document_processed", {
Explanation: Emit an event containing key metrics and result.

                    "filename": file_name,
Explanation: Filename processed.

                    "processing_time_seconds": round(processing_time, 2),
Explanation: Rounded processing time.

                    "trigger": trigger_type,
Explanation: Trigger type.

                    "detection_speed": "real-time" if trigger_type == "pubsub" else "polling",
Explanation: Human-readable detection speed.

                    "result": result
Explanation: Attach processing result.

                })
Explanation: End event emission call.

            else:
Explanation: If result falsey, log failure.

                logger.error(f"REAL-TIME FAILED: {file_name} processing failed")
Explanation: Log real-time failure.

        except Exception as e:
Explanation: Catch exceptions from real-time processing.

            logger.error(f"Real-time processing error for {file_name}: {e}")
Explanation: Log exception.

    def _emit_processing_event(self, event_type: str, data: Dict[str, Any]):
Explanation: Define method to emit events (currently logs) for external dashboards/webhooks.

        """Emit processing events for real-time monitoring dashboard"""
Explanation: Docstring.

        try:
Explanation: Try block for event construction and logging.

            _: Dict[str, Any] = {
Explanation: Build event data structure (assigned to underscore to indicate unused variable).

                "timestamp": datetime.datetime.now().isoformat(),
Explanation: Event timestamp.

                "event_type": event_type,
Explanation: Event type string.

                "bucket": self.bucket_name,
Explanation: Bucket name.

                "data": data
Explanation: Actual event data payload.

            }
Explanation: End event payload.

            # Log the event with emoji indicators
Explanation: Compose logs showing different message for pubsub vs polling triggers.

            trigger = data.get('trigger', 'unknown')
Explanation: Extract trigger value.

            filename = data.get('filename', 'unknown')
Explanation: Extract filename.

            processing_time = data.get('processing_time_seconds', 0)
Explanation: Extract processing time.

            if trigger == "pubsub":
Explanation: If pubsub-triggered event, log as real-time.

                logger.info(f"[EVENT] REAL-TIME EVENT: {filename} processed instantly ({processing_time}s)")
Explanation: Log event.

            else:
Explanation: Else log as polling.

                logger.info(f"[EVENT] POLLING EVENT: {filename} detected and processed ({processing_time}s)")
Explanation: Log event.

            # You can add webhook notifications here for external monitoring
Explanation: Comment suggesting extension to send webhooks.

            # self._send_webhook_notification(event_data)
Explanation: Example code for where to add webhook support.

        except Exception as e:
Explanation: Event emission exception handling.

            logger.error(f"Event emission error: {e}")
Explanation: Log any errors while emitting event.

    def get_realtime_stats(self) -> Dict[str, Any]:
Explanation: Return a summary of real-time monitoring statistics.

        """Get real-time monitoring statistics"""
Explanation: Docstring.

        total_processed = len(self.processed_docs)
Explanation: Compute total processed docs count.

        return {
Explanation: Return dictionary of stats for dashboards.

            "total_processed": total_processed,
Explanation: Total processed count.

            "realtime_processed": self.realtime_processed,
Explanation: Real-time processed count.

            "polling_processed": self.polling_processed,
Explanation: Polling processed count.

            "realtime_percentage": round((self.realtime_processed / max(total_processed, 1)) * 100, 1),
Explanation: Percentage of real-time processed out of total (avoid division by zero).

            "polling_percentage": round((self.polling_processed / max(total_processed, 1)) * 100, 1),
Explanation: Polling percentage.

            "pubsub_enabled": CONFIG.enable_pubsub and self.pubsub_listening,
Explanation: Whether Pub/Sub is enabled and actively listening.

            "monitoring_active": self.is_monitoring,
Explanation: Whether polling monitor is active.

            "last_detection": self.last_detection_time.isoformat() if self.last_detection_time else None,
Explanation: Last detection timestamp ISO string or None.

            "monitoring_interval_seconds": CONFIG.monitoring_interval
Explanation: Current monitoring interval config.

        }
Explanation: End return dict.

    def stop_realtime_monitoring(self):
Explanation: Stop both Pub/Sub listener and polling monitor.

        """Stop all monitoring (Pub/Sub + polling)"""
Explanation: Docstring.

        # Stop Pub/Sub listener
Explanation: Cancel streaming pull future if set and mark listening false.

        if self.streaming_pull_future:
Explanation: If streaming pull object exists.

            self.streaming_pull_future.cancel()
Explanation: Cancel the streaming pull.

        self.pubsub_listening = False
Explanation: Mark Pub/Sub as not listening.

        # Stop polling
Explanation: Stop polling monitor.

        self.stop_monitoring()
Explanation: Call stop_monitoring to stop polling.

    logger.info("Real-time monitoring stopped (Pub/Sub + Polling)")
Explanation: NOTE: This log call appears at module/class indentation level in original source and will execute at import/definition time; likely misplaced. It logs that real-time monitoring stopped.

    # Enhanced pipeline orchestrator with real-time capabilities
Explanation: Comment heading preceding get_pipeline_status method.

    def get_pipeline_status(self) -> Dict[str, Any]:
Explanation: Define method to return current pipeline status including real-time stats.

        """Get current pipeline status with real-time stats"""
Explanation: Docstring.

        runtime = time.time() - self.processor.start_time
Explanation: Compute runtime in seconds since processor start.

        processed = len(self.processed_docs)
Explanation: Count processed docs.

        failed = len(self.processing_errors)
Explanation: Count failures.

        total = processed + failed
Explanation: Total attempts known (processed + failed).

        base_stats: Dict[str, Any] = {
Explanation: Prepare a base stats dict with runtime and throughput measures.

            "processed_count": processed,
Explanation: Processed count.

            "failed_count": failed,
Explanation: Failed count.

            "total_processed": total,
Explanation: Total.

            "success_rate": (processed / total * 100) if total > 0 else 0,
Explanation: Compute success rate, protect divide by zero.

            "average_processing_time": runtime / total if total > 0 else 0,
Explanation: Compute average processing time per document (simplistic), protect divide by zero.

            "total_runtime": runtime,
Explanation: Total runtime.

            "throughput": processed / (runtime / 60) if runtime > 0 else 0,
Explanation: Throughput measured as processed per minute.

            "is_monitoring": self.is_monitoring
Explanation: Whether monitoring is active.

        }
Explanation: End base_stats dict.

        # Add real-time specific stats
Explanation: Merge in real-time stats.

        realtime_stats = self.get_realtime_stats()
Explanation: Call get_realtime_stats.

        base_stats.update({
Explanation: Update base_stats to include realtime_stats and monitoring type.

            "realtime_stats": realtime_stats,
Explanation: Add realtime stats nested.

            "monitoring_type": "real-time" if self.pubsub_listening else "polling"
Explanation: Human readable monitoring type.

        })
Explanation: End update call.

        return base_stats
Explanation: Return combined stats dictionary.

    def list_documents(self) -> List[Any]:
Explanation: List objects in configured GCS bucket and apply folder filtering, returning supported blobs.

        """List documents in bucket with folder filtering"""
Explanation: Docstring.

        try:
Explanation: Try to fetch blob list and filter.

            blobs: List[Any] = list(self.bucket.list_blobs())  # type: ignore
Explanation: Retrieve blobs in bucket and convert to list.

            filtered_blobs = self.apply_folder_filters(blobs)
Explanation: Apply folder include/exclude filters to blobs list.

            supported_blobs = [blob for blob in filtered_blobs if is_supported_file(blob.name)]
Explanation: Filter for supported file extensions.

            logger.info(f"[FILTER] Real-time filtering: {len(blobs)} total -> {len(filtered_blobs)} filtered -> {len(supported_blobs)} supported")
Explanation: Log how many files were filtered and supported.

            if CONFIG.include_folders:
Explanation: If include_folders configured, log it for operator visibility.

                logger.info(f"[CONFIG] Processing only folders: {CONFIG.include_folders}")
Explanation: Log include_folders setting.

            return supported_blobs
Explanation: Return list of supported blobs.

        except Exception as e:
Explanation: Handle errors fetching blob list.

            logger.error(f"Failed to list documents: {e}")
Explanation: Log error.

            return []
Explanation: Return empty list if failed.

    def apply_folder_filters(self, blobs: List[Any]) -> List[Any]:
Explanation: Iterate blob list and keep only those that pass folder filtering rules.

        """Apply folder filtering logic to blob list"""
Explanation: Docstring.

        filtered_blobs: List[Any] = []
Explanation: Initialize result list.

        for blob in blobs:
Explanation: Loop over blobs.

            if self._should_process_file(blob.name):
Explanation: If blob's path passes filter logic, add to list.

                filtered_blobs.append(blob)
Explanation: Append blob to filtered list.

        return filtered_blobs
Explanation: Return filtered list of blobs.

    def _should_process_file(self, file_path: str) -> bool:
Explanation: Determine whether a particular file path should be processed per include/exclude/root/depth rules.

        """Determine if a file should be processed based on folder filters"""
Explanation: Docstring.

        # Root only filter
Explanation: If configured to process only root files, skip ones with slashes.

        if self.root_only and '/' in file_path:
Explanation: Check and skip.

            logger.debug(f"Skipping folder file (root_only=True): {file_path}")
Explanation: Debug log.

            return False
Explanation: Return False to skip.

        # Get folder path
Explanation: Logic to extract folder path and compute folder depth when file in subfolder.

        if '/' in file_path:
Explanation: If path contains folder separators.

            folder_path = '/'.join(file_path.split('/')[:-1]) + '/'
Explanation: Extract folder path and append trailing slash.

            folder_depth = len(folder_path.split('/')) - 1
Explanation: Compute folder depth (# of path segments minus 1).

            # Check folder depth limit
Explanation: If folder depth exceeds limit, skip file.

            if folder_depth > self.folder_depth_limit:
Explanation: Compare depth to limit.

                logger.debug(f"Skipping file (depth {folder_depth} > limit {self.folder_depth_limit}): {file_path}")
Explanation: Debug log for skipping.

                return False
Explanation: Return False.

            # Check include folders (if specified)
Explanation: If include_folders configured, only allow files whose folder_path starts with any include folder prefix.

            if self.include_folders:
Explanation: If include_folders list present.

                should_include = False
Explanation: Initialize flag.

                for include_folder in self.include_folders:
Explanation: Loop include folders.

                    if folder_path.startswith(include_folder):
Explanation: If file's folder_path starts with include folder, mark true and break.

                        should_include = True
Explanation: Mark include flag.

                        break
Explanation: Stop checking further include folders.

                if not should_include:
Explanation: If none matched, skip file.

                    logger.debug(f"Skipping file (not in include_folders): {file_path}")
Explanation: Debug log.

                    return False
Explanation: Return False.

            # Check exclude folders
Explanation: Exclude files if their folder path starts with any exclude folder prefix.

            for exclude_folder in self.exclude_folders:
Explanation: Loop exclude folders.

                if folder_path.startswith(exclude_folder):
Explanation: If matches exclude prefix.

                    logger.debug(f"Skipping file (in exclude_folders): {file_path}")
Explanation: Debug log.

                    return False
Explanation: Skip file.

        else:
Explanation: File is at root level (no slashes).

            # Root file - check if we're only processing specific folders
Explanation: If include_folders is specified, root files should be skipped.

            if self.include_folders:
Explanation: If include_folders present.

                logger.debug(f"Skipping root file (include_folders specified): {file_path}")
Explanation: Debug log.

                return False
Explanation: Skip root file.

        return True
Explanation: If none of the skip conditions were met, return True to indicate file should be processed.

    def process_existing_documents(self) -> List[Optional[Dict[str, Any]]]:
Explanation: Walk the bucket (with filters), process existing documents in parallel using ThreadPoolExecutor, and return list of results.

        """Process all existing documents in the bucket"""
Explanation: Docstring.

        documents = self.list_documents()
Explanation: Get list of supported documents.

        if not documents:
Explanation: If none, log and return empty list.

            logger.info("No documents found to process")
Explanation: Log.

            return []
Explanation: Return empty list.

        logger.info(f"Starting processing of {len(documents)} documents")
Explanation: Log starting processing count.

        results: List[Any] = []
Explanation: Initialize results list.

        with ThreadPoolExecutor(max_workers=CONFIG.max_workers) as executor:
Explanation: Use configured number of workers to process documents concurrently.

            future_to_blob = {
Explanation: Create dict mapping futures to blob objects.

                executor.submit(self._process_single_document, blob): blob 
Explanation: Submit document processing tasks.

                for blob in documents
Explanation: For each blob in documents list.

            }
Explanation: End comprehension creating future_to_blob.

            for future in as_completed(future_to_blob):
Explanation: Iterate futures as they complete.

                blob = future_to_blob[future]
Explanation: Get the corresponding blob for this future.

                try:
Explanation: Try to get result or catch exceptions from future.

                    result = future.result()
Explanation: Obtain result returned by _process_single_document.

                    results.append(result)
Explanation: Append to results list.

                    if result:
Explanation: If processing succeeded, mark as processed, else record error.

                        self.processed_docs.add(blob.name)
Explanation: Add blob name to processed set.

                        logger.info(f"Successfully processed: {blob.name}")
Explanation: Log success.

                    else:
Explanation: If result is None, treat as processing failure.

                        self.processing_errors[blob.name] = Exception("Processing failed")
Explanation: Record processing failure.

                        logger.error(f"Failed to process: {blob.name}")
Explanation: Log failure.

                except Exception as e:
Explanation: If calling future.result() raised exception, catch and record.

                    self.processing_errors[blob.name] = e
Explanation: Store exception for blob.

                    logger.error(f"Error processing {blob.name}: {e}")
Explanation: Log.

                    results.append(None)
Explanation: Append None to results to preserve ordering.

        successful = len([r for r in results if r is not None])
Explanation: Count successful results.

        logger.info(f"Processing complete: {successful}/{len(documents)} successful")
Explanation: Log summary of processing.

        return results
Explanation: Return list of results (some may be None).

    def _process_single_document(self, blob: Any) -> Optional[Dict[str, Any]]:
Explanation: Internal method that processes a single document via DualFlowProcessor and returns compact result summary.

        """Process a single document and store in ChromaDB"""
Explanation: Docstring.

        try:
Explanation: Try block wrapping the processor invocation.

            # Process document using dual-flow
Explanation: Call the dual-flow processing method to perform heavy lifting.

            result = self.processor.process_document_dual_flow(blob)
Explanation: Invoke dual-flow processor.

            if not result:
Explanation: If processing failed, return None.

                return None
Explanation: Return None.

            return {
Explanation: Return compact result metadata for this processed file.

                "filename": blob.name,
Explanation: Filename.

                "doc_id": result["doc_id"],
Explanation: Document id.

                "flow_a_processed": result.get("flow_a_result") is not None,
Explanation: Boolean indicating Flow A completed.

                "flow_b_processed": result.get("flow_b_result") is not None,
Explanation: Boolean indicating Flow B completed.

                "chunks": len(result.get("flow_b_result", {}).get("chunks", [])),
Explanation: Number of chunks processed.

                "summary_length": len(result.get("flow_a_result", {}).get("summary", "")),
Explanation: Length of generated summary.

                "keywords_count": len(result.get("flow_a_result", {}).get("keywords", []))
Explanation: Number of keywords generated.

            }
Explanation: End return dict.

        except Exception as e:
Explanation: Catch processing exceptions.

            logger.error(f"Error processing {blob.name}: {e}")
Explanation: Log error.

            return None
Explanation: Return None on exception.

    def start_monitoring(self):
Explanation: Start continuous polling monitor in a background thread.

        """Start continuous monitoring for new documents (polling)"""
Explanation: Docstring.

        self.is_monitoring = True
Explanation: Flip monitoring flag to True.

        self.monitor_thread = threading.Thread(target=self._monitor_bucket, daemon=True)
Explanation: Create daemon thread to run _monitor_bucket polling loop.

        self.monitor_thread.start()
Explanation: Start polling thread.

        logger.info("Started continuous polling monitoring")
Explanation: Log that polling started.

    def stop_monitoring(self):
Explanation: Stop polling monitoring.

        """Stop continuous monitoring"""
Explanation: Docstring.

        self.is_monitoring = False
Explanation: Set monitoring flag to False which will cause polling loop to exit.

        if self.monitor_thread:
Explanation: If a monitor thread exists, wait for it to finish with timeout.

            self.monitor_thread.join(timeout=5)
Explanation: Join with 5-second timeout to allow graceful shutdown.

        logger.info("Stopped continuous monitoring")
Explanation: Log that monitoring stopped.

    def _monitor_bucket(self):
Explanation: Internal method to periodically poll the bucket for new or updated documents and initiate processing; uses last_check timestamp.

        """Monitor bucket for new documents with enhanced real-time processing"""
Explanation: Docstring.

        last_check = time.time()
Explanation: Initialize last check timestamp.

        while self.is_monitoring:
Explanation: Loop while monitoring flag is True.

            try:
Explanation: Try block to fetch and check documents periodically.

                time.sleep(CONFIG.monitoring_interval)
Explanation: Sleep according to configured monitoring interval between each polling cycle.

                # Get current documents
Explanation: Fetch current set of documents and their update times.

                current_docs = {blob.name: blob.updated for blob in self.list_documents()}
Explanation: Map blob names to their updated timestamp.

                # Check for new or updated documents
Explanation: Build list of new or modified docs since last check.

                new_docs: List[str] = []
Explanation: Initialize list to collect new docs.

                for doc_name, updated_time in current_docs.items():
Explanation: Iterate over current documents and their update times.

                    if doc_name not in self.processed_docs:
Explanation: If document has not been processed yet, consider it new.

                        new_docs.append(doc_name)
Explanation: Add to new_docs.

                    elif updated_time and updated_time.timestamp() > last_check:
Explanation: Or if document updated since last check, treat as new/updated.

                        new_docs.append(doc_name)
Explanation: Add updated doc to list.

                if new_docs:
Explanation: If any new/updated docs were found, process them.

                    logger.info(f"[POLLING] POLLING DETECTION: Found {len(new_docs)} new/updated documents")
Explanation: Log detection count.

                    for doc_name in new_docs:
Explanation: Iterate found documents and process them through _process_new_document_realtime with 'polling' trigger.

                        self._process_new_document_realtime(doc_name, "polling")
Explanation: Immediately process document.

                last_check = time.time()
Explanation: Update last_check timestamp after scanning.

            except Exception as e:
Explanation: Catch errors in monitoring loop and wait before retrying.

                logger.error(f"Monitoring error: {e}")
Explanation: Log monitoring error.

                time.sleep(CONFIG.monitoring_interval)
Explanation: Sleep to avoid tight failure loop.

# ============ REMOVED DUPLICATE PIPELINE ORCHESTRATOR ============
Explanation: Comment explaining duplicate orchestrator code was removed and merged.

# (Functionality merged into RealTimePipelineOrchestrator)
Explanation: Additional explanatory comment.

# ============ DOCUMENT FLOW PROCESSING ============
Explanation: Section header for a standalone function process_document_flow used by API endpoints.

def process_document_flow(blob: Any) -> Optional[Dict[str, Any]]:
Explanation: Define top-level function to process a single blob and store summary/chunks into ChromaDB collections; used by some API endpoints.

    """Complete document processing flow"""
Explanation: Docstring.

    try:
Explanation: Try block implementing a complete flow similar to orchestrator usage.

        # Create temporary processor
Explanation: Instantiate a DualFlowProcessor to process this blob.

        processor = DualFlowProcessor()
Explanation: Create processor instance.

        # Process the document
Explanation: Process document via dual-flow.

        result = processor.process_document_dual_flow(blob)
Explanation: Call dual-flow method to get results.

        if not result:
Explanation: If processing fails, log and return.

            return None
Explanation: Return None.

        # Get collections
Explanation: Get or create both collections used for storing results.

        summary_collection = get_or_create_collection("summary_collection")
Explanation: Get/create summary collection.

        per_doc_collection = get_or_create_collection("per_doc_collection")
Explanation: Get/create per-doc collection for chunks.

        if not summary_collection or not per_doc_collection:
Explanation: If either collection unavailable, log error and return.

            logger.error("Failed to access ChromaDB collections")
Explanation: Log.

            return None
Explanation: Return None.

        # Store summary
Explanation: Store summary into ChromaDB using stored result.

        store_summary_to_chroma(
Explanation: Call helper to store summary into summary_collection.

            summary_collection,
Explanation: Pass collection object.

            result["doc_id"],
Explanation: Document id.

            result["summary"] or result["extracted_text"][:1000],
Explanation: Use summary if available or fallback to leading text.

            result["keywords"],
Explanation: Provide keywords.

            result["summary_embedding"],
Explanation: Provide embedding for summary.

            result["metadata"]
Explanation: Provide metadata.

        )
Explanation: End store_summary_to_chroma call.

        # Process and store chunks
Explanation: Create chunks from full extracted text using chunk_text (rather than header-based), and store per-chunk embeddings and metadata.

        chunks = chunk_text(result["extracted_text"])
Explanation: Chunk the full text with generic chunk_text function.

        doc_id = result["doc_id"]
Explanation: Save doc_id locally.

        for i, chunk in enumerate(chunks):
Explanation: Iterate chunks to store each one.

            content = chunk
Explanation: Local alias to chunk content.

            keywords = result["keywords"][:10]  # Limit keywords per chunk
Explanation: Use top 10 keywords as chunk-level keywords.

            chunk_metadata: Dict[str, Any] = {
Explanation: Build chunk metadata merging document metadata with chunk info.

                **result["metadata"],
Explanation: Unpack parent metadata.

                "chunk_index": i,
Explanation: Chunk index.

                "total_chunks": len(chunks)
Explanation: Total chunks count.

            }
Explanation: End metadata.

            chunk_emb = embed_texts([content])[0] if content else np.zeros(get_embedding_dimension())  # type: ignore
Explanation: Compute embedding for chunk content using embed_texts and take first element; fallback to zero vector if content empty.

            keyword_embs = embed_texts(keywords) if keywords else np.zeros((0, get_embedding_dimension()))  # type: ignore
Explanation: Generate embeddings for keywords for possible future use; if no keywords, create an empty numpy array as placeholder.

            chunk_id = f"{doc_id}_chunk_{i}"
Explanation: Build chunk id string.

            store_chunk_to_chroma(per_doc_collection, chunk_id, content, keywords, chunk_emb, chunk_metadata)
Explanation: Store chunk and metadata into per_doc_collection.

        return {"filename": blob.name, "doc_id": doc_id, "chunks": len(chunks)}
Explanation: Return summary dict indicating file processed and number of chunks stored.

    except Exception as e:
Explanation: Catch exceptions during process_document_flow.

        logger.error(f"Error processing {blob.name}: {e}")
Explanation: Log error.

        return None
Explanation: Return None on failure.

# -------------------- FASTAPI APPLICATION -------------------- #
Explanation: Section header for all FastAPI application components: models, Lifespan, endpoints.

# Pydantic Models for API
Explanation: Define request/response Pydantic models used in endpoints.

class PipelineConfigRequest(BaseModel):
Explanation: Pydantic model for POST /pipeline/start request config.

    bucket_name: str = Field(..., description="GCS bucket name")
Explanation: Required bucket_name field.

    process_existing: bool = Field(True, description="Process existing documents")
Explanation: Whether to process existing documents on start.

    enable_monitoring: bool = Field(True, description="Enable continuous monitoring")
Explanation: Whether to enable monitoring.

    enable_realtime: bool = Field(False, description="Enable real-time Pub/Sub monitoring")
Explanation: Toggle for Pub/Sub-based real-time monitoring.

    # Folder filtering options
Explanation: Additional fields controlling folder filtering.

    include_folders: Optional[List[str]] = Field(None, description="Process only these folders (None = all folders)")
Explanation: Optional include_folders list.

    exclude_folders: List[str] = Field(default=[], description="Exclude these folders from processing")
Explanation: List of folders to exclude.

    root_only: bool = Field(False, description="Process only root files (no subfolders)")
Explanation: Root-only flag.

    folder_depth_limit: int = Field(default=10, ge=1, le=20, description="Maximum folder depth to process")
Explanation: Folder depth limit with validation constraints.

    max_workers: int = Field(default=3, ge=1, le=10, description="Maximum worker threads")
Explanation: Max workers override for pipeline.

    batch_size: int = Field(default=10, ge=1, le=50, description="Batch size for processing")
Explanation: Batch size for batch operations.

    monitoring_interval: int = Field(default=10, ge=5, le=300, description="Monitoring interval in seconds")
Explanation: Monitoring interval with validation constraints.

class PipelineStatusResponse(BaseModel):
Explanation: Pydantic model representing status responses for pipeline control endpoints.

    status: str
Explanation: Status string (e.g., "success" or "active").

    message: str
Explanation: Human-readable message.

    orchestrator_active: bool
Explanation: Whether orchestrator is active.

    monitoring_active: bool
Explanation: Whether monitoring is active.

    processed_documents: int = 0
Explanation: Count of processed documents (default 0).

    total_documents: int = 0
Explanation: Total documents count (default 0).

class DocumentProcessRequest(BaseModel):
Explanation: Model used when a client requests processing of a specific document.

    bucket_name: str
Explanation: Bucket name.

    document_path: str
Explanation: Path of document within bucket.

class SearchRequest(BaseModel):
Explanation: Model used for search endpoints.

    query: str
Explanation: Query text.

    collection_type: str = Field(default="summary", description="'summary' or 'document'")
Explanation: Which collection type to search by default "summary".

    top_k: int = Field(default=5, ge=1, le=20)
Explanation: Number of results to return with validation.

# Global variables for FastAPI
Explanation: Global placeholders for orchestrator instances and monitoring flags used across endpoints.

orchestrator: Optional[RealTimePipelineOrchestrator] = None
Explanation: Global orchestrator instance used by API endpoints.

realtime_orchestrator: Optional[RealTimePipelineOrchestrator] = None
Explanation: Another orchestrator variable referenced in some endpoints (legacy naming).

monitoring_active: bool = False
Explanation: Flag indicating monitoring active state.

realtime_monitoring_active: bool = False
Explanation: Flag indicating realtime monitoring active state.

# Lifespan event handler
Explanation: Use FastAPI asynccontextmanager to run code at startup and shutdown.

@asynccontextmanager
async def lifespan(app: FastAPI):
Explanation: Define asynccontextmanager function used by FastAPI's lifespan parameter.

    """Handle application startup and shutdown events"""
Explanation: Docstring.

    # Startup
Explanation: Begin startup block.

    logger.info("Starting FastAPI application with ChromaDB Cloud")
Explanation: Log startup.

    logger.info(f"Database: {CONFIG.chroma_database}")
Explanation: Log configured database.

    logger.info(f"Host: {CONFIG.chroma_host}")
Explanation: Log configured chroma host.

    # Test ChromaDB Cloud connection
Explanation: Attempt to initialize and verify ChromaDB at startup.

    try:
Explanation: Try block.

        client = get_chroma_client()
Explanation: Attempt to get chroma client.

        if client:
Explanation: If client initialized successfully, attempt to create test collections.

            logger.info("ChromaDB Cloud connection successful")
Explanation: Log success.

            # Test collection creation
Explanation: Create required collections for operations.

            summary_collection = get_or_create_collection("summary_collection")
Explanation: Create/get summary collection.

            per_doc_collection = get_or_create_collection("per_doc_collection")
Explanation: Create/get per-doc collection.

            if summary_collection and per_doc_collection:
Explanation: If both collections available, log readiness.

                logger.info("Collections 'summary_collection' and 'per_doc_collection' ready")
Explanation: Log collections ready.

            else:
Explanation: If either collection missing, warn.

                logger.warning("Collection creation failed")
Explanation: Log warning.

        else:
Explanation: If client is None, log connection failure.

            logger.error("ChromaDB Cloud connection failed")
Explanation: Log error.

    except Exception as e:
Explanation: Catch exceptions during ChromaDB startup test.

        logger.error(f"ChromaDB Cloud startup error: {e}")
Explanation: Log exception.

    # [AUTO-START] REAL-TIME MONITORING SYSTEM
Explanation: Code that auto-starts real-time monitoring on app startup (if GCS available); sets up orchestrator and starts monitoring.

    global orchestrator, realtime_monitoring_active
Explanation: Declare globals to be modified.

    try:
Explanation: Try block to initialize orchestrator and start real-time monitoring.

        if gcs_available and StorageClient is not None:
Explanation: Only attempt if GCS is available.

            logger.info("[AUTO-START] Real-time Monitoring System...")
Explanation: Log about auto-starting monitoring system.

            # Initialize orchestrator
Explanation: Instantiate orchestrator using configured bucket name.

            orchestrator = RealTimePipelineOrchestrator(bucket_name=CONFIG.gcs_bucket_name)
Explanation: Create orchestrator.

            # Configure folder filtering for inspection_docs/
Explanation: Apply bucket prefix filtering if configured.

            if CONFIG.gcs_bucket_prefix:
Explanation: If bucket prefix set in config.

                orchestrator.include_folders = [CONFIG.gcs_bucket_prefix]
Explanation: Restrict orchestrator to that prefix.

                logger.info(f"[MONITOR] Monitoring folder: {CONFIG.gcs_bucket_prefix}")
Explanation: Log folder being monitored.

            else:
Explanation: If no prefix configured.

                logger.info("[MONITOR] Monitoring entire bucket")
Explanation: Log that whole bucket will be monitored.

            orchestrator.root_only = CONFIG.root_only
Explanation: Apply root_only setting.

            orchestrator.folder_depth_limit = CONFIG.folder_depth_limit
Explanation: Apply folder depth limit.

            # Start background monitoring with 1-second interval
Explanation: Configure monitoring active flag and start monitoring (non-async).

            realtime_monitoring_active = True
Explanation: Set flag.

            # Start monitoring directly (it's not async)
Explanation: Note that orchestrator.start_realtime_monitoring is synchronous and will spawn background threads.

            orchestrator.start_realtime_monitoring()
Explanation: Start the orchestrator's monitoring (Pub/Sub + Polling).

            logger.info("[SUCCESS] Real-time monitoring started successfully!")
Explanation: Log success.

            logger.info("[INTERVAL] Interval: 1 second")
Explanation: Log chosen interval.

            logger.info("[PROCESSING] Auto-processing: Flow A (Summaries) + Flow B (Chunks)")
Explanation: Log which flows are auto-processing.

        else:
Explanation: If GCS not available, warn the operator that auto-start is disabled.

            logger.warning("[WARNING] GCS not available - Real-time monitoring disabled")
Explanation: Log warning.

    except Exception as e:
Explanation: Catch any exceptions when trying to start orchestrator.

        logger.error(f"Failed to start real-time monitoring: {e}")
Explanation: Log failure.

    yield
Explanation: Yield control back to FastAPI; the app runs until shutdown.

    # Shutdown (optional cleanup)
Explanation: Code to run on shutdown after yield.

    logger.info("Shutting down FastAPI application")
Explanation: Log shutdown.

# Initialize FastAPI app
Explanation: Instantiate the FastAPI application object providing metadata and the lifespan handler.

app = FastAPI(
Explanation: Create FastAPI instance.

    title="GCS Document Ingestion Pipeline API",
Explanation: API title.

    description="Scalable, fault-tolerant document processing with Gemini 2.5 Pro OCR",
Explanation: API description.

    version="1.0.0",
Explanation: API version.

    lifespan=lifespan
Explanation: Attach the previously defined lifespan context manager to be executed on startup/shutdown.

)
Explanation: End FastAPI instantiation.

# Add CORS middleware
Explanation: Add CORS middleware to allow cross-origin requests.

app.add_middleware(
Explanation: Call to add middleware to FastAPI app.

    CORSMiddleware,
Explanation: Use CORSMiddleware.

    allow_origins=["*"],
Explanation: Allow all origins (open CORS) — be careful for production security.

    allow_credentials=True,
Explanation: Allow credentials such as cookies/authorization headers.

    allow_methods=["*"],
Explanation: Allow all HTTP methods.

    allow_headers=["*"],
Explanation: Allow all HTTP headers.

)
Explanation: End add_middleware call.

# API Endpoints
Explanation: Start defining actual API endpoints (root, pipeline control, document processing, search, analytics, etc.)

@app.get("/", response_class=HTMLResponse)
Explanation: Root endpoint that returns an HTML welcome/documentation page.

async def root():
Explanation: Async function implementing root GET endpoint.

    """API documentation and welcome page"""
Explanation: Docstring.

    return """
Explanation: Return a multi-line HTML string with a human-readable welcome page and quick links.

    <html>
        <head>
            <title>GCS Document Ingestion Pipeline API</title>
            <style>
                body { font-family: Arial, sans-serif; margin: 40px; }
                .header { color: #2c3e50; border-bottom: 2px solid #3498db; padding-bottom: 10px; }
                .section { margin: 20px 0; }
                .endpoint { background: #f8f9fa; padding: 10px; border-left: 4px solid #3498db; margin: 10px 0; }
                code { background: #e9ecef; padding: 2px 6px; border-radius: 3px; }
            </style>
        </head>
        <body>
            <h1 class="header">GCS Document Ingestion - Dual-Flow Pipeline API</h1>
            <p><em>Flow A: Summary Processing | Flow B: Chunk Processing</em></p>
            <p><strong>Gemini 2.5 Pro + 2.0 Flash | spaCy NLP | Sentence Transformers | ChromaDB</strong></p>
            
            <div class="section">
                <h2>Available Endpoints</h2>
                <div class="endpoint">
                    <strong>GET /docs</strong> - Interactive API documentation (Swagger UI)
                </div>
                <div class="endpoint">
                    <strong>GET /redoc</strong> - Alternative API documentation (ReDoc)
                </div>
                <div class="endpoint">
                    <strong>POST /pipeline/start</strong> - Start the document processing pipeline
                </div>
                <div class="endpoint">
                    <strong>POST /pipeline/process</strong> - Process documents
                </div>
                <div class="endpoint">
                    <strong>POST /pipeline/stop</strong> - Stop the pipeline
                </div>
                <div class="endpoint">
                    <strong>GET /pipeline/status</strong> - Get pipeline status
                </div>
                <div class="endpoint">
                    <strong>POST /documents/process</strong> - Process a specific document
                </div>
                <div class="endpoint">
                    <strong>POST /search</strong> - Search processed documents
                </div>
                <div class="endpoint">
                    <strong>GET /analytics</strong> - Get processing analytics
                </div>
            </div>
            
            <div class="section">
                <h2>🔗 Quick Links</h2>
                <ul>
                    <li><a href="/docs">Swagger UI Documentation</a></li>
                    <li><a href="/redoc">ReDoc Documentation</a></li>
                </ul>
            </div>
        </body>
    </html>
    """
Explanation: End multiline HTML payload returned by the root endpoint.

@app.post("/pipeline/start", response_model=PipelineStatusResponse)
Explanation: POST endpoint to initialize and (optionally) start pipeline monitoring; returns PipelineStatusResponse model.

async def start_pipeline(config: PipelineConfigRequest):
Explanation: Async handler receiving PipelineConfigRequest payload.

    """Initialize and start the document processing pipeline"""
Explanation: Docstring.

    global orchestrator, realtime_orchestrator, monitoring_active, realtime_monitoring_active
Explanation: Declare globals modified within endpoint.

    try:
Explanation: Try to initialize orchestrator and validate/chroma collections.

        # Update global CONFIG
Explanation: Apply settings from API request into global CONFIG.

        CONFIG.max_workers = config.max_workers
Explanation: Override global max_workers.

        CONFIG.batch_size = config.batch_size
Explanation: Override batch_size.

        CONFIG.monitoring_interval = config.monitoring_interval
Explanation: Override monitoring interval.

        # Update folder filtering settings
Explanation: Apply folder filter settings.

        CONFIG.include_folders = config.include_folders
Explanation: Update include_folders.

        CONFIG.exclude_folders = config.exclude_folders
Explanation: Update exclude_folders.

        CONFIG.root_only = config.root_only
Explanation: Update root_only flag.

        CONFIG.folder_depth_limit = config.folder_depth_limit
Explanation: Update folder depth limit.

        # Always use RealTimePipelineOrchestrator (with real-time optional)
Explanation: Instantiate orchestrator object.

        orchestrator = RealTimePipelineOrchestrator(config.bucket_name)
Explanation: Create orchestrator instance for requested bucket.

        if config.enable_realtime:
Explanation: Log whether realtime requested.

            logger.info("[INIT] Real-time Enhanced pipeline orchestrator initialized")
Explanation: Log realtime init.

        else:
Explanation: Non-realtime (polling) init log.

            logger.info("[INIT] Real-time pipeline orchestrator initialized (polling mode)")
Explanation: Log.

        # Initialize ChromaDB collections
Explanation: Ensure collections exist for processing.

        summary_collection = get_or_create_collection("summary_collection")
Explanation: Get/create summary collection.

        per_doc_collection = get_or_create_collection("per_doc_collection")
Explanation: Get/create per-doc collection.

        if not summary_collection or not per_doc_collection:
Explanation: If either collection creation failed, raise HTTPException.

            raise HTTPException(status_code=500, detail="Failed to initialize ChromaDB collections")
Explanation: Raise HTTP 500 indicating inability to setup DB.

        pipeline_type = "Real-time Enhanced" if config.enable_realtime else "Standard"
Explanation: Pick a human-readable pipeline type string.

        return PipelineStatusResponse(
Explanation: Return PipelineStatusResponse with initial status values.

            status="success",
Explanation: Status.

            message=f"{pipeline_type} pipeline initialized successfully",
Explanation: Message to client.

            orchestrator_active=True,
Explanation: Orchestrator active flag.

            monitoring_active=False
Explanation: Monitoring not started by this endpoint; toggled separately.

        )
Explanation: End return statement.

    except Exception as e:
Explanation: Catch exceptions during pipeline start.

        logger.error(f"Pipeline initialization error: {e}")
Explanation: Log exception.

        raise HTTPException(status_code=500, detail=f"Failed to initialize pipeline: {str(e)}")
Explanation: Raise HTTP 500 with error info to client.

@app.post("/pipeline/process", response_model=PipelineStatusResponse)
Explanation: Endpoint that triggers processing of existing documents and optionally starts monitoring in background.

async def process_documents(
Explanation: Async function signature.

    background_tasks: BackgroundTasks,
Explanation: FastAPI BackgroundTasks object to schedule background tasks.

    process_existing: bool = True,
Explanation: Query/body parameter controlling processing of existing docs.

    enable_monitoring: bool = True,
Explanation: Whether to start monitoring after initial processing.

    enable_realtime: bool = False
Explanation: Whether to use real-time monitoring (Pub/Sub) when starting monitoring.

):
Explanation: End function signature.

    """Process documents with optional monitoring"""
Explanation: Docstring.

    global orchestrator, realtime_orchestrator, monitoring_active, realtime_monitoring_active
Explanation: Declare globals to read/modify.

    if not orchestrator:
Explanation: If orchestrator not initialized, return error to client.

        raise HTTPException(status_code=400, detail="Pipeline not initialized. Start pipeline first.")
Explanation: Raise HTTP 400 indicating pipeline not initialized.

    try:
Explanation: Try block to perform processing and start monitoring.

        processed
