# Renewal Dashboard & Segmentation — Manager Guide

**Audience:** Sales leadership, operations, product, engineering  
**Source of truth (business):** *Renewal Dashboard Segmentation Requirements* (Retail Sales, Ombrella, 21 Oct 2026)  
**Related product doc (what we shipped):** `RENEWAL_REMINDER_DASHBOARD_AGENT_RANKING_DOC.md`  
**Last updated:** 29 Oct 2026  

---

## 1. Executive summary

Sales wants the **Admin renewal dashboard** to answer four different business questions—not one blended “overdue” number:

| Segment | Who | Question |
|---------|-----|----------|
| **A — Current renewals** | Active customers not yet overdue | Are we collecting **before** they slip? |
| **B — Recently overdue** | 1–60 days past oldest unpaid due | Are we recovering **early** arrears? |
| **C — Mid overdue** | 61–199 days | What sits in the **middle** between recent and lapsed? |
| **D — Long overdue** | 200+ days | What is true **win-back / lapsed** exposure? |

**Two metrics must never be confused:**

1. **Pipeline / exposure (snapshot)** — “As of report date, how much is due and in which segment?”  
   → Uses **today’s** oldest unpaid due date → segment **A / B / C / D**.

2. **Collection performance (events in a period)** — “How much did we collect in October, and **which segment earned credit**?”  
   → Each payment is tagged with the segment the policy was in **on the payment date**, not where it sits **after** payment.

**Today in the product:**

- **Admin → Dashboard → Renewal stats** (carousel tab): **single pool** KPIs for all agents (or one agent). **Does not** break out A–D or credit collections by segment-at-payment-time.  
- **Admin / Agent → Renewal Reminder** (Overdue / Upcoming tabs): **operational work queues** with **age bands** (similar to B/C/D sub-bands on overdue; upcoming sub-bands similar to A). **Not** the full sales dashboard model (no segment-collected matrix, no unified A–D board).

This document explains the **target** (PDF), **current behaviour**, **gaps**, and **how to demo/mock UI** before full implementation.

---

## 2. Definitions (use exactly as in the PDF)

| Term | Meaning |
|------|--------|
| **Renewal due date** | Date the next premium/renewal is due (`policy_instalments.payment_due_date`). A policy can have **several unpaid** instalments; **one oldest unpaid** drives segment. |
| **Report date** | Date the dashboard is viewed, or the **as-of** end of a filter (default: today). |
| **Days overdue** | `report_date − oldest_unpaid_renewal_due_date`. **≤ 0** = not overdue yet (Segment A territory). |
| **Renewal collected** | Payment received against an unpaid renewal due (row becomes `renewal_payment_received` with `payment_date`). |
| **Active customer (PDF)** | In-force policy whose **oldest unpaid** renewal is **not yet overdue** (days overdue ≤ 0). |

**Scope:** All recurring / renewal products (Education, Savings, Life, HMO, Motor, Travel). **New business** is out of scope.

**System filter (already applied on renewal stats & renewal reminder):** leads with `main_status = policy_active` only.

---

## 3. Segments A–D (non-overlapping, full book)

Each **policy (lead)** sits in **exactly one** segment on a given report date.

| Code | Name | Days overdue (oldest unpaid) | Sub-bands (for drill-down) |
|------|------|------------------------------|----------------------------|
| **A** | Current renewals (active) | **−30 to 0** (due in next 30 days or due today) | Due in 15–30 · Due in 1–14 · Due today |
| **B** | Recently overdue | **1–60** | 1–30 · 31–60 |
| **C** | Mid overdue | **61–199** | 61–120 · 121–199 |
| **D** | Long overdue | **200+** | 200–365 · Over 365 |

**Classification rule (snapshot):**

1. List all **unpaid** renewal instalments for the policy (statuses such as `next_renewal`, `renewal_link_sent`, `renewal_overdue`, `renewal_payment_failed`—confirm with ops).  
2. Take **minimum** `payment_due_date` → **oldest unpaid due**.  
3. Compute days overdue vs report date → assign **one** segment.

**Important:** The **row** in Renewal Reminder may still show **multiple** overdue instalments and total premium; the **segment** is from **oldest unpaid only** (worst ageing), not from “newest” due.

