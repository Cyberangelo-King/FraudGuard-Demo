# FraudGuard — Setup & Deployment Guide

## What You've Got

Six HTML files forming a complete fraud detection simulation:

| File | Purpose |
|------|---------|
| `index.html` | Landing page — overview & navigation |
| `device-owner.html` | Device 1: Card owner's main device |
| `device-fraudster.html` | Device 2: Fraudster's device |
| `device-secondary.html` | Device 3: Owner's secondary device |
| `dashboard.html` | Admin dashboard (bank view) |
| `model.html` | Model showcase & research |

---

## Step 1 — Supabase Setup

### 1a. Create the `transactions` table

Go to your Supabase project → **SQL Editor** → run this:

```sql
create table if not exists transactions (
  id uuid default gen_random_uuid() primary key,
  created_at timestamptz default now(),
  device_id text,
  device_label text,
  cardholder text,
  card_last4 text,
  amount numeric,
  merchant text,
  status text,
  fraud_score numeric,
  ip_address text,
  location text,
  device_fingerprint text,
  owner_response text,
  resolved_at timestamptz
);

-- Enable Realtime on the table
alter publication supabase_realtime add table transactions;
```

### 1b. Set Row Level Security (RLS)

If RLS is enabled on your project (it usually is), add a policy to allow anon reads and inserts:

```sql
-- Allow anonymous select
create policy "allow_anon_select" on transactions
  for select using (true);

-- Allow anonymous insert
create policy "allow_anon_insert" on transactions
  for insert with check (true);

-- Allow anonymous update
create policy "allow_anon_update" on transactions
  for update using (true);

-- Allow anonymous delete (for the dashboard Clear button)
create policy "allow_anon_delete" on transactions
  for delete using (true);
```

Or, for a demo-only project, you can simply **disable RLS** on the table:

```sql
alter table transactions disable row level security;
```

---

## Step 2 — Add Your Chart Images

The `model.html` page references three chart images that must be placed in the **same folder** as your HTML files:

- `class_distribution.png`
- `evaluation_results.png`
- `shap_summary.png`

These were uploaded as part of your project — place them alongside the HTML files before deploying.

---

## Step 3 — Deploy to Netlify

1. Create a folder on your computer (e.g. `fraudguard/`)
2. Copy all 6 HTML files + 3 PNG images into it
3. Go to [netlify.com](https://netlify.com) → **Add new site → Deploy manually**
4. Drag and drop the entire `fraudguard/` folder onto the Netlify drop zone
5. Your site goes live instantly at a `*.netlify.app` URL

That's it — no build step, no config files needed.

---

## Step 4 — Demo Walkthrough

### Setup
Open **4 browser tabs** (or windows):
1. `device-owner.html` — This is your "safe phone"
2. `device-fraudster.html` — This is the attacker's device
3. `device-secondary.html` — Your second personal device
4. `dashboard.html` — Keep this visible to see everything in real time

---

### Demo Scenario A — Legitimate Transaction
1. Go to **Device 1** (Owner Main)
2. Click **Pay ₦48,375**
3. Watch the fraud score appear (0.05–0.18 — well below threshold)
4. Transaction auto-approves → green success screen
5. **Dashboard** shows it instantly with a green badge

---

### Demo Scenario B — Fraud Attempt (the dramatic one)
1. Go to **Device 2** (Fraudster)
2. Click **Pay ₦48,375**
3. Device 2 shows "Transaction Flagged" + spinning hold state
4. After ~1.5 seconds, **Device 1** shows a security alert modal:
   > "⚠️ Unusual transaction detected — ₦45,000 at Emunah from an unrecognized device. Was this you?"
5. **Option A:** Click **YES — Approve** on Device 1 within 6 seconds → Device 2 shows "Transaction Approved"
6. **Option B:** Click **NO — It wasn't me** OR wait 6 seconds → Device 2 shows "Transaction Blocked — Fraud Reported"
7. Dashboard updates in real time throughout

---

### Demo Scenario C — Secondary Device Verification
1. Go to **Device 3** (Secondary)
2. Click **Pay ₦48,375**
3. Device 3 enters verification hold
4. After ~1 second, **Device 1** shows confirmation modal:
   > "📱 A transaction of ₦45,000 at Emunah is being initiated from your secondary device. Approve?"
5. Approve or deny → Device 3 updates accordingly
6. Dashboard reflects the result

---

### Reset Between Demos
- Use the **"Reset Demo"** button on any device page to return to checkout state
- Use the **"Clear All Transactions"** button on the dashboard to wipe the feed

---

## Technical Notes

- **Stack:** Vanilla HTML + CSS + JavaScript. No build step. No framework.
- **Backend:** Supabase (Postgres + Realtime broadcast channels)
- **Real-time mechanism:** Supabase Realtime `postgres_changes` for dashboard updates; broadcast channels for device-to-device communication
- **All pages share the same Supabase project** — they communicate through the database and broadcast channels

---

## Project Details

**Author:** Angelo  
**Project:** Final Year Project — Credit Card Fraud Detection Using Ensemble Machine Learning  
**Department:** Computer Science  
**Institution:** Federal University of Technology, Akure (FUTA)  
**Date:** July 2026  

**Model:** Stacking Ensemble (LR + Random Forest + XGBoost → LR Meta-Learner)  
**Dataset:** ULB Credit Card Fraud Detection (284,807 transactions)  
**AUROC:** 0.976 · **F1:** 0.863 · **Threshold:** 0.682
