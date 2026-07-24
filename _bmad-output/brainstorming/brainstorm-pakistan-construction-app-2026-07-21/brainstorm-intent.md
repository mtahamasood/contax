# Contax — Brainstorm Intent

Distilled from the brainstorming session (2026-07-21 → 2026-07-25). Converged, single direction.

## 1. Product concept

**Contax** is a B2B anti-fraud transparency layer — a "trust machine" — for the Pakistani construction industry. It proactively proves a construction company's honesty to its clients each week while giving the company internal control over on-site leakage. Not a dashboard; an engineered-trust instrument that is two-sided (protects the client from being cheated AND protects the company from scope-creep/non-payment/false disputes).

## 2. Who buys it / who uses it

- **Primary buyer & customer:** construction companies. Their interests are PRIMARY and resolve every trade-off.
- **End-users:** the company's clients — notably overseas Pakistanis building back home — plus the company's own staff (management, site engineers, accountants, storekeepers).
- **Value hierarchy (foundational):** end-clients' interests matter only to the degree they create value for the construction company.

## 3. Core job-to-be-done

- **Client:** stop wondering if they're being cheated (ghost expenses, material pilferage, drifting deadlines).
- **Company:** (a) win clients hesitant out of fear of being cheated; (b) get internal transparency over low/mid-level on-site staff who do most of the cheating without senior knowledge; (c) cut day-to-day management load by shifting from polling (chasing people/reports) to push (system self-collects).

## 4. The product spine

- **The wedge:** an automated **weekly "proof-of-innocence" report** (branded PDF to WhatsApp + in-app live view; English + Urdu). Reverses the burden of proof — the company proves innocence weekly with evidence, rather than the client hunting for guilt.
- **The report back-solves the MVP:** to produce it, the system MUST capture the **three atomic capture events**:
  1. **Material-in** — OCR'd delivery challan (quantities/rates off the paper, not a typist).
  2. **Money movement** — ledger entries.
  3. **Geotagged site photo** — trust evidence + %-progress proof + fraud datapoint.
- **Everything else is a computed VIEW** projected from these captures (client app, admin rollup, anomaly engine, PDF). Capture layer designed once; kept open (may grow past three).
- **Architecture:** API-first backend + responsive web app FIRST; native mobile later, built against the same API. Weekly PDFs archived server-side as a numbered, tamper-evident series (missing week is visible). PDF must be branded/self-explanatory and must NOT leak margins/supplier rates (it is forwardable = organic marketing).

## 5. MVP scope (MoSCoW, post-corrections)

