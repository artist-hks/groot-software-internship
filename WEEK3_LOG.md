# 📅 Week 3 Log — ContextCare AI + Colleqo

**Duration:** 01 Jun – 07 Jun 2026  
**Projects:** ContextCare AI (Day 15) · Colleqo (Days 16–21)  
**Tech Focus:** Next.js 14 · App Router · FCM Push Notifications · Razorpay · Twilio · AI Analytics · CSV Import/Export · Parent Portal · Professional Desktop UI · CI/CD

---

## 🎯 Week Goal
Wrap up ContextCare AI with a fully wired Next.js 14 frontend (Day 15), then ship 6 major production features for Colleqo (Days 16–21): push notifications, Razorpay payment gateway, Twilio SMS/WhatsApp alerts, an AI-powered analytics dashboard, bulk CSV import/export + parent portal, and a complete UI rebrand with professional desktop layout + SaaS landing page. End of week: live deployment to `campusos.pages.dev` via GitHub Actions CI/CD.

---

## Day 15 — 01 June 2026 · ContextCare AI — Patient Flow + Doctor Dashboard

### What I Did
Built out the rest of the Next.js 14 app — the same project from Week 2, since the API routes, the Socket.IO server, and the UI all live in one codebase. No separate frontend repo or REST client config needed; pages just call their own `/api/...` routes.

**Patient side:** a multi-step `PatientUploadFlow` (Upload → Processing → Review → Pair) — compress the photo client-side with `browser-image-compression` before it ever leaves the phone, POST it to `/api/scans/extract`, let the patient edit any metric the OCR misread, then `PairStep` collects name + phone and the doctor's QR/pairing code and submits to `/api/scans/pair`.

**Doctor side:** a `QrPairOverlay` that fetches `/api/doctor/qr` and renders the pairing QR right on the dashboard, and a dashboard page that opens a Socket.IO client connection, joins the doctor's own room, and listens for `scan:created` events to update the patient list and trend charts live — with a 5-second polling fallback if the socket ever disconnects, so the feature degrades instead of breaking.

### Key Code / Implementation

```typescript
// components/QrPairOverlay.tsx — doctor's live pairing QR
export default function QrPairOverlay({ open, onClose, pairedPatientName }: Props) {
  const [qr, setQr] = useState<{ code: string; qrDataUrl: string } | null>(null);

  useEffect(() => {
    if (!open) return;
    fetch("/api/doctor/qr")
      .then((r) => r.json())
      .then((d) => (d.error ? setError(d.error) : setQr(d)));
  }, [open]);

  // ...renders qr.qrDataUrl as an <img>, swaps to a success state
  // once `pairedPatientName` is set by the socket handler below.
}
```

```typescript
// app/doctor/dashboard/page.tsx — Socket.IO client + polling fallback
const socket = io({ path: "/socket.io", transports: ["websocket", "polling"] });

socket.on("connect", () => {
  setConnected(true);
  socket.emit("join", doctor.id);
  if (pollRef.current) { clearInterval(pollRef.current); pollRef.current = null; }
});

socket.on("disconnect", () => {
  setConnected(false);
  // Never let the feature silently die — fall back to polling.
  if (!pollRef.current) pollRef.current = setInterval(loadPatients, 5000);
});

socket.on("scan:created", (payload) => {
  loadPatients();
  if (payload.patient?.id === selectedIdRef.current) loadDetail(payload.patient.id);
  if (pairOpenRef.current) setJustPaired(payload.patient?.name);
});

// Heartbeat every 30s so dead sockets don't sit silently connected.
heartbeatRef.current = setInterval(() => socket.emit("ping:heartbeat"), 30000);
```

### Key Learnings
- Compressing the image client-side (`browser-image-compression`, capped at 2MB / 2000px) before upload cuts OCR latency noticeably on a typical phone photo, and is kinder to a clinic's wifi
- A polling fallback on `socket.disconnect()` turned out to be the single most important line for reliability — a doctor on a flaky connection still sees new patients within 5 seconds instead of a dashboard that just looks "stuck"
- Because the QR overlay and the dashboard's `scan:created` listener share the same component tree, I can flip the overlay straight to a "✅ paired with {name}" state the instant the socket event for *that specific pairing session* arrives — no need to ask the patient "did it work?"

---

## Day 16 — 02 June 2026 · Colleqo — FCM Push Notification System

### What I Did
Built end-to-end Firebase Cloud Messaging (FCM) push notifications for Colleqo. The system has two parts: backend dispatch and frontend subscription.

On the backend (Hono/TypeScript), I created a `push_subscriptions` table keyed by `user_id` — each row stores the FCM device token alongside tenant context. A `dispatchPush()` helper wraps the FCM HTTP v1 API call; critically, it uses **graceful degradation** — if a user has no subscription row, the function returns silently rather than throwing. This matters because push permission is optional; the app must not break if a student declines the browser prompt.

On the frontend (Vanilla JS), I wired up a Service Worker `push` event handler that calls `self.registration.showNotification()`, and added push subscription logic that runs at login: it requests `Notification.permission`, calls `serviceWorkerRegistration.pushManager.subscribe()` with the VAPID public key, then POSTs the resulting token to `POST /api/notifications/subscribe`. A notification bell in the header checks `GET /api/notifications?unread=true` and renders a red badge with the unread count.

The announcement route (`POST /api/announcements`) was updated to call `dispatchPush()` for each targeted user after inserting the DB row — so every new announcement automatically fires a push to relevant devices.

