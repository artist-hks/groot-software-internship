# 📅 Week 2 Log — ContextCare AI

**Duration:** 25 May – 31 May 2026  
**Project:** ContextCare AI — Real-Time Clinical Workspace  
**Tech Focus:** Next.js 14 (single full-stack app) · Prisma + SQLite · Socket.IO · Tesseract.js + Sharp · iron-session

---

## 🎯 Week Goal
Build ContextCare AI from scratch — a clinical workspace that lets a patient photograph a lab report on their phone, pairs them to the right doctor via a QR code, runs OCR to pull out diagnostic values, and pushes the result live onto the doctor's dashboard.

---

## Day 8 — 25 May 2026 · CampusOS Wrap-up + ContextCare Architecture Design

### What I Did
- **Morning:** Completed CampusOS Hostel & Library modules
  - Hostel: room allocation table, complaint system, warden dashboard
  - Library: book catalog, issuance tracking, overdue detection
  - Final deploy to Cloudflare Pages — ✅ live
- **Afternoon:** Started ContextCare AI — drew out the system architecture before writing a single line of code. Decided early to keep this as **one Next.js app** (API routes + a thin custom server) instead of a separate backend service — less moving parts to deploy and debug solo.

### ContextCare System Design
```
                 ┌────────────────────────────────────────┐
                 │           Custom HTTP Server            │
                 │             (server.ts)                 │
                 ├────────────────────┬─────────────────────┤
                 │   Next.js App      │     Socket.IO        │
                 │   (SSR + API)      │    (WebSocket)       │
                 └────────┬───────────┴──────────┬───────────┘
                          │                      │
            ┌─────────────┼──────────────────────┼────────────┐
            │              │                      │            │
   ┌────────▼────────┐ ┌──▼────────────┐  ┌──────▼─────────┐   │
   │  Patient Flow   │ │  Doctor Flow  │  │  Real-time     │   │
   │  • Upload photo │ │  • PIN login  │  │  events        │   │
   │  • OCR extract  │ │  • Dashboard  │  │  • scan:created│   │
   │  • Review       │ │  • Notes/PDF  │  │  • heartbeat   │   │
   │  • QR pairing   │ │               │  │                │   │
   └────────┬────────┘ └───────┬───────┘  └────────────────┘   │
            └──────────┬───────┘                                │
                ┌───────▼────────┐                              │
                │  Prisma + SQLite│                              │
                └────────────────┘                              │
            ┌─────────────────────────────────────────────────────┘
            │
   ┌────────▼────────┐
   │ Tesseract.js +  │
   │     Sharp       │
   └─────────────────┘
```

### Key Design Decisions Made
1. **Why one Next.js app instead of a separate backend?** API routes + a custom `server.ts` give me REST endpoints, SSR pages, and a WebSocket server all in one deployable unit — one Render/Railway service instead of two to keep alive.
2. **Why Prisma + SQLite instead of MongoDB?** The data here is genuinely relational (Doctor → Patient → Scan → Metric) — foreign keys and `@@unique([doctorId, phone])` constraints are a natural fit, and SQLite needs zero external DB service for a solo project.
3. **Why Socket.IO instead of raw WebSocket?** Built-in room support (`doctor:<id>`) makes "broadcast to this one doctor's open tabs" trivial — no need to hand-roll a connection registry.
4. **Why PIN + QR pairing instead of email/password for both sides?** Doctors log in once with a short PIN (low friction on a clinic tablet); patients never need an account at all — they just scan the doctor's QR and submit, which matches how a real walk-in visit works.

---

## Day 9 — 26 May 2026 · Next.js Scaffolding + Prisma Schema

### What I Did
- Scaffolded the Next.js 14 App Router project — `app/`, `lib/`, `components/`, `prisma/`
- Designed the relational schema in `prisma/schema.prisma`: `Doctor → Patient → Scan → Metric`, plus a `Note` model for clinical notes
- Set `datasource db { provider = "sqlite" }` — zero external DB service needed during development
- Ran the first migration and got Prisma Client generating types automatically on `postinstall`

