# DealFlow360 — Strategic Blueprint: The "Glass-Box Deal Governance Engine"

## Executive Summary
Build ONE thing exceptionally well: a **transparent, configurable, per-category "Blended Discount Risk Engine"** that is the beating heart of the platform, wrapped in a genuinely separate customer negotiation portal that feeds back into that engine. Every other required feature (multi-warehouse split, hybrid billing, deal-health dashboard, upsell panel) is built to a credible MVP standard, but the risk engine is where we invest disproportionate polish — because the problem statement itself over-specifies it, a deliberate judge tell that most competing teams will fake with a single `if discount > 15%` check.

**Critical reframing (VERIFIED):** the Odoo India Hackathon 2026 is **technology-agnostic** — its own event pages state "All programming languages, frameworks, and database architectures are welcome" — and it is a *recruitment* event (₹5,00,000 prize pool, Grand Finale 5–6 September 2026 at Odoo India Pvt. Ltd., Gandhinagar) judging "problem understanding, innovation, technical implementation, UI/UX design, and team collaboration" (Odoo x Indus Hackathon FAQ). It is NOT an Odoo-addon contest. This overturns the task's central assumption. We do **not** need to build inside the Odoo ORM to win. We should build a clean, fast, custom full-stack app that implements the hard business logic in real application code — which is exactly what the problem statement demands ("core business rules must be implemented in application logic, not hardcoded/faked").

Odoo's own products remain the essential competitive *reference*: we study how Odoo Sales, Subscriptions, and Inventory model these problems and deliberately out-implement the one thing none of them (nor Salesforce CPQ, DealHub, or Conga) actually do: **cumulative, per-category-aware blended risk scoring.**

**The FINAL BET:** win on the Blended Discount Risk Engine as explainable "glass-box" IP + a real (not relabeled) negotiation portal that round-trips through it. Demo those two flows flawlessly.

## Problem & Root Cause
The stated problem: simple tools handle quote→order→invoice, but real B2B sales is messier — multi-level discount approvals, partial stock across warehouses, bundled subscriptions mixed with one-time hardware, portal negotiation, and stalled-deal blindness.

Root cause, correctly diagnosed by the PDF: **reps optimize locally and managers see globally, too late.** A rep can stay under every per-line limit yet give away significant aggregate margin; or blow one thin-margin service line while the order average looks fine. Existing tools enforce *thresholds* — a single line or the order total crossing a fixed number — but do not reason about the *cumulative pattern* of many small violations against *different per-category ceilings*. The PDF's unusually detailed "Blended Discount Risk Score" section (two distinct mechanics: one bad line over its own ceiling, AND many small overages summing to material giveaway) is the real intellectual core of the challenge and the fairest discriminator between teams that understood the domain and teams that pattern-matched a CRUD app.

This problem is expensive in the real world. Per *The Jolt Effect* (Matthew Dixon & Ted McKenna, an analysis of 2.5M recorded B2B sales conversations, reported in Harvard Business Review, 2022), **40–60% of qualified B2B deals end in "no decision" rather than a competitor win** — most pipeline is lost to inertia and silence, not to losing on price. Forrester's *State of Business Buying 2024* similarly found that **86% of B2B purchases stall during the buying process, and 81% of buyers are dissatisfied with the provider they ultimately chose.** A platform that keeps deals moving *and* protects margin attacks both halves of that loss.

## Users & Workflow
Five roles map cleanly to the required flow: **Sales Rep** (builds quotes, sees live margin + risk), **Sales Manager/Approver** (configures tiers/chains, approves L1, monitors deal health), **Finance/Ops** (L2 approval on high-risk, warehouse splits, billing reconciliation), **Customer/Portal User** (separate restricted view: negotiate, counter, confirm), **Admin** (backend config + analytics). The end-to-end flow is the PDF's canonical path: signup → configure → build quote → risk-scored auto-routing → warehouse split → hybrid billing → portal negotiation → auto re-approval on threshold breach → fulfillment/billing → deal-health monitoring → reporting. Our architecture makes the risk engine the hub every state transition passes through.

