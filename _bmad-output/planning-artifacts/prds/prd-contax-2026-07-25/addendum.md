---
title: "Contax PRD — Addendum (downstream technical detail)"
status: final
created: 2026-07-25
updated: 2026-07-25
---

# Contax PRD — Addendum

Depth that belongs downstream (architecture, solution design, ML/eval plan, GTM) but does not fit a capabilities PRD. This is the technical-how and rationale behind the PRD's FRs/NFRs — model picks, execution-location detail, cost/feasibility, benchmark-data source editions, rejected alternatives, and the parked monetization ladder. The PRD references this file; it does not duplicate it. Sourced from the brief addendum and `ai-ml-research.md` (251 lines).

---

## A. AI/ML — Three Execution Locations (detail behind NFR-4/NFR-5)

The governing rule: **all sensitive financial data and all fraud math stay self-hosted; a field-level redaction layer sits in front of any third-party API call.** Location is *config, not code*, via a provider/router abstraction that can swap providers or disable the API tier entirely. "Sending raw financial data to an external LLM is a trust liability for a trust product."

**1. On-device (browser/mobile) — cheap guards, offline-capable:**
- EXIF/geotag/timestamp validation; perceptual-hash (pHash) duplicate/stale-photo detection (largely classical, not ML).
- Small quality/plausibility VLM: **SmolVLM** (256M–2.2B) or **Moondream2** (~1.9B).
- Optional voice-note intent classifier: **SmolLM2** (135M–1.7B) or **Qwen2.5-1.5B** via transformers.js / ONNX Runtime Web.
- Runtimes: ONNX Runtime Web, transformers.js, WebLLM (MLC), TensorFlow.js.
- Constraint: mid-range Pakistani Android runs only small, quantized, single-purpose models — **not** in-browser 7B LLMs. Everything works offline; raw media + results queue for sync (NFR-2/NFR-19).

**2. Self-hosted server (default home for almost everything) — one modest GPU (or rented):**
- **Qwen2.5-VL-7B** (~6 GB, ~125K context, Apache-2.0) — the do-everything workhorse: challan OCR + photo QA + (later, private) CV progress.
- **Whisper large-v3** (1.55B, MIT) / **large-v3-turbo** (809M, ~1.6 GB, ~6 GB VRAM, ~6–8× faster) — Urdu/Punjabi ASR.
- **Qwen2.5-7B-Instruct** (7B, Apache-2.0) — English narrative + NL query.
- **PyOD / rules / Rate-Book** anomaly engine — CPU.
- Bulk OCR: **PaddleOCR (PP-OCRv5)** cheap first pass + **Qwen2-VL Urdu-OCR fine-tune** for Nastaʿlīq/handwriting.
- **Rule: the ledger/anomaly engine is self-hosted, never API.**

**3. Third-party LLM API (narrow, deliberate, redacted) — only two justified launch uses:**
- (a) **Urdu weekly narrative** (Claude / GPT / Gemini) — prose quality matters, inputs are structured/redactable.
- (b) **Hard-OCR escalation** for the small fraction of illegible documents — crop to the document, strip account context.
- A provider-abstraction/router lets the founder swap providers or turn the tier off.

---

## B. Feature-by-Feature Model Picks (behind the PRD's AI/ML FRs)

