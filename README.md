# Renewal dashboard by age group (A, B, C, D) — plain guide for managers

**For:** Sales  
**Date:** October 2026  
**Purpose:** Explain what we want on the renewal dashboard, how numbers are worked out, and what we have today vs what we still need to build.

---

## Why we are doing this

Today, every late renewal (30 days, 100 days, 200+ days) is often shown in **one big “overdue” pile**. That hides the truth:

- Are we collecting from **customers who are not late yet**?
- Are we fixing **recent** lateness (1–60 days)?
- How much sits in the **middle** (61–199 days)?
- How much is **very old** debt (200+ days) that needs win-back?

We split the renewal book into **four age groups (segments)** so each can be managed and measured on its own.

---

## The four segments (each customer sits in only one)

We look at each policy and find the **oldest bill that is still not paid**. How many days that bill is late (or how soon it is due) decides the segment.

| Segment | Plain name | Rule (oldest unpaid bill) |
|---------|------------|---------------------------|
| **A** | Current / not late yet | Due today or in the next 30 days |
| **B** | Recently late | 1 to 60 days late |
| **C** | Mid late | 61 to 199 days late |
| **D** | Very late / win-back | 200 days late or more |

**One policy = one segment.** If someone owes several months, we use the **oldest** unpaid due date, not the newest.

**How customers pay:** In our process, they pay **oldest bill first**, then the next. We do not need special rules for “paying a newer bill while an old one is still open.”

---

## Two different things on the screen (do not mix them)

### 1) Live book — “Who is where **right now**?”

- Shows how many policies are in A, B, C, D **today**.
- Shows how much money is **still to be collected** in each segment **today**.
- **Always live** — refreshes as payments come in.
- **No date range filter** for this part (optional: filter by product type or by agent for admin).
- This is the **work queue**: who to call, how big the problem is in each age group **now**.

When someone pays:

- Money **still owed** in that segment **goes down**.
- The customer may **move** to another segment (e.g. from C to B or A) if their oldest unpaid bill changed.

### 2) Collections in a period — “How much did we **collect**, and which segment **gets the credit**?”

- Shows money **received** in a chosen period: today, this week, this month, or custom dates.
- Each payment is credited to **the segment the customer was in on the day they paid** (using the same oldest-unpaid rule on that day).
- **Needs a date filter** — you are asking “what happened between these dates?”
- That credit **does not move** later if the customer moves to another segment.

**Example:** Customer was in **C** and paid **₦15,000** on Monday.

- **Monday, collections (this week):** Segment **C** shows **+₦15,000** collected.
- **Monday evening, live book:** Segment **C** “still to collect” drops by **₦15,000** for that policy; the customer might no longer be in C.
- **Next week:** If you filter collections **“last week”**, C still shows **₦15,000** for that payment. If you look at the **live book**, the customer might now be in **B** or **A**.

**One sentence for leadership:**  
**Live book = situation now. Collection period = money story for the dates you pick.**

---

## Step-by-step flow (day in the life)

### Morning — agent opens dashboard

1. System looks at every active renewal policy assigned to the agent.
2. For each policy, find **oldest unpaid** renewal due date.
3. Compare to **today** → put policy in **A, B, C, or D**.
4. Add up **still unpaid** premium in each segment → show four boxes: count + **to collect ₦**.

Agent thinks: “I have X people in B, ₦Y still to collect in B.”

### During the day — customer pays

1. Payment succeeds on the **oldest unpaid** bill (our normal process).
2. **Before** marking the bill paid, system checks: on **today’s date**, which segment was this policy in? → e.g. **C**.
3. System **records** on that payment: “This ₦ amount counts as **segment C** collection.”
4. Bill is marked paid; **live book** is recalculated → segments and **to collect** update.

### Evening — manager checks “this week”

1. Manager sets **collection period** = this week.
2. System adds all payments this week, grouped by **segment credit** (A/B/C/D) → “C collected ₦15k this week,” etc.
3. **Live book** is still **as of today** — no week filter on overdue segments.

### One week later

- **Collection filter = last week** → same ₦15k still credited to **C** for that week (history).
- **Live book** → current segments and **to collect** only (customer may have moved).

---

