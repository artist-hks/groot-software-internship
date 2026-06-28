# 📅 Week 6 Log — ContextCare + TaskFlow + Internship Wrap-Up

**Duration:** 22 Jun – 28 Jun 2026  
**Projects:** ContextCare AI (Days 36, 37) · TaskFlow (Day 38) · Documentation/prep (Days 39–42)  
**Tech Focus:** Google Gemini API · @dnd-kit · Recharts · Vercel/Render deployment · Interview prep

---

## 🎯 Week Goal
Close out ContextCare's housekeeping, build a new full-stack project (TaskFlow) end-to-end in a single sprint, then shift into interview prep and internship documentation as the program winds down (internship ends 04 July).

---

## Day 36 — 22 Jun 2026 · ContextCare — Final Docs Pass

### What I Did
Light maintenance commit — kept the README and the repo's activity current after the previous week's deployment + documentation push.

### Key Learnings
- A project doesn't need daily feature commits to stay "alive" — small, honest maintenance commits are still real engineering hygiene, not busywork, as long as they're not standing in for actual work that needs doing elsewhere

---

## Day 37 — 23 Jun 2026 · ContextCare — Repo Cleanup

### What I Did
Re-verified the live deployment was still stable (PIN login → QR pair → scan → dashboard update, full loop) and did final repo cleanup before shifting focus to a new project.

### Key Learnings
- Re-testing a "done" project a week after first deploying it is worth the 10 minutes — confirms a free-tier host hasn't quietly broken something (cold-start DB reset, expired session secret, etc.) before you stop paying attention to it

---

## Day 38 — 24 Jun 2026 · TaskFlow — Full MERN Kanban App, Built in a Day

### What I Did
Built TaskFlow end-to-end: a React 18 + Vite client, an Express/MongoDB backend, JWT auth, drag-and-drop Kanban boards, Recharts analytics, and Google Gemini 2.5 Flash for AI-assisted task effort estimation. Generated the initial scaffold from a detailed one-shot prompt, then spent the rest of the day debugging and polishing the result — config bugs, a routing bug, and a round of dark-mode contrast fixes — until it was actually production-solid, not just demo-solid.

### Gemini AI Estimation — with a Deterministic Fallback
The most interesting piece: an AI service that calls Gemini for effort/due-date estimation, but never leaves the feature broken if the API key is missing, the call times out, or the model returns unparseable output.

```javascript
// server/src/services/aiService.js
function ruleBasedSuggestion(title = '', description = '') {
  const text = `${title} ${description}`.toLowerCase();
  const heavyKeywords = ['refactor', 'migrate', 'architecture', 'design', 'integrate',
                          'research', 'investigate', 'redesign', 'database', 'security'];
  const lightKeywords = ['fix', 'typo', 'update', 'rename', 'tweak', 'copy', 'text'];

  let effort = 'M';
  if (heavyKeywords.some(k => text.includes(k)) || description.length > 240) effort = 'L';
  else if (lightKeywords.some(k => text.includes(k)) || description.length < 60) effort = 'S';

  const isUrgent = /urgent|asap|critical|blocker|high/.test(text);
  const daysOut = isUrgent ? 1 : effort === 'L' ? 5 : 3;

  return { effort, suggestedDueDate: toISODate(addDays(new Date(), daysOut)), fallback: true };
}
// The Gemini call wraps this exact function as its catch-all fallback —
// the feature degrades to keyword heuristics, it never just breaks.
```

### Bug #1 — Gemini API Key Wasn't Being URL-Encoded
Google's newer API keys can start with `AQ.` — and the `.` plus other key characters weren't being escaped when appended to the request URL as a query param, silently producing a malformed request.
```diff
- const url = `${GEMINI_URL}?key=${apiKey}`;
+ const url = `${GEMINI_URL}?key=${encodeURIComponent(apiKey)}`;
```

### Bug #2 — React Router 404s on Page Refresh (Vercel)
A client-side route like `/board/123` 404'd on direct load/refresh because Vercel was looking for a matching static file instead of falling back to `index.html` and letting React Router take over.
```json
// client/vercel.json
{ "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }] }
```

