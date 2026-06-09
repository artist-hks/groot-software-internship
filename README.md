<div align="center">

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=30&duration=3000&pause=1000&color=00D9FF&center=true&vCenter=true&width=700&lines=MERN+Stack+Internship+%F0%9F%9A%80;Groot+Software+%7C+Hemant+Sharma;18+May+2026+%E2%80%94+04+July+2026" alt="Typing SVG" />

</div>

<div align="center">

![Internship](https://img.shields.io/badge/Internship-Groot%20Software-00D9FF?style=for-the-badge&logo=rocket&logoColor=white)
![Stack](https://img.shields.io/badge/Stack-MERN-61DAFB?style=for-the-badge&logo=react&logoColor=black)
![Duration](https://img.shields.io/badge/Duration-45%20Days-FF6B6B?style=for-the-badge&logo=calendar&logoColor=white)
![Status](https://img.shields.io/badge/Status-Active%20%F0%9F%94%A5-4CAF50?style=for-the-badge)

</div>

---

## 🏢 About This Internship

| Field | Details |
|-------|---------|
| **Company** | [Groot Software](https://www.linkedin.com/company/grootsoftware) |
| **Intern** | Hemant Sharma (`@artist-hks`) |
| **Duration** | 18 May 2026 — 04 July 2026 (45 Days) |
| **Domain** | Full-Stack Web Development — MERN Stack |
| **Mode** | Offline / Project-Based / Training |
| **Status** | 🟢 Active |

> This repository serves as my **official internship work log**, documenting daily progress, learnings, code decisions, and deliverables. Individual project codebases are maintained in their own dedicated repositories.

---

## 🚀 Projects Built During Internship

### 1. 🏫 Colleqo — College Management Platform
> *Full-stack PWA for modern educational institutions*

[![Colleqo](https://img.shields.io/badge/Repo-Colleqo-0D1117?style=for-the-badge&logo=github)](https://github.com/artist-hks/Colleqo)
[![Live Demo](https://img.shields.io/badge/Live-campusos.pages.dev-4CAF50?style=for-the-badge&logo=cloudflare&logoColor=white)](https://campusos.pages.dev)
![TypeScript](https://img.shields.io/badge/TypeScript-50%25-3178C6?style=for-the-badge&logo=typescript&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-50%25-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)

**What I Built:**
- 📱 Progressive Web App (PWA) — installable on iOS, Android, Windows with offline support
- 🔐 Role-Based Access Control — Student, Faculty, HOD, Admin, Librarian, Hostel Warden dashboards
- 🏢 Multi-Tenant Architecture — multiple institutions under one deployment via tenant codes
- 📊 Modules: Attendance, Timetable, Exams, Assignments, Fees, Hostel, Library

**Tech Stack:** `Hono` · `Cloudflare D1 (SQLite)` · `Cloudflare Workers/Pages` · `Vite` · `TailwindCSS` · `Chart.js`

---

### 2. 🏥 ContextCare AI — Clinical Workspace Platform
> *Production-grade Medical SaaS with real-time OCR + WebSocket dashboard*

[![ContextCare](https://img.shields.io/badge/Repo-ContextCare-0D1117?style=for-the-badge&logo=github)](https://github.com/artist-hks/ContextCare)
[![Live Demo](https://img.shields.io/badge/Live-context--care.vercel.app-FF6B6B?style=for-the-badge&logo=vercel)](https://context-care.vercel.app)
![TypeScript](https://img.shields.io/badge/TypeScript-85.8%25-3178C6?style=for-the-badge&logo=typescript&logoColor=white)
![Python](https://img.shields.io/badge/Python-9.9%25-3776AB?style=for-the-badge&logo=python&logoColor=white)

**What I Built:**
- 🔬 OCR Pipeline — OpenCV preprocessing → Pytesseract → spaCy NER extraction
- ⚡ Real-Time WebSocket Hub — live patient data broadcasting to doctor dashboards
- 📄 PDF Report Generation — branded diagnostic PDFs via ReportLab
- 🔐 JWT Auth System — bcrypt password hashing, 24-hour token sessions
- 📈 Time-Series FBS Charts — Recharts trend visualizations in live dashboard

**Tech Stack:** `Next.js 14` · `FastAPI` · `MongoDB` · `Motor` · `Docker` · `Recharts` · `WebSocket` · `PyJWT` · `spaCy` · `OpenCV`

---

### 3. 💼 Portfolio (Ongoing Upgrade)
> *Personal developer portfolio — continuously improved throughout internship*

[![Portfolio](https://img.shields.io/badge/Repo-hks--portfolio-0D1117?style=for-the-badge&logo=github)](https://github.com/artist-hks/hks-portfolio)
[![Live](https://img.shields.io/badge/Live-artist--hks.vercel.app-00D9FF?style=for-the-badge&logo=vercel)](https://artist-hks.vercel.app)
![TypeScript](https://img.shields.io/badge/TypeScript-Primary-3178C6?style=for-the-badge&logo=typescript&logoColor=white)

**Upgrades During Internship:**
- Revamped UI/UX with new component architecture
- Added new project showcases from internship work
- Performance optimization and accessibility improvements

---

## 📅 Daily Work Log

> Updated daily. Each entry covers: what was built, what was learned, and any blockers.

---

### 📆 Week 1 — May 18–24, 2026

| Date | Day | Work Done | Key Learnings |
|------|-----|-----------|---------------|
| **18 May** | Day 1 | 🔰 Internship kickoff. Environment setup — Node.js, MongoDB, VS Code extensions. Understood project scope and requirements for Colleqo. Studied existing codebase structure. | MERN project structure, monorepo setup, understanding Hono framework for edge workers |
| **19 May** | Day 2 | ⚙️ Set up Cloudflare Workers + D1 database locally via Wrangler CLI. Initialized Vite build config with TailwindCSS. Created base project scaffolding for Colleqo. | Cloudflare D1 (serverless SQLite), Wrangler CLI workflow, edge computing basics |
| **20 May** | Day 3 | 🗄️ Designed DB schema for Colleqo — users, tenants, roles, attendance, timetable tables. Wrote SQL migrations. Set up role-based middleware logic. | Database schema design, multi-tenant architecture pattern, RBAC implementation |
| **21 May** | Day 4 | 🔐 Built authentication system — login/register endpoints with JWT, role extraction, session management. Integrated `tsconfig.json` for strict TypeScript. | JWT auth flow, TypeScript strict mode, Hono routing patterns |
| **22 May** | Day 5 | 📱 Started PWA implementation — added `manifest.json`, Service Worker registration, IndexedDB setup for offline queueing. | Progressive Web App fundamentals, Service Workers, offline-first design |
| **23 May** | Day 6 | 🎨 Built role-specific dashboard UIs — Student, Faculty, Admin views. Integrated Chart.js for attendance progress rings. | Chart.js integration, conditional rendering by role, responsive TailwindCSS grids |
| **24 May** | Day 7 | 🧪 Testing + bug fixes across Colleqo modules. Fixed offline sync logic — actions queued in IndexedDB now sync on reconnect. Added multi-tenant code system. | Debugging async sync issues, multi-tenant routing via tenant codes |

---

### 📆 Week 2 — May 25–31, 2026

| Date | Day | Work Done | Key Learnings |
|------|-----|-----------|---------------|
| **25 May** | Day 8 | ✅ Colleqo — completed Hostel & Library modules. Room allocation, book issuance, complaint system. Deployed to Cloudflare Pages (`npm run deploy`). | Cloudflare Pages deployment pipeline, Wrangler production config |
| **26 May** | Day 9 | 🏥 New project started: **ContextCare AI**. Designed full system architecture — FastAPI backend, Next.js 14 frontend, MongoDB, WebSocket hub. Studied OCR pipeline requirements. | System architecture design, FastAPI async patterns, clinical NLP basics |
| **27 May** | Day 10 | 🐍 Built FastAPI backend skeleton — project structure, CORS middleware, health endpoint, Pydantic models for `ExtractedDocument`. Set up Docker MongoDB container. | FastAPI project setup, Docker basics, Pydantic data validation |
| **28 May** | Day 11 | 🔬 Built OCR pipeline — OpenCV image preprocessor (grayscale, denoise, CLAHE, deskew) → Pytesseract OCR engine → confidence scoring. Integrated with `/api/extract-intel` endpoint. | Computer vision preprocessing, Tesseract OCR, image quality metrics |
| **29 May** | Day 12 | 🧠 Integrated spaCy NER for medical entity extraction — document classification, date normalization, metric regex, medication extraction. Built `BaseMedicalExtractor` abstract interface (pluggable design). | spaCy rule-based NER, abstract base classes in Python, medical document parsing |
| **30 May** | Day 13 | ⚡ Built WebSocket hub — `ConnectionManager` class broadcasting `ExtractedDocument` payloads to all connected doctor dashboards. Implemented `/ws/doctor-dashboard` endpoint. | WebSocket in FastAPI, broadcast patterns, real-time data streaming |
| **31 May** | Day 14 | 🔐 Completed auth system for ContextCare — bcrypt password hashing, PyJWT token generation, `HTTPBearer` FastAPI dependency injection. Built `/api/auth/register` and `/api/auth/login`. | Bcrypt salt rounds, JWT with expiry windows, FastAPI security dependencies |

---

### 📆 Week 3 — June 1 onwards *(in progress)*

| Date | Day | Work Done | Key Learnings |
|------|-----|-----------|---------------|
| **01 Jun** | Day 15 | 🎨 Built Next.js 14 frontend for ContextCare — App Router setup, dual-state auth form (login + register), route guards on `/doctor` dashboard, localStorage JWT validation. | Next.js App Router, client-side route protection, Tailwind utility composition |
| **02 Jun** | Day 16 | 🔔 Implemented FCM push notification system — backend FCM integration with graceful degradation, `push_subscriptions` DB table, `dispatchPush()` helper. Auto-triggers on new announcement. Frontend: notification bell with badge, push subscription on login. | FCM Web Push API, Service Worker push event handlers, fan-out notification patterns |
| **03 Jun** | Day 17 | 💳 Integrated Razorpay payment gateway into Colleqo fees module — create-order endpoint with HMAC-SHA256 signature verification, `payment_transactions` table, mock mode for demo. Student fee page: Pay Now button triggers Razorpay checkout modal, receipt generation. | Razorpay order flow, HMAC signature verification, payment state machines |
| **04 Jun** | Day 18 | 📱 Built Twilio SMS/WhatsApp alert system — `sendTwilioMessage()` with real `fetch()` to Twilio REST API, auto-triggers on low attendance (<75%) and fee payment confirmation. Alerts admin panel for bulk SMS. User alert preferences per student. | Twilio REST API, Basic Auth with `btoa()`, SMS vs WhatsApp channel routing |
| **05 Jun** | Day 19 | 📊 Built AI-powered analytics dashboard — rule-based insight engine analyzing attendance trends, at-risk students (risk score formula: `attendance×0.4 + exams×0.4 + fees×0.2`), department performance, fee collection trends. Chart.js line + bar charts. CSV export. | Analytics aggregation queries, risk scoring algorithms, Chart.js multi-dataset |
| **06 Jun** | Day 20 | 📂 Built bulk CSV import/export system + Parent Portal. CSV import: parse → validate → insert students/faculty with auto-generated passwords. Parent portal: 6-digit link code system, parent↔student linking, tabbed progress view (Attendance/Results/Fees/Timetable), printable progress report. | CSV parsing without external libs, parent-child DB relationships, print CSS |
| **07 Jun** | Day 21 | 🎨 Major UI upgrade — implemented professional desktop layout (Notion/Linear style): 240px fixed sidebar, 60px header with breadcrumb + user menu, role-based nav, mobile layout preserved via CSS media queries. Rebranded CampusOS → Colleqo. Built SaaS landing page with pricing (₹6,999/₹12,999/₹19,999/month), comparison vs TCS iON/Fedena, FAQ accordion. Deployed to campusos.pages.dev with GitHub Actions CI/CD. | CSS media queries for responsive desktop, Cloudflare Pages CI/CD, SaaS pricing strategy |

---

### 📆 Week 4 — June 8–14, 2026 *(planned)*

| Date | Day | Work Done | Key Learnings |
|------|-----|-----------|---------------|
| **08 Jun** | Day 22 | *(Upcoming)* | |
| **09 Jun** | Day 23 | *(Upcoming)* | |
| **10 Jun** | Day 24 | *(Upcoming)* | |
| **11 Jun** | Day 25 | *(Upcoming)* | |
| **12 Jun** | Day 26 | *(Upcoming)* | |
| **13 Jun** | Day 27 | *(Upcoming)* | |
| **14 Jun** | Day 28 | *(Upcoming)* | |

---

### 📆 Week 5 — June 15–21, 2026 *(planned)*

| Date | Day | Work Done | Key Learnings |
|------|-----|-----------|---------------|
| **15 Jun** | Day 29 | *(Upcoming)* | |
| **16 Jun** | Day 30 | *(Upcoming)* | |
| **17 Jun** | Day 31 | *(Upcoming)* | |
| **18 Jun** | Day 32 | *(Upcoming)* | |
| **19 Jun** | Day 33 | *(Upcoming)* | |
| **20 Jun** | Day 34 | *(Upcoming)* | |
| **21 Jun** | Day 35 | *(Upcoming)* | |

---

### 📆 Week 6 — June 22–28, 2026 *(planned)*

| Date | Day | Work Done | Key Learnings |
|------|-----|-----------|---------------|
| **22 Jun** | Day 36 | *(Upcoming)* | |
| **23 Jun** | Day 37 | *(Upcoming)* | |
| **24 Jun** | Day 38 | *(Upcoming)* | |
| **25 Jun** | Day 39 | *(Upcoming)* | |
| **26 Jun** | Day 40 | *(Upcoming)* | |
| **27 Jun** | Day 41 | *(Upcoming)* | |
| **28 Jun** | Day 42 | *(Upcoming)* | |

---

### 📆 Final Days — June 29 – July 1, 2026

| Date | Day | Work Done | Key Learnings |
|------|-----|-----------|---------------|
| **29 Jun** | Day 43 | *(Upcoming)* | |
| **30 Jun** | Day 44 | *(Upcoming)* | |
| **01 Jul** | Day 45 | 🎓 Internship Completion | |

---

## 🛠️ Tech Stack Covered

```
FRONTEND
├── React.js / Next.js 14          → Component architecture, App Router, SSR
├── TypeScript                     → Type safety, interfaces, generics
├── TailwindCSS                    → Utility-first styling, responsive design
├── Recharts / Chart.js            → Data visualization, time-series charts
└── WebSocket (Browser API)        → Real-time event subscription

BACKEND
├── Node.js + Express              → REST API basics
├── FastAPI (Python)               → Async APIs, dependency injection
├── Hono                           → Edge-ready web framework
├── JWT + bcrypt                   → Auth, session management
└── WebSocket (FastAPI)            → Real-time broadcasting

DATABASE
├── MongoDB + Motor                → Document store, async I/O
├── Cloudflare D1 (SQLite)         → Serverless relational DB
└── IndexedDB                      → Client-side offline storage

AI / ML / NLP
├── OpenCV                         → Image preprocessing pipeline
├── Pytesseract                    → OCR text extraction
├── spaCy                          → Named Entity Recognition (NER)
└── ReportLab                      → PDF generation

DEVOPS
├── Docker                         → Container management
├── Cloudflare Workers/Pages       → Edge deployment
├── Render.com                     → Backend cloud deployment
└── Vercel                         → Frontend deployment
```

---

## 📈 Skills Gained

<div align="center">

| Category | Skills |
|----------|--------|
| **Full-Stack Dev** | MERN architecture, REST APIs, WebSockets, PWA |
| **Backend** | FastAPI, Hono, JWT auth, async programming |
| **Database** | MongoDB schema design, SQL migrations, multi-tenant DB |
| **AI/ML Integration** | OCR pipeline, NLP/NER, medical document parsing |
| **DevOps** | Docker, Cloudflare deployment, Render, Vercel |
| **Engineering Practices** | Abstract interfaces, modular code, system architecture |

</div>

---

## 🔗 Links

<div align="center">

[![GitHub](https://img.shields.io/badge/GitHub-artist--hks-0D1117?style=for-the-badge&logo=github)](https://github.com/artist-hks)
[![Portfolio](https://img.shields.io/badge/Portfolio-artist--hks.vercel.app-00D9FF?style=for-the-badge&logo=vercel)](https://artist-hks.vercel.app)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-artisthks-0A66C2?style=for-the-badge&logo=linkedin)](https://www.linkedin.com/in/artisthks)
[![ContextCare Live](https://img.shields.io/badge/ContextCare-Live%20Demo-FF6B6B?style=for-the-badge&logo=react)](https://context-care.vercel.app)

</div>

---

<div align="center">

*Internship at Groot Software | MERN Stack | 45 Days | May–July 2026*

**Hemant Sharma** · B.Tech CSE · Semester 6

</div>
