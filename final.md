# DealFlow360 — Final Blueprint: The Glass-Box Deal Governance Engine

## 1. Executive Summary
Build ONE thing exceptionally well: a **transparent, configurable, per-category "Blended Discount Risk Engine"** that is the beating heart of the platform, wrapped in a genuinely separate customer negotiation portal that feeds back into that engine. Every other required feature (multi-warehouse split, hybrid billing, deal-health dashboard, upsell panel) is built to a credible MVP standard, but the risk engine is where we invest disproportionate polish — because the problem statement itself over-specifies it, a deliberate tell that most competing teams will fake with a single `if discount > 15%` check.

**Critical reframing (verified):** the Odoo India Hackathon 2026 is **technology-agnostic** — its own event pages state "all programming languages, frameworks, and database architectures are welcome" — and it is judged on "problem understanding, innovation, technical implementation, UI/UX design, and team collaboration," not on Odoo-addon purity. We build a clean, fast, custom full-stack app that implements the hard business logic in real application code — exactly what the problem statement demands ("core business rules must be implemented in application logic, not hardcoded or faked").

Odoo's own products remain the essential competitive *reference*: we study how Odoo Sales, Subscriptions, and Inventory model these problems and deliberately out-implement the one thing none of them — nor Salesforce CPQ, DealHub, or Conga — actually do: **cumulative, per-category-aware blended risk scoring.**

**The final bet:** win on the Blended Discount Risk Engine as explainable "glass-box" IP + a real (not relabeled) negotiation portal that round-trips through it. Demo those two flows flawlessly.

## 2. Problem & Root Cause

### 2.1 The analysis chain
**Stated problem.** Simple sales tools handle quote → order → invoice well. Real B2B sales teams operate in messier conditions: multi-level discount approvals, partial stock spread across warehouses, bundled subscriptions mixed with one-time hardware, customers who want to negotiate inside a portal instead of over email, and managers who only discover a stalled deal after it has lost momentum.