**Challan / receipt OCR (Urdu Nastaʿlīq + English + handwriting) — FR-6:**
- Two-tier: (1) **PaddleOCR (PP-OCRv5)** cheap first pass for clean printed English/numeric (tens of MB, Apache-2.0); (2) **Qwen2-VL / Qwen2.5-VL Urdu-OCR fine-tune** for Nastaʿlīq + handwriting.
- **Qaari-0.1-Urdu-OCR** (Qwen2-VL-2B fine-tune): WER 0.048 / CER 0.029 / BLEU 0.916 on 10k synthetic Nastaʿlīq images (5 fonts) — great for *printed* Urdu; **validate on handwriting**.
- **Arabic-English-handwritten-OCR** (Qwen2.5-VL-3B fine-tune) — supports Persian/Urdu/Turkish + handwriting; strong handwriting candidate.
- **Qwen2.5-VL** (3B/7B/72B; 3B/7B Apache-2.0): SOTA multilingual OCR; 72B tops OCR benchmarks; 7B is the self-host sweet spot; escalate messiest scans to hosted 72B or commercial API.
- Test-first / caveats: **dots.ocr** (1.7B, MIT) strong low-resource but Urdu/Arabic not explicitly listed; **Surya** (90+ langs, commercial terms [VERIFY]); **TrOCR** Latin-centric (Urdu needs fine-tune); **cxfajar197/urdu-ocr** (community).
- **OCR is the flywheel (rank #1 feature bet):** better OCR → richer structured data → better anomaly detection (quantity-vs-BOQ, rate-vs-MRS).

**Photo integrity / CV progress — FR-17, FR-51:**
- Integrity (Tier 2, mostly non-ML): EXIF + pHash + light near-duplicate embedding; on-device via SmolVLM/Moondream2.
- VLM photo QA / stage classification: **Qwen2.5-VL-7B** self-hosted.
- CV progress-% verdict (see §C).

**Ledger / tabular anomaly engine — FR-35..FR-41:**
- **PyOD** (BSD-2, 60+ detectors): **Isolation Forest** (workhorse), **ECOD** (parameter-free, deterministic, explainable per-feature — ideal with no labels to tune against), **LOF** (density/contextual), plus KNN/AutoEncoder.
- **Benford's Law** leading-digit tests on transaction amounts — cheap, two-tailed fraud signal.
- **AutoEncoder** reconstruction error — realistically month-one→year-one (needs rows), not day one.
- Alternate: **scikit-learn** (IsolationForest/LOF/OneClassSVM) if fewer deps wanted.
- v2 learned: hybrid **Autoencoder + Isolation Forest** (~0.98 acc in literature) + SHAP for explainability + temporal models for consumption series.

**LLM weekly narrative — FR-42, NFR-6, NFR-7:**
- English: **Qwen2.5-7B-Instruct** self-hosted (bounded summarization; small model suffices).
- Urdu at launch: **third-party API** (Claude/GPT/Gemini) → migrate to self-hosted **Qwen2.5-7B-Instruct** or **Gemma 3 (4B/12B)** once open Urdu quality is validated.
- **Critical caveat:** no small open model is reliably fluent in high-quality long-form Urdu prose yet; no clean public per-model Urdu benchmark found → **build an internal Urdu eval set before locking self-hosted Urdu** (R-5, NFR-7).
- Guardrail: **templated numeric guardrails** — numbers from the DB; the LLM writes prose only (NFR-6).
- SLM field: **Gemma 3** (1B–27B, 140+ langs) is the main Urdu challenger to Qwen; **Phi-4/3.5** English-centric (weak Urdu); **Mistral/Ministral** weak Urdu; **Llama 3.2** weaker Urdu than Qwen/Gemma.

**Urdu/Punjabi voice-note ASR + intent — FR-52:**
- Default: **Whisper large-v3** (best on conversational Urdu; safe for messy WhatsApp notes); **large-v3-turbo** for throughput.
- Urdu fine-tune: **kingabzpro/whisper-large-v3-urdu** (Common Voice 17) — benchmark vs base.
- **Punjabi (practical path):** few-shot fine-tune **Whisper-small/medium** (2025 CHIPSAL study shows sharp WER cuts on Punjabi/Pashto/Urdu). Punjabi confidence is lower → flag low-confidence transcripts for human confirmation (FR-52 assumption).
- Avoid in production until licensing confirmed: **SeamlessM4T-v2** (~2.3B, CC-BY-NC) and **MMS** (~1B, likely CC-BY-NC) — eval only.
- Intent classification of transcripts: **Qwen2.5 1.5B/3B** or **SmolLM2** on-device.

**NL query over project data (owner Q&A) — convenience layer, not trust core:** text-to-SQL/tool-call via self-hosted **Qwen2.5-7B-Instruct + strict SQL/tool schema**.

**Ranked v1 feature bets** (trust/anti-fraud value × tractability today × frequency): 1) Challan OCR → 2) Ledger/material anomaly engine → 3) Weekly report narrative → 4) Voice-note transcription+intent → 5) Photo integrity → 6) NL query → 7) CV progress → 8) cross-company benchmarking.

**Pragmatic MVP AI starting point:** rules + PyOD + MRS prior (self-host CPU) · PaddleOCR + one Urdu-OCR fine-tune (self-host) · Whisper self-host · on-device EXIF/pHash · **API for Urdu narrative only.** Second wave: Qwen2.5-VL-7B, NL query, on-device VLM. Defer CV progress to a research spike.

---

## C. Anomaly Engine Cold-Start (behind FR-36..FR-40)

ML can ship v1 if "ML" means unsupervised statistical/outlier detection seeded with external priors — **not** learned fraud classifiers (impossible day one: no labeled fraud, almost no data). Anomaly detection is the standard answer to label scarcity (as in intrusion detection, financial-audit tooling). "Normal" is seeded from three priors that exist before any production history:

