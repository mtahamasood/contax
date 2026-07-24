---
title: "Product Brief: Contax"
status: ready-for-review
created: 2026-07-25
updated: 2026-07-25
---

# Product Brief: Contax

> A trust machine for Pakistani construction. Contax proves a construction company's honesty to its client every week — and gives the company control over the on-site leakage it can't otherwise see.

## Executive Summary

In Pakistani construction, the client's defining experience is **suspicion**: they cannot tell whether they are being cheated. Ghost expenses, material that walks off site, a completion date that keeps drifting — and the person best placed to reassure them, the site engineer, is often the person doing the cheating. The client who cannot visit the site (overseas Pakistanis building a house back home are the extreme case) has no defense but the phone call and the benefit of the doubt.

**Contax** inverts the burden of proof. Instead of the client hunting for signs of guilt, the construction company proves its innocence — automatically, every week — through a branded, evidence-backed report delivered over WhatsApp and a live web app. Underneath that client-facing "proof-of-innocence" skin, Contax is an **internal anti-leakage control system**: it captures what happens on site at the three points where money and material move, reconciles those captures against each other and against benchmarks, and flags the anomalies that signal fraud — most of which today happens below senior management's line of sight.

The buyer is the construction company, and its interests resolve every trade-off. Contax opens the owner's wallet three ways: it **wins hesitant clients** who have been burned before ("with us, you see every rupee and every brick"); it gives the owner **transparency into his own organization**; and it **replaces management's constant polling** of staff with a system that collects information on its own. The long game is bigger than the app: the weekly captures accumulate into the one asset nobody in Pakistani construction currently owns — a **cross-company cost-and-productivity dataset** that becomes the industry's benchmark for what things should cost and consume.

## The Problem

Construction is a low-trust, largely informal industry in Pakistan, and the leak points are structural:

- **Material pilferage** — legitimately-delivered material walks off site; deliveries are over-invoiced or ghosted entirely.
- **Labour fraud** — ghost workers and inflated headcounts on the muster roll.
- **Cost padding & fake extras** — inflated rates, and end-of-project "the client approved this" surprise bills the client never knowingly agreed to.
- **Schedule drift** — completion dates slip with no accountable cause.

Crucially, **most of this is done by low- and mid-level on-site staff, constantly present on site, without senior management's knowledge** — because management visits rarely and today operates by *polling*: chasing people, chasing reports. The owner's confidence in his own numbers is therefore thin, and it is exactly that thin confidence he passes to the client.

The client feels the other end of the same problem. What they actually want is not a dashboard — it is to **stop wondering whether they are being cheated**. That wondering is exactly what clients call the site engineer about. For the client who cannot visit — the overseas Pakistani, five to nine time zones away, holding the money most worth defending — the anxiety is sharpest and the current tools (a phone call, a relative sent to "check on things") are weakest.

## Who This Serves

**The buyer and customer is the construction company.** Contax must first be a tool the owner pays for because it serves *him* — the client-facing transparency is the visible skin over an internal control system.

**The end-users are two-sided:**
- **The company's own staff** — management, site engineers, accountants, storekeepers — who operate the capture layer. The app is mandatory for every role, including the storekeeper, and must work for low-literacy users (icon / photo / voice-first).
- **The company's clients** — who receive the weekly report and a live view. The **overseas Pakistani building back home** is the sharpest illustration of the premise (genuinely unable to visit, among the most cheated, app-comfortable, holding money worth defending), but v1 does **not** fork on client segment.

**Beachhead — company-first, segment-agnostic.** v1 is sold to the *company* on internal control and the transparency pitch; whether its clients are overseas or domestic is treated as configuration (e.g. async-approval emphasis, contact handling), not two products. This keeps v1 faithful to the value hierarchy and defers the segment bet rather than baking it into the build. Overseas remains the story that dramatizes the value in sales.

**Value hierarchy (foundational):** the end-client's interests matter *to the degree they create value for the construction company*. When the two conflict, the company wins — but the product's whole logic is that a client's trust in the company is itself the company's most valuable asset.