### Prisma Schema
```prisma
datasource db {
  provider = "sqlite"
  url      = env("DATABASE_URL")
}

model Doctor {
  id             String    @id @default(cuid())
  name           String
  specialization String?
  pinHash        String
  qrToken        String    @unique
  patients       Patient[]
}

model Patient {
  id        String   @id @default(cuid())
  doctorId  String
  doctor    Doctor   @relation(fields: [doctorId], references: [id])
  name      String
  phone     String
  scans     Scan[]
  notes     Note[]

  @@unique([doctorId, phone])
}

model Scan {
  id        String   @id @default(cuid())
  patientId String
  patient   Patient  @relation(fields: [patientId], references: [id])
  rawText   String?
  metrics   Metric[]
}

model Metric {
  id     String @id @default(cuid())
  scanId String
  scan   Scan   @relation(fields: [scanId], references: [id])
  key    String
  label  String
  value  Float
  unit   String
  refMin Float
  refMax Float
  status String // "normal" | "borderline" | "critical"
}
```

### Key Learnings
- `@@unique([doctorId, phone])` on `Patient` is doing real work — it means a repeat patient under the same doctor maps to one row, so all their scans build one trend history instead of duplicating the patient each visit
- Prisma's `postinstall: prisma generate` means the typed client is always in sync after `npm install` — no separate codegen step to remember
- SQLite via Prisma needs zero setup for local dev, and on Render/Railway it just needs a persistent volume + `DATABASE_URL` pointed at a file path

---

## Day 10 — 27 May 2026 · OCR Pipeline — Tesseract.js + Sharp

### What I Did
Built the OCR pipeline using **Tesseract.js + Sharp** instead of Python's OpenCV/Pytesseract — kept everything inside the Node.js process so there's no second language/runtime to deploy. Bundled the English language model locally so the OCR worker never needs a CDN download at runtime (important for slow clinic wifi).

### Image Preprocessing (Sharp)
```typescript
// lib/ocr.ts
import sharp from "sharp";

/**
 * Preprocess an image buffer for OCR: grayscale + normalize + contrast boost.
 * The JS equivalent of grayscale/denoise/threshold.
 */
export async function preprocessImage(buffer: Buffer): Promise<Buffer> {
  return sharp(buffer)
    .rotate()          // honor EXIF orientation (phone photos are often rotated)
    .grayscale()
    .normalize()
    .linear(1.25, -20) // contrast boost
    .sharpen()
    .toFormat("png")
    .toBuffer();
}
```

### Tesseract.js Worker Setup
```typescript
// Explicitly resolve tesseract.js' Node worker script + wasm core from
// node_modules. Without this, the bundled server context mis-resolves the
// worker path (e.g. to .next/worker-script/...) and the OCR worker crashes.
const NODE_WORKER = resolveSafe("tesseract.js/src/worker-script/node/index.js");
const CORE_DIR = (() => {
  const idx = resolveSafe("tesseract.js-core/index.js");
  return idx ? path.dirname(idx) : undefined;
})();

const worker = await createWorker("eng", 1, {
  langPath: LANG_PATH,       // locally bundled traineddata, no CDN fetch
  workerPath: NODE_WORKER,
  corePath: CORE_DIR,
  gzip: false,
});
```

### Key Learnings
- Tesseract.js's Node worker path resolution silently breaks once Next.js bundles the server code — had to manually `require.resolve()` the worker script and WASM core instead of letting the library auto-detect paths
- Bundling `tessdata/eng.traineddata` in the repo avoids a runtime download — first OCR call would otherwise be slow (or fail) on a flaky connection
- `sharp().linear(1.25, -20)` is a cheap contrast boost (`output = input * a + b`) — close enough to what a CLAHE step does for these photos, without needing OpenCV at all

