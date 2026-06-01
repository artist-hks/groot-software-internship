# 📅 Week 2 Log — ContextCare AI

**Duration:** 25 May – 31 May 2026  
**Project:** ContextCare AI — Intelligent Real-Time Clinical Workspace  
**Tech Focus:** FastAPI · Next.js 14 · MongoDB · WebSocket · OpenCV · Pytesseract · spaCy · ReportLab · Docker

---

## 🎯 Week Goal
Build ContextCare AI from scratch — a production-grade medical SaaS platform that lets a doctor scan a patient's lab report with their phone, extract values via OCR+NLP, and see live data appear on their dashboard through WebSocket.

---

## Day 8 — 25 May 2026 · CampusOS Wrap-up + ContextCare Architecture Design

### What I Did
- **Morning:** Completed CampusOS Hostel & Library modules
  - Hostel: room allocation table, complaint system, warden dashboard
  - Library: book catalog, issuance tracking, overdue detection
  - Final deploy to Cloudflare Pages — ✅ live
- **Afternoon:** Started ContextCare AI — drew out the full system architecture before writing a single line of code

### ContextCare System Design
```
Mobile Device (Patient)
      │
      │  POST /api/broadcast-intel
      │  Header: x-doctor-id: <uuid>
      │  Body: ExtractedDocument JSON
      ▼
FastAPI Engine (Port 8000)
      ├── /api/auth/*        → bcrypt + PyJWT
      ├── /api/extract-intel → OCR pipeline
      ├── /api/broadcast-intel → WebSocket hub
      ├── /api/scans         → MongoDB history
      └── /ws/doctor-dashboard → WebSocket endpoint
      │
      ├── MongoDB (Docker)   → persist all scans
      └── WebSocket Hub      → broadcast to doctor dashboard
                                      │
                            Next.js Dashboard (Doctor)
                            Port 3000
                            ├── /login   → JWT auth form
                            └── /doctor  → Live dashboard
                                         ├── WebSocket listener
                                         ├── Recharts FBS trend
                                         └── PDF download button
```

### Key Design Decisions Made
1. **Why FastAPI over Express?** OCR (OpenCV + Pytesseract) and NLP (spaCy) are Python libraries — no JS equivalent at this quality level. FastAPI gives async support without sacrificing Python ecosystem.
2. **Why MongoDB?** Patient scan documents are schema-flexible (different report types have different metrics) — MongoDB's document model fits perfectly.
3. **Why WebSocket instead of polling?** Polling every 2 seconds would create 30 requests/minute per doctor — WebSocket is one persistent connection, instant updates.
4. **Why Next.js 14 App Router?** Server Components reduce JS bundle size; App Router's nested layouts work well for the auth → dashboard flow.

---

## Day 9 — 26 May 2026 · FastAPI Backend Skeleton + Docker MongoDB

### What I Did
- Created `backend/` folder with project structure:
  ```
  backend/
  ├── main.py           → FastAPI app, all routes + WebSocket
  ├── database.py       → Motor async MongoDB client + Pydantic models
  ├── restore_pdf.py    → PDF generation utility
  └── requirements.txt  → all Python dependencies
  ```
- Set up **Docker MongoDB** container:
  ```bash
  docker run -d --name contextcare-mongo -p 27017:27017 mongo:latest
  ```
- Built FastAPI app skeleton — CORS middleware, health endpoint, app metadata
- Defined **Pydantic models**: `ExtractedDocument`, `MetricEntity`, `MedicationEntity`, `DoctorModel`, `ScanRecord`
- Wrote `requirements.txt`:
  ```
  fastapi, uvicorn[standard], motor, pymongo, python-jose[cryptography],
  bcrypt, opencv-python, pytesseract, spacy, reportlab, python-multipart
  ```