### Key Code / Implementation

```typescript
// src/routes/notifications.ts — FCM dispatch helper + subscribe endpoint

export async function dispatchPush(
  db: D1Database,
  userId: string,
  title: string,
  body: string,
  fcmServerKey: string
): Promise<void> {
  const sub = await db
    .prepare('SELECT fcm_token FROM push_subscriptions WHERE user_id = ?')
    .bind(userId)
    .first<{ fcm_token: string }>();

  if (!sub?.fcm_token) return; // graceful degradation — user not subscribed

  await fetch('https://fcm.googleapis.com/fcm/send', {
    method: 'POST',
    headers: {
      Authorization: `key=${fcmServerKey}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      to: sub.fcm_token,
      notification: { title, body },
      data: { click_action: 'FLUTTER_NOTIFICATION_CLICK' },
    }),
  });
}

// Store or update FCM token for logged-in user
notifications.post('/subscribe', async (c) => {
  const user = c.get('user');
  const { fcm_token } = await c.req.json<{ fcm_token: string }>();

  await c.env.DB.prepare(`
    INSERT INTO push_subscriptions (id, user_id, tenant_id, fcm_token, created_at)
    VALUES (?, ?, ?, ?, datetime('now'))
    ON CONFLICT(user_id) DO UPDATE SET fcm_token = excluded.fcm_token
  `).bind(crypto.randomUUID(), user.userId, user.tenantId, fcm_token).run();

  return successResponse(c, { subscribed: true });
});
```

```javascript
// sw.js — Service Worker push event handler
self.addEventListener('push', (event) => {
  const data  = event.data?.json() ?? {};
  const title = data.notification?.title ?? 'Colleqo';
  const body  = data.notification?.body  ?? '';

  event.waitUntil(
    self.registration.showNotification(title, {
      body,
      icon:    '/static/icons/icon-192.png',
      badge:   '/static/icons/badge-72.png',
      vibrate: [200, 100, 200],
      data:    { url: data.data?.url ?? '/' },
    })
  );
});

self.addEventListener('notificationclick', (event) => {
  event.notification.close();
  event.waitUntil(clients.openWindow(event.notification.data.url));
});
```

```javascript
// screens/login.js — push subscription after successful auth
async function subscribeToPush(token) {
  if (!('serviceWorker' in navigator) || !('PushManager' in window)) return;

  const permission = await Notification.requestPermission();
  if (permission !== 'granted') return; // user declined — that's fine

  const reg = await navigator.serviceWorker.ready;
  const sub = await reg.pushManager.subscribe({
    userVisibleOnly: true,
    applicationServerKey: VAPID_PUBLIC_KEY,
  });

  await API.post('/api/notifications/subscribe', { fcm_token: sub.endpoint }, token);
}
```

### Key Learnings
- `ON CONFLICT(user_id) DO UPDATE SET` is the SQLite upsert pattern — one statement handles both first-time subscribe and token refresh
- FCM tokens can rotate (especially on Android) — always upsert rather than insert to keep the stored token current
- `event.waitUntil()` in Service Worker event handlers keeps the SW alive until the Promise resolves — missing it can cause the notification to never appear
- Fan-out (one announcement → many pushes) should ideally be queued; for now, a sequential loop in the announcement handler is acceptable at internship scale

---

## Day 17 — 03 June 2026 · Colleqo — Razorpay Payment Gateway

### What I Did
Integrated the Razorpay payment gateway into Colleqo's fees module. The flow mirrors real e-commerce: backend creates an order with a server-side HMAC signature, the frontend opens the Razorpay checkout modal, and the backend verifies the payment signature before marking the fee as paid.

Implemented three backend endpoints: `POST /api/fees/create-order` (calls Razorpay Orders API via `fetch()` with Basic Auth using `btoa(key:secret)`), `POST /api/fees/verify-payment` (verifies the HMAC-SHA256 signature client-side using `crypto.subtle`), and `GET /api/fees/receipt/:id` (returns a JSON receipt). Also added a `payment_transactions` table to log every gateway interaction for audit purposes.

On the frontend (`screens/fees.js`), clicking "Pay Now" calls create-order, then opens the `Razorpay` constructor (loaded via CDN script tag) with the order ID and amount. The `handler` callback posts the payment IDs back to verify-payment and shows a success toast on confirmation.

A **mock mode** flag in `.dev.vars` bypasses the actual Razorpay API during local development — create-order returns a fake order_id and verify-payment skips signature checking — so the entire payment flow can be tested without real credentials.

### Key Code / Implementation

```typescript
// src/routes/fees.ts — create-order + HMAC signature verification

fees.post('/create-order', async (c) => {
  const user = c.get('user');
  const { fee_record_id } = await c.req.json<{ fee_record_id: string }>();

  const fee = await c.env.DB
    .prepare('SELECT * FROM fee_records WHERE id = ? AND student_id = ? AND tenant_id = ?')
    .bind(fee_record_id, user.userId, user.tenantId)
    .first<FeeRecord>();

  if (!fee) return errorResponse(c, 'Fee record not found', 404);

  const amountPaise = Math.round((fee.amount - fee.amount_paid) * 100);

  // Mock mode for local dev
  if (c.env.RAZORPAY_MOCK === 'true') {
    return successResponse(c, {
      order_id: `mock_order_${Date.now()}`,
      amount:   amountPaise,
      key_id:   'rzp_test_mock',
    });
  }

  const auth  = btoa(`${c.env.RAZORPAY_KEY_ID}:${c.env.RAZORPAY_KEY_SECRET}`);
  const res   = await fetch('https://api.razorpay.com/v1/orders', {
    method:  'POST',
    headers: { Authorization: `Basic ${auth}`, 'Content-Type': 'application/json' },
    body:    JSON.stringify({
      amount:   amountPaise,
      currency: 'INR',
      receipt:  `fee_${fee_record_id}`,
      notes:    { fee_record_id, student_id: user.userId },
    }),
  });
  const order = await res.json() as { id: string };

  return successResponse(c, { order_id: order.id, amount: amountPaise, key_id: c.env.RAZORPAY_KEY_ID });
});