---

## 4. The confusion we clarified: payment changes segment, but collection credit does not move

### 4.1 Snapshot moves when customer pays

When a customer pays renewals, **oldest unpaid due date moves forward** (or vanishes). So **today’s segment** can change:

- Was **C (90 days overdue)** → pays oldest arrears → oldest unpaid might become **B (20 days)** or **A (due in 14 days)** or **none** if fully current.

That is **correct** for “where does this policy sit **now**?”

### 4.2 Collection credit is frozen at payment time (PDF requirement)

For **“Segment B collected ₦X in October”**, you do **not** re-tag old payments when the policy moves to A or C later.

**Rule:** On each `renewal_payment_received` row, look at **payment_date**:

1. Reconstruct **oldest unpaid due** as it was **immediately before** that payment (or equivalent business rule from finance).  
2. Compute **days overdue on payment_date** → that payment counts toward **that segment’s collected** for the period containing `payment_date`.

**Example (simplified amounts):**

| Lead | Unpaid dues on 1 Oct | Segment on 1 Oct |
|------|----------------------|------------------|
| L1 | Due 25 Oct only | **A** (not overdue) |
| L2 | Due 5 Sep (26 days late) | **B** (1–30 band) |
| L3 | Due 1 Jul (92 days late) | **C** |
| L4 | Due 1 Jan 2025 (200+ days late) | **D** |

**15 Oct:** L3 pays ₦10,000 clearing the July due → **October collected** includes **₦10,000 credited to Segment C** (because on 15 Oct, before/against that payment, ageing was in C).  

**1 Nov snapshot:** L3 might now be **A** or **B** if the next unpaid due is near—but **October’s C collected** still includes that ₦10,000.

### 4.3 Multi-instalment lead (one row in UI, one segment)

Lead **X** has unpaid instalments #5–#9 overdue (earliest due 1 Mar) and #10 due next month:

- **Oldest unpaid** = 1 Mar → **days overdue** drives segment (**D** if 200+ days on report date).  
- **Entire lead** appears once in segment **D** on the dashboard snapshot.  
- **Overdue premium** on the row can still show **sum** of all overdue instalments; segment is **not** per-instalment.

**Upcoming tab rule (already implemented):** If **any** instalment is `renewal_overdue`, the lead **does not** appear on Upcoming—even if a future `next_renewal` exists. They stay on **Overdue** until all overdue rows are cleared.

---

## 5. What each dashboard should show (target — PDF)

For **each segment A, B, C, D** (and optional sub-bands), sales expects **three numbers** in the selected period / as-of date:

| Metric | Snapshot meaning | Period meaning |
|--------|------------------|----------------|
| **Renewal due (count / value)** | Policies / ₦ still unpaid in that segment **as of report date** | Optional: dues **falling due** in calendar window |
| **Renewal collected** | N/A as snapshot | ₦ (and count) paid in period, **tagged by segment at payment date** |
| **Renewal rate / collection %** | Collected ÷ (Collected + still due) **within that segment** for the period | Per-segment performance, not one global % |

**Filters (admin):**

- Report / period (week, month, custom)  
- Product type  
- Agent (all agents vs one agent)  

**Not in PDF as primary:** instalment-level ranking without segment breakdown (we have ranking elsewhere).

---

## 6. What we have today (implemented)

### 6.1 Admin → Dashboard → **Renewal stats** tab

**Where:** `/dashboard` → carousel slide **Renewal stats** (`AdminRenewalStatsPanel`).  
**API:** `GET /admin/lead/dashboard/renewal-stats`

**Filters:** `renewal_duration`, custom dates, `renewal_product_type`, optional `agent_id`.

**KPI chips (single aggregate—not A–D):**