- **Layer A — deterministic rules (day one, zero data):** the genuine day-one shippable; transparent (required to tell the client *why*), no drift. Rules in FR-36.
- **Layer B — unsupervised outlier detection (v1, weeks):** over engineered tabular features; no labels. Cold-start mitigations: (1) **cross-line-item within one project** (many line items/challans even on project #1); (2) **company-history bootstrap** at onboarding (import past closed projects, even spreadsheets → per-company baseline of normal rates/consumption/cost-per-sqft) — the *single highest-leverage cold-start move* (FR-38).
- **Layer C — external Rate-Book price prior (v1, "the centerpiece of a credible v1 engine"):** ingest MRS/CSR/MES into a rates reference table (item → provincial fair-rate range + edition/date). Turns "no baseline" into "government-published baseline" per province; feeds both Layer A rules and Layer B features.

**Two-tailed / internal-first:** Benford + deviation scoring flag both above- and below-expected (too-clean, under-consumption); baseline is internal (company history + intra-project) before any cross-company data; cross-company is v2 only.

**v1 vs v2:** v1 = rules + PyOD + Benford + MRS/CSR prior, no new data. v2 = supervised/semi-supervised on thousands of accumulated labeled flags + cross-company anonymized benchmark + continual learning. **Practical instruction (FR-41): log every flag and its human disposition from day one — the label store is the v1 engine's most important output.**

---

## D. CV Progress-% Verdict (behind FR-17, R-3)

- Research is candid: **single-photo % estimation is unreliable.** Trustworthy estimation needs segmentation, multiple viewpoints, or geometry/BIM references. Ranked **Tier 3** (highest value as an independent check on staff-reported progress, but hardest).
- **Recommended v1 guardrail:** start with **coarse stage classification** (VLM prompt: "is this photo consistent with grey-structure / plaster / finishing?"), not precise percentages. BOQ-derived % remains the number of record.
- Precise CV % is a **v2/v3 research bet** — dedicate a later spike; when done, self-hosted **Qwen2.5-VL-7B / segmentation model** (GPU-heavy, kept private).
- Evidence: indoor progress monitoring (ScienceDirect); CV progress review 2025; Mask R-CNN completion % (ASCE).
- **This is retained as a v1 MUST by owner's call, against the research's advice — made safe by the BOQ fallback + shadow-mode (FR-17).**

---

## E. Cost / Feasibility (behind NFR-1/NFR-2)

- No explicit $/page figure in the research; stance is qualitative: per-token API costs are fine for low-volume high-value tasks (weekly narrative, occasional hard OCR) but **expensive at the scale of every challan/photo/voice note** → those self-hosted or on-device. (Brief's market research: self-hosted OCR ~$0.09/1k pages vs ~$1.50 cloud, ~16×.)
- **Qwen2.5-VL-7B** ~6 GB / ~125K context. **Whisper large-v3-turbo** 809M / ~1.6 GB / ~6 GB VRAM / ~6–8× faster.
- **WebLLM** 30–70 tok/s on laptops; marginal on mid-range Android — tiny SLMs only there.
- **Self-hosted server** = one modest GPU comfortably runs 7B-class VLM/SLM/Whisper; anomaly engine on CPU; 70B+ needs hosted API or bigger investment.
- **Async by default:** capture now, process on sync; voice-note volume is bursty/short; cloud calls must tolerate retries + queueing (NFR-2).

---

## F. Data Governance Architecture (behind NFR-4/5/6/12/14)

Baked-in defaults: (1) field-level redaction layer in front of any API call; (2) provider/router abstraction (swap or disable API tier); (3) **label/disposition store capturing every anomaly flag's human outcome from day one** (fuel for v2); (4) templated numeric guardrails on all LLM report generation. Product stance throughout: **opinionated defaults, not rigid** — every default is a reconfigurable starting point; location per task is config, not code.

**[VERIFY] license re-checks before commercial commitment:** Qaari, Arabic-English handwritten OCR, whisper-large-v3-urdu, Surya commercial terms, Qwen2.5-VL-72B card, Llama 3.2 exact terms, Gemma exact terms, Moondream2, SeamlessM4T, MMS. Confirm PaddleOCR Urdu/Arabic-script pack.

---

## G. Change-Order Governance — Full Mechanics (behind FR-27..FR-34)

Purpose: stop an unguarded "the client approved X" from becoming a fraud instrument — a fabricated CO is otherwise perfect cover for pilferage or an inflated bill that lands on the client while management sees a normal record.

- **Separation of powers** — anyone may raise; whoever raised may never ratify or bill (FR-29).
- **Lifecycle** — Draft → Proposed (cost + schedule delta mandatory) → Client-Aware → Approved → Executed → Billed; every transition stamped who/when/evidence; append-only (FR-27, FR-28).
- **Three authority tiers, company-configurable (defaults):**
  - **Tier A** — ≤1% or Rs 50k — site engineer proposes; client **negative confirmation**.
  - **Tier B** — ≤5% or Rs 500k — manager sign-off; client **positive evidence** (voice note / screenshot / in-app tap).
  - **Tier C** — above — owner/admin sign-off; **explicit approval**, no negative confirmation.
- **All three ratification modes always offered per tier** — the mapping is a recommended default; any cell overridable (FR-31). (Proposed hard floor: Tier C never permits negative confirmation — confirm, Open Q5.)
- **Two independent axes** — internal approval (senior-management sign-off) and client ratification; the "executed before approval" gate is internal, separate from client ratification (FR-32).
- **No retroactive billing** — a CO must reach Approved before billable; pre-approval work is logged, flagged "executed before approval," not billable until ratified. Kills the end-of-project surprise bill (FR-33).
- **Segregation of duties** — whoever logs a purchase/consumption against a CO ≠ the proposer; defeating it requires cross-rank collusion (FR-30).
- **Immutable trail** — append-only; edits create versions; no delete (only Cancelled). A fabricated-then-walked-back CO leaves permanent evidence.
- **CO fraud surveillance** — rate-vs-benchmark, CO velocity per engineer (cross-project), round-number bias, just-under-threshold clustering (FR-34).
- **Negative-confirmation mechanics** — decision happens on WhatsApp; the weekly report carries "Decisions recorded this week"; silence over a stated window = accepted, one reply disputes; a week with none says "decisions recorded this week: none" — making unrecorded verbal decisions visible by absence. Attach the client's *own* evidence so the record is the client's voice, not the company's paraphrase (FR-46).

---

## H. The Data-Entry Paradox — Mechanisms (behind R-1)

The fox feeds the henhouse ledger: whoever enters data on site is who cheats. Answers developed (mechanisms proposed, **not yet field-tested** — the pilot must test them):
- **Anchor on what already exists and doesn't lie** — delivery challans, supplier invoices, the geotagged photos, the client's own payments.
- **Even fraudulent data has value** — false entries can still be flagged and investigated; none of that is possible without data at all.
- **"No receipt, no acceptance"** — a cadence refusing material without a challan is enforceable when insisted on by the paying party; CV reads the receipt, so quantities/rates come off paper, not a typist (FR-7).
- **Supplier paper is relatively trustworthy** — supplier–site collusion is low-probability; the supplier's long-term interest is staying in the company's good books.
- **Labour & consumption fraud** (the paperless leak points) caught against industry-average labour productivity / material consumption and against BOQ.
- **Mechanisms:** role-separated double entry; evidence-mandatory entries (geotagged challan + truck photo); append-only ledger with auto-flagged backdating; the client as free auditor; random spot-check photo prompts (v2); day-one rule engine; supplier khata reconciliation via OCR; traceable payment rails; cash-heavy sites flagged.

---

## I. Supplier Reconciliation (behind FR-24..FR-26)

- **Source A (primary, real-time):** OCR'd challan at material-in.
- **Source B (secondary, periodic):** the supplier's **account statement issued to the company** — the customer khata of deliveries + dues, *not* the supplier's private profit ledger. Periodically OCR'd and reconciled against accumulated Source A per supplier: agreement raises credibility; divergence → internal-first investigation.
- Source B has collection friction — confidence-enhancing, not a blocker. Rejected: supplier one-tap confirmation (suppliers won't engage). Workflow (who obtains/scans, cadence, refusal path) undefined — R-7 / Open Q6.

---

## J. Client Delivery & Report Design (behind FR-42..FR-50)

- Client app is built (primary client surface); MVP form factor is a responsive web app; backend API-first so native mobile follows.
- **WhatsApp delivery = a PDF report**, not a link into the app.
- **PDF is a kept artifact:** works without signal, feels official, archived server-side as a numbered series so a missing week is visible.
- **PDF is forwardable = organic marketing:** branded, self-explanatory to a stranger, and **must never leak supplier rates or margins** (forwarding reaches competitors).
- **Company-side two-way WhatsApp Business integration** — provided for in architecture, deferred past MVP. Constraint: WhatsApp Business Platform requires pre-approved templates, a 24-hour session window, metering, and a Business Solution Provider; unofficial libraries get numbers banned. *(Meta changes these rules often — verify current terms before designing on them.)*

---

## K. Benchmark Data Sourcing (moat foundation — behind FR-39, FR-56, R-6)

- **Day one:** provincial **MRS / Composite Schedule of Rates** (Finance and P&D departments) and the **MES schedule**. The *Analysis of Rates* behind them breaks each work item into labour and material coefficients (mason-days per unit of brickwork, cement bags per m³ of a given mix, standard wastage %). Public; the language government and large contractors already argue in.
  - MRS: per province, updated bi-annually by provincial Finance Depts; MRS-2024 ~4,500 items across 27 construction chapters, composite rates incl. overheads/profit.
  - CSR: e.g. Sindh CSR-2022 (NHA + provincial works depts).
  - **MES Analysis of Rates** — [VERIFY: no current public doc retrieved; source latest edition directly.]
- **Month one:** the company's own aggregated project history — a benchmark a site engineer cannot dismiss as "those norms don't apply to our work."
- **Year one:** the cross-company pool — the real, uncopyable moat.
- *Current editions and per-province granularity to be verified (R-6, Open Q7).*

---

## L. The Moat & Parked Monetization Ladder (behind §11, NFR-3)

**Moat:** cross-company anonymized benchmark dataset, built from government rate books → company history → cross-company pool. A single **Expected-Value Service** ("what should X cost/consume here?") feeds BOQ estimation, the anomaly engine, and CO pricing alike. The eventual company owns Pakistan's construction cost-and-productivity dataset, not an app.

**Six parked monetization plays** (architecture kept extensible — pluggable supplier/financier/insurer seam + pooled dataset as a first-class internal service, NFR-3):
1. Group-buying / procurement brokerage (bulk-discount commission).
2. Construction-finance underwriting referral (ledger/BOQ/progress → bank/microfinance lending, revenue-share).
3. Verified-supplier marketplace (placement / badge / leads).
4. Premium benchmarking tier sold back to the companies.
5. AI BOQ auto-estimation / instant quote (upsell).
6. Verified-contractor rating/certification (monetizes trust itself).

Pricing shape mirrors the moat: SaaS entry (per-active-project subscription), data/platform economics later.

---

## M. Competitive Landscape (behind §12)

Bottom line: the whitespace is real but narrow — a *positioning* moat (anti-fraud FOR the remote owner), not a technology one; every mechanic already ships somewhere.

- **Smart Construction (Pakistan)** — closest/most dangerous. Contractor ERP from ~PKR 3,000/mo: procurement/PO governance, site diary, inventory ledgers, HR/payroll, audit logs, **WhatsApp automation, AI Copilot, branded bill PDFs for clients**. Sold to the contractor; no confirmed dedicated client portal, no adversarial framing. One product decision from closing the visible gap.
- **EZYPRO / EzyPMP (Lahore)** — "Pakistan's first cloud construction management software"; contractor mid-market/enterprise PM/ERP (drawings approvals, WBS/Gantt/BOQ, bid & contract mgmt, QA/QC + lab reports, IPC/payment certificates, variation tracking, cost/billing, ticketing/chat, mobile). Clients: ADB, China State Construction (Pakistan). Generic stakeholder-visibility but no dedicated owner portal, no anti-fraud framing; institutional projects. *(Vendor claims; homepages 403 to fetch. Name collision with unrelated Australian "EZYPM".)*
- **Buildertrend / CoConstruct** (global, ~$99–$1,099/mo) — genuine homeowner portals with real-time updates, change-order approvals, budget visibility, AI weekly summary — but *builder-controlled* and framed as engagement, not adversarial anti-fraud.
- **Procore** (enterprise, ~$10k–$50k+/yr, volume-priced) — project controls/financials; no homeowner portal.
- **Powerplay (~₹72k/yr) / Onsite** (India SMB) — material mgmt, attendance, auto progress; contractor-facing.
- **Benchmark datasets** — mature in developed markets (RSMeans, Rate QS); **none for Pakistan/South-Asia SMB residential** → moat space open (compounds only at scale).

**Why-now signals (verified where noted):** smartphone penetration ~62%→72% (2024→mid-2026, directional); WhatsApp ~52M–105M users, PK #3 in downloads; remittances record $38.3B FY2025 +27% (top: Saudi 24%, UAE 20%, UK 15%; caveat: ~70% funds consumption, overseas-construction PKR volume NOT verified); digital payments — EasyPaisa 59M reg/20M MAU, JazzCash ~48M/20M+ MAU, ~88% of retail transactions digital (well-corroborated); OCR/LLM cost collapse (~16×, solid); cost overruns — one study 100% of projects overran avg ~34.6% (15–88%), strong pain evidence but **material theft NOT isolated** in the literature.

**Two claims not to overstate:** material theft as a quantified problem, and overseas-funded home-construction PKR volume. Both unverified; lead with documented cost-overrun pain.