## The Solution

Contax is built on a deliberately small spine and a large computed surface.

**The wedge is the weekly proof-of-innocence report** — an automatically-generated, branded PDF (English + Urdu, with an LLM-written narrative) delivered over WhatsApp, mirrored by a live in-app view. It reverses the burden of proof, and it is an artifact the client *keeps*: it survives without signal, feels official the way a report feels official, and gets forwarded — to the client's family watching the site, into WhatsApp groups, and eventually in front of the next person about to hire a builder. That forwarding is the cheapest marketing channel available, so the report is branded and self-explanatory to a stranger — and it must **never leak supplier rates or margins**, because the same forwarding reaches competitors.

**To produce that report, the system must capture three atomic events** — and these three are the entire mandatory capture layer:

1. **Material-in** — an OCR'd delivery challan (quantities and rates read off the paper, not typed by a clerk who could lie).
2. **Money movement** — ledger entries.
3. **Geotagged site photo** — trust evidence, progress proof, and a fraud datapoint at once.

**Everything else is a computed view projected from those captures** — the client app, the admin cross-project rollup, the anomaly engine, the PDF itself. The capture layer is designed once and kept open (it may grow past three), and this is the architectural discipline that keeps the product coherent.

The **architecture is API-first** — a responsive **web app first**, with native mobile built later against the same API. Weekly PDFs are archived server-side as a numbered, tamper-evident series, so **a missing week is visible** — which disciplines the company's own staff as much as it reassures the client.

## What Makes This Different

An honest reading of the market (research summarized below): **the mechanics are table-stakes; the differentiation is positioning and, later, data.** OCR challans, client portals, weekly branded PDFs, AI summaries, geotagged photos, % completion, change-order approval — every one of these already ships in Buildertrend/CoConstruct (global), Powerplay/Onsite (India), and, most pointedly, in Pakistan's own Smart Construction. Contax cannot win on feature novelty. What is genuinely underserved:

- **It inverts who the software serves.** Incumbent portals are *builder-controlled* ("the client sees what the builder allows") and sold to the contractor as an engagement/productivity tool. Contax points the same mechanics the other way — at the *client's* protection against the contractor's own low/mid site staff — and monetizes the owner's control over his own organization. That inversion of *who it's for* is the real whitespace, and it is a **positioning/go-to-market moat, not a technical one.**
- **It targets a specific low-trust, remote-principal wedge** — the anxious client (overseas Pakistani building back home as the extreme) whom no incumbent centers.
- **Trust is engineered through structure, not forced behavior change.** Users stay on WhatsApp and phone calls; approvals happen *outside* the app; Contax records them after the fact and disciplines that record with negative confirmation, mandatory cost/schedule deltas, and the client's own attached evidence. Nothing asks the client — or the suppliers — to adopt anything.
- **Confidence through redundancy.** Any fact confirmed by two or more independent captures (challan + supplier statement, photo + ledger, milestone + consumption) carries a higher **data-credibility score** — a first-class scored attribute. Fraud is caught not by trusting any single entry but by the disagreement between entries.
- **The primary moat is anti-fraud depth an incumbent structurally cannot match — buyer conflict.** Every incumbent ERP is adopted by the *whole company*, so its usage depends on the site staff liking it; you cannot build genuine surveillance into a tool whose adoption depends on the surveilled. Contax resolves this the opposite way — the **owner mandates it top-down to watch his own organization**, and the design is *adversarial toward its own operators by construction* (append-only ledgers, separation of powers, evidence-mandatory entries, spot-check prompts). A contractor-productivity tool can't bolt that on without alienating the users it depends on. This is the defense Contax leads with.
- **The durable second moat is data, not features.** The one thing that cannot be cloned is the cross-company cost-and-productivity dataset (below) — but it compounds only at scale, so it is a *year-two+* reinforcement, not a day-one defense.

