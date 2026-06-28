# 📅 Week 5 Log — Colleqo + CropCortex + ContextCare

**Duration:** 15 Jun – 21 Jun 2026  
**Projects:** Colleqo (Days 29, 30) · CropCortex (Day 31) · ContextCare AI (Days 32–35)  
**Tech Focus:** CSS 3D transforms · XSS-safe rendering · Cloudflare Pages (Expo web) · Railway/Render deployment

---

## 🎯 Week Goal
Ship the Apple-style sidebar redesign + a security pass on Colleqo, get CropCortex's Expo web build actually deploying on Cloudflare Pages, and take ContextCare from "working locally" to a live, production deployment with a proper public README.

---

## Day 29 — 15 Jun 2026 · Sidebar Redesign + Security Pass — Planning

### What I Did
Planned two things before touching code: an iOS-style sidebar push animation to replace the current flat slide-in, and a security pass on the sidebar/header rendering — both `innerHTML`-based and built from user-controlled fields (name, role) without any escaping.

### Key Learnings
- Grepping for every `innerHTML = ` assignment that includes a template literal with a user field is a fast way to scope an XSS pass — most of the actual fixes turned out to be 3 call sites (sidebar header, desktop header, profile dropdown)

---

## Day 30 — 16 Jun 2026 · Apple Sidebar Animation + Security Fixes

### What I Did
Three commits, same day: the sidebar redesign itself, a couple of bugs it introduced, and the security pass planned on Day 29.

**Apple-style sidebar:** rebuilt the mobile drawer to push/scale/rotate the *entire app* instead of just sliding a panel over it — the same visual language iOS uses for its app-switcher-adjacent menus.

```css
/* src/index.tsx — injected styles */
#app-wrapper {
  transition: transform 0.42s cubic-bezier(0.32, 0.72, 0, 1),
              border-radius 0.42s cubic-bezier(0.32, 0.72, 0, 1),
              box-shadow 0.42s cubic-bezier(0.32, 0.72, 0, 1);
  transform-origin: right center;
}
#app-wrapper.sidebar-open {
  transform: translateX(72%) scale(0.91) rotateY(-4deg);
  border-radius: 16px;
  box-shadow: -20px 0 60px rgba(0,0,0,0.35);
  will-change: transform;
  overflow: hidden;
}
```

**Bug found immediately after shipping it:** tapping "Logout" from inside the open sidebar left the sidebar visually stuck mid-animation because the page navigated away before the close transition could run.

```javascript
// Before: onclick="logout()"
// After:
<button onclick="closeSidebar();logout()">Logout</button>
```

**Security fixes**, same day:
```javascript
function escapeHtml(value) {
  return String(value ?? '').replace(/[&<>"']/g, (char) => ({
    '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;',
  }[char]));
}

// applied to every user-controlled field rendered into the sidebar/header:
const fullName = escapeHtml(`${user?.first_name || ''} ${user?.last_name || ''}`.trim() || 'User');
```

Also added `resetSidebar({ clear })` called from the router on every navigation — closes the drawer and (on logout / unauthenticated routes) wipes its `innerHTML` entirely, so a logged-out session can never render a stale logged-in user's name from a cached DOM node.

### Key Learnings
- `transform-origin: right center` + a combined `translateX + scale + rotateY` is what actually sells the "the whole app is a physical card being pushed" effect — animating just `translateX` alone looks flat by comparison
- A UI bug and a security bug can come from the same root cause: neither `closeSidebar()` nor a route change was clearing/escaping the sidebar's `innerHTML` reliably, so fixing one properly (the `resetSidebar` reset-and-clear pattern) fixed both
- Escaping at render time (not at input time) is the safer default — it means *every* place that renders user data is protected, instead of relying on every input path having sanitized first

---

## Day 31 — 17 Jun 2026 · CropCortex — Cloudflare Pages Deployment

### What I Did
Got CropCortex's Expo web export actually deployable on Cloudflare Pages. `npx expo export -p web` outputs assets under `_expo/` and `assets/node_modules/` — and Cloudflare Wrangler silently ignores both underscore-prefixed paths and anything named `node_modules` when uploading a Pages deployment. The build looked fine locally and then 404'd half its JS chunks in production.

### The Fix — `fix-dist.js`
```javascript
// Run after `expo export -p web`, before `wrangler pages deploy dist`.
// Renames the two path patterns Wrangler ignores, then rewrites every
// reference to them across the exported html/js/json/css files.
const oldExpoDir = path.join(distDir, '_expo');
const newExpoDir = path.join(distDir, 'expo');
const oldNodeModulesDir = path.join(distDir, 'assets', 'node_modules');
const newNodeModulesDir = path.join(distDir, 'assets', 'vendor');

if (fs.existsSync(oldExpoDir)) fs.renameSync(oldExpoDir, newExpoDir);
if (fs.existsSync(oldNodeModulesDir)) fs.renameSync(oldNodeModulesDir, newNodeModulesDir);

function replaceInFile(filePath) {
  let content = fs.readFileSync(filePath, 'utf8');
  content = content.replace(/\/_expo\//g, '/expo/').replace(/\/node_modules\//g, '/vendor/');
  fs.writeFileSync(filePath, content, 'utf8');
}
// ...walks the whole dist/ tree calling replaceInFile on every html/js/json/css file
```