fees.post('/verify-payment', async (c) => {
  const { order_id, payment_id, signature, fee_record_id } =
    await c.req.json<{
      order_id: string; payment_id: string;
      signature: string; fee_record_id: string;
    }>();

  // HMAC-SHA256 — Razorpay signs `order_id|payment_id` with the key secret
  const encoder = new TextEncoder();
  const key = await crypto.subtle.importKey(
    'raw', encoder.encode(c.env.RAZORPAY_KEY_SECRET),
    { name: 'HMAC', hash: 'SHA-256' }, false, ['sign']
  );
  const sigBuffer  = await crypto.subtle.sign('HMAC', key, encoder.encode(`${order_id}|${payment_id}`));
  const expectedSig = Array.from(new Uint8Array(sigBuffer))
    .map(b => b.toString(16).padStart(2, '0')).join('');

  if (c.env.RAZORPAY_MOCK !== 'true' && expectedSig !== signature) {
    return errorResponse(c, 'Payment signature mismatch', 400);
  }

  await c.env.DB.prepare(`
    UPDATE fee_records
    SET status = 'paid', amount_paid = amount,
        payment_gateway = 'razorpay', payment_id = ?, paid_at = datetime('now')
    WHERE id = ?
  `).bind(payment_id, fee_record_id).run();

  await c.env.DB.prepare(`
    INSERT INTO payment_transactions
      (id, fee_record_id, gateway, gateway_order_id, gateway_payment_id, status, created_at)
    VALUES (?, ?, 'razorpay', ?, ?, 'success', datetime('now'))
  `).bind(crypto.randomUUID(), fee_record_id, order_id, payment_id).run();

  return successResponse(c, { verified: true });
});
```

```javascript
// screens/fees.js — Pay Now button + Razorpay modal
register('fees', async (app) => {
  const { data: fees } = await API.get('/api/fees/my-fees');

  fees.forEach(fee => {
    if (fee.status !== 'pending') return;
    const card    = app.querySelector(`[data-fee-id="${fee.id}"]`);
    const payBtn  = card?.querySelector('.pay-btn');
    if (!payBtn) return;

    payBtn.addEventListener('click', async () => {
      const { data } = await API.post('/api/fees/create-order', { fee_record_id: fee.id });

      const rzp = new window.Razorpay({
        key:        data.key_id,
        order_id:   data.order_id,
        amount:     data.amount,
        currency:   'INR',
        name:       'Colleqo',
        description: `${fee.category} — ${fee.semester ?? ''}`,
        theme:      { color: '#00D9FF' },
        handler: async (response) => {
          await API.post('/api/fees/verify-payment', {
            order_id:      data.order_id,
            payment_id:    response.razorpay_payment_id,
            signature:     response.razorpay_signature,
            fee_record_id: fee.id,
          });
          showToast('Payment successful! 🎉', 'success');
          router.navigate('fees'); // refresh the screen
        },
      });
      rzp.open();
    });
  });
});
```

### Key Learnings
- Razorpay's HMAC is computed server-side from `order_id|payment_id` — never trust the client to do this; always re-verify on the backend
- `crypto.subtle` is available natively in Cloudflare Workers — zero npm packages needed for HMAC-SHA256
- `btoa(key:secret)` produces the Basic Auth header string; this is the same pattern Twilio and many REST APIs use
- A mock mode flag in env vars is worth the 10 extra lines — it makes testing fee flows in local dev instant without burning real Razorpay test credits

---

## Day 18 — 04 June 2026 · Colleqo — Twilio SMS/WhatsApp Alert System

### What I Did
Built a full Twilio messaging integration for SMS and WhatsApp alerts. The core is a `sendTwilioMessage()` helper that uses `fetch()` directly to the Twilio REST API with Basic Auth (`btoa(SID:token)`) and URL-encoded form body — no Twilio SDK needed, keeping the Cloudflare Workers bundle minimal.

Auto-trigger logic: when faculty or admin calls the attendance route to close a session, the worker now checks for students below 75% attendance and fires SMS alerts automatically. Similarly, on fee payment confirmation the student receives a WhatsApp receipt message. The channel routing (SMS vs WhatsApp) is as simple as prefixing the phone number — `whatsapp:+91…` for WhatsApp, plain number for SMS.

Added an **Alerts Admin Panel** screen (`screens/alerts_admin.js`) where college admins can compose custom bulk messages, select target groups (all students, specific batch, low-attendance students), preview recipient count, and fire. Each dispatched message is logged to a `sms_whatsapp_logs` table with status, delivery timestamp, and channel.

Also implemented per-student alert preferences — students can toggle SMS and WhatsApp alerts on/off from their profile, stored in a `users.alert_prefs` JSON column — the dispatch function respects these flags before calling Twilio.

### Key Code / Implementation

```typescript
// src/lib/twilio.ts — core helper