## Competitive Landscape & Gap (with sources)
I verified how mature commercial and Odoo-ecosystem tools handle each pillar, with primary-source confirmation of the central gap.

**Discount governance — the gap is real and specific.** Salesforce CPQ Advanced Approvals evaluates **rule-by-rule, condition-by-condition**, each condition a tested field compared to a fixed value/threshold, with multiple conditions combined by Boolean AND/OR logic (Salesforce Trailhead, "Manage Approval Logic with Approval Rules, Conditions, and Variables"). Its nearest aggregation capability is the "Approval Variable," which can compute an *average* or *max* discount across quote lines and test that against one threshold — but this is a single aggregate vs. a single limit, not a per-category-ceiling-aware composite; it would not reliably catch many lines each 2–3 points over their *own differing* ceilings. DealHub states plainly that "DealHub's standard approval workflows rely on discount percentage thresholds—not specific price points" (DealHub Community, Configuring Floor Pricing). Conga uses Boolean entry-criteria rules combined with AND/OR (Conga Documentation, Approval Flow Settings). PandaDoc offers per-line, per-section, and "total discount (sum of all discounts across the quote)" conditions tested against thresholds (PandaDoc Help Center, Conditional Approvals). HubSpot out of the box provides essentially one predefined approver, with threshold-based routing via workflows (HubSpot Community; Quotivity). Across all of them: **no tool computes a cumulative, per-category-weighted blended risk score that catches many small per-category overages that individually don't alarm.** That is precisely what the PDF asks for — and precisely the differentiator.

Odoo marketplace discount-approval modules are even simpler: nearly all are single-threshold, single-or-two-level, per-line checks — e.g., "If any product in a sales order has a discount greater than 10%, the system blocks confirmation of the order" (Odoo Apps, `sales_approval_enhancement`); `l4l_sale_discount_approval` enters a "Waiting For Discount Approval" state when a line exceeds one configured max. The OCA `base_tier_validation` module (ForgeFlow/Ecosoft) is the most sophisticated: configurable multi-tier "Tier Definitions" with sequenced "approve by sequence" reviews reusable on any model — but it triggers on a Python-domain condition, not a computed cross-category risk score. This is the closest existing art and worth citing as what we surpass.

**Upsell/cross-sell.** Odoo natively has "Optional Products," "Accessory Products," and "Alternative Products" — but these are **manually configured per product on the product form** (Odoo 18/19 docs), not learned from historical co-purchase data. Our market-basket approach (below) is a genuine step up and trivially demo-able.

**Subscriptions/proration.** Odoo Subscriptions genuinely handles proration, but with caveats we can exploit as talking points: proration works through the **upsell wizard / `prepare_upsell_order()` path** and is bypassed by manual line edits; proration commonly **requires the product to be a Service** (Octura Solutions; oduist.com Odoo Experience 2025 summary). Hybrid one-time + recurring on a single order is supported natively — but demonstrating it cleanly separated is where teams stumble.

**Deal health.** Gong and Clari define the commercial standard: deal-health scoring against historical benchmarks, stall detection, push-count tracking (Clari, "Top Sales Metrics to Identify At-Risk Deals"). Their acknowledged limitation: Clari flags that a deal stalled but "doesn't diagnose why at the conversational level" (Hyperbound). These tools are also expensive, separate systems: Gong's 2026 Foundations license runs **~$1,300–$1,600 per user/year plus a mandatory platform fee reported between $5,000 and $50,000/year** (2026 buyer-data teardowns, e.g., PitchMonster, Sybill), making effective per-user cost 2–3x the seat rate. A lightweight, integrated, rep-baseline-relative anomaly view is a credible hackathon-scale differentiator.

**Warehouse split.** Odoo supports multi-warehouse delivery and split/backorder natively (route rules; "Destination location origin from rule" in v17.2+), but real-world users report it "mandate[s] manual intervention" and is not a clean auto-split-to-minimize-shipments optimizer (Odoo Forum). So an explicit shipment-count-minimizing allocation algorithm is genuinely additive.