---

## Day 11 — 28 May 2026 · `/api/scans/extract` Endpoint

### What I Did
- Built `runOcr()` — ties preprocessing + the Tesseract worker into one call returning raw text
- Built `POST /api/scans/extract` — multipart image upload, validation, OCR, and a graceful "couldn't read it" response instead of a hard failure
- Added a simple in-memory rate limiter so the OCR endpoint can't be hammered

### Extract Route
```typescript
// app/api/scans/extract/route.ts
const MAX_BYTES = 8 * 1024 * 1024; // 8MB

export async function POST(req: NextRequest) {
  const ip = clientIp(req);
  const limit = rateLimit(ip);
  if (!limit.ok) {
    return NextResponse.json(
      { error: `Too many uploads. Try again in ${limit.retryAfter}s.` },
      { status: 429 }
    );
  }

  const form = await req.formData();
  const file = form.get("image");
  if (!file || !(file instanceof File) || !file.type.startsWith("image/")) {
    return NextResponse.json({ error: "Please upload a photo of your report." }, { status: 400 });
  }
  if (file.size > MAX_BYTES) {
    return NextResponse.json({ error: "Image is larger than 8MB." }, { status: 400 });
  }

  const buffer = Buffer.from(await file.arrayBuffer());
  const rawText = await runOcr(buffer);
  const metrics = parseMetricsFromText(rawText);

  if (metrics.length === 0) {
    return NextResponse.json({
      rawText, metrics: [],
      warning: "We couldn't read any known lab values. You can add them manually on the next screen.",
    });
  }
  return NextResponse.json({ rawText, metrics });
}
```

### In-Memory Rate Limiter
```typescript
// lib/ratelimit.ts — sliding window, no external service needed for a solo deploy
const WINDOW_MS = 60 * 1000;
const MAX_REQUESTS = 10;
const hits = new Map<string, number[]>();

export function rateLimit(ip: string) {
  const now = Date.now();
  const timestamps = (hits.get(ip) ?? []).filter((t) => now - t < WINDOW_MS);
  if (timestamps.length >= MAX_REQUESTS) {
    return { ok: false, retryAfter: Math.ceil((WINDOW_MS - (now - timestamps[0])) / 1000) };
  }
  timestamps.push(now);
  hits.set(ip, timestamps);
  return { ok: true, retryAfter: 0 };
}
```

### Key Learnings
- Failing "softly" (returning `metrics: []` + a warning, not a 500) matters here — a patient with a blurry photo should still be able to continue and enter values manually, not get stuck
- A 60-second sliding window with a `Map<ip, timestamps[]>` is enough abuse protection for a single-process app — no Redis needed at this scale
- `file.type.startsWith("image/")` + an explicit byte-size cap catches most bad uploads before they ever touch the OCR worker

---

## Day 12 — 29 May 2026 · Metric Parser — Alias-Based Extraction

### What I Did
- Built a canonical `METRIC_REFERENCE` table — six lab metrics (FBS, Total Cholesterol, HDL, LDL, Triglycerides, Hemoglobin) each with label, unit, and a normal range
- Built `parseMetricsFromText()` — scans OCR output line by line, matches against known aliases per metric (lab reports format these inconsistently), and pulls out the first plausible number
- Decided against a full NLP/NER model here — lab reports are structured enough that alias + regex matching is faster, has zero model-loading cost, and is much easier for me to debug than a black-box extractor