## How each number is calculated (simple formulas)

### Live segment (which group is the policy in?)

1. Oldest unpaid due date = earliest due date among all unpaid renewal bills for that policy.
2. Days difference = **today minus that date** (if due is in the future, they are in **A**, not “late” yet).
3. Map days to **A, B, C, or D** using the table above.

### Live “to collect” per segment

- Sum of **unpaid renewal premium** for all policies that sit in that segment **right now** (rules for which bills count in the sum should match finance — usually all unpaid renewal amounts on that policy while it stays in that segment).

### Collected per segment in a period

1. Take every renewal payment with **payment date** inside the chosen period.
2. Each payment already has **segment credit** saved when it was paid (e.g. credited to **C**).
3. Sum amounts **by segment** → “B collected ₦…,” “C collected ₦…,” etc.

### Optional: collection rate per segment (future)

- For a period: **collected in segment B** ÷ (**collected in B** + **still was due in B at start of period**) — agree exact definition with sales before building.

---

## What to store in the system (so reports stay honest)

| What | When it is saved | Why |
|------|------------------|-----|
| **Segment credit on payment** | When payment succeeds | So “C collected ₦15k last week” stays true forever |
| **Live segments** | Calculated when dashboard opens | Always current; no daily history required at first |
| **Daily snapshot table** | Optional later | Only if you need “what did segment C look like on 31 Oct?” frozen after months pass |

**Start simple:** live book + save segment on each payment. Add daily snapshots only if leadership needs frozen historical books.

---

## Who sees what

| User | Live book (A–D, to collect) | Collections by segment | Extra filters |
|------|----------------------------|-------------------------|---------------|
| **Agent** | Own customers only | Own customers, chosen period | Product type |
| **Admin** | All agents or one agent | Same | Agent, product type, period |

**Renewal Reminder** (existing screens): good for **calling lists** with similar age bands (e.g. 1–30 days late). That is **not** the full dashboard with **collections credited by segment** — it complements the new view.

---

## What we have today (the problem)

| What sales wants | What we have now |
|------------------|------------------|
| Four segments A–D on admin renewal dashboard | One combined renewal stats view — not split by A–D |
| Live “to collect” per segment | Partially similar on **Renewal Reminder** age bands, not the same as A–D dashboard |
| Collections credited to segment when paid | **Not built** — we only total collections by payment date, not by segment credit |
| Agent sees both live book + period collections | Agent has renewal stats totals and reminder lists, not this full model |

**Risk if we don’t build segment credit on payment:**  
“We collected ₦X in segment C this month” cannot be reported accurately after customers move to other segments.

---

## What we will build (recommended order)

1. **Save segment credit** on every renewal payment (which of A/B/C/D on payment day).
2. **Live dashboard block:** four segments, policy count + to collect ₦ (agent + admin).
3. **Collection block:** period filter → collected ₦ (and count) per A/B/C/D.
4. **Lead list:** show each customer’s **current segment**; optional column “last payment credited to segment X.”
5. **Later (optional):** nightly snapshot if you need frozen “book size on date X.”

---

## Filters — quick reference

| Filter | Used for | Example |
|--------|----------|---------|
| **Collection period** (today / week / month / custom) | **Collected** ₦ by A/B/C/D | “How much did B recover in October?” |
| **Product type** | Both blocks (narrow which policies) | Life only |
| **Agent** (admin) | Both blocks | One agent’s book |
| **Date range on overdue segments** | **Not used** — live book is **always current** | — |

---

## Short summary for a meeting

We split renewals into **four age groups (A–D)** using the **oldest unpaid bill**. The dashboard shows **two stories**: (1) **live** — who owes what in each group **today**; (2) **period** — money **collected** in a chosen week/month, **credited to the group they were in when they paid**. Live book updates after every payment; collection history for a week does not change. We need to **record segment on each payment** and **build the split dashboard**; today we only have **combined** renewal totals and **reminder lists**, not full A–D collection credit.

---

*Related technical note for engineering:* `docs/RENEWAL_DASHBOARD_SEGMENTATION_MANAGER_GUIDE.md`  
*Sales source PDF:* Renewal Dashboard Segmentation Requirements, 21 Oct 2026