**The honest risk this creates:** a domestic incumbent — Smart Construction is already at ~PKR 3,000/mo with WhatsApp automation, an AI copilot, and branded client PDFs — is *one product decision* from copying the visible skin (a client-facing transparency layer). Contax's bet is that they *won't follow into adversarial anti-fraud depth* because it conflicts with their buyer. **The tension this exposes, which the pilot must resolve:** Contax needs the very staff it surveils to enter data, so the whole model rests on the **owner's top-down mandate carrying adoption where an incumbent's voluntary buy-in cannot**. If mandate doesn't drive honest capture, the moat and the product fail together — this is the single assumption most worth testing first.

## Why Now & Competitive Reality

_(Grounded in market research, July 2026; sourced detail in the addendum.)_

**Why now** — the enabling conditions are real and verifiable: WhatsApp is ubiquitous in Pakistan (the primary delivery channel is where users already are); overseas remittances hit a record ~$38.3B in FY2025 (+27% YoY), the money pool behind remote home-building; digital payments (JazzCash/EasyPaisa) now dominate retail transactions, making traceable payment rails viable; and OCR/LLM costs have collapsed (self-hosted OCR ~16× cheaper than cloud), making document-capture of every challan economically trivial. Cost overruns in Pakistani construction are near-universal (one study: 100% of projects, averaging ~35% overrun) — strong evidence of the pain.

**Where Contax sits** — Pakistan already has capable local players, so this is *not* a greenfield market. **Smart Construction** (SMB, ~PKR 3,000/mo, WhatsApp + AI copilot + branded client PDFs) is the closest direct overlap; **EZYPRO / EzyPMP** (Lahore — "Pakistan's first cloud construction PM software"; clients include ADB and China State Construction) is a mid-market/enterprise contractor PM/ERP. Crucially, **both are sold to the contractor as operations software** — neither centers a client-facing, adversarial "prove innocence to the owner" transparency layer, and EZYPRO serves large institutional projects rather than Contax's SMB-residential / absent-owner wedge. Global and Indian tools are likewise contractor-productivity plays with builder-controlled client portals. No one found is building a Pakistan construction cost/productivity benchmark dataset — the moat space is open. **Takeaway:** the *positioning* whitespace is real, but a capable local incumbent moving into it is a live threat, not a hypothetical — which is why the defense rests on anti-fraud depth (buyer conflict) and speed, not features.

## Business Model

_(Working hypothesis — to be validated against market/pricing research; flagged for the PRD and GTM.)_

- **Pricing model:** a **per-active-project monthly subscription**, tiered at the company level by number of concurrent projects. Value scales with projects (each running project generates the weekly reports and the leakage-catching), and per-project deliberately avoids per-seat pricing — which would fight the "app mandatory for every role including storekeeper" requirement.
- **Who pays:** the construction company owner/director — the same person the internal-control pitch targets. Buyer, champion, and beneficiary are one person.
- **Pricing power:** the ROI is **self-funding** — but frame it on the *documented* pain (cost overruns average ~35% in Pakistani projects), not on "pilferage," since material theft specifically is not quantified in the literature. "Reduce a 35% overrun" is a defensible pitch; "catch pilferage" is a hypothesis. Plus the softer "win one hesitant client."
- **Pricing anchor & risk:** the reference price in-market is **Smart Construction at ~PKR 3,000/mo** — so Contax is pricing into an existing, cheap anchor, not a blank market. Emerging-market SMBs have low SaaS willingness-to-pay (they can revert to running without software), which pushes toward a **flat low-PKR subscription possibly paired with a transaction/commission hook** rather than pure per-seat SaaS.
- **An option the beachhead opens:** because the *client* (often an overseas Pakistani with hard-currency, higher WTP) is a distinct payer from the local contractor, Contax could charge the **client** for the transparency guarantee rather than only the contractor — worth testing against the company-first buyer model.
- **Revenue evolution:** subscription is the *wedge* revenue; the parked platform plays (procurement brokerage take-rate, finance-referral revenue-share, benchmarking tier — see Vision) are the *scale* revenue. Pricing shape mirrors the product's moat: SaaS entry, data/platform economics later.
- **Open:** exact PKR price points, Pakistani SMB-contractor willingness-to-pay, and construction-company margins remain unvalidated.

