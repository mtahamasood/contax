# Contax — AI/ML Architecture Research

**Decision-support document for a technical founder**
Prepared: 2026-07-23 · Scope: v1 feasibility, feature-fit ranking, concrete open-source model picks, and execution-location mapping (client-side / self-hosted / third-party API).

> **How to read this.** Sections 1–4 answer the four research questions directly. Each model recommendation names a concrete artifact, its approximate size, license, and where it should run. Where a claim could not be fully verified via web search, it is flagged inline with **[VERIFY]**. Contax's stated product stance — *"opinionated defaults, not rigid"* — is respected throughout: every default below is a starting point the founder can reconfigure.

---

## Executive summary (TL;DR)

- **ML anomaly detection CAN ship in v1** — but "ML in v1" means *unsupervised statistical/outlier detection seeded with external priors*, not learned fraud classifiers. The cold-start is solved by (a) a deterministic rule engine, (b) statistical outlier methods (Isolation Forest / ECOD / LOF via PyOD) run per-project and cross-line-item, and (c) an **external price prior** built from Pakistan's published Market Rate System (MRS) / Composite Schedule of Rates (CSR). Learned, company-specific anomaly models arrive in v2 once labeled outcomes accumulate.
- **Top AI/ML bets, ranked:** (1) OCR/document extraction of challans & receipts, (2) rule+statistical fraud/anomaly engine on ledger + material consumption, (3) LLM report-narrative generation (English + Urdu), (4) voice-note transcription + intent classification for client approvals. Photo *understanding* (progress estimation) is high-value but high-difficulty — start with cheap photo *integrity* checks (EXIF/geotag/duplicate) and defer CV progress estimation.
- **Headline model picks:** OCR → **Qwen2.5-VL-7B** (general) + a **Qaari/Qwen2-VL-2B Urdu-OCR** fine-tune for Nastaʿlīq challans; VLM photo QA → **Qwen2.5-VL-7B** with **Moondream2 / SmolVLM** on-device; SLM for reports/NL query → **Qwen2.5-7B-Instruct** self-hosted (Gemma 3 / Llama 3.x as alternates), **Claude/GPT for the hard long-form narrative** at first; ASR → **Whisper large-v3 / large-v3-turbo** and a **Whisper-large-v3-Urdu** fine-tune, with **SeamlessM4T-v2** as an alternate for read-speech; tabular anomaly → **PyOD** (Isolation Forest + ECOD + LOF) plus Benford's-law digit tests.
- **Execution-location default:** sensitive fraud math and ledger data stay **self-hosted**; cheap integrity checks and quick classification run **on-device**; only the hardest language generation and messy-document OCR go to a **third-party API**, and only with field-level redaction. Rationale and full mapping in Section 4.

---

## 1. Can an ML anomaly-detection engine realistically be in v1? (Cold-start)