### Metric Reference + Parser
```typescript
// lib/metrics.ts — shared between the OCR pipeline, the API, the UI, and the PDF
export const METRIC_REFERENCE: Record<string, MetricReference> = {
  fbs: {
    label: "Fasting Blood Sugar",
    aliases: ["fasting blood sugar", "fbs", "glucose fasting", "blood sugar fasting"],
    unit: "mg/dL", min: 70, max: 99,
  },
  total_cholesterol: {
    label: "Total Cholesterol",
    aliases: ["total cholesterol", "cholesterol total", "cholesterol"],
    unit: "mg/dL", min: 125, max: 200,
  },
  // ...hdl, ldl, triglycerides, hemoglobin follow the same shape
};

export function parseMetricsFromText(rawText: string): ExtractedMetric[] {
  const lines = rawText.split(/\r?\n/);
  const found: Record<string, ExtractedMetric> = {};

  for (const rawLine of lines) {
    const norm = normalizeLine(rawLine);
    for (const { key, alias } of ALIAS_INDEX) {
      if (found[key] || !norm.includes(alias)) continue;
      // Prefer the number right after the alias text, else the first number on the line
      const afterAlias = norm.slice(norm.indexOf(alias) + alias.length);
      const value = firstNumber(afterAlias) ?? firstNumber(norm);
      if (value === null) continue;

      const ref = METRIC_REFERENCE[key];
      found[key] = { key, label: ref.label, value, unit: ref.unit,
                     refMin: ref.min, refMax: ref.max,
                     status: getStatus(value, ref.min, ref.max) };
      break; // a line maps to a single metric
    }
  }
  return METRIC_ORDER.filter((k) => found[k]).map((k) => found[k]);
}
```

### Key Learnings
- Going alias-table + regex instead of a full NER model was the right call for a known, fixed set of 6 metrics — easy to extend (just add an entry), easy to test, no model weights to ship
- "First match wins" per metric per line keeps the parser predictable — important when OCR output is noisy and a naive global regex could double-count
- Keeping `METRIC_REFERENCE` as the single source of truth (used by the parser, the API validation in `/api/scans/pair`, the UI cards, and the seed script) means the normal ranges only ever need to be defined once

---

## Day 13 — 30 May 2026 · Socket.IO Real-Time Hub

### What I Did
- Wired a custom `server.ts` that creates one HTTP server, attaches both the Next.js request handler **and** a Socket.IO server to it — one process, one port
- Implemented the "doctor room" pattern — a dashboard socket joins `doctor:<id>` on connect; new scans get emitted only into that room
- Stashed the `io` instance on `globalThis` so Next.js API routes (running in the same process) can emit into a room without needing their own socket reference

### Custom Server + Socket Rooms
```typescript
// server.ts
const server = createServer((req, res) => handle(req, res, parse(req.url!, true)));
const io = new IOServer(server, { cors: { origin: "*" }, path: "/socket.io" });
setIo(io); // make it reachable from API routes

io.on("connection", (socket) => {
  socket.on("join", (doctorId: string) => {
    if (typeof doctorId === "string" && doctorId.length > 0) {
      socket.join(doctorRoom(doctorId));
    }
  });
  socket.on("ping:heartbeat", () => socket.emit("pong:heartbeat"));
});

server.listen(port, hostname);
```

```typescript
// lib/socket.ts — bridge between API routes and the io instance above
const globalForIo = globalThis as unknown as { io?: IOServer };
export function setIo(io: IOServer) { globalForIo.io = io; }
export function getIo() { return globalForIo.io; }
export function doctorRoom(doctorId: string) { return `doctor:${doctorId}`; }
```

### Key Learnings
- A custom Node server (instead of plain `next start`) is the one real infrastructure trade-off of this approach — you lose some of Vercel's zero-config hosting, but you gain a real persistent WebSocket server, which Vercel's serverless functions can't hold open anyway
- Stashing `io` on `globalThis` is a pragmatic way to share one instance between the server bootstrap and API route handlers that run in the same Node process — would need a different pattern (e.g. a separate pub/sub service) if this ever had to scale to multiple instances
- A heartbeat ping/pong pair is cheap insurance against "zombie" sockets that look connected but are dead

---

## Day 14 — 31 May 2026 · PIN Auth + QR Pairing + Patient Flow