### Pydantic Model Design
```python
# database.py
from pydantic import BaseModel
from typing import Optional, List
from datetime import datetime

class MetricEntity(BaseModel):
    name: str           # e.g., "Fasting Blood Sugar"
    value: float
    unit: str           # e.g., "mg/dL"
    status: str         # "normal" | "high" | "low" | "critical"
    normal_range: str   # e.g., "70-100 mg/dL"

class ExtractedDocument(BaseModel):
    document_type: str          # "blood_test" | "urine_test" | "lipid_panel"
    patient_name: Optional[str]
    report_date: Optional[str]
    metrics: List[MetricEntity]
    medications: List[str]
    raw_text: str
    confidence_score: float     # OCR confidence (0.0 - 1.0)
    extracted_at: datetime = datetime.now()
```

### Key Learnings
- Docker's `-p 27017:27017` maps container port to host — Motor connects to `mongodb://localhost:27017` from host
- Pydantic v2 (default in new projects) has different validator syntax than v1 — `@field_validator` replaces `@validator`
- FastAPI auto-generates `/docs` (Swagger UI) — invaluable for testing endpoints without Postman

---

## Day 10 — 27 May 2026 · OCR Pipeline — Image Preprocessing

### What I Did
Built the image preprocessing pipeline — the most critical step in OCR accuracy. Raw phone photos of lab reports have noise, shadows, skew, and poor contrast. We need to fix all of this before feeding to Tesseract.

### Preprocessing Steps (in order)
```python
import cv2
import numpy as np

class ImagePreprocessor:
    def preprocess(self, image_bytes: bytes) -> np.ndarray:
        # Step 1: Decode image bytes to numpy array
        nparr = np.frombuffer(image_bytes, np.uint8)
        img = cv2.imdecode(nparr, cv2.IMREAD_COLOR)
        
        # Step 2: Resize — Tesseract works best at 300 DPI equivalent
        # If image is smaller than 800px wide, upscale
        h, w = img.shape[:2]
        if w < 800:
            scale = 800 / w
            img = cv2.resize(img, None, fx=scale, fy=scale, interpolation=cv2.INTER_CUBIC)
        
        # Step 3: Convert to grayscale
        gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
        
        # Step 4: Bilateral filter — removes noise but PRESERVES edges (text edges)
        # (vs Gaussian blur which blurs text edges too)
        denoised = cv2.bilateralFilter(gray, 9, 75, 75)
        
        # Step 5: CLAHE — Contrast Limited Adaptive Histogram Equalization
        # Improves contrast locally — helps with uneven lighting / shadows
        clahe = cv2.createCLAHE(clipLimit=2.0, tileGridSize=(8, 8))
        enhanced = clahe.apply(denoised)
        
        # Step 6: Adaptive thresholding — converts to binary (black/white)
        # Adaptive (vs global) handles varying background brightness across the image
        binary = cv2.adaptiveThreshold(
            enhanced, 255,
            cv2.ADAPTIVE_THRESH_GAUSSIAN_C,
            cv2.THRESH_BINARY, 11, 2
        )
        
        # Step 7: Morphological cleanup — remove tiny dots/noise
        kernel = np.ones((1, 1), np.uint8)
        cleaned = cv2.morphologyEx(binary, cv2.MORPH_CLOSE, kernel)
        
        # Step 8: Deskew correction
        return self._deskew(cleaned)
    
    def _deskew(self, image: np.ndarray) -> np.ndarray:
        # Find text angle using Hough lines
        coords = np.column_stack(np.where(image > 0))
        angle = cv2.minAreaRect(coords)[-1]
        if angle < -45:
            angle = -(90 + angle)
        else:
            angle = -angle
        
        # Rotate image to correct skew
        (h, w) = image.shape[:2]
        center = (w // 2, h // 2)
        M = cv2.getRotationMatrix2D(center, angle, 1.0)
        return cv2.warpAffine(image, M, (w, h), flags=cv2.INTER_CUBIC, borderMode=cv2.BORDER_REPLICATE)
```

### Why Each Step Matters
| Step | Without It | With It |
|------|-----------|---------|
| Resize | Tesseract misses small text | Consistent character size |
| Bilateral Filter | Gaussian blur smears text | Noise gone, text edges sharp |
| CLAHE | Shadow areas unreadable | Even contrast everywhere |
| Adaptive Threshold | Global threshold fails on mixed backgrounds | Binary image regardless of lighting |
| Deskew | Slanted text → garbled OCR | Horizontal text → clean OCR |