## Scope — First Version

The MVP is defined by one question: *what is the minimum that produces a credible weekly proof-of-innocence report and the internal control beneath it?* (Governance mechanics, AI/ML detail, and monetization are expanded in the addendum.)

**MUST — v1**
- Weekly proof-of-innocence report: branded PDF over WhatsApp + live in-app view, English + Urdu, LLM-generated narrative; archived as a numbered tamper-evident series.
- The three capture events: OCR'd challan material-in, money movement, geotagged site photo.
- Milestones with **computed** % completion, derived from BOQ line items marked done — not a typed number.
- Accounts + client outstanding balance. OCR posts material stock automatically, but the **accounts-payable entry is a recommendation the accountant edits and signs off** before commit (human-in-loop + separation of duties on money).
- Material-vs-BOQ tracking.
- Photo evidence timeline.
- **Change-order recording with tiered, company-configurable governance** (separation of powers, mandatory cost + schedule deltas, no retroactive billing, append-only trail — see addendum).
- Admin cross-project rollup.
- **Anomaly flags v1** — rule engine + unsupervised (PyOD) + government rate-book price prior; internal-first (manager audits before any client disclosure), two-tailed (flags suspiciously-clean as well as over-consumption).
- Versioned drawings with an "issued for construction" marker.
- Multi-auth identity (phone + email + username/password); a client is one identity with multiple concern-tagged contact numbers.
- Low-literacy-friendly UX (icon / photo / voice-first); app mandatory for every role including storekeeper.
- Negative-confirmation approval loop in the report; Urdu/Punjabi voice-note ASR into a decision audit log; cash-heavy-site risk flag / traceable payment rails.
- **Supplier reconciliation** — Source A (OCR'd challan, primary, real-time) reconciled against Source B (the supplier's periodic account statement issued *to the company*), raising credibility on agreement and an internal investigation on divergence. Source B carries collection friction but is **kept in v1** as confidence-through-redundancy (owner's call).

**MUST — but flagged as known-risk**
- **CV progress-% estimation from photos** (and on-device inference for integrity checks) — **kept as a v1 MUST (owner's call)**, against the AI/ML research's judgment that single-photo CV progress-% is unreliable. Mandatory guardrail: BOQ-derived % is the fallback and the client-facing number of record until CV demonstrably earns trust — the PRD should implement CV in *shadow-mode behind the BOQ number*, not as the sole source.

**COULD — v2**
- Learned ML anomaly models + cross-company benchmarking; quality checklists / inspections / snag lists / test reports; deep document vault; quiet-mode (exceptions-only) client view; random spot-check photo prompts.

**WON'T — explicitly rejected**
- Client-family parallel data entry; in-app client chat; supplier one-tap confirmation; no-app SMS/IVR capture (app is mandatory for all roles); direct third-party sale of market-index data; two-way company-side WhatsApp Business integration (designed for in the architecture, deferred past MVP).

## Key Product Principles

These are the invariants a PRD and architecture must preserve. (Full governance and AI detail in the addendum.)

- **North star:** prefer a *configurable choice with a recommended, pre-filled default* over a hard-coded single answer. Opinionated, not rigid — every governance threshold ships as a default the company can reconfigure.
- **Two separate axes, never conflated:** internal-facing control vs. client-facing transparency; internal approval vs. client ratification. Both configurable, independently.
- **Anomalies are internal-first.** The manager has discretion over whether any given flag is ever disclosed to the client.
- **Data credibility is scored, driven by source-agreement.** Redundancy, not any single trusted entry, is the fraud control.
- **AI/ML is a standing evaluation lens on every feature**, across three execution locations (on-device integrity checks; self-hosted open-source SLMs for sensitive/fraud data; third-party LLM API only for Urdu narrative and hard OCR, behind field-level redaction).

## Success Criteria

The make-or-break criterion comes first, because if it fails nothing else matters.

- **The mandate test (make-or-break):** on a pilot site, do the three capture events actually happen — driven by the owner's mandate, without management chasing — at high completeness (target: ≥90% of deliveries produce an OCR'd challan; ledger and photo capture on the same cadence)? This validates the single assumption the whole model rests on. Everything below is moot if this fails.
- **Client trust:** measurable drop in client "checking-up" contacts to the site engineer; weekly-report open and forward rates; client-reported confidence (before/after).
- **Owner value:** anomalies surfaced *and acted on* (documented leakage or overrun caught); at least one sale the owner attributes to the transparency pitch; management hours reclaimed from polling.
- **Business:** first pilot converted to paid; paying companies and projects under management; logo/project retention across renewal.

_(Targets are first-pass and should be calibrated with the pilot company; the point is that each is measurable, not aspirational.)_

## Vision & Moat

If Contax works, it stops being an app vendor and becomes **the outfit that owns Pakistan's construction cost-and-productivity dataset**. The moat is built in three stages: government rate books (day one) → the company's own project history (month one) → the cross-company pool (year one, the real moat). Companies will trade their anonymized numbers to see how they compare against the market. A single "expected value" service — *what should this cost / consume here?* — then feeds BOQ estimation, the anomaly engine, and change-order pricing alike.

That dataset unlocks a parked ladder of monetization (detailed in the addendum): procurement/group-buying brokerage, construction-finance underwriting referral, a verified-supplier marketplace, a premium benchmarking tier sold back to the companies, AI BOQ auto-estimation, and verified-contractor certification. The architecture is kept extensible — a pluggable supplier/financier/insurer seam and the pooled dataset as a first-class internal service — so any of these can switch on later without a rebuild.

## Open Questions & Known Risks

_(Carried forward for the PRD. These are the cracks — several are addressed in the attack phase.)_

- **Validation status** — the concept is well-reasoned but only lightly validated: there is *some real signal* (an owner/site-level indication that the premise holds) but **no committed pilot customer yet**. The brief should be read as a strong thesis with early signal, not a market-proven plan. Securing a first pilot company is the highest-leverage next step.
- **The data-entry paradox** — the people who must enter data on site are the same people doing the cheating. The concept's answer is to anchor on what already exists and doesn't lie (challans, supplier invoices, photos, client payments), role-separated double entry, and even-fraudulent-data-has-value. *Whether that holds in practice is the central product risk, and it is exactly what a pilot must test first.*
- **MVP scope weight** — the v1 MUST list is deliberately large (the owner's call: the trust proposition is held to require the full set, including CV progress-% and Source-B reconciliation despite their known risks). This is an examined, accepted timeline risk, not an unexamined one.
- **CV progress-% reliability** — kept as a v1 MUST against the research's own advice; carries a hard dependency on the BOQ-derived fallback as the number of record, and a PRD decision to run CV in shadow-mode until it earns trust.
- **Benchmark data sourcing** — the moat rests on obtaining MES *Analysis of Rates* and provincial MRS/CSR coefficients; current editions and per-province granularity are unverified.
- **Supplier statement (Source B) collection friction** — Source B is a v1 MUST, but the workflow (who obtains and scans the periodic statement, how often, what happens if the supplier refuses) is undefined and must be designed.
- **Urdu quality** — an internal Urdu eval set for narrative and ASR quality does not yet exist.
- **Competitive threat (resolved, now a live risk):** Smart Construction (Pakistan, ~PKR 3,000/mo) already overlaps heavily and could add a client-transparency layer; the differentiation is positioning + eventual data, not features. Defensibility depends on speed, anti-fraud depth, and dataset head start.
- **Two premise claims the brief leans on but research could NOT verify:** (a) material *theft/pilferage* as a *quantified* problem (cost overruns are documented at ~35%, but theft is not isolated) — so lead ROI messaging with overruns, not theft; (b) the PKR volume of *overseas-funded* home construction — do not cite a specific figure until verified.