| UI label | Meaning |
|----------|--------|
| **Amount Collected / Amount Due** | Left: sum of `amount` where `renewal_payment_received` and **`payment_date`** in range. Right: sum of premium/amount for **unpaid** statuses (`link_sent`, `overdue`, `failed`) whose **`payment_due_date`** falls in range. |
| **Renewals Collected / Unpaid (due in range)** | Count of collected instalments (payment date in range) / count of unpaid instalments (due date in range). |
| **Overdue Renewals** | Count of instalments with status **`renewal_overdue`** and due date in range. |
| **Next Renewal Links Sent** | Count with **`renewal_link_sent`**, due in range. |
| **Overdue Premium** | Sum of **`premium`** on **`renewal_overdue`** only, due in range. |
| **Premium value collection %** | `collected_amount ÷ (collected_amount + overdue_premium)` — **not** segment-aware. |
| **Instalments collected %** | `collected_count ÷ (collected_count + unpaid_due_count)` in window. |
| **Ranking** | Agent ordinal by **count** of renewals collected (payment date in range) when one agent selected. |

**Key implementation detail:** **Collected** uses **`payment_date`** (good for PDF “when paid”). **Due / unpaid** uses **`payment_due_date`** in the same calendar window—**not** the same as “outstanding by segment A–D as of today.”

### 6.2 Agent dashboard — Renewal stats section

Same logic as admin renewal stats but **scoped to logged-in agent** via `GET /agent/dashboard` renewal fields.

### 6.3 Admin & Agent — **Renewal Reminder** module

**Where:** `/renewal-reminder` (admin) and `/agent/renewal-reminder`.

**Purpose:** Daily **work queue**, not the full segmented performance dashboard.

| Tab | List unit | Segment filter |
|-----|-----------|----------------|
| **Upcoming** | One row per lead (nearest upcoming due) | `upcoming_today`, `upcoming_1_14`, `upcoming_15_30` |
| **Overdue** | One row per lead (`max_overdue_days` from **earliest** overdue due) | `overdue_1_30`, … `overdue_366_plus` |
| **Paid** | Instalment rows | Payment date filters |

**KPI strip:** Lead count + ₦ to collect (overdue) or advance premium (upcoming) for **selected segment band**.

**Mapping to PDF (approximate, not identical):**

| PDF segment | Renewal Reminder analogue |
|-------------|---------------------------|
| A sub-bands | Upcoming segments (due today / 1–14 / 15–30) |
| B sub-bands | Overdue 1–30, 31–60 |
| C sub-bands | Overdue 61–120, 121–199 |
| D sub-bands | Overdue 200–365, 366+ |

**Gap:** Reminder segments are **overdue/upcoming tabs**, not a unified **A–D board** with **per-segment collected** in period.

### 6.4 Agent ranking

**Where:** `/agent/agent-ranking`. Ranks agents by renewal collection % in a date range—**global**, not per segment A–D.

---

## 7. Gap analysis (PDF vs today)

| PDF requirement | Status |
|-----------------|--------|
| Four top-level segments A–B–C–D on **admin renewal dashboard** | **Not built** |
| Sub-bands under each segment | Partially mirrored in **Renewal Reminder** filters only |
| Outstanding / due by segment **as of report date** | **Not built** (stats use due-date **window**, not oldest-unpaid segment) |
| Collected credited to **segment at payment time** | **Not built** (only total collected by payment date) |
| Per-segment renewal rate | **Not built** |
| Agent-specific views with same segmentation | **Not built** on dashboard (agent has reminder bands only) |

---

## 8. Recommended target UI (for mockups / Gemini)

Use this layout for **non-functional demo** screens. Label clearly: **“Target design — segmentation model”**.

### 8.1 Admin — Dashboard — Renewal stats (redesign)

```
┌─────────────────────────────────────────────────────────────────────────┐
│  Renewal performance          [Period ▼] [Product ▼] [Agent ▼] [Apply]   │
│  Report as of: 29 Oct 2026 · Selected: 1–29 Oct 2026                     │
├─────────────────────────────────────────────────────────────────────────┤
│  SEGMENT SNAPSHOT (as of report date)                                    │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐                  │
│  │ A Current│ │ B Recent │ │ C Mid    │ │ D Long   │                  │
│  │ Leads 120│ │ Leads 45 │ │ Leads 18 │ │ Leads 7  │                  │
│  │ Due ₦…   │ │ Due ₦…   │ │ Due ₦…   │ │ Due ₦…   │                  │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘                  │
├─────────────────────────────────────────────────────────────────────────┤
│  COLLECTION IN PERIOD (credit at payment time)                           │
│  Segment │ Collected ₦ │ Paid count │ Rate % │ vs prior period          │
│  A       │ …           │ …          │ …      │ ▲                        │
│  B       │ …           │ …          │ …      │ ▼                        │
│  C       │ …           │ …          │ …      │                          │
│  D       │ …           │ …          │ …      │                          │
│  TOTAL   │ …           │ …          │ …      │                          │
├─────────────────────────────────────────────────────────────────────────┤
│  [Optional drill-down: sub-band tabs under selected segment]             │
│  Table: Lead ID | Policy | Agent | Oldest unpaid due | Days overdue | ₦  │
└─────────────────────────────────────────────────────────────────────────┘
```