**Bottom line gap:** Even mature CPQ tools are threshold engines with opaque, rule-by-rule approvals. Nobody ships a transparent, explainable, per-category **cumulative** risk score — and nobody pairs it with a real negotiation portal that re-scores on counter-offers.

## Winning Product (ONE product)
**DealFlow360 as a "Glass-Box Deal Governance Engine."** A sales operations platform whose singular identity is that *every pricing decision is scored, explained, and auditable in real time*, and whose customer portal is a real negotiation surface that feeds the same engine. We deliberately reject the "build all 7 features equally" trap. We build all 7 to functional MVP, but the risk engine and the negotiation round-trip are the product's soul and get 60% of our polish budget.

## Core Innovation
**The Blended Discount Risk Engine (BDRE): a transparent, configurable rules + scoring engine.** It implements BOTH mechanics the PDF specifies, which most teams will miss:

1. **Per-line, per-category ceiling breach** — each line scored against its *category's* ceiling (Hardware 15%, Services 10%), not one global order limit. One 8-points-over service line flags the whole quote even if the average looks fine.
2. **Cumulative aggregate pattern** — a weighted sum of every line's overage (2pts + 3pts + 2pts …) so distributed small giveaways that individually pass still escalate when their total margin impact crosses a configured threshold.

The score is **explainable**: the UI shows a line-by-line contribution breakdown ("Setup Service: 8.0 pts over → +40 risk; 4 lines each 2–3 pts over → +18 cumulative; blended score 58 → routes to Manager + Finance"). It is **configurable**: ceilings per customer tier and per category, weightings, and routing bands are admin-editable data, not code. It produces a **deterministic routing decision** (None / Manager / Manager+Finance) and a full **audit log** (user, timestamp, reason, before/after). This is exactly the "deterministic guardrails" best practice the CPQ industry itself now emphasizes (Salesforce; digitalapplied.com CPQ 2026 guide) — but made transparent and cumulative.

Formula (defensible, simple, tunable):
`risk = Σ_lines [ max(0, discount_i − ceiling(category_i, tier)) × margin_weight_i × line_value_share_i ]`, with a separate hard-flag if any single term exceeds a per-line escalation cap. Bands map score → approval level. **No ML is used here — and we say so explicitly**, because forcing ML into a governance rule would reduce trust and explainability.

## 3–5 Killer Features
1. **Blended Discount Risk Engine (glass-box).** *User value:* managers stop reviewing every quote; reps can't game per-line limits. *Competitive advantage:* out-implements Salesforce CPQ / DealHub / Conga (all threshold/Boolean, per verified vendor docs) on the exact axis the PDF stresses. *Feasibility:* pure application logic over quote lines; a day's work to a polished state; demos in 20 seconds.
2. **Real negotiation portal that round-trips through the BDRE.** *User value:* customer counters a discount in a restricted portal; on submit, the quote re-scores and, if it now breaches thresholds, auto-re-enters approval. *Competitive advantage:* redlining tools (PandaDoc, DealHub DealRoom) negotiate *text*; none re-run a governance score on a customer counter-offer. Satisfies the PDF's explicit "must be a real, separate, restricted view" rule. *Feasibility:* separate authenticated view + one re-score call; maps directly to Quick Test Flow step 6.
3. **Shipment-minimizing warehouse auto-split.** *User value:* fewer shipments = lower cost, honest ETAs. *Competitive advantage:* Odoo's native split needs manual intervention; we run an explicit greedy allocation minimizing shipment count with a "Consolidate Remaining Backorder" prompt. *Feasibility:* greedy set-cover heuristic over live stock; bounded and demo-safe.
4. **Co-purchase upsell panel (market-basket).** *User value:* ranked, data-driven suggestions with live margin delta, not a static hand-maintained list. *Competitive advantage:* beats Odoo's manually-configured Optional/Accessory/Alternative products. *Feasibility:* precomputed association rules (support/confidence/lift) via Apriori on seed order history; lookup at quote-build time is instant.
5. **Deal-health & discount-anomaly dashboard (rep-relative).** *User value:* surfaces stalled deals (inactive > configured days), discount anomalies vs. the rep's *own* historical average (z-score), and delivery-promise slippage, with one-click nudge/escalate. *Competitive advantage:* Gong/Clari-style value, integrated and free, with the "why" attached to each flag. *Feasibility:* z-score is a few lines; stall detection is a date diff.