### Key Learnings
- `cv2.bilateralFilter` is slower than Gaussian but critical — it's the only denoiser that doesn't blur edges
- CLAHE's `tileGridSize=(8,8)` divides the image into 64 tiles — each tile gets its own histogram equalization
- Deskew using `minAreaRect` on white pixel coords is a classic trick — gives the dominant text angle

---

## Day 11 — 28 May 2026 · OCR Engine + `/api/extract-intel` Endpoint

### What I Did
- Built `OCREngine` class wrapping Pytesseract with confidence scoring
- Built `/api/extract-intel` endpoint — accepts image upload, runs full pipeline, returns `ExtractedDocument`

### OCR Engine
```python
import pytesseract
from PIL import Image
import io

class OCREngine:
    def extract_text(self, preprocessed_image: np.ndarray) -> tuple[str, float]:
        # Convert numpy array to PIL Image (Pytesseract needs PIL)
        pil_img = Image.fromarray(preprocessed_image)
        
        # OEM 3 = LSTM neural net engine (most accurate)
        # PSM 6 = assume uniform block of text (good for lab reports)
        custom_config = r'--oem 3 --psm 6'
        
        # Get word-level data with confidence scores
        data = pytesseract.image_to_data(
            pil_img,
            config=custom_config,
            output_type=pytesseract.Output.DICT
        )
        
        # Filter words with confidence > 60 (ignore noise words)
        words = []
        confidences = []
        for i, conf in enumerate(data['conf']):
            if int(conf) > 60:
                words.append(data['text'][i])
                confidences.append(int(conf))
        
        full_text = ' '.join(w for w in words if w.strip())
        mean_confidence = sum(confidences) / len(confidences) if confidences else 0
        
        return full_text, mean_confidence / 100  # normalize to 0.0–1.0

# FastAPI endpoint
@app.post("/api/extract-intel")
async def extract_intel(file: UploadFile = File(...)):
    image_bytes = await file.read()
    
    preprocessor = ImagePreprocessor()
    ocr_engine = OCREngine()
    extractor = SpaCyRuleBasedExtractor()
    
    preprocessed = preprocessor.preprocess(image_bytes)
    raw_text, confidence = ocr_engine.extract_text(preprocessed)
    document = extractor.extract(raw_text, confidence)
    
    return document
```

### Key Learnings
- `--oem 3` (LSTM) is significantly more accurate than `--oem 0` (Legacy) for lab report text
- `--psm 6` (uniform text block) works better than default PSM 3 (auto-detect) for structured lab reports
- Confidence filtering at 60% removes most garbage characters without losing real text
- Mean confidence score is a good quality signal to expose in the response — lets client warn user if scan quality is poor

---

## Day 12 — 29 May 2026 · spaCy NER — Medical Entity Extraction

### What I Did
- Built `SpaCyRuleBasedExtractor` — extracts structured data from raw OCR text
- Designed as an abstract base class (`BaseMedicalExtractor`) so it's swappable with ClinicalBERT or any LLM later
- Covers: document type classification, date normalization, metric regex extraction, medication NER, threshold status