export async function sendTwilioMessage(
  accountSid: string,
  authToken:  string,
  from:       string,
  to:         string,
  body:       string,
  channel:    'sms' | 'whatsapp' = 'sms'
): Promise<boolean> {
  const fromNumber = channel === 'whatsapp' ? `whatsapp:${from}` : from;
  const toNumber   = channel === 'whatsapp' ? `whatsapp:${to}`   : to;

  const auth = btoa(`${accountSid}:${authToken}`);

  const res = await fetch(
    `https://api.twilio.com/2010-04-01/Accounts/${accountSid}/Messages.json`,
    {
      method:  'POST',
      headers: {
        Authorization:  `Basic ${auth}`,
        'Content-Type': 'application/x-www-form-urlencoded',
      },
      body: new URLSearchParams({
        From: fromNumber,
        To:   toNumber,
        Body: body,
      }).toString(),
    }
  );

  return res.ok;
}
```

```typescript
// src/routes/alerts.ts — auto low-attendance SMS trigger

alerts.post('/attendance-alert', async (c) => {
  const user = c.get('user');
  if (user.role !== 'faculty' && user.role !== 'college_admin') {
    return errorResponse(c, 'Unauthorized', 403);
  }

  // Students below 75% attendance in this tenant
  const { results } = await c.env.DB.prepare(`
    SELECT u.phone, u.full_name, u.alert_prefs,
           ROUND(
             SUM(CASE WHEN ar.status = 'present' THEN 100.0 ELSE 0 END) / COUNT(*), 1
           ) AS pct
    FROM attendance_records ar
    JOIN users u ON ar.student_id = u.id
    WHERE ar.tenant_id = ? AND u.tenant_id = ? AND u.role = 'student'
    GROUP BY ar.student_id
    HAVING pct < 75
  `).bind(user.tenantId, user.tenantId).all<{
    phone: string; full_name: string; alert_prefs: string; pct: number;
  }>();

  let sent = 0;
  for (const student of results) {
    const prefs = JSON.parse(student.alert_prefs ?? '{}');
    if (prefs.sms === false) continue; // respect user preference

    const msg = `Dear ${student.full_name}, your attendance is ${student.pct}% — below the 75% requirement. Please attend classes regularly. — Colleqo`;

    const ok = await sendTwilioMessage(
      c.env.TWILIO_ACCOUNT_SID,
      c.env.TWILIO_AUTH_TOKEN,
      c.env.TWILIO_PHONE_NUMBER,
      student.phone,
      msg,
      'sms'
    );

    if (ok) {
      sent++;
      // Log to audit table
      await c.env.DB.prepare(`
        INSERT INTO sms_whatsapp_logs (id, tenant_id, recipient_phone, channel, body, status, created_at)
        VALUES (?, ?, ?, 'sms', ?, 'sent', datetime('now'))
      `).bind(crypto.randomUUID(), user.tenantId, student.phone, msg).run();
    }
  }

  return successResponse(c, { sent, total: results.length });
});
```

### Key Learnings
- Twilio's REST API accepts `application/x-www-form-urlencoded` — `new URLSearchParams({…}).toString()` is the cleanest way to build that body without a library
- WhatsApp via Twilio just needs the `whatsapp:` prefix — same API endpoint, same auth, different From/To format
- `btoa()` for Basic Auth works in Cloudflare Workers (and browsers) without any imports — no need for `Buffer.from()`
- Checking `alert_prefs` before dispatching is good UX and also reduces Twilio bill — always give users an opt-out

---

## Day 19 — 05 June 2026 · Colleqo — AI-Powered Analytics Dashboard

### What I Did
Built the analytics dashboard (`screens/analytics.js` + `src/routes/analytics.ts`) — the most data-heavy feature of the week. The "AI" is rule-based: a `generateInsights()` function that runs deterministic checks on aggregated DB data and produces plain-English observations, avoiding the latency and cost of a real LLM at this stage.

The **at-risk score** formula — `attendance × 0.4 + exam_avg × 0.4 + fee_status × 0.2` — weighs academics equally and fee compliance lightly. Students scoring below 60 appear in the at-risk table. This required four correlated sub-queries in a single SQL statement pulling from `attendance_records`, `results`, `fee_records`, and `users` — the most complex query I've written in this project.

Frontend renders three Chart.js visualisations: a 30-day attendance trend (line chart with fill), a department comparison bar chart, and a fee collection donut. A CSV export button assembles the at-risk table as a comma-separated string and triggers a download via a temporary `<a>` element with a `data:text/csv` href — no server round-trip needed.

The insights panel fires alongside the charts: if attendance dropped more than 5% week-over-week, or more than 10 students are at risk, or a department has average attendance below 65%, the engine surfaces a highlighted card with the observation and a suggested action.

### Key Code / Implementation

```typescript
// src/routes/analytics.ts — at-risk query + insight engine

