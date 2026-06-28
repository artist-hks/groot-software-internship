<div align="center">

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=30&duration=3000&pause=1000&color=00D9FF&center=true&vCenter=true&width=700&lines=MERN+Stack+Internship+%F0%9F%9A%80;Groot+Software+45+Days;Full-Stack+Web+Development" alt="Typing SVG" />

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
![TypeScript](https://img.shields.io/badge/TypeScript-47.1%25-3178C6?style=for-the-badge&logo=typescript&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-44.2%25-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![HTML](https://img.shields.io/badge/HTML-8.7%25-E34C26?style=for-the-badge&logo=html5&logoColor=white)

**What I Built:**
- 📱 Progressive Web App (PWA) — installable on iOS, Android, Windows with offline support
- 🔐 Role-Based Access Control — Student, Faculty, HOD, Admin, Librarian, Hostel Warden dashboards
- 🏢 Multi-Tenant Architecture — multiple institutions under one deployment via tenant codes
- 📊 Modules: Attendance, Timetable, Exams, Assignments, Fees, Hostel, Library
- 💳 Payment Integration (Razorpay) with order verification & HMAC signatures
- 🔔 FCM Push Notifications + Twilio SMS/WhatsApp alerts
- 📂 Bulk CSV import/export system for students & faculty
- 👨‍👩‍👧 Parent Portal with 6-digit link codes
- 📊 AI-powered analytics dashboard with at-risk scoring

**Tech Stack:** `Hono` · `Cloudflare D1 (SQLite)` · `Cloudflare Workers/Pages` · `Vite` · `TailwindCSS` · `Chart.js` · `Razorpay` · `FCM` · `Twilio`

---

### 2. 🏥 ContextCare AI — Clinical Workspace Platform
> *Single-stack Next.js medical SaaS — OCR lab-report digitization + real-time doctor dashboard*

[![ContextCare](https://img.shields.io/badge/Repo-ContextCare-0D1117?style=for-the-badge&logo=github)](https://github.com/artist-hks/ContextCare)
[![Live Demo](https://img.shields.io/badge/Live-contextcare.onrender.com-FF6B6B?style=for-the-badge&logo=render&logoColor=white)](https://contextcare.onrender.com)
![TypeScript](https://img.shields.io/badge/TypeScript-93.1%25-3178C6?style=for-the-badge&logo=typescript&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-5.4%25-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)

**What I Built:**
- 📸 OCR Pipeline — Sharp image preprocessing (grayscale, normalize, contrast, sharpen) → Tesseract.js text extraction → regex/alias-based metric parser
- 🔗 QR Doctor-Patient Pairing — each doctor gets a unique QR token; patients scan it to securely link their scan to the right physician, no patient login needed
- ⚡ Real-Time Dashboard — Socket.IO room-per-doctor (`doctor:<id>`) pushes new scans to the dashboard instantly, no polling
- 🔐 PIN-Based Doctor Auth — bcrypt-hashed 4–6 digit PIN login, `iron-session` encrypted cookie sessions
- 📈 Trend Charts + Notes — Recharts metric trend lines per patient, append-only clinical notes ledger
- 📄 PDF Export — `@react-pdf/renderer` report generation, run in an isolated child process (its WASM layout engine doesn't survive Next.js bundling)
- 🛡️ In-memory rate limiting on OCR/auth endpoints to block abuse on a single-process deployment

**Tech Stack:** `Next.js 14` · `Prisma` · `SQLite` · `Socket.IO` · `Tesseract.js` · `Sharp` · `iron-session` · `@react-pdf/renderer` · `Recharts` · `Railway/Render`


---

### 3. ✅ TaskFlow — AI-Powered Kanban Task Manager
> *Full-stack MERN kanban board with Gemini-powered effort estimation*

[![TaskFlow](https://img.shields.io/badge/Repo-TaskFlow-0D1117?style=for-the-badge&logo=github)](https://github.com/artist-hks/TaskFlow)
[![Live Demo](https://img.shields.io/badge/Live-taskflow--hks.vercel.app-4CAF50?style=for-the-badge&logo=vercel)](https://taskflow-hks.vercel.app)
![JavaScript](https://img.shields.io/badge/JavaScript-96.6%25-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![CSS](https://img.shields.io/badge/CSS-2.8%25-1572B6?style=for-the-badge&logo=css3&logoColor=white)
![HTML](https://img.shields.io/badge/HTML-0.6%25-E34C26?style=for-the-badge&logo=html5&logoColor=white)

**What I Built:**
- 🗂️ Kanban boards — drag-and-drop tasks across To Do / In Progress / Done using `@dnd-kit`
- ✨ AI Task Estimation — Google Gemini 2.5 Flash predicts task effort & suggests due dates from the task description, with a deterministic offline fallback
- 🔐 Full auth system — JWT + bcrypt password hashing, protected routes
- 📊 Dashboard analytics — status & priority breakdowns via Recharts (donut + bar charts)
- 🌗 Dark mode — class-based theming across the entire app
- ⚙️ REST API — Node.js/Express + MongoDB/Mongoose, validated with `express-validator`

**Tech Stack:** `React 18 (Vite)` · `Node.js/Express` · `MongoDB` · `Mongoose` · `TailwindCSS` · `Google Gemini API` · `@dnd-kit` · `Recharts` · `JWT`

---

### 4. 🌾 CropCortex — AgTech Digital Assistant
> *Mobile-first AI farming assistant for Indian farmers — multilingual & offline-first*

[![CropCortex](https://img.shields.io/badge/Repo-CropCortex-0D1117?style=for-the-badge&logo=github)](https://github.com/artist-hks/CropCortex)
[![Live Demo](https://img.shields.io/badge/Live-cropcortex--app.pages.dev-4CAF50?style=for-the-badge&logo=cloudflare&logoColor=white)](https://cropcortex-app.pages.dev)
![TypeScript](https://img.shields.io/badge/TypeScript-77%25-3178C6?style=for-the-badge&logo=typescript&logoColor=white)
![HTML](https://img.shields.io/badge/HTML-22%25-E34C26?style=for-the-badge&logo=html5&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-1%25-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)

**What I Built:**
- 👨‍⚕️ Crop Doctor — photo-based AI disease diagnosis with organic + chemical treatment suggestions
- 📈 Mandi price intelligence — live mandi rates, 30-day trend charts, custom price alerts
- 🛰️ Satellite field monitoring — NDVI imagery to flag crop stress before it's visible to the eye
- 🗓️ AI crop advisory — hyper-local weekly task plans from crop stage + live weather
- 🌍 Multilingual UI — Hindi, Marathi, Tamil, Telugu support, built mobile-first for India
- 📶 Offline-first — core features work with no network, background sync on reconnect

**Tech Stack:** `React Native + Expo` · `Expo Router` · `Zustand` · `Hono` · `Cloudflare Workers/Pages` · `Wrangler`

---

### 5. 💼 Portfolio (Ongoing Upgrade)
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
| **18 May** | Day 1 | 🔰 Internship kickoff. Environment setup — Node.js, MongoDB, VS Code extensions. Understood project scope and requirements for Colleqo. Studied existing codebase structure. | Project scoping, dev environment best practices |
| **19 May** | Day 2 | ⚙️ Set up Cloudflare Workers + D1 database locally via Wrangler CLI. Initialized Vite build config with TailwindCSS. Created base project scaffolding for Colleqo. | Cloudflare ecosystem, Wrangler local dev, Vite setup |
| **20 May** | Day 3 | 🗄️ Designed DB schema for Colleqo — users, tenants, roles, attendance, timetable tables. Wrote SQL migrations. Set up role-based middleware logic. | Database schema design, multi-tenant patterns, SQLite |
| **21 May** | Day 4 | 🔐 Built authentication system — login/register endpoints with JWT, role extraction, session management. Integrated `tsconfig.json` for strict TypeScript. | JWT auth flow, password hashing, TypeScript strict mode |
| **22 May** | Day 5 | 📱 Started PWA implementation — added `manifest.json`, Service Worker registration, IndexedDB setup for offline queueing. | Progressive Web App fundamentals, Service Workers, offline storage |
| **23 May** | Day 6 | 🎨 Built role-specific dashboard UIs — Student, Faculty, Admin views. Integrated Chart.js for attendance progress rings. | Chart.js integration, conditional rendering based on roles |
| **24 May** | Day 7 | 🧪 Testing + bug fixes across Colleqo modules. Fixed offline sync logic — actions queued in IndexedDB now sync on reconnect. Added multi-tenant code system. | Debugging offline flows, IndexedDB sync patterns |

---

### 📆 Week 2 — May 25–31, 2026

| Date | Day | Work Done | Key Learnings |
|------|-----|-----------|---------------|
| **25 May** | Day 8 | ✅ Colleqo — completed Hostel & Library modules. Room allocation, book issuance, complaint system. Deployed to Cloudflare Pages (`npm run deploy`). | Cloudflare Pages deployment, production PWA checklist |
| **26 May** | Day 9 | 🏥 New project started: **ContextCare AI**. Designed system architecture — single Next.js 14 app (API routes + custom server), Prisma + SQLite, Socket.IO. Scaffolded the project and the Doctor/Patient/Scan/Metric schema. | System architecture design, relational schema design, medical document workflows |
| **27 May** | Day 10 | 🧵 Built the OCR pipeline — Sharp image preprocessing (grayscale, normalize, contrast, sharpen) → Tesseract.js worker, with the English model bundled locally to avoid runtime CDN downloads. | Tesseract.js worker-path resolution in a bundled Node server, image preprocessing |
| **28 May** | Day 11 | 🔬 Built `/api/scans/extract` — multipart image upload, validation, OCR call, graceful fallback when no metrics are found. Added an in-memory sliding-window rate limiter. | API file upload handling, abuse-resistant rate limiting |
| **29 May** | Day 12 | 🧠 Built the metric parser — canonical `METRIC_REFERENCE` table + alias-based regex extraction for 6 lab metrics (FBS, cholesterol, HDL/LDL, triglycerides, hemoglobin), with automatic normal/borderline/critical status. | Alias-table parsing vs. ML/NER trade-offs, single-source-of-truth reference data |
| **30 May** | Day 13 | ⚡ Built the real-time layer — custom `server.ts` running Socket.IO alongside Next.js, doctor-room pattern (`doctor:<id>`), `emitScanCreated()` helper reachable from API routes via a shared `io` instance. | Socket.IO rooms, sharing state between a custom server and Next.js API routes |
| **31 May** | Day 14 | 🔐 Built PIN-based doctor auth (bcrypt + `iron-session` cookie sessions) and the QR pairing flow — `/api/doctor/qr` + `/api/scans/pair`, with find-or-create patient logic keyed by doctor+phone. | iron-session vs. JWT trade-offs, QR-based pairing UX, idempotent patient lookups |

---

### 📆 Week 3 — June 1–7, 2026

| Date | Day | Work Done | Key Learnings |
|------|-----|-----------|---------------|
| **01 Jun** | Day 15 | 🎨 Built the rest of ContextCare's Next.js app — patient upload flow (compress → OCR review → QR pairing), doctor `QrPairOverlay`, Socket.IO client on the dashboard with a polling fallback on disconnect. | Client-side image compression, Socket.IO client patterns, graceful real-time degradation |
| **02 Jun** | Day 16 | 🔔 Implemented FCM push notification system — backend FCM integration with graceful degradation, `push_subscriptions` DB table, `dispatchPush()` helper. Auto-triggers on announcement creation. | Firebase Cloud Messaging, Service Worker push events, upsert patterns |
| **03 Jun** | Day 17 | 💳 Integrated Razorpay payment gateway into Colleqo fees module — create-order endpoint with HMAC-SHA256 signature verification, `payment_transactions` table, mock mode for local dev. | Razorpay integration, crypto.subtle HMAC, payment order flow |
| **04 Jun** | Day 18 | 📱 Built Twilio SMS/WhatsApp alert system — `sendTwilioMessage()` with real `fetch()` to Twilio REST API, auto-triggers on low attendance (<75%) and fee payment confirmations. | Twilio REST API, URLSearchParams encoding, multi-channel messaging |
| **05 Jun** | Day 19 | 📊 Built AI-powered analytics dashboard — rule-based insight engine analyzing attendance trends, at-risk students (risk score: `attendance×0.4 + exams×0.4 + fees×0.2`). CSV export with no server round-trip. | SQL aggregations, risk scoring algorithms, client-side CSV generation |
| **06 Jun** | Day 20 | 📂 Built bulk CSV import/export system + Parent Portal. CSV import: parse → validate → insert students/faculty with auto-generated passwords. Parent portal: 6-digit link codes (24-hour TTL). | Character-by-character CSV parsing, parent-child linking model, token expiry |
| **07 Jun** | Day 21 | 🎨 Major UI upgrade — implemented professional desktop layout (Notion/Linear style): 240px fixed sidebar, 60px header with breadcrumb + user menu, role-based nav. Rebranded to Colleqo. SaaS landing page. | CSS Grid layout patterns, mobile-responsive hamburger menu, print CSS |

---

### 📆 Week 4 — June 8–14, 2026

| Date | Day | Work Done | Key Learnings |
|------|-----|-----------|---------------|
| **08 Jun** | Day 22 | 🧪 Colleqo — regression-tested the Week 3 desktop UI overhaul (sidebar, breadcrumb, role-based nav) across all roles. Fixed minor responsive bugs on tablet breakpoints. | Cross-role UI testing, responsive breakpoint debugging |
| **09 Jun** | Day 23 | 📋 Colleqo — triaged bug reports from the new SaaS landing page & pricing tiers. Reviewed CropCortex (personal AgTech side-project) codebase to write up its architecture docs. | Bug triage workflow, documenting an existing Hono/Cloudflare backend |
| **10 Jun** | Day 24 | 🎨 Colleqo — full logo rebrand: replaced every logo with `colleqo_logo.svg` app-wide, restructured README. CropCortex — fixed README formatting in the system architecture section. | Brand asset rollout across a multi-screen app, README structuring |
| **11 Jun** | Day 25 | 🔲 Colleqo — designed the QR-based attendance flow: backend endpoint contracts for session creation + scan verification, ahead of implementation. | QR attendance system design, session-token patterns |
| **12 Jun** | Day 26 | ✅ Colleqo — shipped QR attendance: backend endpoints + in-session QR code image display. Added a proprietary license to the repo. | QR code generation/display in-app, licensing a proprietary codebase |
| **13 Jun** | Day 27 | 🧪 Colleqo — end-to-end testing of the new QR attendance flow; fixed edge cases (duplicate scans, expired sessions). | Edge-case testing for time-bound tokens |
| **14 Jun** | Day 28 | 📚 Placement prep — DSA practice + resume refinement alongside Week 4 wrap-up on Colleqo. | Balancing internship delivery with placement prep |

---

### 📆 Week 5 — June 15–21, 2026

| Date | Day | Work Done | Key Learnings |
|------|-----|-----------|---------------|
| **15 Jun** | Day 29 | 🎨 Colleqo — planned the Apple-style mobile sidebar redesign + a security pass (XSS escaping, login guards) ahead of the Week 5 push. | Planning a UI redesign + security checklist for auth-gated nav |
| **16 Jun** | Day 30 | 📱 Colleqo — shipped Apple-style sidebar animation, fixed the QR scanner bug, seeded parent portal data. Fixed bottom-nav-breaks-after-sidebar-close + sidebar-stuck-on-logout bugs. Added XSS escaping and sidebar login guards. Redesigned mobile sidebar to the Colleqo indigo palette. | CSS animation timing, defensive state resets on logout, XSS-safe rendering |
| **17 Jun** | Day 31 | 🚀 CropCortex — Cloudflare Pages deployment: added `fix-dist.js` to work around Wrangler's hidden-folder ignore rules for the Expo web export. Polished README with live deployment links. | Cloudflare Pages + Expo web export quirks, Wrangler ignore-rule workarounds |
| **18 Jun** | Day 32 | 🏥 ContextCare — prepped production deployment: Railway config with `DATABASE_URL`, an auto-seed script for free-tier hosting, added the live demo URL to the README. | Railway deployment config, free-tier DB seeding strategy |
| **19 Jun** | Day 33 | 📝 ContextCare — full README overhaul: fixed a duplicate title, replaced Unicode ASCII art with plain ASCII for consistent cross-platform rendering, restructured into a professional layout. | Cross-platform terminal rendering quirks, technical writing for a production README |
| **20 Jun** | Day 34 | 🔧 ContextCare — repo housekeeping and documentation upkeep; verified the live deployment was stable end-to-end. | Maintaining repo hygiene on a shipped project |
| **21 Jun** | Day 35 | ✅ Wrapped Week 5 with three live, deployed projects running in parallel — Colleqo, ContextCare, and CropCortex. Final ContextCare housekeeping pass. | Managing multiple live deployments simultaneously |

---

### 📆 Week 6 — June 22–28, 2026

| Date | Day | Work Done | Key Learnings |
|------|-----|-----------|---------------|
| **22 Jun** | Day 36 | 📝 ContextCare — final documentation pass to keep the README and repo activity current. | Long-term repo maintenance habits |
| **23 Jun** | Day 37 | 🔍 ContextCare — repo cleanup, re-verified live deployment stability. | Post-launch monitoring on a side project |
| **24 Jun** | Day 38 | ✅ Built TaskFlow — a full-stack MERN kanban app from scratch: React 18 + Vite client, Express/MongoDB backend, JWT auth, drag-and-drop boards with `@dnd-kit`, Recharts analytics, and Google Gemini 2.5 Flash for AI task-effort estimation. Shipped to Vercel (frontend) + Render (backend); fixed Gemini API key URL-encoding, React Router 404s on refresh, and dark-mode contrast issues the same day. | Full MERN app in a single sprint, Gemini API integration, SPA routing on Vercel, dark-mode contrast tuning |
| **25 Jun** | Day 39 | 🧠 Did a full code walkthrough of TaskFlow — auth flow, the AI estimation service, and the Kanban drag-and-drop state — to be able to explain every part confidently. | Explaining AI-assisted code with the same depth as hand-written code |
| **26 Jun** | Day 40 | 📚 Continued placement prep (mock interview practice, DSA revision) alongside light Colleqo maintenance. | Interview communication, time-boxing prep across multiple goals |
| **27 Jun** | Day 41 | 🗂️ Internship documentation — updated this work log and refreshed READMEs across Colleqo, ContextCare, CropCortex, and TaskFlow for the placement portfolio. | Technical documentation as part of a portfolio narrative |
| **28 Jun** | Day 42 | *(Upcoming)* | |

---

### 📆 Final Days — June 29 – July 4, 2026

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
├── React Native + Expo            → Cross-platform mobile (Android/iOS/Web)
├── Zustand                        → Lightweight global state management
├── Vanilla JavaScript             → DOM manipulation, localStorage, IndexedDB
└── WebSocket (Browser API)        → Real-time event subscription

BACKEND
├── Node.js + Express              → REST API basics
├── Hono                           → Edge-ready web framework (Cloudflare Workers)
├── Next.js API Routes             → Full-stack routes + custom Node HTTP server
├── JWT + bcrypt / iron-session    → Auth, encrypted session cookies
└── Socket.IO                      → Real-time broadcasting (room-per-doctor)

DATABASE
├── MongoDB + Mongoose             → Document store, ODM modeling
├── Prisma + SQLite                → Type-safe ORM, embedded relational DB
├── Cloudflare D1 (SQLite)         → Serverless relational DB
└── IndexedDB                      → Client-side offline storage

AI / ML / OCR
├── Tesseract.js + Sharp           → In-browser/Node OCR + image preprocessing
├── Google Gemini API              → AI task-effort estimation (TaskFlow)
├── NDVI Satellite Imagery         → Crop stress detection (CropCortex)
└── @react-pdf/renderer            → PDF report generation (isolated child process)

INTEGRATIONS
├── Firebase Cloud Messaging       → Push notifications
├── Razorpay                       → Payment gateway
├── Twilio                         → SMS & WhatsApp messaging
├── qrcode / html5-qrcode          → QR generation & in-browser scanning

DEVOPS
├── Cloudflare Workers/Pages       → Edge deployment
├── Render.com / Railway           → Backend cloud deployment
└── Vercel                         → Frontend deployment
```

---

## 📈 Skills Gained

<div align="center">

| Category | Skills |
|----------|--------|
| **Full-Stack Dev** | MERN architecture, REST APIs, real-time sockets, PWA, system design |
| **Mobile Dev** | React Native, Expo Router, Zustand state management, offline-first design |
| **Backend** | Node.js/Express, Hono, Next.js API routes, custom HTTP+WebSocket servers, JWT/session auth |
| **Database** | MongoDB schema design, Prisma ORM, SQL migrations, multi-tenant DB, query optimization |
| **AI/ML Integration** | OCR pipeline (Tesseract.js), Gemini API task estimation, NDVI satellite imagery, image preprocessing |
| **Integrations** | FCM, Razorpay, Twilio, QR generation/scanning, payment verification, real-time messaging |
| **DevOps** | Cloudflare deployment, Render, Railway, Vercel, CI/CD basics |
| **Engineering Practices** | Abstract interfaces, modular code, system architecture, testing patterns |

</div>

---

## 🔗 Links

<div align="center">

[![GitHub](https://img.shields.io/badge/GitHub-artist--hks-0D1117?style=for-the-badge&logo=github)](https://github.com/artist-hks)
[![Portfolio](https://img.shields.io/badge/Portfolio-artist--hks.vercel.app-00D9FF?style=for-the-badge&logo=vercel)](https://artist-hks.vercel.app)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-artisthks-0A66C2?style=for-the-badge&logo=linkedin)](https://www.linkedin.com/in/artisthks)
[![ContextCare Live](https://img.shields.io/badge/ContextCare-Live%20Demo-FF6B6B?style=for-the-badge&logo=render&logoColor=white)](https://contextcare.onrender.com)
[![Colleqo Live](https://img.shields.io/badge/Colleqo-Live%20Demo-4CAF50?style=for-the-badge&logo=cloudflare)](https://campusos.pages.dev)
[![CropCortex Live](https://img.shields.io/badge/CropCortex-Live%20Demo-00D9FF?style=for-the-badge&logo=cloudflare)](https://cropcortex-app.pages.dev)
[![TaskFlow Live](https://img.shields.io/badge/TaskFlow-Live%20Demo-B45309?style=for-the-badge&logo=vercel)](https://taskflow-hks.vercel.app)

</div>

---

<div align="center">

*Internship at Groot Software | MERN Stack | 45 Days | May–July 2026*

**Hemant Sharma** · B.Tech CSE · Semester 6

</div>