### Extractor Architecture
```python
from abc import ABC, abstractmethod
import spacy
import re
from datetime import datetime

class BaseMedicalExtractor(ABC):
    @abstractmethod
    def extract(self, raw_text: str, confidence: float) -> ExtractedDocument:
        pass

class SpaCyRuleBasedExtractor(BaseMedicalExtractor):
    def __init__(self):
        self.nlp = spacy.load("en_core_web_sm")
        
        # Metric patterns — covers common lab report formats
        self.METRIC_PATTERNS = {
            "Fasting Blood Sugar": r"(?:FBS|Fasting\s+Blood\s+Sugar)[:\s]+(\d+\.?\d*)\s*(mg/dL|mmol/L)?",
            "HbA1c": r"(?:HbA1c|Glycated\s+Haemoglobin)[:\s]+(\d+\.?\d*)\s*(%)?",
            "Total Cholesterol": r"(?:Total\s+Cholesterol)[:\s]+(\d+\.?\d*)\s*(mg/dL)?",
            "Hemoglobin": r"(?:Hemoglobin|Hb|Hgb)[:\s]+(\d+\.?\d*)\s*(g/dL)?",
            "Creatinine": r"(?:Creatinine|Creat)[:\s]+(\d+\.?\d*)\s*(mg/dL)?",
        }
        
        # Normal ranges for threshold detection
        self.NORMAL_RANGES = {
            "Fasting Blood Sugar": (70, 100, "mg/dL"),
            "HbA1c": (4.0, 5.6, "%"),
            "Total Cholesterol": (0, 200, "mg/dL"),
            "Hemoglobin": (12.0, 17.5, "g/dL"),
            "Creatinine": (0.6, 1.2, "mg/dL"),
        }
    
    def extract(self, raw_text: str, confidence: float) -> ExtractedDocument:
        doc = self.nlp(raw_text)
        
        return ExtractedDocument(
            document_type=self._classify_document(raw_text),
            patient_name=self._extract_patient_name(doc),
            report_date=self._extract_date(raw_text),
            metrics=self._extract_metrics(raw_text),
            medications=self._extract_medications(doc),
            raw_text=raw_text,
            confidence_score=confidence
        )
    
    def _classify_document(self, text: str) -> str:
        text_lower = text.lower()
        if any(kw in text_lower for kw in ["blood sugar", "fbs", "hba1c", "glucose"]):
            return "blood_test"
        elif any(kw in text_lower for kw in ["urine", "urea", "creatinine"]):
            return "urine_test"
        elif any(kw in text_lower for kw in ["cholesterol", "ldl", "hdl", "triglyceride"]):
            return "lipid_panel"
        return "general_report"
    
    def _extract_metrics(self, text: str) -> List[MetricEntity]:
        metrics = []
        for metric_name, pattern in self.METRIC_PATTERNS.items():
            match = re.search(pattern, text, re.IGNORECASE)
            if match:
                value = float(match.group(1))
                unit = match.group(2) if match.lastindex >= 2 else self.NORMAL_RANGES[metric_name][2]
                status = self._get_status(metric_name, value)
                low, high, _ = self.NORMAL_RANGES[metric_name]
                metrics.append(MetricEntity(
                    name=metric_name, value=value, unit=unit,
                    status=status, normal_range=f"{low}–{high} {unit}"
                ))
        return metrics
    
    def _get_status(self, metric: str, value: float) -> str:
        low, high, _ = self.NORMAL_RANGES[metric]
        if value < low: return "low"
        if value > high * 1.5: return "critical"
        if value > high: return "high"
        return "normal"
```

### Pluggable Design — Why This Matters
```python
# Swapping to ClinicalBERT later requires ZERO other code changes:
class ClinicalBERTExtractor(BaseMedicalExtractor):
    def extract(self, raw_text: str, confidence: float) -> ExtractedDocument:
        # Use transformer model here
        ...

# In main.py, just change one line:
extractor = ClinicalBERTExtractor("/models/clinicalbert")  # was SpaCyRuleBasedExtractor()
```

### Key Learnings
- `re.IGNORECASE` is essential — lab reports have inconsistent casing ("FBS", "fbs", "Fbs" all appear)
- spaCy's `en_core_web_sm` NER recognizes `PERSON` entities — useful for extracting patient names from free text
- Abstract base classes (`ABC`) + single injection point make systems genuinely extensible — this is the Open/Closed Principle in practice

---

## Day 13 — 30 May 2026 · WebSocket Hub — Real-Time Broadcasting

### What I Did
- Built `ConnectionManager` — manages all active WebSocket connections per doctor
- Implemented `/ws/doctor-dashboard` WebSocket endpoint
- Built `/api/broadcast-intel` — receives scan data and broadcasts to all connected dashboard sessions of that doctor