## AI Strategy (AI only where it earns its place)
We use lightweight, explainable statistics/ML in exactly two places and **explicitly refuse** to add chatbots, LLM agents, or blockchain — none solve a real problem here, and forcing them would hurt trust and demo reliability.

- **Upsell recommendations — market-basket / association-rule mining (Apriori).**
  - *Problem:* which product to suggest next while building a quote.
  - *Data:* historical order lines (seed data of co-purchases).
  - *AI approach:* Apriori (Agrawal & Srikant, 1994) generates association rules; rank by **lift**, filter by **confidence** and a minimum-margin threshold — the exact documented use case for this technique (e.g., ResearchGate, "Smart Product Recommendations… Apriori for Market Basket Analysis"). Precompute offline; serve instant lookups.
  - *Decision → Action:* top-N suggestions shown with margin delta and promo tag; "Add to Quote" updates total/margin live.
  - *Value:* measurable AOV/margin uplift; genuinely better than manual optional-product lists.
- **Discount & deal-health anomaly detection — z-score (with modified z-score / MAD as robustness note).**
  - *Problem:* is this rep's discount unusual *for them*; is this deal stalling.
  - *Data:* the rep's historical discount distribution; deal activity timestamps.
  - *AI approach:* `z = (x − μ)/σ` vs. the rep's own baseline; flag |z| > ~3 (standard practice — Tinybird; GeeksforGeeks). Note MAD-based **modified z-score** as more robust to small, non-Gaussian samples (arXiv 2103.12323) — a smart caveat to raise with judges.
  - *Decision → Action:* anomaly badge + automated nudge/escalation.
  - *Value:* catches silent margin erosion and stalls early; explainable, not a black box.

**Why not ML for the risk score itself:** governance demands determinism and auditability; a learned model would undermine both. We say this out loud — it signals judgment.

## Odoo-Native Strategy (honest positioning)
**Verified reframing:** the Odoo India Hackathon 2026 explicitly welcomes "all programming languages, frameworks, and database architectures" and judges "problem understanding, innovation, technical implementation, UI/UX, and team collaboration" (Odoo hackathon event pages; Odoo x Indus Hackathon FAQ). It is a recruitment sprint, not an Odoo-addon contest. Therefore **we do not bet on building inside the Odoo ORM.** We build a clean custom full-stack app.

