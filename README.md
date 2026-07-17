# FraudGuard — Real-Time Credit Card Fraud Detection Simulation

**Angelo · Final Year Project · Computer Science · FUTA · July 2026**

A fully browser-based, zero-backend simulation of a real-time fraud detection system. Tabs communicate with each other using the native **BroadcastChannel API** — no Supabase, no database, no build step required.

---

## Files

| File | Purpose |
|---|---|
| `index.html` | Landing page — start here |
| `device-owner.html` | Owner's main device (legitimate, low fraud score) |
| `device-fraudster.html` | Fraudster's device (high fraud score, flagged) |
| `device-secondary.html` | Owner's secondary device (moderate fraud score) |
| `dashboard.html` | Admin dashboard — live transaction feed |
| `model.html` | ML model showcase — metrics, architecture, charts |
| `class_distribution.png` | Chart: class imbalance visualization |
| `evaluation_results.png` | Chart: AUROC, AUPRC, F1, MCC results |
| `shap_summary.png` | Chart: SHAP feature importance |

---

## How to Run

> ⚠️ **BroadcastChannel requires a shared origin.** It does NOT work when files are opened as `file://` from different browser windows. You must serve the files via a local server or Netlify.

### Option A — Local server (recommended for development)

```bash
# Python 3
python -m http.server 8080

# Node.js (npx)
npx serve .

# Then open: http://localhost:8080
```

### Option B — Deploy to Netlify

1. Drag the entire folder into [Netlify Drop](https://app.netlify.com/drop)
2. Netlify gives you a public URL — open it in your browser
3. Open 4 tabs at that URL, each pointing to a different page

---

## Running the Demo (4-Tab Setup)

Open all four pages **in the same browser**, as separate tabs:

| Tab | URL |
|---|---|
| Tab 1 | `device-owner.html` |
| Tab 2 | `device-fraudster.html` |
| Tab 3 | `device-secondary.html` |
| Tab 4 | `dashboard.html` |

Also open `model.html` for the model showcase (no BroadcastChannel needed there).

---

## Demo Walkthrough

### Scenario 1 — Legitimate Transaction (Owner's Device)

1. Go to **Tab 1** (`device-owner.html`)
2. Click **Pay ₦48,375**
3. Transaction is immediately approved (fraud score 0.05–0.18 — below threshold 0.682)
4. Watch **Tab 4** (Dashboard) — a green "Approved" card appears instantly

### Scenario 2 — Fraudster Attempt

1. Go to **Tab 2** (`device-fraudster.html`)
2. Click **Pay ₦48,375**
3. Transaction is flagged immediately (fraud score 0.82–0.97)
4. After 1.5 seconds, a verification request is sent to the owner
5. Switch to **Tab 1** — a modal appears:
   - **"YES — Approve"** → Fraudster tab shows green "Approved"; Dashboard updates to Confirmed
   - **"NO — Deny"** or wait 6 seconds → Fraudster tab shows red "Blocked"; Dashboard updates to Blocked

### Scenario 3 — Secondary Device Payment

1. Go to **Tab 3** (`device-secondary.html`)
2. Click **Pay ₦48,375**
3. Transaction is flagged (fraud score 0.71–0.79 — moderate risk)
4. After 1 second, a different verification request goes to the owner
5. Switch to **Tab 1** — a modal appears with secondary device context:
   - **"YES — Approve"** → Secondary tab shows green "Approved"; Dashboard updates
   - **"NO — Deny"** or wait 6 seconds → Secondary tab shows red "Denied"
   - If **Tab 3** gets no response in 8 seconds → auto-blocks itself

### Resetting

- Click **"Clear All"** on the Dashboard — broadcasts `reset-all` to all tabs
- Or click the individual Reset buttons on each device tab

---

## BroadcastChannel Protocol

All inter-tab communication uses a single channel: `new BroadcastChannel('fraudguard')`

```
tx-new          → Device broadcasts a new transaction
verify-request  → Flagged device asks owner to confirm
owner-decision  → Owner sends approved/denied
tx-update       → Status change broadcast (for dashboard)
reset-all       → Wipe all state across all tabs
```

No SQL. No setup. No external dependencies.

---

## Technical Stack

- Vanilla HTML, CSS, JavaScript — no framework, no build step
- Google Fonts (DM Sans) — CSS only, no JS
- BroadcastChannel API — all browsers (Chrome, Firefox, Safari 15.4+, Edge)
- Zero backend — works entirely in-browser

---

*Built for the FraudGuard Final Year Project — FUTA Computer Science Department, July 2026.*