analytics.get('/insights', async (c) => {
  const user = c.get('user');

  // 30-day attendance trend
  const { results: trend } = await c.env.DB.prepare(`
    SELECT DATE(created_at) AS date,
           ROUND(AVG(CASE WHEN status = 'present' THEN 100.0 ELSE 0 END), 1) AS avg_pct
    FROM attendance_records
    WHERE tenant_id = ? AND created_at >= datetime('now', '-30 days')
    GROUP BY DATE(created_at)
    ORDER BY date ASC
  `).bind(user.tenantId).all<{ date: string; avg_pct: number }>();

  // At-risk students — risk score formula
  const { results: atRisk } = await c.env.DB.prepare(`
    SELECT u.id, u.full_name, u.email,
      COALESCE(att.pct, 0) AS attendance_pct,
      COALESCE(ex.avg_marks, 0) AS exam_avg,
      CASE WHEN f.status = 'paid' THEN 100.0 ELSE 0.0 END AS fee_score,
      ROUND(
        COALESCE(att.pct, 0) * 0.4 +
        COALESCE(ex.avg_marks, 0) * 0.4 +
        (CASE WHEN f.status = 'paid' THEN 100.0 ELSE 0.0 END) * 0.2,
        1
      ) AS risk_score
    FROM users u
    LEFT JOIN (
      SELECT student_id,
             ROUND(SUM(CASE WHEN status='present' THEN 100.0 ELSE 0 END) / COUNT(*), 1) AS pct
      FROM attendance_records WHERE tenant_id = ? GROUP BY student_id
    ) att ON att.student_id = u.id
    LEFT JOIN (
      SELECT student_id,
             ROUND(AVG(marks_obtained * 100.0 / NULLIF(total_marks, 0)), 1) AS avg_marks
      FROM results WHERE tenant_id = ? GROUP BY student_id
    ) ex ON ex.student_id = u.id
    LEFT JOIN fee_records f ON f.student_id = u.id AND f.tenant_id = ?
    WHERE u.role = 'student' AND u.tenant_id = ?
    HAVING risk_score < 60
    ORDER BY risk_score ASC
    LIMIT 20
  `).bind(user.tenantId, user.tenantId, user.tenantId, user.tenantId)
    .all<{ id: string; full_name: string; email: string; attendance_pct: number; exam_avg: number; fee_score: number; risk_score: number }>();

  const insights = generateInsights(trend, atRisk);

  return successResponse(c, { trend, at_risk: atRisk, insights });
});

function generateInsights(trend: { avg_pct: number }[], atRisk: { risk_score: number }[]): string[] {
  const msgs: string[] = [];

  if (atRisk.length > 10) {
    msgs.push(`⚠️ ${atRisk.length} students are at high academic risk — immediate intervention recommended.`);
  }

  if (trend.length >= 14) {
    const recent   = trend.slice(-7).reduce((s, d) => s + d.avg_pct, 0) / 7;
    const previous = trend.slice(-14, -7).reduce((s, d) => s + d.avg_pct, 0) / 7;
    const drop     = previous - recent;
    if (drop > 5) {
      msgs.push(`📉 Attendance dropped by ${drop.toFixed(1)}% over the last 7 days. Consider sending SMS reminders.`);
    } else if (recent > previous + 3) {
      msgs.push(`📈 Attendance improved by ${(recent - previous).toFixed(1)}% this week. Keep it up!`);
    }
  }

  if (atRisk.length === 0) {
    msgs.push('✅ No students currently at high risk. Institution performance is on track.');
  }

  return msgs;
}
```

```javascript
// screens/analytics.js — Chart.js trend + CSV export

register('analytics', async (app) => {
  const { data } = await API.get('/api/analytics/insights');

  // Attendance trend — line chart
  const trendCtx = app.querySelector('#trend-chart').getContext('2d');
  new Chart(trendCtx, {
    type: 'line',
    data: {
      labels:   data.trend.map(d => d.date),
      datasets: [{
        label:           'Avg Attendance %',
        data:            data.trend.map(d => d.avg_pct),
        borderColor:     '#00D9FF',
        backgroundColor: 'rgba(0,217,255,0.08)',
        fill:            true,
        tension:         0.4,
        pointRadius:     3,
      }],
    },
    options: { scales: { y: { min: 0, max: 100 } } },
  });

  // CSV export — no server round-trip
  app.querySelector('#export-risk-btn').addEventListener('click', () => {
    const header = 'Name,Email,Attendance %,Exam Avg,Risk Score';
    const rows   = data.at_risk.map(s =>
      [s.full_name, s.email, s.attendance_pct, s.exam_avg, s.risk_score].join(',')
    );
    const csv = [header, ...rows].join('\n');

    const a   = document.createElement('a');
    a.href    = `data:text/csv;charset=utf-8,${encodeURIComponent(csv)}`;
    a.download = `at-risk-students-${new Date().toISOString().slice(0,10)}.csv`;
    a.click();
  });
});
```

### Key Learnings
- `NULLIF(total_marks, 0)` prevents division-by-zero in the exam average sub-query — a common SQL trap
- SQLite's `HAVING` clause runs after `GROUP BY` but before `ORDER BY` and `LIMIT` — using it to filter on a calculated column (`HAVING risk_score < 60`) is valid and efficient
- `data:text/csv;charset=utf-8,` + `encodeURIComponent()` is the cleanest client-side CSV download — no Blob API, no dependencies
- Rule-based insight engines look "AI" enough for institutional dashboards at this scale, and they're deterministic and debuggable

---

## Day 20 — 06 June 2026 · Colleqo — Bulk CSV Import/Export + Parent Portal

### What I Did

**Bulk CSV Import/Export:**
Built a complete data pipeline with zero external dependencies. The CSV parser handles quoted fields with embedded commas using a character-by-character state machine. Import validates required column headers first, then iterates rows — each row inserts a `users` record plus a `student_profiles` record with a batch lookup. Auto-generated passwords follow `${roll_number}@Colleqo` convention so the admin can share them with students. Any row that fails (duplicate email, missing batch) is collected into an `errors[]` array returned to the frontend rather than aborting the whole import.

CSV export goes the other direction: query all students/faculty from D1, map to CSV rows, and return with `Content-Disposition: attachment; filename=...`.

**Parent Portal:**
Parents link to their children via a 6-digit link code the student generates (valid 24 hours, stored in `parent_link_codes`, marked `used=1` after one use). Once linked, the parent dashboard shows a tabbed view: Attendance (last 30 days calendar), Results (subject-wise marks), Fees (outstanding + paid), and Timetable (weekly grid). A "Print Report" button uses `window.print()` with `@media print` CSS that hides sidebar/header and formats the content into a clean A4 printable layout — no PDF generation needed.

All four child data queries (attendance, results, fees, timetable) run in `Promise.all()` to keep the page load snappy.

### Key Code / Implementation

```typescript
// src/routes/bulk_data.ts — CSV parser + student bulk import