**Colour coding suggestion for mocks:**

- **A** — teal / green (healthy pipeline)  
- **B** — amber (early recovery)  
- **C** — orange  
- **D** — red (lapsed / win-back)  

### 8.2 Agent — Dashboard slice (future)

Same **A–D snapshot** for **my book only**, plus link **“Open in Renewal Reminder”** pre-filtered to matching band.

### 8.3 Keep existing carousel tab (interim)

Until segmentation ships, keep current chips but add a **banner**:  
*“Totals are not split by segment A–D. See Renewal Reminder for age bands.”*

---

## 9. Data model reference (engineering)

| Entity | Fields used |
|--------|-------------|
| `policy_instalments` | `payment_due_date`, `payment_date`, `status`, `amount`, `premium`, `lead_id` |
| `leads` | `main_status`, `plan_type`, `agent_id` |
| `lead_reports` | `policy_number`, display fields for reminder table |

**Statuses:** `next_renewal`, `renewal_link_sent`, `renewal_overdue`, `renewal_payment_received`, `renewal_payment_failed`.

**Collected query (today):** `renewal_payment_received` + `payment_date` in `[start, end]`.  
**Unpaid due in window (today):** unpaid statuses + `payment_due_date` in window.

**Segment snapshot (to build):** group by `lead_id`, compute oldest unpaid due, days overdue vs report date → bucket A/B/C/D.  
**Segment collected (to build):** for each paid row in period, compute segment at `payment_date` (historical oldest unpaid).

---

## 10. Edge cases (confirm with sales before build)

| Case | Question for sales |
|------|-------------------|
| Partial payment on one instalment | PDF says track separately—define whether one instalment can split across segments. |
| Payment clears multiple instalments at once | One payment event or multiple segment credits? |
| Policy not `policy_active` | Exclude from renewal dashboard? (current code excludes non-active.) |
| Travel / single-pay products | In scope per PDF—confirm instalment pattern in data. |
| Agent reassignment | Segment snapshot by current agent vs agent at payment time? |

---

## 11. Summary for your manager (one paragraph)

We already measure **total** renewal collections and overdue exposure on the **Admin Renewal stats** tab, but sales’s PDF requires **four non-overlapping age segments (A–D)** based on each policy’s **oldest unpaid due date**, plus **collection credited to the segment the policy was in when paid**—so performance in “recently overdue” is not mixed with “200+ day lapsed.” **Renewal Reminder** gives agents **similar age bands** for daily calling, but it is **not** the full segmented dashboard with per-segment collected and renewal rates. Next step is to implement the **A–D board and payment-time segment tagging** on admin (then agent) renewal stats while keeping Renewal Reminder as the operational drill-down.

---

## 12. Related files in repo

| Area | Path |
|------|------|
| Admin renewal stats API | `insurance-service/application/routes/admin/lead_management.py` → `get_admin_renewal_stats` |
| Collection / due window helpers | `insurance-service/application/utilities/agent_ranking.py` |
| Reminder segment bands | `insurance-service/application/service/renewal_reminder_segments.py` |
| Reminder aggregation | `insurance-service/application/service/renewal_reminder_service.py` |
| Admin renewal stats UI | `admin/component/dashboard/admin-renewal-stats-panel.js` |
| Reminder segment KPI | `admin/component/agent-component/renewal-reminder/renewal-reminder-segment-kpi.js` |
| PDF requirements | `Renewal_Dashboard_Segmentation_Requirements.pdf` (Sales, 21 Oct 2026) |
