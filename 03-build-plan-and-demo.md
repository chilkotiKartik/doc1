# DealFlow360 — Part 3: What We're Building, Who Does What, and How We Explain It

*(For the team of 3, and for the "explain everything in depth" round)*

---

## 1. Our Team (3 people, all full-stack)

Since all 3 of us can do both frontend and backend, we're not splitting by "frontend person / backend person." We're splitting by **feature**, so each person owns something start to finish and can explain it fully:

| Person | Owns (frontend + backend together) |
|---|---|
| **Person A** | The Discount Risk Engine + Approval flow — the core brain of the app. This is the most important piece. |
| **Person B** | The Customer Portal — separate login, negotiation screen, counter-offer flow. |
| **Person C** | Warehouse split + Billing/Subscriptions + Upsell panel + Deal-health dashboard |

Everyone reviews everyone's work at the end and can answer questions on any part — this matters because judges may ask any of us anything.

## 2. What We Build First, Next, Later

| Stage | What's in it | Why |
|---|---|---|
| **Must-have (build first)** | Discount Risk Engine (both checks — single line AND total add-up), approval flow, customer portal with counter-offer, one-time + subscription billing on the same order | This is what makes the whole idea work — without this, we don't have a product |
| **Should-have (build next)** | Warehouse split screen, deal-health dashboard, upsell suggestions | Required by the brief, but simpler logic — build once the core is solid |
| **Nice-to-have (only if time left)** | PDF/Excel export, multiple currencies | Brief itself says these are bonus, not required |

**Rule we follow:** if we're running out of time, we cut from the bottom of this list first. We never touch the Must-have section.

## 3. Folder Structure (so anyone opening our code understands it fast)

```
dealflow360/
├── README.md                  → setup steps + how to run the demo
├── docker-compose.yml         → one command to start the local database
│
├── backend/
│   ├── models/                → database tables as code (product, quote, tier, warehouse...)
│   ├── logic/
│   │   ├── risk_engine.py     → ⭐ the core discount-checking brain
│   │   ├── approval_flow.py
│   │   ├── warehouse_split.py
│   │   ├── billing.py
│   │   ├── upsell.py
│   │   └── deal_health.py
│   ├── tests/
│   │   └── test_risk_engine.py → ⭐ proves the math actually works, not just "looks right in demo"
│   └── api/
│       ├── internal_routes.py  → for rep/manager/finance/admin
│       └── portal_routes.py    → for the customer, kept separate on purpose
│
├── frontend-internal/          → rep, manager, finance, admin screens
└── frontend-portal/             → customer-only app, separate login
```

**Why we're pointing this out to a mentor:** anyone can open `risk_engine.py` and `test_risk_engine.py` side by side and see it's real, tested logic — not something faked just for the demo. And `frontend-portal/` being a totally separate folder (separate app, separate login) proves the negotiation screen is genuinely a different, restricted thing — not a copy-pasted internal screen.

## 4. How We'll Demo It (5 minutes)

1. **(30 sec)** Show the settings: Gold customer, Hardware allowed 15% off, Services allowed 10% off. Point out these are just settings — not hardcoded.
2. **(1 min)** Build a quote live: Laptop at 12% off (fine) → Setup Service at 18% off — watch it instantly flag as "8% over its limit, needs Manager + Finance."
3. **(30 sec)** Add 3 more small discounts, each just a little over their own limit. Show the total score cross the line too — say clearly: "none of these looked bad by themselves, but together they do."
4. **(45 sec)** Accept an upsell suggestion, watch total + profit update live.
5. **(45 sec)** Manager approves, Finance approves — show the history log of who did what and when.
6. **(45 sec)** Switch to the customer portal (different login, different screen). Customer asks for even MORE discount. Watch it automatically re-check and send it back for approval — no one has to click anything to make that happen.
7. **(45 sec)** Confirm the order, show the warehouse split, show the invoice with one-time and subscription parts kept separate, mark it paid.

## 5. Deep Explanation Round — Questions We're Ready For

**Q: Explain the discount logic in your own words, slowly.**
> Every product belongs to a category. Every category has its own discount limit, based on the customer's tier. We check every line against ITS OWN limit — not one big limit for the whole order. Then we add up how far over the limit each line went, and if that total is too high, OR if even one line is way too far over, the system asks a human to check it. That's the whole idea.

**Q: Why is this hard / why can't a simple `if` statement do this?**
> A simple `if discount > 15%` only catches ONE big mistake. It completely misses 4 different lines that are each only a little over their limit — but added up, that's real money lost. Our system catches both cases. That's the actual improvement.

**Q: Show me it's not faked.**
> We have a test file that runs the exact numbers from our example and checks the output matches. It's not us clicking through a script — it's automated proof the math works.

**Q: Why did you use AI only in 2 small places?**
> Because AI should solve a real problem, not be added to look impressive. For suggesting extra products, comparing past order history is a known, simple, explainable technique. For catching weird discounts, comparing a person's current behavior to their own past average is simple and explainable too. Everywhere else — especially the discount approval — we kept it as plain, checkable math, because that's something Finance actually needs to trust.

**Q: Why not build this directly inside Odoo?**
> This hackathon allows any tech stack — it's not required to be an Odoo module. We looked at what Odoo already offers (basic single-line discount checks, manually-set product suggestions) and built something that goes further, while copying Odoo's proven way of organizing data (products, price lists, orders). We chose our own stack so we could move fast and have full control over the customer portal in the time we had.

**Q: What does the customer portal actually stop someone from doing?**
> It's a separate login. A customer can only view the quote, comment, propose a different discount, or confirm. They cannot see other customers' data, cannot change product prices directly, and cannot skip approval — every action they take goes back through the same checking engine the internal app uses.

**Q: What would you add with more time?**
> A more advanced version of the "weird discount" check for very small data sets, real-time refreshing of the upsell suggestions instead of doing it overnight, a smarter warehouse-splitting method for when there are hundreds of products, and support for multiple currencies and multiple companies (the brief itself says this is a bonus, not required).

## 6. Quick Recap — What Exists vs. What We Built

| Thing | What already exists | What we did differently |
|---|---|---|
| Discount approval | One number vs. one limit (Salesforce, DealHub, Conga, Odoo add-ons all do this) | Per-category limits AND an add-it-all-up total check |
| Customer negotiation | Slow email, or basic "sign here" portals | Real separate portal that re-checks itself the moment the customer counters |
| Upsell suggestions | Manually picked by admin (Odoo does this) | Learned automatically from past order history |
| Deal-health tracking | Separate paid tools (Gong, Clari) costing $1,300+/user/year plus platform fees | Built directly into the same app, free, tied to the rep's own history |
| Warehouse splitting | Needs manual work in most tools including Odoo | Automatic split that tries to use the fewest shipments possible |

## 7. Last Word

We're not trying to build 7 average features. We're trying to build 1 feature (the smart discount checker) so well that a judge can ask us anything about it and we can answer in real detail — and then show the other 6 features working properly around it. That's the plan, that's the demo, and that's what we can defend in the next round.