Deploy sequence became: `npx expo export -p web` → `node fix-dist.js` → `npx wrangler pages deploy dist`.

### Key Learnings
- Static-host ignore rules (dotfiles, underscore-prefixed paths, `node_modules`) exist for good reasons in general, but they collide hard with bundlers that weren't written with that specific host in mind — Expo's web export has no idea Cloudflare Pages exists
- The bug was invisible locally because `expo start --web` serves files directly, never going through Wrangler's upload filter — it only ever showed up *after* deploying, which is a good reminder to actually test the deployed build, not just the local dev server
- A small Node script run as a deploy-time step is sometimes the pragmatic fix — didn't need to fight Expo's export config or Wrangler's ignore rules, just normalized the output between the two

---

## Day 32 — 18 Jun 2026 · ContextCare — Production Deployment Prep

### What I Did
Prepped ContextCare for a real (not just local) deployment. Three small but important changes:

1. Pointed `DATABASE_URL` at an env var instead of a hardcoded path, so the same Prisma schema works locally (`file:./dev.db`) and on a host with a persistent volume (`file:/data/dev.db` on Railway)
2. Chained the production start script so a fresh deploy with an empty DB auto-seeds itself — important on a free hosting tier where the filesystem isn't guaranteed to persist between deploys
3. Added the live demo URL to the README once the first deploy was confirmed working

```diff
- "start": "NODE_ENV=production tsx server.ts",
+ "start": "npx prisma migrate deploy && tsx prisma/seed.ts && NODE_ENV=production tsx server.ts",
```

```env
# .env.example
# For local dev: "file:./dev.db"
# For Railway (persistent volume): "file:/data/dev.db"
DATABASE_URL="file:./dev.db"
```

### Key Learnings
- "Auto-seed on every start" is a deliberate trade-off for a demo app on free hosting — in a real product you'd never want a restart to silently reset data, but here it guarantees the live demo always has the seeded doctors/patients to show, even after a free-tier cold restart wipes the disk
- Chaining `migrate deploy && seed && start` in one script line means there's no manual deployment runbook to forget — `npm start` is the entire deploy story

---

## Day 33 — 19 Jun 2026 · ContextCare — README Overhaul

### What I Did
Rewrote the README from a working-notes doc into something a recruiter or evaluator could actually read end-to-end: proper Tech Stack table, an ASCII architecture diagram, a features table, API reference, and a database schema section. Also fixed three smaller rendering bugs the previous version had: a duplicated project title, inconsistent emoji breaking the ASCII box alignment, and Unicode box-drawing characters that rendered fine on GitHub's web view but garbled in some terminals — replaced with plain ASCII for consistent rendering everywhere.

### Key Learnings
- Unicode box-drawing characters (`┌─┐│└┘`) look great on GitHub's rendered view but are a gamble in a plain terminal or a different font — plain ASCII (`+--+|`) is less pretty but renders identically everywhere
- Writing the "How It Works" step-by-step flow forced me to describe the patient→OCR→DB→dashboard pipeline in plain language, which is a good check that the architecture actually makes sense to someone who didn't build it

---

## Day 34 — 20 Jun 2026 · ContextCare — Repo Housekeeping

### What I Did
Light day — verified the live deployment end-to-end (PIN login, QR pairing, a full scan-to-dashboard round trip) and did small README upkeep to keep the repo's contribution activity current.

### Key Learnings
- A "does it actually still work end-to-end" pass a couple of days after the first deploy caught nothing broken this time, but it's worth doing as a habit — free-tier hosts restart/sleep instances in ways that can silently break a stateful demo

---

## Day 35 — 21 Jun 2026 · Week 5 Wrap-Up

### What I Did
Closed out the week with three live, deployed projects running in parallel: Colleqo (Cloudflare Pages), CropCortex (Cloudflare Pages), and ContextCare (Railway-configured, live on Render). Final small ContextCare housekeeping commit.

### Key Learnings
- Running three separate deployed projects at once is its own kind of overhead that's easy to underestimate — each has its own deploy command, its own env vars, and its own "is it actually still up" check; worth keeping a short personal checklist rather than relying on memory

---

## 📊 Week 5 Summary

| Metric | Count |
|--------|-------|
| Projects Deployed/Re-deployed | 3 (Colleqo, CropCortex, ContextCare) |
| Security Fixes Shipped | 2 (XSS escaping, sidebar state leak on logout) |
| Deployment Blockers Solved | 1 (Wrangler ignore-rule collision with Expo web export) |
| README Rewrites | 1 (ContextCare, +389 lines) |

**What went well:** The CropCortex deploy bug (Day 31) had a clean, satisfying root cause once found — a static host's ignore rules colliding with a bundler's default output paths — and a small standalone fix script solved it without fighting either tool's config.

**What was hard:** The sidebar bug + security pass on Day 30 took longer than expected because they turned out to share a root cause (the drawer's `innerHTML` was neither reliably cleared nor escaped) — fixing them as two separate patches first, then realizing one `resetSidebar()` function should own both concerns.

**Next week:** Final ContextCare housekeeping, then a new build — TaskFlow, a full-stack MERN kanban app with Gemini-powered AI task estimation — followed by internship wrap-up and documentation.