### MUST (v1) — SHOULD tier was emptied into MUST
- Weekly proof-of-innocence report (PDF/WhatsApp + in-app, branded, Eng+Urdu) with **LLM-generated narrative**.
- The 3 capture events: OCR challan material-in, money movement, geotagged site photo.
- Milestones with **computed %** derived from BOQ line items marked done (not manually typed).
- Accounts + client balance; OCR posts material stock automatically, but the **accounts-payable entry is a recommendation the accountant edits and signs off** before commit (human-in-loop + separation of duties on money).
- Material vs BOQ tracking.
- Photo evidence timeline.
- **Change-order recording with tiered, configurable governance** (see §6).
- Admin cross-project rollup.
- **Anomaly flags v1:** rule engine + PyOD unsupervised + government rate-book price prior; internal-first (manager audit), two-tailed (flags under-consumption / suspiciously-clean too).
- Versioned drawings (with "issued for construction" marker).
- **Multi-auth identity:** phone + email + username/password; a client is one identity with **multiple concern-tagged contact numbers** (e.g. one for accounting, one for architecture).
- Low-literacy-friendly icon/photo/voice-first UX; app MANDATORY for every role incl. storekeeper.
- **Promoted to MUST with known-risk caveat:** CV progress-% estimation from photos, and on-device inference for integrity checks. *(AI/ML research judged single-photo CV progress-% unreliable — carry as known-risk MUST; fallback = manual / BOQ-derived %.)*
- Negative-confirmation approval loop in the report; Urdu/Punjabi voice-note ASR → decision audit log; decision audit log for out-of-app approvals; cash-heavy-site risk flag / traceable payment rails.
- **Supplier reconciliation (reinstated as MUST)** for confidence-through-redundancy: **Source A** (primary, real-time) = OCR'd delivery challan at material-in; **Source B** (secondary, periodic) = the supplier's **account statement issued TO the company** (customer khata of deliveries + dues — NOT the supplier's private profit ledger). Periodically OCR Source B and reconcile against accumulated Source A per supplier: agreement raises a data-credibility score; divergence raises an internal-first investigation. Source B has collection friction — it is confidence-enhancing, not a blocker; the challan stays the primary control.

### COULD (v2)
ML learned anomaly models + cross-company benchmarking; quality checklists / inspections / snag lists / test reports; deep document vault; quiet-mode (exceptions-only) client view; random spot-check photo prompts.

### WON'T / rejected
- Client-family parallel data entry (rejected — "never-ending story" in PK context).
- In-app client chat (rejected — users stay on WhatsApp).
- Supplier one-tap confirmation (rejected — suppliers won't engage).
- No-app SMS/IVR capture (rejected — app mandatory for all roles).
- Direct 3rd-party market-index data sale (not sellable in PK).
- Two-way WhatsApp Business integration on the company side (designed-for in architecture, deferred past MVP).

### Parked
Six monetization plays (see §8) — architecture kept extensible so any can switch on later.

## 6. Key design principles / decisions

- **Trust engineered through structure, not forced behavior change:** users stay on WhatsApp/calls; approvals happen OUTSIDE the app, the app records them after the fact.
- **Change-order governance (finalized):**
  - Separation of powers — anyone may RAISE a change; nobody who raises it may RATIFY it.
  - Lifecycle: Draft → Proposed (cost + schedule delta mandatory) → Client-Aware → Approved → Executed → Billed; each transition stamped who/when/evidence.
  - **Tiered, company-configurable authority** (defaults, all overridable): Tier A (≤1% or Rs50k) engineer proposes + client negative-confirm; Tier B (≤5% or Rs500k) manager sign-off + client evidence; Tier C (above) owner sign-off + explicit client approval, no negative-confirm.
  - **No retroactive billing:** a CO must reach Approved before its costs are billable; pre-approval work is flagged "executed before approval" and can't be billed until ratified — kills the end-of-project surprise bill.
  - **Append-only / immutable:** edits create versions, no deletion (only "cancelled"); a fabricated-then-walked-back CO leaves permanent evidence.
  - Segregation of duties: whoever logs cost against a CO can't be who proposed it.
  - CO fraud surveillance: rate-vs-benchmark, velocity per engineer (cross-project), round-number bias, "just-under-threshold" clustering.
- **Negative confirmation:** silence over a stated window = accepted; one tap / WhatsApp reply to dispute (allowed only for Tier A). Attach the client's OWN evidence (voice note / screenshot) to records so it's the client's voice, not a company paraphrase.
- **Data credibility as a first-class scored attribute** driven by source-agreement: any fact confirmed by 2+ independent captures carries higher confidence (extends beyond materials — e.g. photo+ledger, milestone+consumption).
- **Configurable defaults, not rigid rules:** every governance threshold ships as a recommended pre-filled default the company can reconfigure. Opinionated, not rigid.
- **Two separate axes:** internal-approval vs client-facing ratification are independent, both configurable.
- **North star (meta-requirement):** prefer configurable choice + recommended default over hard-coded answers; keep internal and client-facing concerns as separate axes; coach decides best option then notifies for review.
- **Anomalies are internal-first:** manager has authority/judgement over whether to disclose any given anomaly to the client (discretionary).
- Low-literacy-friendly UX (icon/photo/voice-first); app mandatory for all roles incl. storekeeper.

## 7. AI/ML strategy

AI/ML is a **standing default evaluation lens** on every feature, across **3 execution locations**:
1. **On-device** (browser/mobile) — integrity checks.
2. **Self-hosted open-source SLMs** — fraud/sensitive data kept in-house.
3. **Third-party LLM API** — only Urdu narrative + hard OCR, behind field-level redaction.

- **ML anomaly engine is feasible in v1** via 3-layer cold-start: rules + PyOD unsupervised + government rate-book price prior (no baseline wait needed).
- **Headline model picks:** Qwen2.5-VL (challan OCR + photo integrity + later progress-est, multi-task one deployment), Qwen2.5-7B (SLM narrative/reasoning), Whisper-v3 (Urdu/Punjabi ASR), PyOD (tabular anomaly).
- **Top feature bets:** challan OCR, ledger anomaly engine, LLM weekly report, voice-note transcription.
- Fuller detail in `ai-ml-research.md` (251 lines).

## 8. The moat & parked monetization

- **Long-term moat:** cross-company **anonymized benchmark dataset** (built via govt rate books → company history → cross-company pool). A single "expected value" service ("what should X cost/consume here?") feeds BOQ estimation, the anomaly engine, and CO pricing alike.
- **6 parked monetization plays** (keep architecture extensible — pluggable supplier/financier/insurer seam + pooled dataset as a first-class internal service):
  1. Group-buying / procurement brokerage (bulk-discount commission).
  2. Construction-finance underwriting referral (ledger/BOQ/progress data → bank/microfinance lending, revenue-share).
  3. Verified-supplier marketplace (placement/badge/leads).
  4. Premium benchmarking tier sold back TO the companies.
  5. AI BOQ auto-estimation / instant quote (upsell).
  6. Verified-contractor rating/certification (monetizes trust itself).

## 9. Open items / known risks

- **CV progress-% reliability** — promoted to v1 against research advice; needs manual / BOQ-derived fallback.
- **Sourcing MES Analysis of Rates** (and provincial MRS/CSR coefficients) for the benchmark price prior.
- **Build an internal Urdu eval set** for narrative/ASR quality.
- **Supplier account-statement (Source B) collection friction** — someone must obtain + scan periodic statements.
