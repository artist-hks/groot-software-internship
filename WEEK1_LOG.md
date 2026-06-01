# 📅 Week 1 Log — CampusOS

**Duration:** 18 May – 24 May 2026  
**Project:** CampusOS — Next-Generation College Management System  
**Tech Focus:** Cloudflare Workers · Hono · D1 SQLite · Vite · TailwindCSS · PWA

---

## 🎯 Week Goal
Set up the CampusOS project from scratch — architecture, DB schema, auth system, role dashboards, PWA support, and core modules.

---

## Day 1 — 18 May 2026 · Environment Setup & Scope Understanding

### What I Did
- Installed and configured dev tools: Node.js 18+, Wrangler CLI, VS Code extensions (ESLint, Prettier, Tailwind IntelliSense)
- Read through the project brief — CampusOS needs to serve Students, Faculty, HOD, Admin, Librarian, Hostel Warden, and Super Admin under one roof
- Studied **Hono framework** docs — understood why it's ideal for Cloudflare Workers (zero cold starts, edge-native)
- Explored **Cloudflare D1** — serverless SQLite that runs at the edge alongside the Worker
- Mapped out initial module list: Auth, Attendance, Timetable, Exams, Assignments, Fees, Hostel, Library

### Key Learnings
- Hono's routing API is Express-like but ~10x faster at the edge because it has no Node.js dependency
- Cloudflare D1 uses the same SQL syntax as SQLite — no ORM needed for simple queries
- PWA requirement means we need `manifest.json` + Service Worker from day one, not bolted on later

### Commands Run
```bash
npm create cloudflare@latest campusos -- --framework=none
npm install hono
npx wrangler d1 create campusos-db
```

---

## Day 2 — 19 May 2026 · Project Scaffolding + Build Config

### What I Did
- Initialized **Vite** as the frontend build tool — configured `vite.config.ts` with path aliases and build output pointing to `public/static`
- Set up **TailwindCSS** v3 with custom config — defined color palette, font families, spacing scale
- Created folder structure:
  ```
  src/
  ├── pages/          → HTML entry points per role
  ├── components/     → Reusable vanilla JS modules
  ├── api/            → Hono route handlers
  ├── db/             → D1 query helpers
  └── workers/        → Service Worker + background sync
  ```
- Configured `wrangler.jsonc` — bound D1 database, set compatibility flags, defined routes
- Created `tsconfig.json` with strict mode enabled

### Key Learnings
- Vite's `build.rollupOptions.input` can take multiple HTML entry points — useful for multi-page apps
- TailwindCSS JIT mode (default in v3) generates only used classes, keeping CSS bundle tiny
- `wrangler.jsonc` allows comments unlike regular JSON — helpful for documenting binding names

### Config Snippet
```jsonc
// wrangler.jsonc
{
  "name": "campusos",
  "compatibility_date": "2024-01-01",
  "d1_databases": [
    {
      "binding": "DB",
      "database_name": "campusos-db",
      "database_id": "..." // from wrangler d1 create output
    }
  ]
}
```

---

## Day 3 — 20 May 2026 · Database Schema Design & Migrations

### What I Did
- Designed the full relational schema for CampusOS — 12 tables covering all modules
- Wrote SQL migration files in `/migrations/` folder (Wrangler picks these up automatically)
- Implemented multi-tenant logic: every table has a `tenant_code` column — queries always filter by tenant
- Set up `npm run db:reset` script that applies migrations + seeds demo data

### Schema Overview
```sql
-- Core tables
CREATE TABLE tenants (
  id TEXT PRIMARY KEY,
  name TEXT NOT NULL,
  tenant_code TEXT UNIQUE NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE users (
  id TEXT PRIMARY KEY,
  email TEXT UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  role TEXT NOT NULL, -- student | faculty | hod | admin | librarian | warden | super_admin
  tenant_code TEXT NOT NULL,
  full_name TEXT,
  FOREIGN KEY (tenant_code) REFERENCES tenants(tenant_code)
);

CREATE TABLE attendance (
  id TEXT PRIMARY KEY,
  student_id TEXT NOT NULL,
  subject_code TEXT NOT NULL,
  date DATE NOT NULL,
  status TEXT NOT NULL, -- present | absent | late
  tenant_code TEXT NOT NULL
);

-- + timetable, exams, assignments, fees, hostel_rooms, books, complaints
```