### Bug #3 — Dark Mode Contrast
Cards and the Recharts tooltip blended into the dark background with no visible edge.
```diff
- .card { @apply rounded-2xl bg-surface shadow-soft dark:bg-surface-dark; }
+ .card { @apply rounded-2xl bg-surface shadow-soft dark:bg-surface-dark dark:border dark:border-hairline-dark; }

- backgroundColor: theme === 'dark' ? '#1C1C1E' : '#FFFFFF', border: 'none',
+ backgroundColor: theme === 'dark' ? '#2C2C2E' : '#FFFFFF',
+ border: theme === 'dark' ? '1px solid #38383A' : 'none',
```

Shipped to Vercel (client) + Render (server) the same day.

### Key Learnings
- A deterministic rule-based fallback isn't a lesser version of an "AI feature" — it's what makes the AI feature trustworthy. If Gemini times out or the key is wrong, the user still gets a reasonable estimate instantly, not a spinner or an error
- API keys aren't always safe to interpolate raw into a URL — `encodeURIComponent()` on any externally-issued credential going into a query string should be a default habit, not an afterthought caught by a bug
- Dark mode contrast bugs are easy to miss if you build primarily in light mode — a `1px` border that's invisible in light mode can be the only thing separating a card from its background once the background gets dark too

---

## Day 39 — 25 Jun 2026 · TaskFlow Code Walkthrough — Interview Prep

### What I Did
Went through TaskFlow's own codebase end-to-end — the auth flow, the Gemini service + its fallback logic, and the `@dnd-kit` board state management — specifically to be able to explain every part confidently, not just "it works." Given the scaffold came from a one-shot AI prompt, this was about closing the gap between "I can demo it" and "I can defend it in an interview."

### Key Learnings
- Explaining AI-assisted code out loud to yourself is a genuinely useful test — if a line makes you pause and think "wait, why does it do that," that's exactly the line an interviewer is going to ask about

---

## Day 40 — 26 Jun 2026 · Placement Prep + Light Maintenance

### What I Did
Continued placement prep — mock interview practice and DSA revision — alongside light maintenance across the now-deployed projects.

### Key Learnings
- Treating placement prep as its own scheduled block (not something squeezed into leftover time after project work) made it easier to actually track progress on it day to day

---

## Day 41 — 27 Jun 2026 · Internship Documentation

### What I Did
Updated this work log and refreshed the README/architecture docs across Colleqo, ContextCare, CropCortex, and TaskFlow — including correcting a real documentation-accuracy issue found while doing it: ContextCare's README and earlier weekly logs described a Python/FastAPI/MongoDB backend that didn't match the project's actual Next.js/Prisma/Socket.IO implementation, so those got rewritten to match the real codebase.

### Key Learnings
- Documentation drift is a real risk on a fast-moving, AI-assisted workflow — it's worth periodically diffing what the docs claim against what the repo actually contains, especially before anything goes in front of a recruiter
- Catching and fixing a documentation/reality mismatch is a better outcome than not looking — an inaccurate but unexamined README is a bigger risk than the time it takes to audit and correct one

---

## Day 42 — 28 Jun 2026

*(Upcoming)*

---

## 📊 Week 6 Summary

| Metric | Count |
|--------|-------|
| New Project Shipped | 1 (TaskFlow — full MERN app + AI integration, single-day build) |
| Production Bugs Fixed Same-Day | 3 (Gemini key encoding, SPA routing on Vercel, dark-mode contrast) |
| Documentation Issues Found & Fixed | 1 (ContextCare's tech-stack docs vs. real codebase) |
| Live Projects by Week's End | 4 (Colleqo, ContextCare, CropCortex, TaskFlow) |

**What went well:** Building TaskFlow's AI estimation with a deterministic fallback baked in from the start (Day 38) meant the feature was never actually fragile, even though it depends on an external API. Auditing the internship docs (Day 41) caught a real accuracy problem before it could surface badly in an interview.

**What was hard:** Closing the gap between "generated TaskFlow in one shot" and "can defend every line of it in an interview" (Day 39) took real, focused effort — AI-assisted scaffolding is fast to produce but doesn't come with the same built-in understanding that writing it line-by-line would.

**Internship wrap-up:** With the program ending 04 July, the remaining days are documentation, final testing, and handoff notes across all four shipped projects.