**Root problem.** Reps optimize *locally* (per line, per interaction) while risk exists *globally and cumulatively* (across the whole order, across time, across a rep's pattern of behavior) — and the system has no mechanism to reconcile the two in real time. A rep can stay under every individual limit yet quietly give away significant order-level margin; or one thin-margin line can blow past its own ceiling while the *order average* looks perfectly acceptable. Existing tooling enforces static thresholds on a single number (one line, or one order total). None of it reasons about the *cumulative pattern* of many small violations measured against *different per-category ceilings*, which is precisely the two-part mechanic the problem statement spells out in unusual depth (a single bad line, AND many small overages that sum to a material giveaway). That depth is the real intellectual core of the challenge and the fairest way to separate teams that understood the domain from teams that pattern-matched a CRUD app.

**User pain (by role).**
- *Sales Rep:* wants to move fast and close deals, but has no real-time visibility into whether a discount combination will trigger approval delay — so deals stall in review queues they didn't see coming.
- *Sales Manager/Approver:* is stuck manually reviewing quotes that don't actually need review, while quotes that *should* be flagged (many small overages) slip through because no single line looks alarming.
- *Finance/Ops:* only sees risk after it has already been baked into a confirmed order, and reconciling one-time and recurring billing on the same order is manual and error-prone.
- *Customer:* is forced into slow email back-and-forth to negotiate, with no live view of quotation status or way to counter-propose directly.
- *Everyone:* deals go quiet ("no decision") and nobody notices until it's too late — the platform has no mechanism to catch a stalling deal early.

This pain has measurable scale in the real world: per *The Jolt Effect* (Matthew Dixon & Ted McKenna, analysis of 2.5M recorded B2B sales conversations, reported in Harvard Business Review, 2022), **40–60% of qualified B2B deals end in "no decision" rather than a competitor win** — most pipeline is lost to inertia and silence, not to losing on price. Forrester's *State of Business Buying 2024* similarly found that **86% of B2B purchases stall during the buying process, and 81% of buyers are dissatisfied with the provider they ultimately chose.** A platform that keeps deals moving *and* protects margin attacks both halves of that loss.

**Existing solutions (what's already out there — see full detail in Section 4).** Salesforce CPQ, DealHub, Conga, and PandaDoc all offer discount-approval automation, but each is a threshold-or-Boolean engine: one number (or one aggregate average/max) compared against one limit. Odoo's native and marketplace tooling is even simpler — most modules block confirmation past a single per-line percentage. The most sophisticated open-source option, OCA's `base_tier_validation`, supports configurable multi-tier sequenced approvals but triggers on a single Python-domain condition, not a computed cross-category cumulative score. Odoo natively supports hybrid billing, multi-warehouse fulfillment, and manually-configured upsells — each real, but each with real limitations (detailed below).

**Competitive gap.** Nobody — not Odoo, not any OCA module, not any mature commercial CPQ tool we could verify — computes a **cumulative, per-category-weighted blended risk score** that catches many small per-category overages that individually don't alarm. That is precisely what the problem statement asks for, and precisely the gap this product exploits.

**Opportunity.** Build the Blended Discount Risk Engine (BDRE) as a first-class, explainable, configurable governance IP — not a buried validation rule — and pair it with a real customer negotiation portal that re-triggers the same engine on every counter-offer. This is the one place a hackathon-scale team can credibly out-execute both the Odoo ecosystem and commercial CPQ incumbents, because it rewards clean logic and domain understanding over integration breadth or infrastructure scale.

## 3. Users & Workflow
Five roles map cleanly to the required flow: **Sales Rep** (builds quotes, sees live margin + risk), **Sales Manager/Approver** (configures tiers/chains, approves L1, monitors deal health), **Finance/Ops** (L2 approval on high-risk, warehouse splits, billing reconciliation), **Customer/Portal User** (separate restricted view: negotiate, counter, confirm), **Admin** (backend config + analytics).

End-to-end flow: signup/login → admin configures backend (products, price lists, tiers, chains, warehouses, subscription plans) → rep creates quotation → adds products/discounts/reviews upsells → quote auto-routes for approval if the blended score crosses a threshold (Manager, then Finance if required) → warehouse fulfillment split suggested → order may include subscription lines generating a billing schedule alongside a one-time invoice → customer negotiates via portal → if terms change beyond thresholds, quote automatically re-enters approval → once confirmed, order proceeds to fulfillment and billing → manager monitors the Deal Health dashboard throughout → reports reviewed via filters (period/team/status/product). Our architecture makes the risk engine the hub every state transition passes through.

## 4. Competitive Landscape & Gap
Verified how mature commercial and Odoo-ecosystem tools handle each pillar of the problem, with primary-source confirmation of the central gap.

**Discount governance — the gap is real and specific.** Salesforce CPQ Advanced Approvals evaluates **rule-by-rule, condition-by-condition**, each condition a tested field compared to a fixed value/threshold, with multiple conditions combined by Boolean AND/OR logic (Salesforce Trailhead, "Manage Approval Logic with Approval Rules, Conditions, and Variables"). Its nearest aggregation capability is the "Approval Variable," which can compute an *average* or *max* discount across quote lines and test that against one threshold — a single aggregate vs. a single limit, not a per-category-ceiling-aware composite; it would not reliably catch many lines each 2–3 points over their *own differing* ceilings. DealHub states plainly that "DealHub's standard approval workflows rely on discount percentage thresholds—not specific price points" (DealHub Community, Configuring Floor Pricing). Conga uses Boolean entry-criteria rules combined with AND/OR (Conga Documentation, Approval Flow Settings). PandaDoc offers per-line, per-section, and "total discount (sum of all discounts across the quote)" conditions tested against thresholds (PandaDoc Help Center, Conditional Approvals). HubSpot ships with essentially one predefined approver and threshold-based routing via workflows (HubSpot Community; Quotivity). Across all of them: **no tool computes a cumulative, per-category-weighted blended risk score that catches many small per-category overages that individually don't alarm.**

Odoo marketplace discount-approval modules are simpler still — nearly all are single-threshold, per-line checks: e.g., "If any product in a sales order has a discount greater than 10%, the system blocks confirmation of the order" (Odoo Apps, `sales_approval_enhancement`); `l4l_sale_discount_approval` enters a "Waiting For Discount Approval" state when any line exceeds one configured max. The OCA `base_tier_validation` module (ForgeFlow/Ecosoft) is the most sophisticated open-source option — configurable multi-tier "Tier Definitions" with sequenced "approve by sequence" reviews reusable on any model — but it triggers on a single Python-domain condition, not a computed cross-category cumulative score.

**Upsell/cross-sell.** Odoo natively offers "Optional Products," "Accessory Products," and "Alternative Products" — but these are **manually configured per product on the product form** (Odoo docs), not learned from historical co-purchase data. A market-basket approach is a genuine step up and trivially demo-able.

**Subscriptions/proration.** Odoo Subscriptions genuinely handles proration, but proration flows through the **upsell wizard / `prepare_upsell_order()` path** and is bypassed by manual line edits, and commonly **requires the product to be a Service** (Octura Solutions; oduist.com, Odoo Experience 2025 summary). Hybrid one-time + recurring on a single order is supported natively — but demonstrating it cleanly separated is where teams stumble.

**Deal health.** Gong and Clari define the commercial standard: deal-health scoring against historical benchmarks, stall detection, push-count tracking (Clari, "Top Sales Metrics to Identify At-Risk Deals"). Their acknowledged limitation: Clari flags that a deal stalled but "doesn't diagnose why at the conversational level" (Hyperbound). These tools are also expensive and separate: Gong's 2026 Foundations license runs **~$1,300–$1,600 per user/year plus a mandatory platform fee reported between $5,000 and $50,000/year** (2026 buyer-data teardowns — PitchMonster, Sybill), making effective per-user cost 2–3x the seat rate. A lightweight, integrated, rep-baseline-relative anomaly view is a credible differentiator at hackathon scale.

**Warehouse split.** Odoo supports multi-warehouse delivery and split/backorder natively (route rules; "Destination location origin from rule" in v17.2+), but real-world users report it "mandate[s] manual intervention" and is not a clean auto-split-to-minimize-shipments optimizer (Odoo Forum). An explicit shipment-count-minimizing allocation algorithm is genuinely additive.

**Bottom line gap.** Even mature CPQ tools are threshold engines with opaque, rule-by-rule approvals. Nobody ships a transparent, explainable, per-category **cumulative** risk score — and nobody pairs it with a real negotiation portal that re-scores on counter-offers.

## 5. Winning Product
**DealFlow360 as a "Glass-Box Deal Governance Engine."** A sales operations platform whose singular identity is that *every pricing decision is scored, explained, and auditable in real time*, and whose customer portal is a real negotiation surface feeding the same engine. We build all 7 required feature areas to functional MVP, but the risk engine and the negotiation round-trip are the product's soul and receive the majority of our polish.

## 6. Core Innovation
**The Blended Discount Risk Engine (BDRE): a transparent, configurable rules + scoring engine.** It implements both mechanics the problem statement specifies, which most teams will miss:

1. **Per-line, per-category ceiling breach** — each line scored against its *category's* ceiling (Hardware 15%, Services 10%), not one global order limit. One 8-points-over service line flags the whole quote even if the average looks fine.
2. **Cumulative aggregate pattern** — a weighted sum of every line's overage (2pts + 3pts + 2pts …) so distributed small giveaways that individually pass still escalate once their total margin impact crosses a configured threshold.

The score is **explainable**: the UI shows a line-by-line contribution breakdown ("Setup Service: 8.0 pts over → +40 risk; 4 lines each 2–3 pts over → +18 cumulative; blended score 58 → routes to Manager + Finance"). It is **configurable**: ceilings per customer tier and per category, weightings, and routing bands are admin-editable data, not code. It produces a **deterministic routing decision** (None / Manager / Manager+Finance) and a full **audit log** (user, timestamp, reason, before/after) — the "deterministic guardrails" best practice the CPQ industry itself now emphasizes, made transparent and cumulative.

Formula (defensible, simple, tunable):

`risk = Σ_lines [ max(0, discount_i − ceiling(category_i, tier)) × margin_weight_i × line_value_share_i ]`

with a separate hard-flag if any single term exceeds a per-line escalation cap. Bands map score → approval level. **No ML is used here, deliberately** — forcing ML into a governance rule would reduce trust and explainability.

## 7. Killer Features
1. **Blended Discount Risk Engine (glass-box).** *User value:* managers stop reviewing every quote; reps can't game per-line limits. *Competitive advantage:* out-implements Salesforce CPQ / DealHub / Conga (all threshold/Boolean, per verified vendor docs) on the exact axis the problem statement stresses. *Feasibility:* pure application logic over quote lines; demos in 20 seconds.
2. **Real negotiation portal that round-trips through the BDRE.** *User value:* customer counters a discount in a restricted portal; on submit, the quote re-scores and, if it now breaches thresholds, auto-re-enters approval. *Competitive advantage:* redlining tools (PandaDoc, DealHub DealRoom) negotiate *text*; none re-run a governance score on a customer counter-offer. Satisfies the explicit "must be a real, separate, restricted view" requirement. *Feasibility:* separate authenticated view + one re-score call.
3. **Shipment-minimizing warehouse auto-split.** *User value:* fewer shipments, honest ETAs. *Competitive advantage:* Odoo's native split needs manual intervention; we run an explicit greedy allocation minimizing shipment count, with a "Consolidate Remaining Backorder" prompt. *Feasibility:* bounded greedy heuristic over live stock.
4. **Co-purchase upsell panel (market-basket).** *User value:* ranked, data-driven suggestions with live margin delta, not a static hand-maintained list. *Competitive advantage:* beats Odoo's manually-configured Optional/Accessory/Alternative products. *Feasibility:* precomputed association rules (support/confidence/lift) via Apriori on seed order history.
5. **Deal-health & discount-anomaly dashboard (rep-relative).** *User value:* surfaces stalled deals, discount anomalies vs. the rep's *own* historical average, and delivery-promise slippage, with one-click nudge/escalate. *Competitive advantage:* Gong/Clari-style value, integrated and free, with the "why" attached to each flag. *Feasibility:* z-score is a few lines; stall detection is a date diff.

## 8. AI Strategy
AI is used in exactly two places, each justified end to end; we deliberately do not add chatbots, LLM agents, or blockchain — none solve a real problem here.

**Upsell recommendations — market-basket / association-rule mining (Apriori).**
- *Problem:* which product to suggest next while building a quote.
- *Data:* historical order lines (seed data of co-purchases).
- *AI:* Apriori (Agrawal & Srikant, 1994) generates association rules; rank by **lift**, filter by **confidence** and a minimum-margin threshold — a documented use case for this technique (ResearchGate, "Smart Product Recommendations… Apriori for Market Basket Analysis"). Precompute offline; serve instant lookups.
- *Decision → Action:* top-N suggestions shown with margin delta and promo tag; "Add to Quote" updates total/margin live.
- *Value:* measurable AOV/margin uplift, genuinely better than a manual optional-product list.

**Discount & deal-health anomaly detection — z-score.**
- *Problem:* is this rep's discount unusual *for them*; is this deal stalling.
- *Data:* the rep's historical discount distribution; deal activity timestamps.
- *AI:* `z = (x − μ)/σ` against the rep's own baseline; flag |z| > ~3 (standard practice — Tinybird; GeeksforGeeks). Modified z-score / MAD noted as a more robust variant for small, non-Gaussian samples (arXiv 2103.12323).
- *Decision → Action:* anomaly badge + automated nudge/escalation.
- *Value:* catches silent margin erosion and stalls early, explainably.

**Why not ML for the risk score itself:** governance demands determinism and auditability; a learned model would undermine both, so we say this out loud — it signals judgment, not a gap.

## 9. Odoo-Native Strategy
**Verified reframing:** the Odoo India Hackathon explicitly welcomes "all programming languages, frameworks, and database architectures" and judges "problem understanding, innovation, technical implementation, UI/UX, and team collaboration" — it is not an Odoo-addon contest. We therefore build a clean custom full-stack app rather than betting on the Odoo ORM.

Odoo remains essential in two concrete ways: (1) as the **domain reference model** — we mirror proven Odoo data structures (product templates + variants, pricelist rules with per-category/per-tier discounts, `sale.order`/`sale.order.line`, subscription recurring plans, stock routes) so our schema is battle-tested, not invented; and (2) as the **competitive baseline we beat** — we can point to specific Odoo limitations (manual optional products; proration requiring Service products and the upsell wizard; manual warehouse-split intervention; single-threshold marketplace approval modules; OCA `base_tier_validation`'s condition-trigger rather than cumulative scoring) and show exactly where DealFlow360 goes further. If a team wanted to build inside Odoo, `base_tier_validation` plus custom computed fields on `sale.order` would be the path — worth naming to prove platform understanding, before justifying the independent build for velocity, UI control, and demo reliability under time pressure.

## 10. Technical Architecture
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
**Stack recommendation:** a single-language full stack (e.g., Django/FastAPI + PostgreSQL + React, or equivalent) so the "business logic in real application code" mandate is unmistakable. PostgreSQL because the schema is genuinely relational and judges reward dynamic, queryable data over static JSON.

## 11. Data / Odoo Models & Key Workflows
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

**Key workflow — the scored round-trip:** on any line edit or portal counter-offer, recompute margin + BDRE score + breakdown → persist → if the score band requires approval, set state and create approval steps + audit entries → on approval, trigger warehouse split + billing schedule → on fulfillment/payment, update invoice status. This is the exact end-to-end path the deliverable's Quick Test Flow requires.

## 12. MVP & Build Priorities
**Must-win (P0):** BDRE with per-category ceilings + cumulative aggregation + explainable breakdown + configurable tiers/chains + audit log; auto-routing (Manager, then Finance); the separate negotiation portal with counter-offer re-scoring and auto-re-approval; hybrid order showing one-time + recurring lines billed separately; seed data.
**Should-have (P1):** warehouse split with shipment-count display + backorder/consolidate prompt; deal-health dashboard with stall + z-score anomaly; upsell panel with live margin update.
**Nice-to-have (P2):** Apriori-learned rules (fall back to seeded rules if time-constrained), XLS/PDF export, multi-currency (explicitly a bonus).
Cut ruthlessly before touching P0. A flawless BDRE + portal beats seven half-working features.

## 13. Demo Flow & WOW Moment
1. **(0:00–0:30)** Admin shows configured tiers: Gold, Hardware 15% / Services 10%, chain bands.
2. **(0:30–1:30) WOW #1 — Glass-box scoring.** Rep builds a quote: Laptop at 12% (fine), Setup Service at 18%. Live margin ticks down; the risk panel animates a breakdown: "Service 8 pts over ceiling → routes to Manager+Finance," even though the order average looks acceptable. Then add three more lines each 2–3% over — show the *cumulative* meter crossing the band a single-line view wouldn't have caught. The single most differentiating minute of the demo.
3. **(1:30–2:15)** Accept one upsell suggestion (margin delta shown) → total + margin update instantly.
4. **(2:15–3:00)** Quote auto-routes (no manual "request approval") → Manager approves → Finance approves; audit trail shown.
5. **(3:00–3:40)** Confirm → warehouse split across two warehouses with shipment count + cost; trigger backorder consolidate prompt.
6. **(3:40–4:30) WOW #2 — Living negotiable document.** Switch to the separate customer portal (distinct login/URL). Customer counters for a bigger discount → submit → quote re-scores → **automatically re-enters approval** on-screen. Confirm with one click.
7. **(4:30–5:00)** Show hybrid invoice (one-time invoiced now, subscription schedule listed separately), record payment, invoice status flips to Paid; glance at deal-health dashboard flagging a stalled/anomalous deal.

## 14. Judge Objections & Answers
- *"Isn't this just an if-statement on discount?"* → No: per-category ceilings AND cumulative aggregation catch distributed small overages — the exact scenario the problem statement calls out, which Salesforce CPQ / DealHub / Conga cannot do (verified: threshold/Boolean per vendor docs).
- *"Why not Odoo modules?"* → We mirror Odoo's proven data model and can cite where we surpass native Odoo and OCA `base_tier_validation`; the hackathon explicitly allows any stack and judges implementation quality, so we optimized for velocity, UI control, and demo reliability.
- *"Is the portal real or relabeled?"* → Separate authenticated app, restricted fields, its own URL; the counter-offer re-runs the governance engine — a relabeled internal screen literally couldn't do that.
- *"Where's the AI, and why so little?"* → Apriori for upsell and z-score for anomalies, each with a problem→data→action justification; the governance score stays deterministic for auditability, matching CPQ industry best practice.
- *"Does proration actually work?"* → Yes, computed on remaining days; we note the real Odoo caveat (Service-product/upsell-wizard dependency) to show domain depth.

## 15. Risks & Mitigations
- **Over-scoping (biggest risk).** → Ruthless P0/P1/P2; BDRE + portal are non-negotiable, everything else is sacrificable.
- **Warehouse optimizer edge cases eat time.** → Greedy heuristic with bounded seed data; manual-override path always available.
- **Apriori under-delivers on tiny seed data.** → Ship curated seeded association rules; present Apriori as the generating method with lift/confidence shown.
- **Proration correctness under time pressure.** → Simple daily proration with a visible formula; correctness over generality.
- **Demo fragility.** → Deterministic seed data; rehearse the exact path; keep P2 features off the critical path.
- **Judges expected Odoo-native build.** → Pre-empt with the explicit Odoo-reference + "any stack allowed" framing.

## 16. Competitive Scorecard
| Capability | DealFlow360 (us) | Likely generic team | Salesforce CPQ / DealHub / Conga | Odoo native + OCA |
|---|---|---|---|---|
| Discount approval | **Per-category + cumulative blended score, explainable, configurable** | Single `if discount>X%` | Threshold/Boolean, rule-by-rule; avg/max variable at best | Single-threshold modules; `base_tier_validation` = condition trigger |
| Explainable routing | **Line-by-line breakdown + audit** | Opaque flag | Rule audit, not cumulative rationale | Basic state + approver |
| Negotiation portal | **Real, restricted, re-scores counter-offers** | Relabeled internal screen | Text redlining, no re-scoring | Online sign/pay only, no counter |
| Upsell | **Apriori co-purchase + live margin** | Static list | Some AI (enterprise) | Manual optional products |
| Warehouse split | **Shipment-minimizing + consolidate prompt** | Manual pick | N/A (CPQ) | Manual intervention |
| Hybrid billing | **One-time + recurring, prorated, on one order** | Often skipped | Add-on/CLM | Yes (Service caveat) |
| Deal health | **Rep-relative z-score + stall + nudge** | Absent | Gong/Clari (separate; ~$1.3–1.6k/user/yr + platform fee) | Absent |

## 17. Why We Can Win
The problem statement telegraphs its own rubric: it spends more words on the Blended Discount Risk Score than on any other feature, and separately warns that the negotiation portal must be *real* and that business rules must not be *faked*. Most competing teams will build a competent quote-to-cash CRUD app with a single-threshold approval and a static upsell list — the exact generic version we designed against. We win by making the one feature judges will probe hardest into transparent, cumulative, configurable IP; pairing it with the one integration (portal → re-score → re-approve) nobody else will wire end to end; and demoing both in under five minutes with a visible "aha." It is more innovative, more technically credible, and more demo-able than the field — on the axes an engineering-focused jury actually scores.

## 18. Final Bet
Build the **Blended Discount Risk Engine** as a transparent, per-category, cumulative "glass-box" governance engine — implemented in real application code, configurable as data, fully audited — and wire it to a genuinely separate customer negotiation portal that re-scores counter-offers and auto-re-enters approval. Ship the other five features to honest MVP, but pour your polish here. Demo two flows: (1) a Gold quote where one 18% service line AND several 2–3% overages each independently and cumulatively drive the score across the Manager+Finance band with a live line-by-line explanation; (2) a customer in the portal countering for a deeper discount that instantly re-scores and bounces the deal back into approval on screen. That is the one thing to build, it is the thing no other team and no commercial CPQ tool (verified: Salesforce CPQ, DealHub, Conga are all threshold/Boolean) does well, and it is the thing that will make the judges remember you.

---

### Source & Confidence Notes
**Verified (primary/vendor sources):** Odoo hackathon is technology-agnostic (hackathon.odoo.com event pages; Indus/NMIT FAQ pages); Odoo native optional/accessory/alternative products are manually configured (Odoo docs); Odoo Subscriptions proration mechanics and Service-product caveat (Odoo docs; Octura Solutions; oduist.com); Odoo online signature/Sign & Pay portal confirmation (Odoo docs); Odoo multi-warehouse split needs manual intervention (Odoo Forum); Odoo pricelist per-category/per-tier discount rules (Odoo docs); marketplace discount-approval modules are single-threshold (Odoo Apps Store listings); OCA `base_tier_validation` capabilities (PyPI/OCA); Salesforce CPQ Advanced Approvals is rule-by-rule/threshold with avg/max aggregate variables only (Salesforce Trailhead); DealHub "thresholds—not specific price points" (DealHub Community); Conga Boolean entry criteria (Conga docs); PandaDoc per-line/section/total-discount threshold conditions (PandaDoc Help Center); Gong pricing structure (2026 buyer teardowns — PitchMonster/Sybill); 40–60% no-decision stat (*The Jolt Effect* / HBR 2022); 86% stall / 81% dissatisfaction (Forrester State of Business Buying 2024); Apriori/market-basket and z-score/MAD techniques (ResearchGate; Tinybird; GeeksforGeeks; arXiv 2103.12323).

**Reasoned assumptions (labeled):** that judges will specifically probe whether the blended risk score is real vs. faked (inferred from the problem statement's disproportionate detail and its explicit "not hardcoded/faked" instruction, not a stated rubric line); the "~1,000 competing teams" figure is the user's premise and was not independently verified; the recommended tech stack is engineering judgment, not a hackathon requirement.

**Could not verify:** any published Odoo hackathon judging weightings specific to this problem statement; exact past-winner patterns for Odoo India hackathons (event pages state criteria only in general terms).