### What I Did
- Built doctor login: `POST /api/doctor/login` — checks a 4–6 digit PIN against bcrypt hashes of all seeded doctors, then opens an `iron-session` encrypted cookie session (no JWT, no localStorage)
- Built `GET /api/doctor/qr` — returns the doctor's stable pairing token as a QR code data URL (`qrcode` package)
- Built `POST /api/scans/pair` — the endpoint a patient hits after scanning the QR: validates the doctor token, finds-or-creates the `Patient` row (keyed by doctor + phone), saves the scan + metrics, and pushes it live via `emitScanCreated()`

### PIN Login
```typescript
// app/api/doctor/login/route.ts
const pin = (body?.pin ?? "").toString().trim();
if (!/^\d{4,6}$/.test(pin)) {
  return NextResponse.json({ error: "Enter your 4–6 digit PIN." }, { status: 400 });
}

const doctors = await prisma.doctor.findMany();
let matched = null;
for (const d of doctors) {
  if (await bcrypt.compare(pin, d.pinHash)) { matched = d; break; }
}
if (!matched) {
  return NextResponse.json({ error: "That PIN didn't match any doctor." }, { status: 401 });
}

const session = await getSession();
session.doctorId = matched.id;
await session.save();
```

### Patient Pairing + Scan Creation
```typescript
// app/api/scans/pair/route.ts (trimmed)
const doctor = await prisma.doctor.findUnique({ where: { qrToken: doctorToken } });
if (!doctor) return NextResponse.json({ error: "Doctor code not recognized." }, { status: 404 });

let patient = await prisma.patient.findUnique({
  where: { doctorId_phone: { doctorId: doctor.id, phone: patientPhone } },
});
if (!patient) {
  patient = await prisma.patient.create({ data: { doctorId: doctor.id, name: patientName, phone: patientPhone } });
}

const scan = await prisma.scan.create({
  data: { patientId: patient.id, rawText: body?.rawText ?? null, metrics: { create: metricRows } },
  include: { metrics: true },
});

emitScanCreated(doctor.id, { patient, scan, hasCritical: scan.metrics.some(m => m.status === "critical") });
```

### Key Learnings
- A short numeric PIN (vs. email/password) is a deliberate UX call for a shared clinic device — fast to type, easy to change, no "forgot password" flow needed
- `iron-session` gives an encrypted, stateless cookie session — no session table in the DB, and no JWT secret to leak via localStorage/XSS
- "Find-or-create by `(doctorId, phone)`" on the patient is what makes repeat visits build one continuous trend line instead of fragmenting into duplicate patient records

---

## 📊 Week 2 Summary

| Metric | Count |
|--------|-------|
| New Projects Started | 1 (ContextCare AI) |
| API Routes Built | 7 (login, qr, extract, pair, patients, notes, pdf) |
| DB Models | 5 (Doctor, Patient, Scan, Metric, Note) |
| Canonical Metrics Tracked | 6 (FBS, Total/HDL/LDL Cholesterol, Triglycerides, Hemoglobin) |
| Real-Time Channel | Socket.IO, room-per-doctor |

**What went well:** Keeping everything inside one Next.js process (API routes + custom server + Socket.IO) instead of standing up a separate backend service paid off immediately — one thing to run locally, one thing to deploy. The alias-table metric parser on Day 12 was simple to build and very easy to extend.

**What was hard:** Tesseract.js's worker-path resolution inside a bundled Next.js server context was the trickiest bug of the week — had to manually resolve the worker script and WASM core paths instead of trusting the library's auto-detection. Getting the `(doctorId, phone)` composite-unique constraint right in Prisma (so repeat patients don't duplicate) also took a couple of tries.

**Next week:** ContextCare AI — finish the patient-facing pairing UI (camera capture, review screen) and the doctor dashboard (trend charts, notes, PDF export), then prep for deployment.