### Key Learnings
- Multi-tenant with a `tenant_code` column (vs separate DB per tenant) is the right call for small-scale SaaS — simpler ops, same performance
- Wrangler migrations run in order by filename prefix — naming them `0001_init.sql`, `0002_seed.sql` keeps order deterministic
- SQLite doesn't have `UUID()` function — generating UUIDs in application code (crypto.randomUUID()) before inserting

---

## Day 4 — 21 May 2026 · Authentication System

### What I Did
- Built complete JWT-based auth flow in Hono:
  - `POST /api/auth/login` — validates credentials, returns signed JWT
  - `POST /api/auth/register` — hashes password, inserts user, returns JWT
  - Auth middleware that verifies JWT on every protected route
- Used **Web Crypto API** for JWT (no external library needed in Cloudflare Workers — `crypto` is built-in)
- Role is embedded in JWT payload — middleware extracts it and attaches to `c.set('user', payload)`

### Auth Flow
```
Client → POST /api/auth/login { email, password, tenant_code }
       → Worker validates tenant exists
       → Fetches user from D1 by email + tenant_code
       → Compares bcrypt hash (using SubtleCrypto)
       → Signs JWT with role + userId + tenantCode
       → Returns { token, role, user }

Client → Stores JWT in localStorage
       → Sends as Authorization: Bearer <token> on every API call
```

### Hono Middleware Pattern
```typescript
// Auth middleware
const authMiddleware = async (c: Context, next: Next) => {
  const authHeader = c.req.header('Authorization');
  if (!authHeader?.startsWith('Bearer ')) {
    return c.json({ error: 'Unauthorized' }, 401);
  }
  const token = authHeader.slice(7);
  const payload = await verifyJWT(token, JWT_SECRET);
  c.set('user', payload);
  await next();
};
```

### Key Learnings
- Cloudflare Workers have `crypto.subtle` available globally — no need for `jsonwebtoken` npm package
- Hono's `c.set()` / `c.get()` is the idiomatic way to pass data between middleware and handlers
- Storing tenant_code in JWT means one fewer DB lookup per request

---

## Day 5 — 22 May 2026 · PWA Implementation

### What I Did
- Created `public/static/manifest.json` with app name, icons (192px + 512px), theme color, display mode
- Wrote Service Worker (`sw.js`) with:
  - **Cache-first strategy** for static assets (JS, CSS, images)
  - **Network-first strategy** for API calls
  - **Background sync** — queues failed POST requests (attendance mark, assignment submit) in IndexedDB and replays when online
- Registered Service Worker in app entry point with proper scope

### Service Worker Cache Strategy
```javascript
// sw.js
const CACHE_NAME = 'campusos-v1';
const STATIC_ASSETS = ['/index.html', '/app.js', '/app.css', '/manifest.json'];

self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME).then(cache => cache.addAll(STATIC_ASSETS))
  );
});

self.addEventListener('fetch', (event) => {
  const isAPI = event.request.url.includes('/api/');
  event.respondWith(
    isAPI ? networkFirst(event.request) : cacheFirst(event.request)
  );
});
```

### IndexedDB Offline Queue
```javascript
// When offline, queue action
async function queueAction(payload) {
  const db = await openDB('campusos-offline', 1);
  await db.add('pending-actions', { payload, timestamp: Date.now() });
}

// On reconnect, replay queued actions
self.addEventListener('sync', async (event) => {
  if (event.tag === 'sync-attendance') {
    const pending = await getAllPending();
    for (const action of pending) {
      await fetch('/api/attendance', { method: 'POST', body: JSON.stringify(action) });
    }
  }
});
```

### Key Learnings
- `display: standalone` in manifest.json removes browser chrome — app feels native when installed
- Background Sync API needs `navigator.serviceWorker.ready` before registering a sync tag
- IndexedDB has a complex callback API — wrapping it in a Promise-based helper makes life much easier