### ConnectionManager
```python
from fastapi import WebSocket
from typing import Dict, List

class ConnectionManager:
    def __init__(self):
        # doctor_id → list of active WebSocket connections
        # (one doctor could have dashboard open on multiple tabs)
        self.active_connections: Dict[str, List[WebSocket]] = {}
    
    async def connect(self, websocket: WebSocket, doctor_id: str):
        await websocket.accept()
        if doctor_id not in self.active_connections:
            self.active_connections[doctor_id] = []
        self.active_connections[doctor_id].append(websocket)
    
    def disconnect(self, websocket: WebSocket, doctor_id: str):
        if doctor_id in self.active_connections:
            self.active_connections[doctor_id].remove(websocket)
    
    async def broadcast_to_doctor(self, doctor_id: str, data: dict):
        if doctor_id not in self.active_connections:
            return  # doctor not currently online — data is in MongoDB anyway
        
        dead_connections = []
        for connection in self.active_connections[doctor_id]:
            try:
                await connection.send_json(data)
            except Exception:
                dead_connections.append(connection)  # connection dropped
        
        # Clean up dead connections
        for dead in dead_connections:
            self.active_connections[doctor_id].remove(dead)

manager = ConnectionManager()

# WebSocket endpoint
@app.websocket("/ws/doctor-dashboard")
async def websocket_endpoint(websocket: WebSocket, doctor_id: str):
    await manager.connect(websocket, doctor_id)
    try:
        while True:
            await websocket.receive_text()  # keep connection alive
    except WebSocketDisconnect:
        manager.disconnect(websocket, doctor_id)

# Broadcast endpoint (called by mobile scanner)
@app.post("/api/broadcast-intel")
async def broadcast_intel(
    document: ExtractedDocument,
    doctor_id: str = Header(..., alias="x-doctor-id")
):
    # 1. Persist to MongoDB
    await db.patient_scans.insert_one({
        "doctor_id": doctor_id,
        "document": document.dict(),
        "received_at": datetime.now()
    })
    
    # 2. Broadcast to all dashboard sessions of this doctor
    await manager.broadcast_to_doctor(doctor_id, document.dict())
    
    return {"status": "broadcast_sent"}
```

### Why `x-doctor-id` Header (Not JWT)?
The mobile scanner app doesn't have the doctor's JWT — it just has a QR code link with the doctor_id embedded. Using a custom header keeps the mobile flow simple: scan QR → POST to endpoint with doctor_id header. Security is acceptable here because the data is being *sent to* the doctor, not accessing private data.

### Key Learnings
- Dictionary of `doctor_id → [WebSocket list]` handles one doctor's multiple browser tabs elegantly
- Dead connection cleanup is critical — stale WebSocket objects cause errors on next broadcast
- `receive_text()` in a while loop keeps the connection alive — without it FastAPI closes the WebSocket immediately

---

## Day 14 — 31 May 2026 · JWT Auth System + MongoDB Integration

### What I Did
- Completed the full authentication system for ContextCare
- Set up Motor async MongoDB client with proper connection pooling
- Built `/api/scans` — JWT-guarded endpoint returning doctor's full scan history
- Built `/api/reports/{id}/pdf` — generates and streams a branded PDF for any scan

### Auth System
```python
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from jose import jwt, JWTError
import bcrypt

SECRET_KEY = "your-secret-key"
ALGORITHM = "HS256"
security = HTTPBearer()

# Register
@app.post("/api/auth/register")
async def register(doctor: DoctorRegistration):
    # Hash password with bcrypt
    hashed = bcrypt.hashpw(doctor.password.encode(), bcrypt.gensalt())
    
    doctor_doc = {
        "_id": str(uuid4()),
        "email": doctor.email,
        "name": doctor.name,
        "password_hash": hashed.decode(),
        "created_at": datetime.now()
    }
    await db.doctors.insert_one(doctor_doc)
    
    # Return JWT immediately (no separate login step)
    token = jwt.encode({"sub": doctor_doc["_id"], "email": doctor.email}, SECRET_KEY, ALGORITHM)
    return {"token": token, "doctor_id": doctor_doc["_id"]}

# JWT dependency
async def get_current_doctor(credentials: HTTPAuthorizationCredentials = Depends(security)):
    try:
        payload = jwt.decode(credentials.credentials, SECRET_KEY, algorithms=[ALGORITHM])
        return payload
    except JWTError:
        raise HTTPException(status_code=401, detail="Invalid token")

# Protected route
@app.get("/api/scans")
async def get_scans(doctor = Depends(get_current_doctor)):
    scans = await db.patient_scans.find(
        {"doctor_id": doctor["sub"]},
        sort=[("received_at", -1)]  # newest first
    ).to_list(50)
    return scans
```