Odoo remains essential in two concrete ways we will articulate to judges: (1) as the **domain reference model** — we mirror proven Odoo data structures (product templates + variants, pricelist rules with per-category/per-tier discounts, `sale.order`/`sale.order.line`, subscription recurring plans, stock routes) so our schema is battle-tested, not invented; and (2) as the **competitive baseline we beat** — we can point to specific Odoo limitations (manual optional products; proration requiring Service products and the upsell wizard; manual warehouse-split intervention; single-threshold marketplace approval modules; OCA `base_tier_validation`'s condition-trigger rather than cumulative scoring) and show exactly where DealFlow360 goes further. If a team *wanted* to build in Odoo, `base_tier_validation` + custom computed fields on `sale.order` would be the path — we mention this to prove we understand the platform, then justify our independent build for velocity, UI control, and demo reliability under time pressure.

## Technical Architecture
```mermaid
flowchart TD
    subgraph Client
      RepUI[Rep Workspace SPA]
      PortalUI[Customer Negotiation Portal - separate restricted app]
      AdminUI[Admin & Config]
      DashUI[Manager Deal-Health Dashboard]
    end
    subgraph API[Application Server - business logic layer]
      Auth[Auth: internal login + portal magic-link]
      BDRE[[Blended Discount Risk Engine]]
      Route[Approval Router - tiers & chains]
      Split[Warehouse Split Optimizer]
      Bill[Hybrid Billing & Proration]
      Reco[Upsell Engine - Apriori rules]
      Health[Deal-Health & Anomaly - z-score]
      Audit[Audit Log Service]
    end
    subgraph Data[(PostgreSQL)]
      Prod[(products / variants / pricelists)]
      Quote[(quotations / lines)]
      Tier[(discount tiers / category ceilings / chains)]
      WH[(warehouses / stock)]
      Subs[(subscription plans / schedules)]
      Rules[(assoc_rules)]
      Log[(audit_log / activity)]
    end
    RepUI --> Auth --> API
    PortalUI --> Auth
    RepUI --> BDRE
    PortalUI -->|counter-offer| BDRE
    BDRE --> Route --> Audit
    BDRE --> Tier
    Route --> Quote
    RepUI --> Reco --> Rules
    Quote --> Split --> WH
    Quote --> Bill --> Subs
    DashUI --> Health --> Log
    API --> Data
    Audit --> Log
```
**Stack recommendation** (for a small team optimizing velocity + polish): a single-language full stack (e.g., Django/FastAPI + PostgreSQL + React, or equivalent) so the "business logic in real application code" mandate is unmistakable. PostgreSQL because the schema is relational and the judges reward "dynamic data, avoid static JSON."

## Data / Models & Key Workflows
Concrete models (mirroring Odoo semantics):
- `product` (id, name, category_id, cost, list_price, is_subscription, min_margin) + `product_variant`.
- `pricelist_rule` (tier, category_id/product_id, discount_ceiling_pct) — drives ceilings.
- `discount_tier` (Bronze/Silver/Gold, overall ceiling) + `category_ceiling` (tier_id, category_id, max_pct).
- `approval_chain` (score_band_low, score_band_high, required_levels[]) — configurable routing.
- `quotation` (customer, tier, state ∈ {Draft, RiskScored, PendingManager, PendingFinance, UnderNegotiation, Confirmed, Fulfilling, Billed}, blended_risk_score, risk_breakdown_json).
- `quotation_line` (product, qty, discount_pct, unit_cost, margin, line_overage_pts, risk_contribution).
- `warehouse` (name, shipping_weight) + `stock_level` (warehouse, product, qty).
- `subscription_plan` (period, proration_rule, cancel_refund_rule) + `billing_schedule`.
- `assoc_rule` (antecedent, consequent, support, confidence, lift).
- `audit_log` (quote_id, user, timestamp, action, reason, before/after) — append-only.
- `deal_activity` (quote_id, last_activity, stall_flag, anomaly_z).

**Key workflow — the scored round-trip:** on any line edit or portal counter-offer, recompute margin + BDRE score + breakdown → persist → if score band requires approval, set state and create approval steps + audit entries → on approval, trigger warehouse split + billing schedule → on fulfillment/payment, update invoice status. This is the exact 8-step Quick Test Flow.

## MVP & Build Priorities (time-boxed)
**Must-win (P0):** BDRE with per-category ceilings + cumulative aggregation + explainable breakdown + configurable tiers/chains + audit log; auto-routing (Manager, then Finance); the separate negotiation portal with counter-offer re-scoring and auto-re-approval; hybrid order showing one-time + recurring lines billed separately; seed data. These map directly to Quick Test Flow steps 1–3, 6, 8.
**Should-have (P1):** warehouse split with shipment-count display + backorder/consolidate prompt (steps 4–5); deal-health dashboard with stall + z-score anomaly; upsell panel with live margin update (part of step 3).
**Nice-to-have (P2):** Apriori-learned rules (fall back to seeded rules if time-constrained), XLS/PDF export, multi-currency (explicitly a bonus per PDF).
Cut ruthlessly before touching P0. A flawless BDRE + portal beats seven half-working features.

## Demo Flow & WOW Moment (5 minutes, ≥2 full flows)
1. **(0:00–0:30)** Admin shows configured tiers: Gold, Hardware 15% / Services 10%, chain bands.
2. **(0:30–1:30) WOW #1 — Glass-box scoring.** Rep builds a quote: Laptop at 12% (fine), Setup Service at 18%. Live margin ticks down; the risk panel animates a breakdown: "Service 8 pts over ceiling → routes to Manager+Finance," even though the order average looks acceptable. Then add three more lines each 2–3% over — show the *cumulative* meter crossing the band that the single-line view wouldn't have caught. This is the single most differentiating 60 seconds of the demo.
3. **(1:30–2:15)** Accept one upsell suggestion (margin delta shown) → total + margin update instantly.
4. **(2:15–3:00)** Quote auto-routes (no manual "request approval") → Manager approves → Finance approves; audit trail shown.
5. **(3:00–3:40)** Confirm → warehouse split across two warehouses with shipment count + cost; trigger backorder consolidate prompt.
6. **(3:40–4:30) WOW #2 — Living negotiable document.** Switch to the separate customer portal (distinct login/URL). Customer counters for a bigger discount → submit → quote re-scores → **automatically re-enters approval** on-screen. Confirm with one click.
7. **(4:30–5:00)** Show hybrid invoice (one-time invoiced now, subscription schedule listed separately), record payment, invoice status flips to Paid; glance at deal-health dashboard flagging a stalled/anomalous deal.

## Judge Objections & Answers
- *"Isn't this just an if-statement on discount?"* → No: show the per-category ceilings AND the cumulative aggregation catching distributed small overages — the exact scenario the PDF calls out and that Salesforce CPQ / DealHub / Conga cannot do (verified: they're threshold/Boolean, per vendor docs).
- *"Why not Odoo modules?"* → We mirror Odoo's proven data model and can cite where we surpass native Odoo and OCA `base_tier_validation`; the hackathon explicitly allows any stack and judges implementation quality, so we optimized for velocity, UI control, and demo reliability.
- *"Is the portal real or relabeled?"* → Separate authenticated app, restricted fields, its own URL; the counter-offer re-runs the governance engine — a relabeled internal screen literally couldn't do that.
- *"Where's the AI, and why so little?"* → Apriori for upsell and z-score for anomalies, each with a problem→data→action justification; we deliberately kept the governance score deterministic for auditability, matching the CPQ industry's own "deterministic guardrails" best practice.
- *"Does proration actually work?"* → Yes, computed on remaining days; we note the real Odoo caveat (Service-product/upsell-wizard dependency) to show domain depth.

## Risks & Mitigations
- **Over-scoping (biggest risk).** → Ruthless P0/P1/P2; BDRE + portal are non-negotiable, everything else is sacrificable.
- **Warehouse optimizer edge cases eat time.** → Greedy heuristic with bounded seed data; manual-override path always available so the demo can't dead-end.
- **Apriori under-delivers on tiny seed data.** → Ship curated seeded association rules; present Apriori as the generating method with lift/confidence shown.
- **Proration correctness under time pressure.** → Simple daily proration with a visible formula; correctness over generality.
- **Demo fragility.** → Deterministic seed data; rehearse the exact 8-step path; keep P2 features off the critical path.
- **Judges expected Odoo-native build.** → Pre-empt with the explicit Odoo-reference + "any stack allowed" framing; keep a one-slide note on how it maps to Odoo models.

## Competitive Scorecard
| Capability | DealFlow360 (us) | Likely generic team | Salesforce CPQ / DealHub / Conga | Odoo native + OCA |
|---|---|---|---|---|
| Discount approval | **Per-category + cumulative blended score, explainable, configurable** | Single `if discount>X%` | Threshold/Boolean, rule-by-rule; avg/max variable at best | Single-threshold modules; `base_tier_validation` = condition trigger |
| Explainable routing | **Line-by-line breakdown + audit** | Opaque flag | Rule audit, not cumulative rationale | Basic state + approver |
| Negotiation portal | **Real, restricted, re-scores counter-offers** | Relabeled internal screen | Text redlining, no re-scoring | Online sign/pay only, no counter |
| Upsell | **Apriori co-purchase + live margin** | Static list | Some AI (enterprise) | Manual optional products |
| Warehouse split | **Shipment-minimizing + consolidate prompt** | Manual pick | N/A (CPQ) | Manual intervention |
| Hybrid billing | **One-time + recurring, prorated, on one order** | Often skipped | Add-on/CLM | Yes (Service caveat) |
| Deal health | **Rep-relative z-score + stall + nudge** | Absent | Gong/Clari (separate; ~$1.3–1.6k/user/yr + platform fee) | Absent |

## Why We Can Win
The problem statement telegraphs its own rubric: it spends more words on the Blended Discount Risk Score than on any other feature, and separately warns that the negotiation portal must be *real* and that business rules must not be *faked*. Most competing teams will build a competent quote-to-cash CRUD app with a single-threshold approval and a static upsell list — the exact generic version we designed against. We win by making the one feature judges will probe hardest into transparent, cumulative, configurable IP; pairing it with the one integration (portal → re-score → re-approve) nobody else will wire end-to-end; and demoing both in under five minutes with a visible "aha." It is more innovative, more technically credible, and more demo-able than the field — on the axes an engineering-recruitment jury actually scores (innovation, technical implementation, UI/UX).

## FINAL BET
Build the **Blended Discount Risk Engine** as a transparent, per-category, cumulative "glass-box" governance engine — implemented in real application code, configurable as data, fully audited — and wire it to a genuinely separate customer negotiation portal that re-scores counter-offers and auto-re-enters approval. Ship the other five features to honest MVP, but pour your polish here. Demo two flows: (1) a Gold quote where one 18% service line AND several 2–3% overages each independently and cumulatively drive the score across the Manager+Finance band with a live line-by-line explanation; (2) a customer in the portal countering for a deeper discount that instantly re-scores and bounces the deal back into approval on screen. That is the one thing to build, it is the thing no other team and no commercial CPQ tool (verified: Salesforce CPQ, DealHub, Conga are all threshold/Boolean) does well, and it is the thing that will make the judges remember you.

---

### Source & Confidence Notes
**VERIFIED (primary/vendor sources):** Odoo hackathon is technology-agnostic and recruitment-focused (hackathon.odoo.com event pages; Indus/NMIT FAQ pages); Odoo native optional/accessory/alternative products are manually configured (Odoo 18/19 docs); Odoo Subscriptions proration mechanics and Service-product caveat (Odoo 18 docs; Octura Solutions; oduist.com); Odoo online signature/Sign & Pay portal confirmation (Odoo 17/19 docs); Odoo multi-warehouse split needs manual intervention (Odoo Forum); Odoo pricelist per-category/per-tier discount rules and sale-order margin fields (Odoo docs); marketplace discount-approval modules are single-threshold (Odoo Apps Store listings); OCA `base_tier_validation` capabilities (PyPI/OCA); Salesforce CPQ Advanced Approvals is rule-by-rule/threshold with avg/max aggregate variables only (Salesforce Trailhead); DealHub "thresholds—not specific price points" (DealHub Community); Conga Boolean entry criteria (Conga docs); PandaDoc per-line/section/total-discount threshold conditions (PandaDoc Help Center); Gong pricing structure (2026 buyer teardowns — PitchMonster/Sybill, corroborated); 40–60% no-decision stat (*The Jolt Effect* / HBR 2022); 86% stall / 81% dissatisfaction (Forrester State of Business Buying 2024); Apriori/market-basket and z-score/MAD techniques (ResearchGate; Tinybird; GeeksforGeeks; arXiv 2103.12323).

**REASONED ASSUMPTIONS (labeled):** that judges will specifically probe whether the blended risk score is real vs. faked (inferred from the PDF's disproportionate detail and its explicit "not hardcoded/faked" instruction, not a stated rubric line); the "~1,000 competing teams" figure is the user's premise — no authoritative published participant count was found, so it is not independently verified; the recommended tech stack is our engineering judgment, not a hackathon requirement.

**COULD NOT VERIFY:** any published Odoo hackathon judging weightings specific to this problem statement; exact past-winner patterns for Odoo India hackathons (event pages state criteria only in general terms).