---

## Day 6 — 23 May 2026 · Role-Based Dashboards + Chart.js

### What I Did
- Built 7 distinct dashboard views — one per role — each showing only relevant modules
- **Student Dashboard:** Attendance progress rings, today's timetable, pending assignments, exam schedule
- **Faculty Dashboard:** Mark attendance (QR or manual), view class list, upload assignments
- **Admin Dashboard:** Fee tracking, notices, user management
- Integrated **Chart.js** for attendance progress rings (doughnut chart)
- Built a reusable `renderAttendanceRing(canvas, percentage)` function

### Attendance Ring Component
```javascript
// components/AttendanceRing.js
export function renderAttendanceRing(canvasId, percentage, subject) {
  const ctx = document.getElementById(canvasId).getContext('2d');
  const color = percentage >= 75 ? '#4CAF50' : percentage >= 60 ? '#FF9800' : '#F44336';
  
  new Chart(ctx, {
    type: 'doughnut',
    data: {
      datasets: [{
        data: [percentage, 100 - percentage],
        backgroundColor: [color, '#1E2327'],
        borderWidth: 0
      }]
    },
    options: {
      cutout: '75%',
      plugins: {
        legend: { display: false },
        tooltip: { enabled: false }
      }
    }
  });
  
  // Center text overlay
  ctx.fillStyle = color;
  ctx.font = 'bold 20px monospace';
  ctx.textAlign = 'center';
  ctx.fillText(`${percentage}%`, canvas.width/2, canvas.height/2);
}
```

### Key Learnings
- Chart.js doughnut with high `cutout` value looks like a progress ring — no extra library needed
- Role-based UI: one HTML file per role with conditional JS loading (lazy imports) keeps initial bundle small
- Color-coded attendance: green ≥75%, orange 60–74%, red <60% — immediately communicates urgency

---

## Day 7 — 24 May 2026 · Testing, Offline Sync Fix + Multi-Tenant Code System

### What I Did
- Full end-to-end testing of all CampusOS modules across roles
- **Critical bug found:** Background sync was replaying actions even after successful submission → Fixed by clearing IndexedDB queue only after confirmed 2xx response
- Implemented **Tenant Code System** — every institution gets a unique 4-letter code (e.g., `DEMO`, `RAIT`, `JIET`) used in login screen + embedded in all DB queries
- Added seed data for `DEMO` tenant with test credentials for all roles
- Ran `npm run deploy` — live on Cloudflare Pages

### Bug Fix — Double Submission
```javascript
// BEFORE (buggy): cleared queue before confirming success
await clearPending(action.id);
await fetch('/api/attendance', { ... });

// AFTER (fixed): clear ONLY after confirmed success
const response = await fetch('/api/attendance', { ... });
if (response.ok) {
  await clearPending(action.id); // ✅ safe to clear now
}
```

### Demo Credentials Added
```sql
INSERT INTO tenants VALUES ('demo-001', 'Demo University', 'DEMO', CURRENT_TIMESTAMP);
INSERT INTO users VALUES ('admin-001', 'admin@demouniversity.edu', '<hashed>', 'admin', 'DEMO', 'Demo Admin');
```

### Key Learnings
- Always confirm server success before clearing local queue — network can partially succeed
- Tenant codes in login screen act as a "namespace" — same email can exist across two tenants without conflict
- Cloudflare Pages deployment is instant — `wrangler pages deploy ./dist` takes under 30 seconds

---

## 📊 Week 1 Summary

| Metric | Count |
|--------|-------|
| Features Built | 8 core modules |
| DB Tables Designed | 12 |
| API Endpoints | 18+ |
| Roles Supported | 7 |
| Days to Deployment | 7 |

**What went well:** Hono + Cloudflare D1 combo is extremely fast to work with. PWA offline sync works reliably after the double-submission bug fix.

**What was hard:** Multi-tenant schema design took longer than expected — had to redo 3 tables after realizing tenant_code needed to be on every table, not just `users`.

**Next week:** New project — ContextCare AI (medical SaaS with OCR pipeline + WebSocket dashboard)
