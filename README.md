# PostgreSQL + pgvector Setup Guide

## Your Current Setup
- PostgreSQL running at: localhost:5434
- Database: OCR_POC_DB
- Issue: pgvector extension not installed

## Solution Options

### Option 1: Use Docker with pgvector (EASIEST)

1. Install Docker Desktop for Windows from: https://www.docker.com/products/docker-desktop/

2. After Docker is installed, run this command:
```bash
docker run -d --name pgvector-db ^
  -p 5434:5432 ^
  -e POSTGRES_PASSWORD=$antAbi1209 ^
  -e POSTGRES_DB=OCR_POC_DB ^
  -e POSTGRES_USER=postgres ^
  ankane/pgvector
```

3. Wait 10 seconds for the container to start, then run:
```bash
python setup_pgvector.py
```

### Option 2: Install pgvector on Windows PostgreSQL

1. Download pgvector for Windows from:
   https://github.com/pgvector/pgvector/releases

2. Extract the files to your PostgreSQL extension directory:
   - Usually: C:\Program Files\PostgreSQL\{version}\share\extension\

3. Restart PostgreSQL service

4. Run: python setup_pgvector.py

### Option 3: Use a simpler vector store (QUICK FIX)

We can switch to using ChromaDB (local file-based) instead of PostgreSQL.
This doesn't require any database setup.

Which option would you like to proceed with?
