# Installing pgvector on PostgreSQL 18.0 (Windows)

## Your PostgreSQL Information
- **Version**: PostgreSQL 18.0 (x86_64-windows)
- **Installation Path**: `C:\Program Files\PostgreSQL\18\`
- **Data Directory**: `C:\Program Files\PostgreSQL\18\data`

---

## Installation Steps

### Method 1: Pre-built Binaries (RECOMMENDED - Easiest)

#### Step 1: Download pgvector
1. Go to: https://github.com/pgvector/pgvector/releases
2. Download the **Windows binaries** for your PostgreSQL version (look for `pgvector-v0.7.4-windows-x64.zip` or latest)
   - Note: If exact version isn't available, download the closest version

#### Step 2: Extract Files
Extract the downloaded ZIP file to a temporary location (e.g., `C:\Temp\pgvector`)

#### Step 3: Copy Files to PostgreSQL Directory
You need **Administrator privileges** for this step.

Open PowerShell **as Administrator** and run:

```powershell
# Navigate to the extracted pgvector folder
cd C:\Temp\pgvector

# Copy vector.dll to lib directory
Copy-Item -Path "vector.dll" -Destination "C:\Program Files\PostgreSQL\18\lib\" -Force

# Copy extension control file
Copy-Item -Path "vector.control" -Destination "C:\Program Files\PostgreSQL\18\share\extension\" -Force

# Copy all SQL files
Copy-Item -Path "vector--*.sql" -Destination "C:\Program Files\PostgreSQL\18\share\extension\" -Force
```

**Manual Alternative:**
1. Open File Explorer **as Administrator** (right-click -> Run as administrator)
2. Navigate to extracted pgvector folder
3. Copy `vector.dll` to: `C:\Program Files\PostgreSQL\18\lib\`
4. Copy `vector.control` to: `C:\Program Files\PostgreSQL\18\share\extension\`
5. Copy all `vector--*.sql` files to: `C:\Program Files\PostgreSQL\18\share\extension\`

#### Step 4: Restart PostgreSQL Service

**Option A - Using Services GUI:**
1. Press `Win + R`, type `services.msc`, press Enter
2. Find "postgresql-x64-18" service
3. Right-click → **Restart**

**Option B - Using PowerShell (as Administrator):**
```powershell
Restart-Service -Name "postgresql-x64-18"
```

#### Step 5: Verify Installation
Run the setup script again:
```powershell
.venv\Scripts\python.exe setup_pgvector.py
```

If successful, you should see:
```
✓ Created pgvector extension
✓ Created table 'data_rag_vectors'
✓ Created vector index 'rag_vectors_idx'
```

---

### Method 2: Build from Source (Advanced)

**Requirements:**
- Visual Studio 2022 (Community Edition is free)
- Visual Studio C++ Build Tools

#### Step 1: Install Visual Studio
1. Download from: https://visualstudio.microsoft.com/downloads/
2. Install with "Desktop development with C++" workload

#### Step 2: Clone pgvector
```powershell
git clone https://github.com/pgvector/pgvector.git
cd pgvector
git checkout v0.7.4  # or latest stable version
```

#### Step 3: Build
Open **Developer Command Prompt for VS 2022** and run:
```cmd
set PGROOT=C:\Program Files\PostgreSQL\18
nmake /F Makefile.win
nmake /F Makefile.win install
```

#### Step 4: Restart PostgreSQL (see Method 1, Step 4)

---

## Troubleshooting

### Issue: "Access Denied" when copying files
**Solution**: Run PowerShell or File Explorer as Administrator

### Issue: "Service not found: postgresql-x64-18"
**Solution**: Check actual service name:
```powershell
Get-Service -Name "*postgres*"
```

### Issue: Extension still not available after installation
**Solution**: 
1. Verify files are in correct locations:
   ```powershell
   Test-Path "C:\Program Files\PostgreSQL\18\lib\vector.dll"
   Test-Path "C:\Program Files\PostgreSQL\18\share\extension\vector.control"
   ```
2. Check PostgreSQL logs: `C:\Program Files\PostgreSQL\18\data\log\`
3. Ensure PostgreSQL service restarted successfully

### Issue: DLL compatibility error
**Solution**: 
- Ensure downloaded binary matches PostgreSQL version (18.x)
- Check if binary is 64-bit (matches your PostgreSQL installation)

---

## Quick Verification Commands

After installation, connect to your database and run:

```sql
-- Create extension
CREATE EXTENSION IF NOT EXISTS vector;

-- Verify installation
SELECT * FROM pg_extension WHERE extname = 'vector';

-- Test vector operations
SELECT '[1,2,3]'::vector;
```

---

## Next Steps

Once pgvector is installed:

1. Run setup script:
   ```powershell
   .venv\Scripts\python.exe setup_pgvector.py
   ```

2. Test document upload via Postman:
   - POST http://localhost:8000/upload/
   - Upload a PDF file

3. Query your documents:
   - POST http://localhost:8000/query/
   - Body: `{"query": "your question here"}`

---

## Support Links

- pgvector GitHub: https://github.com/pgvector/pgvector
- pgvector Releases: https://github.com/pgvector/pgvector/releases
- PostgreSQL Windows Download: https://www.postgresql.org/download/windows/