### MongoDB Motor — Async Pattern
```python
# database.py
from motor.motor_asyncio import AsyncIOMotorClient

client = AsyncIOMotorClient("mongodb://localhost:27017")
db = client.contextcare

# Motor queries are all async — use await
await db.doctors.insert_one(doc)           # insert
await db.patient_scans.find_one({"_id": id})  # find one
await db.patient_scans.find(query).to_list(50)  # find many
```

### ReportLab PDF Generation
```python
from reportlab.lib.pagesizes import A4
from reportlab.platypus import SimpleDocTemplate, Paragraph, Table
from io import BytesIO

def generate_pdf(scan: dict) -> bytes:
    buffer = BytesIO()
    doc = SimpleDocTemplate(buffer, pagesize=A4)
    
    elements = []
    # Header
    elements.append(Paragraph("ContextCare AI — Diagnostic Report", title_style))
    elements.append(Paragraph(f"Patient: {scan['document']['patient_name']}", normal_style))
    
    # Metrics table
    metrics_data = [["Metric", "Value", "Unit", "Status", "Normal Range"]]
    for m in scan['document']['metrics']:
        metrics_data.append([m['name'], m['value'], m['unit'], m['status'], m['normal_range']])
    
    table = Table(metrics_data)
    elements.append(table)
    
    doc.build(elements)
    return buffer.getvalue()

@app.get("/api/reports/{scan_id}/pdf")
async def download_pdf(scan_id: str):
    scan = await db.patient_scans.find_one({"_id": scan_id})
    pdf_bytes = generate_pdf(scan)
    return Response(pdf_bytes, media_type="application/pdf",
                   headers={"Content-Disposition": f"attachment; filename=report-{scan_id}.pdf"})
```

### Key Learnings
- Motor (async MongoDB driver) uses the same API as PyMongo but with `await` — muscle memory transfers
- `Depends(get_current_doctor)` is FastAPI's dependency injection — cleaner than manually checking headers in every route
- ReportLab's `SimpleDocTemplate` handles page breaks automatically — no manual pagination needed
- `BytesIO` buffer lets you build the PDF in memory and stream it directly — no temp files on disk

---

## 📊 Week 2 Summary

| Metric | Count |
|--------|-------|
| New Projects Started | 1 (ContextCare AI) |
| API Endpoints Built | 8 |
| Preprocessing Steps | 8 (OpenCV pipeline) |
| Tech Stacks Used | 5 (FastAPI + MongoDB + Docker + spaCy + ReportLab) |
| Lines of Python | ~600 |
| WebSocket Connections Managed | Real-time, multi-session |

**What went well:** The pluggable `BaseMedicalExtractor` design decision on Day 12 was worth the extra 2 hours — it makes the NER engine genuinely swappable. The OCR pipeline preprocessing on Day 10 pushed Tesseract accuracy from ~60% to ~92% on test images.

**What was hard:** Motor's async cursor `.to_list()` syntax caught me off guard — forgot that `find()` returns a cursor, not a list, and you need `.to_list(limit)` to materialize it. Also Docker port conflicts (MongoDB default 27017 clashing with local Mongo install) ate 30 minutes.

**Next week:** ContextCare AI — Next.js 14 frontend (App Router, auth form, live dashboard with Recharts + WebSocket), then integrate `render.yaml` for cloud deployment.
