# 01. File Handling Mastery: Uploads, Downloads, and Security

## 1. Receiving Files: `UploadFile` vs `bytes`

FastAPI gives you two ways to receive files.

### The "Quick & Dirty" Way: `bytes`
Reads the *entire* file into memory.
- **Pros**: Simple.
- **Cons**: Will crash your server if someone uploads a 5GB video.

### The "Professional" Way: `UploadFile`
Uses Python's `SpooledTemporaryFile`. It keeps small files in memory but writes large files to disk automatically.
- **Pros**: Safe, efficient, exposes metadata (filename, content_type).

```python
from fastapi import FastAPI, UploadFile, File

app = FastAPI()

@app.post("/upload")
async def upload_file(file: UploadFile = File(...)):
    # file.file is a file-like object
    contents = await file.read() 
    
    return {
        "filename": file.filename,
        "content_type": file.content_type,
        "size": len(contents)
    }
```

---

## 2. Streaming Uploads (Saving to Disk)

Don't read the whole file into RAM (`await file.read()`) if you are just saving it to disk. Stream it chunk by chunk.

```python
import shutil
from pathlib import Path

UPLOAD_DIR = Path("uploads")
UPLOAD_DIR.mkdir(exist_ok=True)

@app.post("/save-file")
async def save_file(file: UploadFile = File(...)):
    destination = UPLOAD_DIR / file.filename
    
    # DANGER: See Security Section below about file.filename!
    
    with destination.open("wb") as buffer:
        shutil.copyfileobj(file.file, buffer)
        
    return {"path": str(destination)}
```

---

## 3. Security: The "Zip Slip" & Path Traversal

**NEVER TRUST USER INPUT.** especially filenames.

If a hacker uploads a file named `../../../../windows/system32/hack.exe`, and you blindly join it with your upload dir, you might overwrite critical system files.

### The Fix: Sanitize Filenames
Always generate your own safe filename (e.g., UUID) or strictly sanitize the input.

```python
import uuid

@app.post("/safe-upload")
async def safe_upload(file: UploadFile = File(...)):
    # Ignore the user's filename. Make a new one.
    extension = file.filename.split(".")[-1]
    new_filename = f"{uuid.uuid4()}.{extension}"
    
    destination = UPLOAD_DIR / new_filename
    
    with destination.open("wb") as buffer:
        shutil.copyfileobj(file.file, buffer)
        
    return {"id": new_filename}
```

---

## 4. Serving Files (Downloads)

FastAPI provides `FileResponse` and `StreamingResponse`.

### `FileResponse`
Best for files that exist on disk. It handles `Content-Type`, `Content-Length`, and `Last-Modified` headers automatically.

```python
from fastapi.responses import FileResponse

@app.get("/download/{file_id}")
async def download_file(file_id: str):
    file_path = UPLOAD_DIR / file_id
    if not file_path.exists():
        return {"error": "File not found"}
        
    return FileResponse(path=file_path, filename=f"download_{file_id}")
```

### `StreamingResponse`
Best for:
- Large files (videos).
- Files generated on-the-fly (CSV exports).
- Proxying files from S3 without saving to local disk.

```python
from fastapi.responses import StreamingResponse
import io

@app.get("/generate-csv")
async def generate_csv():
    # Generator function yields chunks of data
    def iter_csv():
        yield "id,name,email\n"
        for i in range(1000):
            yield f"{i},User{i},user{i}@example.com\n"
            
    return StreamingResponse(iter_csv(), media_type="text/csv")
```

---

## 5. Inductive Example: A Secure File Server

We want a system where:
1.  Users upload images.
2.  We validate they are *actually* images (Magic Numbers).
3.  We save them safely.

```python
import filetype # pip install filetype

@app.post("/upload-image")
async def upload_image(file: UploadFile = File(...)):
    # 1. Read first 2KB to check file signature
    head = await file.read(2048)
    kind = filetype.guess(head)
    
    if kind is None or not kind.mime.startswith("image/"):
        return {"error": "Invalid image type"}
        
    # Reset cursor to start
    await file.seek(0)
    
    # 2. Save safely
    filename = f"{uuid.uuid4()}.{kind.extension}"
    path = UPLOAD_DIR / filename
    
    with path.open("wb") as buffer:
        shutil.copyfileobj(file.file, buffer)
        
    return {"url": f"/static/{filename}"}
```