function parseCSV(csvText: string): string[][] {
  return csvText.trim().split('\n').map(line => {
    const fields: string[] = [];
    let inQuote = false, field = '';
    for (const ch of line) {
      if (ch === '"')          { inQuote = !inQuote; continue; }
      if (ch === ',' && !inQuote) { fields.push(field.trim()); field = ''; continue; }
      field += ch;
    }
    fields.push(field.trim());
    return fields;
  });
}

bulk_data.post('/import/students', async (c) => {
  const user = c.get('user');
  if (user.role !== 'college_admin') return errorResponse(c, 'Unauthorized', 403);

  const formData = await c.req.formData();
  const file     = formData.get('file') as File;
  const rows     = parseCSV(await file.text());

  const headers  = rows[0].map(h => h.toLowerCase().replace(/\s+/g, '_'));
  const required = ['name', 'email', 'phone', 'roll_number', 'batch'];
  const missing  = required.filter(col => !headers.includes(col));
  if (missing.length) return errorResponse(c, `Missing columns: ${missing.join(', ')}`, 400);

  const dataRows = rows.slice(1).filter(r => r.some(cell => cell.trim()));
  let inserted = 0;
  const errors: string[] = [];

  for (let i = 0; i < dataRows.length; i++) {
    const rec: Record<string, string> = Object.fromEntries(
      headers.map((h, j) => [h, dataRows[i][j] ?? ''])
    );
    const tempPassword = `${rec.roll_number}@Colleqo`;

    try {
      const userId = crypto.randomUUID();
      await c.env.DB.prepare(`
        INSERT INTO users (id, full_name, email, phone, role, password_hash, tenant_id, created_at)
        VALUES (?, ?, ?, ?, 'student', ?, ?, datetime('now'))
      `).bind(userId, rec.name, rec.email, rec.phone, tempPassword, user.tenantId).run();

      const batch = await c.env.DB
        .prepare('SELECT id FROM batches WHERE name = ? AND tenant_id = ?')
        .bind(rec.batch, user.tenantId)
        .first<{ id: string }>();

      await c.env.DB.prepare(`
        INSERT INTO student_profiles (id, user_id, tenant_id, roll_number, batch_id)
        VALUES (?, ?, ?, ?, ?)
      `).bind(crypto.randomUUID(), userId, user.tenantId, rec.roll_number, batch?.id ?? null).run();

      inserted++;
    } catch (e: any) {
      errors.push(`Row ${i + 2}: ${e.message}`);
    }
  }

  return successResponse(c, { inserted, errors, total: dataRows.length });
});
```

```typescript
// src/routes/parent.ts — link code generation + child data

parent.post('/generate-link-code', async (c) => {
  const user = c.get('user');
  if (user.role !== 'student') return errorResponse(c, 'Only students can generate link codes', 403);

  const code      = Math.floor(100_000 + Math.random() * 900_000).toString();
  const expiresAt = new Date(Date.now() + 24 * 60 * 60 * 1000).toISOString();

  await c.env.DB.prepare(`
    INSERT INTO parent_link_codes (id, student_id, tenant_id, code, expires_at, used, created_at)
    VALUES (?, ?, ?, ?, ?, 0, datetime('now'))
    ON CONFLICT(student_id) DO UPDATE SET code = excluded.code, expires_at = excluded.expires_at, used = 0
  `).bind(crypto.randomUUID(), user.userId, user.tenantId, code, expiresAt).run();

  return successResponse(c, { code, expires_at: expiresAt });
});