**Short answer: yes — if "ML" in v1 is defined honestly.** You cannot train a supervised fraud classifier on day one because you have no labeled fraud examples and almost no production data. But *anomaly detection* is precisely the family of techniques designed for the "no labels, define normal, flag deviations" problem. Unsupervised methods are the industry-standard answer to label scarcity — the same reason they dominate network-intrusion and financial-audit tooling ([PyOD](https://pyod.readthedocs.io/); [continual unsupervised auditing, arXiv](https://arxiv.org/pdf/2112.13215)).

The trick for Contax is that "normal" doesn't have to come from *your* data. It can be seeded from **three external/internal priors** that exist before you have any production history.

### 1.1 The three-layer cold-start architecture

**Layer A — Deterministic rule engine (day one, zero data).**
Hard, explainable rules encoding domain fraud patterns. Examples:
- Challan quantity > BOQ-remaining quantity for that material.
- Milestone % completion increases while zero corresponding material consumed.
- Payment recorded with no challan / no photo within N days.
- Rate on a purchase exceeds MRS composite rate by > X%.
- Duplicate challan number, or same photo hash reused across milestones.
- Cash outflow timing clustered suspiciously before report generation.

Rules are the backbone: they are transparent (critical for a *trust* product — you must be able to tell a client *why* something was flagged), need no data, and never "drift." This is your genuine day-one shippable.

**Layer B — Statistical / unsupervised outlier detection (v1, weeks not months).**
Run classic outlier detectors over engineered tabular features (unit rate, quantity-per-milestone, consumption-vs-BOQ ratio, cost-per-sqft, vendor concentration, transaction cadence). These need *no labels* — they model the empirical distribution and score deviation:
- **Isolation Forest** — isolates outliers via random partitioning; fast, robust, the default workhorse ([PyOD models](https://deepwiki.com/yzhao062/pyod/3-models)).
- **ECOD** (Empirical-CDF Outlier Detection) — **parameter-free**, deterministic, explainable per-feature; PyOD's recommended beginner default and ideal when you cannot tune hyperparameters against labels ([PyOD](https://pyod.readthedocs.io/)).
- **Local Outlier Factor (LOF)** — density-based, catches contextual outliers a global method misses.
- **Benford's Law digit tests** on transaction amounts — a classic, cheap, well-evidenced audit red-flag: fraudulent/altered amounts deviate from the expected leading-digit distribution ([Benford + ML for fraud, arXiv 2025](https://arxiv.org/pdf/2507.08650); [Benford + ML case study, ResearchGate](https://www.researchgate.net/publication/321024564)).

The cold-start weakness of Layer B is that with one project you have few rows. Two mitigations:
1. **Cross-line-item, not cross-project:** within a single project you have *many* BOQ line items, transactions, and challans — enough rows to find intra-project outliers (one vendor's cement rate vs all cement rates) even on project #1.
2. **Company-historical bootstrap:** at onboarding, import the construction company's *past* closed projects (even as spreadsheets) to establish a per-company baseline of normal rates, consumption ratios, and cost-per-sqft. This is the single highest-leverage cold-start move.

**Layer C — External price prior from Pakistani rate schedules (v1, huge leverage).**
Pakistan publishes authoritative construction rate books that give you a *ground-truth "fair price" distribution* for materials and labor before you have any customer data:
- **Market Rate System (MRS)** — issued per province, updated bi-annually by provincial Finance Departments; MRS-2024 covers ~4,500 items across 27 construction chapters (cement, bricks, sand, concrete, plaster, tiles, electrical, etc.) with composite rates including overheads/profit ([KP Finance MRS-2024](https://www.finance.gkp.pk/attachments/b8e8f730589111efa87555dca6816d3d/download); [KP MRS portal](https://communication_works.kp.gov.pk/page/market_rate_system_mrs/page_type/citizen); [Punjab MRS Rawalpindi 2024](https://www.slideshare.net/slideshow/government-of-punjab-market-rate-system-rawalpindi-1-2024_2-pdf/269436526); [Daily Ausaf: new govt rates](https://dailyausaf.com/en/business/new-list-of-government-rates-for-cement-bricks-and-labour-released/)).
- **Composite Schedule of Rates (CSR)** — e.g. Sindh CSR-2022 used by NHA and provincial works departments ([Sindh CSR 2022, NHA PDF](https://nha.gov.pk/uploads/topics/16569807987105.pdf)).
- **MES Analysis of Rates** — the military/cantonment analogue **[VERIFY: could not pull a current public MES AoR document in this research pass; treat as "known to exist, source the latest edition directly."]**

Ingest these into a **rates reference table** (item → provincial fair-rate range, with the edition/date). Now Layer A rules and Layer B features both have a principled "expected" value on day one, per province. This turns "we have no baseline" into "we have a government-published baseline." **This is the centerpiece of a credible v1 anomaly engine.**

### 1.2 v1 vs v2 recommendation for the anomaly engine

| Aspect | **v1 (day-one → month-one)** | **v2 (year-one, learned)** |
|---|---|---|
| Core method | Rule engine + PyOD (Isolation Forest / ECOD / LOF) + Benford + MRS/CSR price prior | Supervised & semi-supervised models trained on accumulated *labeled* outcomes (flag → confirmed fraud / false alarm) |
| Data needed | None new; company history + public rate books | Thousands of labeled flags across many projects |
| Baseline of "normal" | External (MRS/CSR) + per-company historical import | Learned per-company and cross-company (anonymized benchmark dataset) |
| Explainability | High (rules + per-feature ECOD scores) — essential for trust | Add SHAP/feature-attribution to keep flags explainable |
| Advanced models | (optional) tabular **autoencoder** reconstruction error once enough rows exist | Autoencoder / hybrid **Autoencoder + Isolation Forest** (well-evidenced combo, ~0.98 acc in literature) ([hybrid AE+IF, Springer 2025](https://link.springer.com/article/10.1007/s10115-025-02580-6); [AE+IF edge, arXiv 2025](https://arxiv.org/html/2511.18235v1)); temporal models for consumption time-series |
| Cross-company | Not yet | Anonymized benchmark → each company scored vs peer distribution; continual-learning to handle drift ([continual auditing, arXiv](https://arxiv.org/pdf/2112.13215)) |
| Feedback loop | Capture human dispositions from day one (label store) — this *is* what makes v2 possible | Retrain on the label store; monitor precision/recall of flags |

**Practical instruction:** Ship v1 as rules + statistics + MRS prior, and — critically — **log every flag and its human disposition from day one**. The v1 engine's most important output isn't just alerts; it's the labeled dataset that unlocks v2. Autoencoders are listed as an *option* in v1 (they're unsupervised and need no labels) but they need enough rows to learn a reconstruction manifold, so they realistically belong at the month-one → year-one boundary, not day one.

---

## 2. Which product features are the best fits for AI/ML? (Ranked)

Ranking criteria: value to the core trust/anti-fraud job × technical tractability today × frequency of use.

### Tier 1 — Do these first (highest ROI, tractable now)

**1. OCR / document extraction from challans & receipts (Urdu + English, handwriting).**
*What the model does:* read a photographed delivery challan or receipt and extract structured fields — vendor, material, quantity, unit, rate, date, challan number — into the ledger/material tracker.
*Why it fits:* it's the highest-friction manual step on site, it directly feeds the fraud engine (quantity vs BOQ, rate vs MRS), and modern multilingual VLM-OCR handles mixed Urdu/English and (imperfectly) handwriting. This is the flywheel: better OCR → richer structured data → better anomaly detection. **Rank #1.**

**2. Anomaly / fraud detection on ledger + material consumption.**
*What it does:* Section 1's engine — scores transactions, rates, and consumption ratios; raises explainable flags.
*Why it fits:* it *is* the product's reason to exist. Tractable in v1 via rules+stats. **Rank #2.**

**3. Auto-generating the weekly report narrative (LLM summarization, English + Urdu).**
*What it does:* turn structured weekly deltas (milestones advanced, spend, materials received, flags, change orders) into a clean human narrative + the WhatsApp-friendly PDF, in English and Urdu.
*Why it fits:* recurring, bounded, and forgiving — a summarization task LLMs are excellent at; templated numbers guard against hallucination. High perceived value for overseas clients. **Rank #3.**

**4. Voice-note transcription + intent classification for client approvals.**
*What it does:* transcribe Urdu/Punjabi WhatsApp voice notes (change-order approvals, instructions), then classify intent (approve / reject / question / instruction) and link to the relevant change order.
*Why it fits:* clients genuinely communicate this way; capturing an auditable approval trail from a voice note is a real trust artifact. ASR for Urdu is now workable (Section 3.4). **Rank #4.**

### Tier 2 — High value, more effort or dependency

**5. Photo *integrity* checks (EXIF/geotag/timestamp validation, duplicate/stale-photo detection).**
*What it does:* verify a site photo's GPS is on-site, timestamp is fresh, and the image isn't a reused/duplicate of an earlier upload (perceptual hashing + embedding similarity).
*Why it fits:* cheap, mostly *non-ML* (EXIF parse + pHash) with a light embedding model for near-duplicate detection; directly anti-fraud (stops "recycled progress photos"). Ranked in Tier 2 only because much of it is classical, not because it's low value — arguably do it alongside Tier 1.

**6. Natural-language querying of project data for the owner.**
*What it does:* "How much cement is left on the DHA project?" → SQL/tool call over project data → answer. Text-to-query + retrieval, not free-form generation.
*Why it fits:* delightful for the multi-project admin/owner view; tractable with a self-hosted SLM + strict tool/SQL schema. Slightly lower rank because it's a convenience layer, not the trust core.

### Tier 3 — High value but hardest; stage carefully

**7. Progress estimation from site photos (computer vision → % completion).**
*What it does:* infer construction stage / element completion from photos to cross-check the claimed BOQ-derived milestone %.
*Why it fits (and why it's Tier 3):* enormously valuable as an *independent* check on staff-reported progress — but the research is candid that image-based % estimation is unreliable from single representative photos and needs segmentation, multiple viewpoints, or geometry/BIM references to be trustworthy ([indoor progress monitoring, ScienceDirect](https://www.sciencedirect.com/science/article/pii/S2666721524000346); [CV progress review 2025, ScienceDirect](https://www.sciencedirect.com/science/article/pii/S2666165925002327); [Mask R-CNN completion %, ASCE](https://ascelibrary.org/doi/10.1061/9780784483961.074)). Start with **coarse stage classification** ("is this photo consistent with 'grey-structure / plaster / finishing'?") via a VLM prompt rather than precise percentages. Precise CV % estimation is a v2/v3 research bet.

**8. Cross-company anonymized benchmarking.**
Future ML; depends on multi-tenant data volume. Correctly a later generation.

**Ranked list (compact):**
1. Challan/receipt OCR (Urdu+English) → 2. Ledger/material anomaly engine → 3. Weekly report narrative (LLM) → 4. Voice-note transcription + intent → 5. Photo integrity (EXIF/geo/duplicate) → 6. NL query over project data → 7. CV progress estimation → 8. Cross-company benchmarking.

---

## 3. Specific suitable open-source models

All sizes/licenses verified via web search where noted; **[VERIFY]** flags where the model card should be re-checked before committing (licenses and Urdu support in particular change and vary by fine-tune).

### 3.1 OCR / Document AI (multilingual incl. Urdu/Arabic + handwriting)

| Model | Task | ~Size | License | Exec location | Notes |
|---|---|---|---|---|---|
| **Qwen2.5-VL** (3B / 7B / 72B) | VLM-based OCR + doc parsing | 3B / 7B / 72B | Apache-2.0 (3B/7B); 72B check card **[VERIFY]** | Self-host (7B) / API (72B) | SOTA multilingual OCR & document parsing; 72B tops OCR parsing benchmarks; 7B is the practical self-host sweet spot (~6 GB) ([Qwen2.5-VL report](https://arxiv.org/pdf/2502.13923)) |
| **Qaari-0.1-Urdu-OCR** (Qwen2-VL-2B fine-tune) | **Urdu Nastaʿlīq OCR** | 2B | Inherits Qwen2-VL-2B (Apache-2.0) **[VERIFY]** | Self-host / on-device (quantized) | Purpose-built for Urdu: **WER 0.048, CER 0.029, BLEU 0.916**; trained on 10k synthetic Nastaʿlīq images, 5 fonts. Great for *printed* Urdu challans; validate on *handwriting* ([Qaari HF](https://huggingface.co/oddadmix/Qaari-0.1-Urdu-OCR-VL-2B-Instruct)) |
| **dots.ocr** (rednote-hilab) | Unified layout + OCR | 1.7B LLM / ~3B total | **MIT** | Self-host | Single-model layout detection + reading order + OCR; strong on low-resource languages (100-lang internal bench) but **Urdu/Arabic not explicitly listed** — test before trusting ([dots.ocr HF](https://huggingface.co/rednote-hilab/dots.ocr)) |
| **PaddleOCR (PP-OCRv5)** | Detection + recognition + tables/handwriting | Small (det+rec, tens of MB) | Apache-2.0 | Self-host / edge | Best "classic" default for printed text & broad languages; handwriting, rotated/curved text; PP-StructureV3 for tables. Very light vs VLMs. Confirm Urdu/Arabic-script recognition pack **[VERIFY]** ([PP-OCRv5 HF](https://huggingface.co/PaddlePaddle/PP-OCRv5_server_det); [Modal OCR comparison](https://modal.com/blog/8-top-open-source-ocr-models-compared)) |
| **Surya** (Datalab) | Detection + recognition + layout, 90+ langs | Small | Open weights (check commercial terms) **[VERIFY]** | Self-host | Strong multilingual line OCR + layout; core of the Marker pipeline ([KDnuggets OCR](https://www.kdnuggets.com/top-7-open-source-ocr-models)) |
| **TrOCR** (microsoft) | Single-line printed/handwritten OCR | 334M (base) / 558M (large) | MIT | Self-host / on-device | Transformer OCR for single-line crops; Latin-centric out of the box — Urdu needs fine-tuning. Good as a *component* after line detection |
| **cxfajar197/urdu-ocr** | Urdu OCR / handwriting | Small **[VERIFY]** | Check card **[VERIFY]** | Self-host | Community Urdu OCR model; candidate to benchmark/fine-tune for handwritten challans ([HF](https://huggingface.co/cxfajar197/urdu-ocr)) |
| **Arabic-English-handwritten-OCR** (Qwen2.5-VL-3B fine-tune) | Handwritten Arabic-script OCR | 3B | Inherits Qwen2.5-VL **[VERIFY]** | Self-host | Explicitly supports Persian/**Urdu**/Turkish + handwriting — a strong handwriting candidate ([HF](https://huggingface.co/sherif1313/Arabic-English-handwritten-OCR-v3)) |

**OCR recommendation:** two-tier. (1) **PaddleOCR** as a cheap first pass for clean printed English/numeric fields. (2) A **Qwen2-VL/Qwen2.5-VL Urdu-OCR fine-tune (Qaari or the Arabic-English-handwritten model)** for Urdu Nastaʿlīq and handwriting. For the messiest scans, escalate to a hosted VLM (Qwen2.5-VL-72B or a commercial API). Note: 2026 guidance explicitly recommends olmOCR / Qwen2.5-VL-class models for messy layouts & handwriting over classic engines ([Unstract 2026 OCR guide](https://unstract.com/blog/best-opensource-ocr-tools/)).

### 3.2 Vision-Language Models (photo understanding)

| Model | Task | ~Size | License | Exec location | Notes |
|---|---|---|---|---|---|
| **Qwen2.5-VL-7B** | Site-photo QA, stage classification, doc VQA | 7B (~6 GB) | Apache-2.0 | Self-host | Best perf/efficiency balance; 7B beats Llama-3.2-11B-Vision on several benches; ~125K context; good for "is this photo consistent with stage X?" ([Labellerr comparison](https://www.labellerr.com/blog/qwen-2-5-vl-vs-llama-3-2/)) |
| **Llama 3.2 Vision** (11B / 90B) | Photo understanding, doc VQA | 11B / 90B | Llama 3.2 Community License (restrictions) **[VERIFY]** | Self-host (11B) / API (90B) | Strong OCR/doc-VQA, 128K context; license is not fully open — check for your commercial use ([BentoML VLM guide](https://www.bentoml.com/blog/multimodal-ai-a-guide-to-open-source-vision-language-models)) |
| **Moondream2** | Lightweight photo QA, structured output | ~1.9B | Apache-2.0 **[VERIFY]** | **On-device / edge** | Tiny, fast; good for on-device integrity/quick captioning; structured output support ([Roboflow local VLMs](https://blog.roboflow.com/local-vision-language-models/)) |
| **SmolVLM** (256M / 500M / 2.2B) | Lightweight photo QA | 256M–2.2B | Apache-2.0 | **On-device / browser** | Designed for resource-constrained & browser/mobile real-time; ideal for on-device duplicate/quality screening ([Labellerr VLMs](https://www.labellerr.com/blog/top-open-source-vision-language-models/)) |

**VLM recommendation:** **Qwen2.5-VL-7B** self-hosted is the do-everything workhorse (doubles as the OCR engine). Put **SmolVLM/Moondream2** on-device for instant, offline photo screening (blurry? duplicate? plausibly a construction site?). Reserve cloud 72B/90B for occasional hard cases.

### 3.3 Small Language Models / SLMs (report generation, NL query)

| Model | Task | ~Size | License | Exec location | Notes |
|---|---|---|---|---|---|
| **Qwen2.5-7B-Instruct** | Report narrative, NL→SQL, summarization | 7B | Apache-2.0 | Self-host | Leads open benchmarks on reasoning/math/**multilingual**; strongest non-English of the small set — best self-host default for Urdu+English generation ([MLMastery SLMs](https://machinelearningmastery.com/top-7-small-language-models-you-can-run-on-a-laptop/)) |
| **Qwen2.5 1.5B / 3B** | Lightweight NL query, classification | 1.5B / 3B | Apache-2.0 | Self-host / edge | Multilingual; good for cheap intent classification of transcribed voice notes |
| **Gemma 3** (1B/4B/12B/27B) | Report narrative, multilingual | 1B–27B | Gemma license (open-ish, terms apply) **[VERIFY]** | Self-host / edge (4B) | **140+ languages**; 4B is multilingual+multimodal and light; strong Urdu candidate — benchmark vs Qwen ([DataCamp SLMs](https://www.datacamp.com/blog/top-small-language-models)) |
| **Llama 3.2** (1B / 3B) | Edge NL query, classification | 1B / 3B | Llama 3.2 license **[VERIFY]** | On-device / edge | Smallest footprints; 1B for mobile/edge; multilingual but weaker Urdu than Qwen/Gemma — verify |
| **Phi-4 / Phi-3.5** (mini ~3.8B) | Reasoning-heavy summarization | ~3.8B | MIT | Self-host / edge | Strong reasoning per-param; English-centric — Urdu is a weak spot **[VERIFY]** |
| **Mistral 7B / Ministral** | General generation | 7B / 3B–8B | Apache-2.0 (7B) | Self-host | Solid English; weaker Urdu; keep as fallback |
| **SmolLM2** (135M–1.7B) | Tiny on-device classification | ≤1.7B | Apache-2.0 | On-device / browser | For transformers.js/WebGPU intent classification; not for long-form Urdu |

**SLM recommendation:** self-host **Qwen2.5-7B-Instruct** as the default generation/NL-query engine (best small-model Urdu). Evaluate **Gemma 3 4B/12B** head-to-head on your Urdu report prose — it's the main challenger. **Important caveat:** no small open model is reliably fluent in high-quality *Urdu long-form prose* yet; for the client-facing weekly narrative in Urdu, **start on a third-party API (Claude/GPT/Gemini)** and migrate to self-hosted Qwen/Gemma as you validate quality. General web sources did not give clean per-model Urdu benchmarks — **[VERIFY: run your own Urdu eval set before locking this in.]**

### 3.4 Speech-to-Text (Urdu / Punjabi)

| Model | Task | ~Size | License | Exec location | Notes |
|---|---|---|---|---|---|
| **Whisper large-v3** | Urdu/Punjabi ASR (conversational) | 1.55B | **MIT** | Self-host | Best on *conversational* Urdu in benchmarks; the safe default for messy WhatsApp voice notes ([WER We Stand, ACL/arXiv](https://arxiv.org/html/2409.11252v3)) |
| **Whisper large-v3-turbo** | Faster ASR | 809M (4 decoder layers) | MIT | Self-host / stronger edge | ~6–8× faster, ~1.6 GB, ~6 GB VRAM, minimal accuracy loss — great throughput default ([turbo HF](https://huggingface.co/openai/whisper-large-v3-turbo)) |
| **kingabzpro/whisper-large-v3-urdu** | Urdu fine-tune | 1.55B | Check card (base MIT) **[VERIFY]** | Self-host | Fine-tuned on Common Voice 17 Urdu; benchmark against base v3 ([HF](https://huggingface.co/kingabzpro/whisper-large-v3-urdu)) |
| **SeamlessM4T-v2-large** | Urdu ASR (read speech) + translation | ~2.3B | CC-BY-NC (non-commercial) **[VERIFY — likely blocks commercial use]** | Self-host (eval only) | Best on *read* Urdu speech and ASR→translation; big **license caveat**: Seamless has historically been non-commercial — verify before any production use ([SeamlessM4T](https://arxiv.org/pdf/2308.11596)) |
| **Whisper small + few-shot fine-tune** | Punjabi / Pashto low-resource | 244M | MIT | Edge / self-host | 2025 study shows few-shot fine-tuning of Whisper-small sharply cuts WER on Punjabi/Pashto/Urdu — the practical path for **Punjabi** ([CHIPSAL 2025](https://aclanthology.org/2025.chipsal-1.20/)) |
| **MMS** (Meta) | Very-low-resource ASR | ~1B | CC-BY-NC **[VERIFY]** | Self-host (eval) | Broad language coverage incl. South Asian; license likely non-commercial — verify |

**ASR recommendation:** self-host **Whisper large-v3** (or **large-v3-turbo** for throughput) as default. For **Punjabi** and dialectal robustness, fine-tune Whisper-small/medium on a modest labeled set (few-shot gains are documented). Avoid Seamless/MMS in production until you confirm licensing. Because voice-note volume is bursty and files are short, self-hosting is cost-effective; a hosted ASR API is an acceptable stopgap for launch.

### 3.5 Classic tabular anomaly detection + on-device runtimes

| Tool / Model | Task | ~Size | License | Exec location | Notes |
|---|---|---|---|---|---|
| **PyOD** (Isolation Forest, ECOD, LOF, KNN, AutoEncoder) | Tabular/ledger outlier detection | Library | BSD-2 | Self-host | 60+ detectors, consistent API; **ECOD** parameter-free default, **IsolationForest** workhorse, **AutoEncoder** for v2 ([PyOD](https://pyod.readthedocs.io/); [GitHub](https://github.com/yzhao062/pyod)) |
| **scikit-learn** (IsolationForest, LOF, OneClassSVM) | Same, minimal deps | Library | BSD | Self-host | If you want fewer dependencies than PyOD |
| **Benford's-law digit test** | Amount-manipulation red flag | Trivial | — | Self-host / on-device | Cheap statistical fraud signal on transaction amounts |
| **ONNX Runtime Web** | Run exported models in browser/mobile | Runtime | MIT | **On-device** | Widest model support; export sklearn/torch → ONNX for on-device inference ([Intel in-browser LLM guide](https://www.intel.com/content/www/us/en/developer/articles/technical/web-developers-guide-to-in-browser-llms.html)) |
| **transformers.js** | HF models in browser (NLP/vision) | Runtime | Apache-2.0 | **On-device / browser** | Best for on-device classification, embeddings, small VLM/OCR; WebGPU-accelerated |
| **WebLLM (MLC)** | LLM in browser via WebGPU | Runtime | Apache-2.0 | **On-device** | 30–70 tok/s on laptops; heavier on mid-range Android — use only tiny SLMs there ([Pockit WebGPU guide](https://pockit.tools/blog/run-llms-browser-webgpu-transformers-js-chrome-built-in-ai-guide/)) |
| **TensorFlow.js** | On-device ML (vision/tabular) | Runtime | Apache-2.0 | On-device | Mature; good for lightweight image/tabular models on device |

**On-device reality check for Pakistan mid-range Android:** WebGPU/WebLLM shine on laptops but are marginal on cheap phones (limited GPU, RAM, thermal). On-device on such phones should mean **small, quantized, single-purpose models** (EXIF/pHash logic, a SmolVLM/Moondream quality check, a SmolLM/Qwen-1.5B intent classifier via transformers.js/ONNX) — **not** in-browser 7B LLMs. Anything heavy runs server-side.

---

## 4. Execution-location mapping (client-side / self-hosted / third-party API)

### 4.1 Decision factors

- **Privacy / data-sovereignty:** ledger, transactions, and fraud flags are the product's crown jewels and are commercially sensitive to the construction company. Default these **off third-party APIs**; keep on infrastructure you control. Sending raw financial data to an external LLM is a trust liability for a *trust* product.
- **Latency & connectivity:** Pakistani sites have intermittent/low bandwidth. On-device inference works offline (photo capture, integrity checks, quick classification); anything cloud must tolerate retries and queueing. Favor async (capture now, process on sync).
- **Cost:** per-token API costs are fine for low-volume, high-value tasks (the weekly narrative, occasional hard OCR) but expensive at the scale of every challan/photo/voice note — those should be self-hosted or on-device.
- **Device capability:** mid-range Android → only tiny quantized models on-device. Server (a single GPU box or rented GPU) comfortably runs 7B-class VLM/SLM/Whisper. 70B+ → hosted API or a bigger self-host investment.
- **Quality ceiling:** frontier hosted models still beat small open models on hard reasoning and polished Urdu prose — pay for them where quality is worth it and data is redactable.

### 4.2 Recommended mapping

| Task | Recommended location | Model(s) | Why |
|---|---|---|---|
| Photo integrity (EXIF/geotag/timestamp/duplicate) | **On-device** | EXIF parse + perceptual hash + SmolVLM/Moondream2 | Instant, offline, zero data egress; blocks recycled-photo fraud at source |
| Quick voice-note intent triage | **On-device** (optional) | SmolLM2 / Qwen2.5-1.5B via transformers.js/ONNX | Cheap first-pass classification; full transcription can wait for sync |
| Challan/receipt OCR (bulk) | **Self-hosted** | PaddleOCR + Qwen2-VL Urdu-OCR fine-tune | High volume → API cost prohibitive; keep vendor/rate data private |
| Challan/receipt OCR (messy/hard cases) | **Third-party API** (redacted) | Qwen2.5-VL-72B host / Claude / Gemini vision | Rare, high-difficulty; crop to the document, strip account context |
| **Ledger + material anomaly engine** | **Self-hosted** (never API) | PyOD (IF/ECOD/LOF), Benford, rules, MRS prior | Most sensitive data; must be explainable and sovereign |
| Voice-note transcription (Urdu/Punjabi) | **Self-hosted** | Whisper large-v3 / turbo (+ Urdu fine-tune) | Bursty short files; self-host is cheap and private |
| Weekly report narrative (English) | **Self-hosted** | Qwen2.5-7B-Instruct | Bounded summarization; small model is enough |
| Weekly report narrative (**Urdu**, launch) | **Third-party API** initially → migrate self-host | Claude / GPT / Gemini → Qwen2.5-7B / Gemma 3 | Best Urdu prose today via API on redacted/structured inputs; move in-house as open Urdu quality is validated |
| NL query over project data (owner) | **Self-hosted** | Qwen2.5-7B-Instruct + strict SQL/tool schema | Convenience over private data; keep in-house |
| CV progress estimation (v2/v3) | **Self-hosted** | Qwen2.5-VL-7B / segmentation model | GPU-heavy; research-grade; keep private |

### 4.3 Recommended default architecture (opinionated, reconfigurable)

**A three-tier "route by sensitivity and difficulty" design:**

1. **On-device (mobile app):** capture + cheap guards. EXIF/geo/timestamp validation, perceptual-hash duplicate detection, a small quality/plausibility VLM (SmolVLM/Moondream2), optional quick intent classifier. Everything works offline; results and raw media queue for sync. *Purpose: instant fraud guards + bandwidth resilience.*

2. **Self-hosted server (the default home for almost everything):** one modest GPU (or rented GPU) running **Qwen2.5-VL-7B** (OCR + photo QA), **Whisper large-v3-turbo** (ASR), **Qwen2.5-7B-Instruct** (English narrative + NL query), and the **PyOD/rules/MRS anomaly engine** (CPU). All sensitive financial data and all fraud math stay here. *Purpose: privacy, cost control, sovereignty.*

3. **Third-party API (narrow, deliberate, redacted):** only two justified uses at launch — (a) the **Urdu weekly narrative** where prose quality matters and inputs are structured/redactable, and (b) **hard OCR escalation** for the small fraction of illegible documents. A provider-abstraction layer lets the founder swap Claude/OpenAI/Google or turn the tier off entirely. *Purpose: buy top-tier quality only where it pays and data can be protected.*

**Guardrails baked into the default:**
- A **field-level redaction layer** in front of any API call (strip account balances, client identities, totals — send only what the task needs).
- A **provider/router abstraction** so location per task is config, not code (honoring "opinionated defaults, not rigid").
- A **label/disposition store** capturing every anomaly flag's human outcome from day one — the fuel for v2 learned models.
- **Templated numeric guardrails** on all LLM report generation (numbers come from the DB, the LLM only writes prose around them) to prevent financial hallucination.

**Pragmatic starting point (MVP):** rules + PyOD + MRS prior anomaly engine (self-host CPU) · PaddleOCR + one Urdu-OCR fine-tune (self-host) · Whisper self-host · on-device EXIF/pHash checks · **API for the Urdu narrative only**. Add Qwen2.5-VL-7B, NL query, and on-device VLM as the second wave. Defer CV progress estimation to a dedicated later research spike.

---

## Sources

- Cold-start / anomaly detection: [PyOD docs](https://pyod.readthedocs.io/) · [PyOD GitHub](https://github.com/yzhao062/pyod) · [PyOD models](https://deepwiki.com/yzhao062/pyod/3-models) · [Hybrid AE+IF, Springer 2025](https://link.springer.com/article/10.1007/s10115-025-02580-6) · [AE+IF edge, arXiv 2025](https://arxiv.org/html/2511.18235v1) · [Continual unsupervised auditing, arXiv](https://arxiv.org/pdf/2112.13215) · [Benford robust inference, arXiv 2025](https://arxiv.org/pdf/2507.08650) · [Benford + ML money laundering](https://www.researchgate.net/publication/321024564)
- Pakistani rate schedules: [KP MRS-2024 PDF](https://www.finance.gkp.pk/attachments/b8e8f730589111efa87555dca6816d3d/download) · [KP MRS portal](https://communication_works.kp.gov.pk/page/market_rate_system_mrs/page_type/citizen) · [Punjab MRS Rawalpindi 2024](https://www.slideshare.net/slideshow/government-of-punjab-market-rate-system-rawalpindi-1-2024_2-pdf/269436526) · [Sindh CSR 2022 (NHA)](https://nha.gov.pk/uploads/topics/16569807987105.pdf) · [Daily Ausaf govt rates](https://dailyausaf.com/en/business/new-list-of-government-rates-for-cement-bricks-and-labour-released/)
- OCR: [Qaari Urdu-OCR HF](https://huggingface.co/oddadmix/Qaari-0.1-Urdu-OCR-VL-2B-Instruct) · [dots.ocr HF](https://huggingface.co/rednote-hilab/dots.ocr) · [Qwen2.5-VL report](https://arxiv.org/pdf/2502.13923) · [PP-OCRv5 HF](https://huggingface.co/PaddlePaddle/PP-OCRv5_server_det) · [cxfajar197/urdu-ocr](https://huggingface.co/cxfajar197/urdu-ocr) · [Arabic-English handwritten OCR](https://huggingface.co/sherif1313/Arabic-English-handwritten-OCR-v3) · [Modal OCR comparison](https://modal.com/blog/8-top-open-source-ocr-models-compared) · [Unstract 2026 OCR guide](https://unstract.com/blog/best-opensource-ocr-tools/) · [KDnuggets OCR](https://www.kdnuggets.com/top-7-open-source-ocr-models)
- VLMs / SLMs: [Labellerr Qwen vs Llama](https://www.labellerr.com/blog/qwen-2-5-vl-vs-llama-3-2/) · [Labellerr top VLMs](https://www.labellerr.com/blog/top-open-source-vision-language-models/) · [BentoML VLM guide](https://www.bentoml.com/blog/multimodal-ai-a-guide-to-open-source-vision-language-models) · [Roboflow local VLMs](https://blog.roboflow.com/local-vision-language-models/) · [MLMastery SLMs](https://machinelearningmastery.com/top-7-small-language-models-you-can-run-on-a-laptop/) · [DataCamp SLMs](https://www.datacamp.com/blog/top-small-language-models)
- ASR: [WER We Stand (Urdu ASR), arXiv](https://arxiv.org/html/2409.11252v3) · [CHIPSAL 2025 Pashto/Punjabi/Urdu](https://aclanthology.org/2025.chipsal-1.20/) · [Whisper large-v3-turbo HF](https://huggingface.co/openai/whisper-large-v3-turbo) · [whisper-large-v3-urdu HF](https://huggingface.co/kingabzpro/whisper-large-v3-urdu) · [SeamlessM4T](https://arxiv.org/pdf/2308.11596)
- CV progress monitoring: [Indoor progress, ScienceDirect](https://www.sciencedirect.com/science/article/pii/S2666721524000346) · [CV monitoring review 2025](https://www.sciencedirect.com/science/article/pii/S2666165925002327) · [Mask R-CNN completion %, ASCE](https://ascelibrary.org/doi/10.1061/9780784483961.074)
- On-device runtimes: [Intel in-browser LLM guide](https://www.intel.com/content/www/us/en/developer/articles/technical/web-developers-guide-to-in-browser-llms.html) · [Pockit WebGPU/transformers.js guide](https://pockit.tools/blog/run-llms-browser-webgpu-transformers-js-chrome-built-in-ai-guide/)

## Items flagged as not fully verified
- **MES Analysis of Rates**: known to exist as the military/cantonment rate schedule but no current public document retrieved this pass — source the latest edition directly.
- **Licenses** for several fine-tunes (Qaari, Arabic-English OCR, whisper-large-v3-urdu, Surya commercial terms, Llama 3.2 / Gemma exact terms) inherit or vary — re-check each model card before commercial commitment.
- **Urdu long-form generation quality** of small open models (Qwen2.5-7B, Gemma 3): no clean public Urdu benchmark found — build an internal Urdu eval set before relying on self-hosted Urdu narrative.
- **SeamlessM4T / MMS** are likely non-commercial (CC-BY-NC) — verify licensing before any production use.