parent.get('/child/:studentId', async (c) => {
  const user      = c.get('user');
  const studentId = c.req.param('studentId');

  const linked = await c.env.DB
    .prepare('SELECT 1 FROM parent_links WHERE parent_id = ? AND student_id = ? AND tenant_id = ?')
    .bind(user.userId, studentId, user.tenantId)
    .first();
  if (!linked) return errorResponse(c, 'Not authorized', 403);

  const [attendance, results, fees, timetable] = await Promise.all([
    c.env.DB.prepare(`
      SELECT DATE(created_at) AS date, status FROM attendance_records
      WHERE student_id = ? ORDER BY created_at DESC LIMIT 60
    `).bind(studentId).all(),
    c.env.DB.prepare(`
      SELECT subject_name, marks_obtained, total_marks, grade, semester
      FROM results WHERE student_id = ? ORDER BY created_at DESC
    `).bind(studentId).all(),
    c.env.DB.prepare(`
      SELECT category, amount, amount_paid, status FROM fee_records
      WHERE student_id = ? ORDER BY created_at DESC
    `).bind(studentId).all(),
    c.env.DB.prepare(`
      SELECT day_of_week, start_time, end_time, subject_name, room_no FROM timetable_slots
      WHERE batch_id = (SELECT batch_id FROM student_profiles WHERE user_id = ?)
      ORDER BY day_of_week, start_time
    `).bind(studentId).all(),
  ]);

  return successResponse(c, {
    attendance: attendance.results,
    results:    results.results,
    fees:       fees.results,
    timetable:  timetable.results,
  });
});
```

```css
/* Print CSS for parent progress report */
@media print {
  .sidebar, .app-header, .bottom-nav, .no-print { display: none !important; }
  .main-content { margin: 0; padding: 0; }
  .print-report { font-family: serif; color: #000; }
  .print-report table { width: 100%; border-collapse: collapse; }
  .print-report th, .print-report td { border: 1px solid #000; padding: 6px 10px; }
}
```

### Key Learnings
- A character-by-character CSV parser is ~25 lines and handles edge cases (quoted commas) cleanly — `split(',')` is wrong for production CSV
- Collecting errors per-row instead of aborting the whole import is critical for bulk operations — an admin shouldn't have to fix one typo and re-upload 200 rows
- `Promise.all()` with four D1 queries saves ~3× the latency vs sequential awaits — important for a dashboard page load
- `window.print()` + print CSS is genuinely underrated — produces clean PDF-quality reports without any server-side PDF generation

---

## Day 21 — 07 June 2026 · Professional Desktop Layout + Rebrand + SaaS Landing Page

### What I Did
The biggest UI overhaul of the project — three interrelated deliverables in one day.

**Professional Desktop Layout:** Replaced the mobile-first full-screen layout with a Notion/Linear-style desktop shell: 240px fixed sidebar with role-based nav icons + labels, a 60px sticky header with breadcrumb navigation and user menu (name + logout), and a scrollable main content area that fills the remaining viewport. CSS Grid drives the layout — `grid-template-columns: 240px 1fr`. On mobile (≤768px), the sidebar collapses off-screen and a hamburger button toggles it via a CSS class — preserving the existing mobile experience without a separate code path.

**Rebrand CampusOS → Colleqo:** Updated every visible string, `<title>`, `<meta>` tag, `manifest.json` (name, short_name), and brand mark. The Vite build config's HTML plugin handles the title injection, so it propagates to all entry HTML files automatically.

**SaaS Landing Page:** Built a standalone `landing.html` at the domain root for Colleqo's SaaS offering. Sections: hero with CTA, feature highlights grid, comparison table vs TCS iON and Fedena, three-tier pricing (₹6,999 / ₹12,999 / ₹19,999 per month), FAQ accordion, and a contact/demo form. Deployed to `campusos.pages.dev` via a GitHub Actions workflow that triggers on push to `main`.

### Key Code / Implementation

```css
/* core/layout.css — professional desktop shell */
:root {
  --sidebar-w:      240px;
  --header-h:       60px;
  --surface-1:      #111318;
  --border-color:   rgba(255,255,255,0.08);
  --nav-active-bg:  rgba(0,217,255,0.12);
  --nav-active-fg:  #00D9FF;
}

.app-layout {
  display:               grid;
  grid-template-columns: var(--sidebar-w) 1fr;
  grid-template-rows:    var(--header-h) 1fr;
  height:                100vh;
  overflow:              hidden;
}

/* Fixed sidebar — spans both grid rows */
.sidebar {
  grid-row:   1 / -1;
  position:   fixed;
  inset:      0 auto 0 0;
  width:      var(--sidebar-w);
  background: var(--surface-1);
  border-right: 1px solid var(--border-color);
  overflow-y: auto;
  z-index:    100;
}

.sidebar .brand { display: flex; align-items: center; gap: 10px; padding: 18px 16px; }
.sidebar .brand img  { width: 28px; border-radius: 6px; }
.sidebar .brand span { font-weight: 700; font-size: 16px; color: #fff; }

.nav-item a {
  display:     flex;
  align-items: center;
  gap:         10px;
  padding:     9px 16px;
  border-radius: 8px;
  color:       rgba(255,255,255,0.6);
  text-decoration: none;
  transition:  background 0.15s, color 0.15s;
}
.nav-item a:hover   { background: rgba(255,255,255,0.05); color: #fff; }
.nav-item.active a  { background: var(--nav-active-bg);  color: var(--nav-active-fg); }

/* Sticky header */
.app-header {
  grid-column: 2;
  position:    sticky;
  top:         0;
  height:      var(--header-h);
  background:  var(--surface-1);
  border-bottom: 1px solid var(--border-color);
  display:     flex;
  align-items: center;
  padding:     0 24px;
  gap:         12px;
  z-index:     99;
}

.breadcrumb { display: flex; align-items: center; gap: 6px; font-size: 13px; color: rgba(255,255,255,0.45); }
.breadcrumb .bc-cur { color: #fff; font-weight: 500; }

/* Scrollable content */
.main-content {
  grid-column: 2;
  padding:     24px;
  overflow-y:  auto;
  height:      calc(100vh - var(--header-h));
}

/* Mobile — collapse sidebar */
@media (max-width: 768px) {
  .app-layout   { grid-template-columns: 1fr; }
  .sidebar      { transform: translateX(-100%); transition: transform 0.25s ease; }
  .sidebar.open { transform: translateX(0); }
  .main-content { grid-column: 1; }
}
```

```javascript
// core/ui.js — desktopLayout() wrapper
export function desktopLayout(pageTitle, contentHTML, breadcrumb = []) {
  const user     = State.user;
  const navItems = getNavByRole(user.role);

  const crumbs = ['Colleqo', ...breadcrumb]
    .map((b, i, arr) =>
      i === arr.length - 1
        ? `<span class="bc-cur">${b}</span>`
        : `<span>${b}</span><span class="bc-sep">›</span>`
    ).join('');

  return `
    <div class="app-layout">
      <nav class="sidebar">
        <div class="brand">
          <img src="/static/icons/icon-192.png" alt="Colleqo" />
          <span>Colleqo</span>
        </div>
        <ul class="nav-list">
          ${navItems.map(item => `
            <li class="nav-item ${State.route === item.route ? 'active' : ''}">
              <a href="#${item.route}">
                <i class="fa ${item.icon}"></i>
                <span>${item.label}</span>
              </a>
            </li>`).join('')}
        </ul>
      </nav>

      <header class="app-header">
        <button class="menu-btn" id="sidebar-toggle"><i class="fa fa-bars"></i></button>
        <nav class="breadcrumb">${crumbs}</nav>
        <div style="flex:1"></div>
        <span class="user-name">${user.full_name}</span>
        <button class="btn-ghost" onclick="Auth.logout()">Logout</button>
      </header>

      <main class="main-content">${contentHTML}</main>
    </div>
  `;
}
```

```html
<!-- landing.html — SaaS pricing section -->
<section class="pricing" id="pricing">
  <div class="container">
    <h2>Simple, Transparent Pricing</h2>
    <p class="sub">No setup fees. No hidden charges. Cancel anytime.</p>

    <div class="pricing-grid">
      <div class="plan">
        <h3>Starter</h3>
        <div class="price">₹6,999<span>/mo</span></div>
        <ul>
          <li>✓ Up to 500 students</li>
          <li>✓ Core modules (Attendance, Exams, Fees)</li>
          <li>✓ 1 institution</li>
          <li>✓ Email support</li>
        </ul>
        <a href="#demo" class="btn-outline">Request Demo</a>
      </div>

      <div class="plan featured">
        <span class="badge">Most Popular</span>
        <h3>Professional</h3>
        <div class="price">₹12,999<span>/mo</span></div>
        <ul>
          <li>✓ Up to 2,000 students</li>
          <li>✓ All modules + Parent Portal</li>
          <li>✓ Push, SMS & WhatsApp alerts</li>
          <li>✓ AI analytics dashboard</li>
          <li>✓ Priority support</li>
        </ul>
        <a href="#demo" class="btn-primary">Request Demo</a>
      </div>

      <div class="plan">
        <h3>Enterprise</h3>
        <div class="price">₹19,999<span>/mo</span></div>
        <ul>
          <li>✓ Unlimited students</li>
          <li>✓ Multi-campus / group colleges</li>
          <li>✓ Placements module</li>
          <li>✓ Dedicated account manager</li>
          <li>✓ Custom integrations</li>
        </ul>
        <a href="#contact" class="btn-outline">Contact Sales</a>
      </div>
    </div>
  </div>
</section>
```

### Key Learnings
- CSS Grid `grid-template-columns: 240px 1fr` + `grid-row: 1 / -1` on the sidebar is the cleanest way to build a fixed sidebar that spans full height without `position: fixed` hacks
- `transform: translateX(-100%)` + a toggled class is the right mobile drawer pattern — it preserves layout flow and is hardware-accelerated
- A SaaS landing page benefits from showing pricing upfront — Indian institutional buyers compare numbers first, contact sales second
- GitHub Actions with `wrangler pages deploy` in the workflow file gets CI/CD live in under an hour; the hardest part was adding `CLOUDFLARE_API_TOKEN` and `CLOUDFLARE_ACCOUNT_ID` as repo secrets

---

## 📊 Week 3 Summary

| Metric | Count |
|--------|-------|
| **Features Shipped** | 8 major features |
| **New DB Tables** | 12 |
| **API Endpoints Added** | 25+ |
| **Frontend Screens** | 6 new screens |
| **Platforms Deployed** | Cloudflare Pages (CI/CD) |

**What went well:** The Razorpay HMAC verification using native `crypto.subtle` (no npm package) was a clean win — stays within Cloudflare Workers' zero-dependency philosophy. Bulk CSV import with per-row error collection (vs abort-on-first-error) turned out to be genuinely useful during self-testing. The `Promise.all()` pattern on the Parent Portal child data endpoint cut the API response time noticeably.

**What was hard:** The at-risk query with four correlated sub-queries was the trickiest SQL of the internship so far — `NULLIF` for division-by-zero guard, `HAVING` on a derived column, and getting tenant isolation right across all four sub-selects. Took 3 iterations before the results were correct. The CSS desktop layout also required careful testing to ensure the mobile experience wasn't broken — the `@media` breakpoint approach worked cleanly but the sidebar overlay on mobile needed an explicit `z-index` stack.

**Next week:** Week 4 planning — likely email notification system (SendGrid/Resend), student/faculty mobile app exploration (React Native), and further polish on the analytics dashboard with real data visualisations.
