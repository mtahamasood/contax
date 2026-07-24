---
title: "Contax PRD — Adversarial Anti-Fraud / Security-of-Design Review"
status: draft
created: 2026-07-25
reviewer: "Adversarial anti-fraud reviewer (red-team of the FR/NFR set)"
scope: "prd.md + addendum.md, prd-contax-2026-07-25"
review_question: "Can a motivated, colluding on-site staffer defeat the controls as specified?"
---

# Contax — Adversarial Anti-Fraud Review

## 0. Method & framing

Contax's stated principle is that it is **"adversarial toward its own operators by construction."** This review takes that literally and asks the sharp question: given the FRs/NFRs exactly as written, **where can a motivated, colluding on-site staffer still steal, and have the system show normal?**

Findings are stated as concrete exploit paths (actor → action → why the record looks clean), each with a severity and the FR/NFR that *should* close it but does not. Severity is anti-fraud severity: **critical** = defeats a make-or-break control or the core trust claim; **high** = defeats a named control with modest effort; **medium** = real but narrower or requires more collusion; **low** = hardening.

**Overall verdict.** The control *architecture* is unusually strong for this market — append-only trails, separation of powers, source-redundancy scoring, an internal-first two-tailed engine. But the FR set has **four structural gaps that are individually sufficient to defeat the whole promise**, and every one of them lives at the *boundary between the physical world and the first captured bit*. Contax is adversarial toward operators who *lie in the record*; it is largely blind to operators who *never create the record* (omission), who *control both "independent" sources* (collusion), or who *supply the instrument that measures them* (Source B). The system's own §9 invariant — "redundancy, not any single trusted entry, is the fraud control" — is only as true as the *independence* of the redundant sources, and no FR establishes or verifies that independence. That is the thread running through the critical findings below.

---

## 1. The three-capture layer (FR-6..FR-12) — "no receipt, no acceptance" (FR-7)

### F-1 — [CRITICAL] There is no consumption / material-OUT capture; stock-on-hand theft is invisible by design
**Exploit.** Kamran (storekeeper) + Bilal (site engineer) collude. A genuine 100-bag cement challan is captured cleanly (FR-6): OCR reads 100, stock posts 100 (FR-18), the AP entry is honest (FR-19), Source B will later agree (FR-25) — **maximum credibility**. They physically divert 30 bags. The theft happens at *consumption/storage*, which **is not one of the three atomic captures**. The Capture Layer records material *in* (challan) but never material *out*; consumption is only *tracked* against BOQ (FR-15), i.e. inferred from line-items the site engineer himself marks done (FR-14). Book-stock says 30 remain; physically 0 remain. Nothing in the FRs performs a **physical stock count reconciled against book stock**, so the shrinkage is never surfaced. FR-36's rule "Milestone % up with zero consumption" catches only the *zero*-consumption case; here consumption is reported normally.
**Why it matters.** §9 calls the three-capture spine "closed by discipline." The discipline has closed off the single capture that would catch the market's most-cited leak (pilferage). Addendum H concedes "labour & consumption fraud (the paperless leak points)" are caught only "against industry-average productivity / BOQ" — i.e. against a self-reported denominator, not a measured one.
**Should be closed by:** a new capture event (material-OUT / periodic physical stock count) in the extensible registry (FR-12), and a book-stock-vs-physical-stock reconciliation rule in FR-36/FR-15 that flags unexplained shrinkage. Until then SM-1's "≥90% of deliveries captured" measures only the *in*-side and says nothing about theft.

### F-2 — [HIGH] Geotag and timestamp are device-supplied; there is no independent location/time anchor
**Exploit.** FR-9/FR-11 geotag and timestamp "from device geolocation" and "device time," stored "immutably." On a mid-range Pakistani Android, mock-location apps and a manually-set clock let the operator inject any coordinate and any time. FR-51 validates the geotag as "consistent with the Project location" — but the project location *is* the site, so a spoof set to the exact site coordinates **passes**. FR-10 explicitly "preserves original capture timestamp/geotag, not the sync time," so a photo shot elsewhere/earlier and synced later keeps its spoofed metadata. Immutability then locks the lie in as gospel.
**Why it matters.** "Provenance everywhere" (NFR-11) makes the record tamper-evident *after* write, but the write ingests untrusted device inputs. A site engineer can stage a progress photo off-site, or backdate a capture, and the "immutable provenance" launders it.
**Should be closed by:** a server-side received-timestamp stamped at sync (in addition to device time, with the delta flagged); a network/cell-derived location cross-check where signal exists; and treating mock-location / clock-skew as a first-class Anomaly Flag in FR-51/FR-36. As written, FR-9/FR-11/FR-51 trust the attacker's phone.

### F-3 — [HIGH] "No receipt, no acceptance" (FR-7) proves paper existed, not that material arrived — and the challan itself is forgeable
**Exploit.** FR-7 closes the *no-paper* hole (a challan-less delivery is flagged, never clean). It does **not** close the *fabricated-paper* hole. Nothing in FR-6/FR-7 requires the corroborating truck/material photo (UJ-1 shows two photos but FR-6 mandates only the challan image), nor requires the photo's content to match the challan quantity. A colluding storekeeper can OCR a **forged or borrowed challan** (fresh number, plausible rate) with no matching delivery, or photograph one real challan as two events. FR-36 catches *duplicate* challan numbers and *reused* photo hashes — a novel forgery evades both.
**Should be closed by:** making the geotagged material photo mandatory *and* cross-checked against declared quantity (VLM count sanity) in FR-6; and per-supplier challan-number sequence/continuity checks in FR-36 (a fabricated number breaks the supplier's own numbering run). Today the paper is trusted because "quantities come off paper, not a typist" — but a forged paper is a typist with extra steps.

---

## 2. Separation of powers / segregation of duties (FR-19, FR-29, FR-30, NFR-9)

### F-4 — [HIGH] The controls check *identity ≠ identity*, not *human ≠ human*; a small firm defeats SoD with two logins
**Exploit.** FR-19/FR-29/FR-30/NFR-9 block *self-*approval and raise-then-ratify by comparing the acting **identity**. In a small residential builder there may be one back-office person. The Owner simply issues that person **two accounts** — a Storekeeper login and an Accountant login (FR-1 even normalizes multi-method identities). Kamran captures on the Storekeeper login and "Ayesha" signs off on the Accountant login — same human, two identities, control satisfied. The system has no notion that the two identities are one person: no shared-device / shared-IP / same-phone detection, no minimum-distinct-operators check, no attestation that roles map to distinct humans.
**Why it matters.** NFR-9 promises separation "enforced by access control, not policy alone." It is enforced only at the identity layer; the identity↔human binding is pure policy. The "cross-rank collusion required" defense (FR-30) collapses to "one person with two passwords."
**Should be closed by:** device/session fingerprinting that flags two "segregated" roles operating from one device or in impossible-travel patterns; an onboarding attestation of distinct humans per segregated role; and a warning when a Project runs below the operator count SoD needs.

### F-5 — [CRITICAL, by design boundary] A complicit Owner nullifies every client-facing control; "proof of innocence" becomes proof-laundering
**Exploit.** The Owner holds Tier-C authority, sets *every* configurable threshold (FR-31, north-star §9), owns anomaly-disclosure discretion (FR-40/FR-53, "nothing disclosed by default"), sets the cash-heavy threshold (FR-22), and imports the history that seeds "normal" (FR-38). If the Owner is the fraudster padding the client's bill, Contax has **no control that protects the client from the Owner** — and the value hierarchy (§0) explicitly resolves company-vs-client conflicts in the company's favor. The Owner reconfigures all COs to Tier A negative-confirmation, discloses no flags, and Contax still emits a branded, numbered, tamper-evident "Proof-of-Innocence Report" over padded numbers.
**Why it matters.** This is arguably *by design* (Contax is adversarial toward operators, not the Owner). But it is a **direct contradiction of the client-facing marketing claim** ("you see every rupee and every brick"). The client's only Owner-independent anchors are their own payment records and the raw photos. The PRD should state this boundary explicitly rather than let "proof of innocence" imply a guarantee the architecture does not provide.
**Should be closed by:** an honest scoping statement in §0/§13 that Contax defends the client against *operators*, not against a complicit *Owner*; and (if the client-trust claim is to be real) a small set of Owner-uncircumventable invariants — e.g. the negative-confirmation cumulative cap (F-8) and the tamper-evident series (FR-45) — that even the Owner cannot reconfigure away.

### F-6 — [MEDIUM] No collusion-ring / pairing detection
**Exploit.** FR-34 tracks CO velocity *per engineer*. Nothing tracks the *pair*: if the same capturer→approver dyad (e.g. Storekeeper-A always signed off by Accountant-B, or Site-Engineer-X's COs always ratified by Manager-Y) recurs on every suspect transaction, that is the signature of a ring, and it is invisible. Segregation is treated as a per-transaction gate, never as a *quality metric* over time.
**Should be closed by:** a rollup signal (FR-55) on recurring capture/approve and raise/ratify pairings, and periodic-rotation nudges.

---

## 3. Change-order governance (FR-27..FR-34)

### F-7 — [CRITICAL] Negative Confirmation treats a *silent/absent* client as a *consenting* client — with no proof of receipt
**Exploit.** Tier A defaults to Negative Confirmation: "silence over a stated window = accepted" (FR-31, FR-46). The flagship persona (Naeem, UJ-4) is "genuinely unable to visit," five time zones away, and may go weeks without opening WhatsApp — or change numbers, or simply trust the builder and never read. Two-way WhatsApp is out of scope (§5); delivery is a **fire-and-forget PDF** (FR-43) with **no read/delivery receipt requirement anywhere in the FRs**. So the system **cannot distinguish "client read it and accepted" from "client never saw it."** A colluding site engineer routes every fraudulent extra through Tier A; the client's inattention auto-ratifies it; the append-only trail then reads "ratified by negative confirmation" — fraud laundered into a legitimate-looking approval. FR-46's "attach the client's own evidence where possible" is definitionally empty here: silence produces no client evidence.
**Why it matters.** This is the load-bearing weakness in the *client-facing* trust claim. Sustained silence is being read as sustained consent, when sustained silence is more likely a **dead channel** — the exact condition a fraudster wants.
**Should be closed by:** (a) require a WhatsApp *delivered/read* signal before a silence-window can start counting; (b) treat N consecutive silent windows as a **channel-health alarm** that forces a mode upgrade (Positive Evidence / Explicit Approval) rather than continued auto-accept; (c) the cumulative cap in F-8. None of these exist in FR-31/FR-46 today.

### F-8 — [HIGH] Threshold-splitting is *named* but only *clustering* is caught; cumulative exposure under the weakest mode is uncapped
**Exploit.** FR-34 flags "just-under-threshold clustering." But the tier system caps the *rigor per CO*, not the *cumulative value* passing under the weakest mode. Ten Tier-A COs at 0.9% of contract value each = **9% of the contract auto-ratified by client silence**, each one individually legitimate and not necessarily "clustered" (spread across weeks, line-items, or engineers to defeat the clustering heuristic). The flag is also a soft signal whose Flag Disposition is a *human* call (FR-41) that a complicit Manager dispositions as "false alarm."
**Should be closed by:** a hard **aggregate cap** — cumulative negative-confirmation CO value per Project per window escalates to a stronger ratification mode regardless of individual sizes — added to FR-31/FR-34. "Just-under clustering" detection is necessary but not sufficient.

### F-9 — [MEDIUM] "Executed before approval" flag doesn't recover the consumed material
**Exploit.** FR-33 makes pre-approval work "logged, flagged, not billable until ratified." But the *material* for that work is already consumed. Do the work, let the CO quietly die (never ratified) — the material is gone and now appears as unexplained base-contract consumption or stock shrinkage (compounding F-1), while the client is never billed (so no client-side red flag). The flag governs *billing*, not *material recovery*.
**Should be closed by:** tying an unratified-but-executed CO's consumption to the shrinkage reconciliation of F-1.

---

## 4. Supplier reconciliation (FR-24..FR-26) as the SM-1 denominator (§8, RR-1)

RR-1 promoted Source B from "credibility bonus" to **the measuring instrument of the make-or-break metric**. That promotion moved the entire success criterion onto an instrument with three fatal, in-scope failure modes.

### F-10 — [CRITICAL] The audited party can supply its own audit baseline; a colluding or fraudster-collected Source B *raises* credibility
**Exploit.** Two variants, both open in the FRs:
- **Supplier colludes.** The supplier issues a statement covering only the captured challans (omitting diverted deliveries) or a wholly fabricated one. Source A = Source B → FR-25 raises the **Data-Credibility Score**. The fraud is now a *high-credibility* fact. Redundancy only works if the two sources are independent; two colluding sources agreeing is indistinguishable from two honest sources agreeing, and no FR verifies independence.
- **Fraudster collects Source B.** Open Q6 / R-7 leave *who obtains and scans the supplier statement* **undefined**. If the storekeeper or site engineer (the very party being policed) collects and OCRs Source B, they forge it to match Source A. The instrument is gathered by its target.
**Why it matters.** RR-1 made this the denominator of SM-1 (make-or-break). Addendum H rests it on the bare *assumption* "supplier–site collusion is low-probability." The entire primary success metric now depends on an unverified assumption, and the collection workflow that would enforce independence does not exist.
**Should be closed by:** define in FR-24 an **independent collection channel** (statement obtained directly from the supplier by head-office/Accountant, never by the site operator being measured; ideally supplier emails/uploads it to a channel operators can't author); and add a cross-check that Source B's numbering/formatting is continuous and supplier-originated, not operator-reconstructed. Until Open Q6 is answered adversarially, SM-1 is measured against an instrument the fraudster can hold.

### F-11 — [CRITICAL] "Source B unavailable is graceful" (FR-26) contradicts "Source B is the make-or-break denominator" (RR-1) — and the fraudster controls the availability
**Exploit.** FR-26 says a missing Source B is benign ("no redundancy bonus," "not a blocker"). RR-1 says Source B *is* the SM-1 denominator. These cannot both hold: where Source B is absent, **SM-1 is unmeasurable** — there is no independent count, so the ≥90% capture-completeness target is measured against nothing. The fraudster's optimal move is therefore to **ensure Source B is never collected** (lean on collection friction R-7, "supplier won't give it," lose the scan). SM-C1 tries to counter by watching the low-credibility share, but if Source B is absent *everywhere* there is no miss-rate signal at all — the counter-metric also goes blind.
**Why it matters.** The make-or-break metric has an easy opt-out, and the party who benefits from the opt-out controls it.
**Should be closed by:** make Source-B coverage itself a **first-class monitored metric** (Source-B collection rate per supplier), treat a *falling or zero* coverage as a fraud-risk alarm (not a neutral "unavailable"), and require a minimum Source-B coverage before SM-1 is reported as valid. FR-26 must stop framing absence as neutral now that RR-1 made presence load-bearing.

### F-12 — [MEDIUM/HIGH] Supplier khata statements are amount/balance-based; per-delivery *quantity* diversion may be unreconcilable
**Exploit.** FR-24 promises reconciliation of "missing/extra deliveries, quantity/amount mismatches." But a supplier *account statement* (customer khata, per Glossary/Addendum I) typically shows dated **amounts and a running balance**, not per-delivery bag/tonne counts. A fraudster who diverts *quantity* while the rupee total still ties (e.g. renegotiated rate, or the statement aggregates) leaves Source A and Source B in amount-agreement while the physical quantity is short. The instrument may only reconcile money, not material.
**Should be closed by:** FR-24 must specify the minimum line-item granularity Source B must carry to be a valid quantity instrument, and mark amount-only statements as *not* satisfying the SM-1 quantity denominator.

---

## 5. Anomaly engine (FR-35..FR-41), append-only (NFR-8), credibility scoring

### F-13 — [CRITICAL] The Data-Credibility Score measures *agreement*, not *independence* or *truth* — so controlling both sources yields maximally-credible fraud
**Exploit.** FR-35 raises the score when "two or more independent Captures agree." "Independent" is asserted, never enforced. A single colluding operator (or an operator + colluding supplier, F-10) who controls both a photo and a ledger entry, or a challan and a supplier statement, produces two "agreeing" captures and the fact scores *high*. The fraud engine's own headline defense — §9's "redundancy, not any single trusted entry, is the fraud control" — inverts into a **fraud amplifier**: the most-collusive fabrications carry the highest credibility.
**Should be closed by:** the score must weight **source-independence** — captures from the same identity/device/session, or from a supplier flagged for prior divergence, must *not* count as independent corroboration. FR-35 needs an independence predicate, not just an agreement predicate.

### F-14 — [MEDIUM/HIGH] Rate-Book-hugging padding evades both the over-threshold rule and the too-clean detector
**Exploit.** FR-36 flags "rate exceeds Rate Book by > X%." The Rate Book is *public* (Addendum K) and its composite rates *include overheads/profit* — so there is legitimate headroom. A fraudster prices padding to sit **just under** the Rate Book +X% margin. FR-40's two-tailed engine is meant to catch "rate exactly at benchmark every time" as too-clean — but small deliberate *jitter* around the benchmark defeats the over-threshold rule and the too-clean detector simultaneously. The attacker knows the exact number to hide beneath because the anchor is published.
**Should be closed by:** make the deviation margin (FR-39) and the too-clean band non-public/per-company-jittered, and add cross-signal correlation (a rate near-ceiling *and* a vendor-concentration *and* a cash-heavy pattern together) rather than single-rule thresholds.

### F-15 — [MEDIUM/HIGH] Consistent-from-day-one fraud is invisible to outlier detection, and history import can poison the baseline
**Exploit.** FR-37's Layer B catches *outliers* — inconsistency. If every cement delivery is padded 8% from day one, the padding *is* the intra-project norm and nothing deviates. Cross-line-item comparison (FR-37) doesn't help when the padding is uniform. The mitigations lean on FR-38 company-history import to seed "normal" — but that history is imported by the (possibly complicit) Owner and may itself embed the same padding, so importing it **normalizes the fraud as the baseline**. Layer C (Rate Book) is the only external anchor and is evadable per F-14.
**Should be closed by:** treat the *level* (not just the variance) against Layer C as a standing check even when intra-project variance is zero; and quarantine/flag imported history that itself diverges from Rate Book priors rather than trusting it as ground-truth "normal."

### F-16 — [MEDIUM] Append-only stops cover-up but not omission, and "correction versions" can restate the client-facing number
**Exploit.** NFR-8 immutability defends against a caught fraudster *deleting* evidence — genuinely valuable. It does nothing against a fraudster who *never creates* the record (omission, F-1/F-11). Separately: FR-8/FR-23 allow "corrections create a new versioned entry." The FRs do **not** say who may author a correction version or whether a correction needs its own SoD/second-party sign-off, nor whether the *computed* client-facing figure reads the latest version. If a raiser can push a self-authored "correction" that the report then reflects, the append-only trail preserves the original while the *shown* number is quietly restated — laundering through the correction channel.
**Should be closed by:** apply FR-19-style separation to *corrections* (a correction is a new commit requiring a different approver), and state explicitly that anomaly checks re-run on the correction, comparing it to the superseded version.

### F-17 — [HIGH] Milestone completion is still self-reported at line-item granularity; the only independent check (CV) is shadow-mode and may not ship
**Exploit.** FR-14 makes % "computed, never typed" — but computed from "BOQ Line Items **marked done**," and the site engineer marks them. "Computed" means *aggregated*, not *verified*. A site engineer marks items done to inflate the client-facing % (drawing down milestone payments) ahead of real work. The independent cross-check — CV progress (FR-17) — is explicitly **Shadow-Mode, Management-only, never client-facing**, and per RR-2 is a re-filed research spike that is *not build-blocking* and may not ship at all. FR-36's "milestone up with zero consumption" catches only the zero case; if material was received and marked consumed (F-1), the books look consistent while the physical work isn't done.
**Should be closed by:** require photo/CV evidence *linked to each line-item marked done* as a completion precondition in FR-14, and surface the CV-vs-reported-progress discrepancy as a hard Management flag (FR-17) even in shadow mode.

### F-18 — [MEDIUM] Photo integrity is defeatable and likely not cross-project
**Exploit.** FR-51 flags perceptual-hash duplicates "across milestones/weeks" — scope reads as *within a project*. A site engineer reuses a photo from a *different* project (or a stock/internet photo of the right stage); if the pHash store isn't cross-project/cross-company, it passes. A minor crop/recolor/reshoot defeats pHash. A genuinely novel fake (photograph a different real site at the correct stage) has no duplicate and, with a spoofed geotag (F-2), passes every integrity check.
**Should be closed by:** cross-project/cross-company pHash scope, near-duplicate embedding robustness, and coupling to the F-2 location anchor so an off-site novel photo is caught by geolocation rather than by hashing.

---

## 6. Redaction / data-residency (NFR-4, NFR-5, NFR-12)

### F-19 — [HIGH] The redaction rule and the report's purpose are in tension: the narrative API must see the numbers the report exists to show
**Exploit.** The Urdu weekly narrative runs on a **third-party API** at launch (Addendum A/B). NFR-5 strips "account balances, client identities, and totals." But the report's entire job is to show the client this week's spend and progress — those figures must survive to appear in the PDF, which means they are in the narrative-generation path. NFR-6's "templated numeric guardrail" helps *only if* numbers are inserted into fixed slots *after* generation; if the LLM is handed the figures to write naturally around (the more fluent path the Addendum favors for Urdu prose quality), then client spend/progress — and whatever else sits in the prompt/context window — leaves the controlled environment. The boundary between "redacted crown jewels" and "the report content the API must see" is **unspecified**, and it is exactly the client's financial picture.
**Should be closed by:** FR-42/NFR-5/NFR-6 must mandate the *strict-slot* templating variant for any API-tier narrative (LLM receives only category labels and pre-formatted opaque tokens, never raw financials), and forbid the "hand the model the numbers" variant while the API tier is in use. As written, the fluent path leaks.

### F-20 — [MEDIUM/HIGH] Hard-OCR escalation sends supplier rates/identity to a third-party API by construction
**Exploit.** NFR-4 permits "hard-OCR escalation" of illegible challans to a commercial API ("crop to the document, strip account context," Addendum A/B). But a challan *is* supplier name + unit rates + quantities — the precise data NFR-17 says must "never leak." You cannot redact an image you are sending *to be read*; the sensitive content is the payload. So for the subset of escalated challans, the "supplier rates never leave" posture is structurally violated, and NFR-4 *authorizes* it.
**Should be closed by:** keep hard-OCR on a self-hosted larger model (Addendum already lists Qwen2.5-VL-72B self-host as an option) rather than a third-party API; if an API is unavoidable, mask rate columns before send and OCR them separately on-device/self-host. NFR-4/NFR-17 currently contradict each other on this path.

### F-21 — [MEDIUM] "Config, not code" makes crown-jewel residency a default, not a wall
**Exploit.** NFR-4 makes execution location "config, not code," and NFR-5 puts ledger/fraud data "default OFF" third-party APIs. §9's north-star says *every default is reconfigurable*. So a misconfiguration, a router bug, or a well-meaning override can route ledger/anomaly data to the API tier — "default off" is not "impossible off." For a trust product whose crown jewels are financial and fraud data (NFR-12), residency should be an architectural hard-wall, not a toggle.
**Should be closed by:** NFR-12 should specify that ledger/anomaly/fraud-math egress to any third-party endpoint is **not a reconfigurable cell** (compile-time/deploy-time deny), mirroring the FR-31 Tier-C hard-floor pattern the PRD already accepts for governance.

---

## 7. Cross-cutting observations

- **The recurring root cause** across F-1, F-2, F-3, F-10, F-13 is the **physical-to-digital boundary**: Contax is rigorous about what happens to a bit *after* capture (append-only, provenance, redundancy scoring) and credulous about the bit's *birth* (device-supplied metadata, self-collectable second sources, unverified independence). An operator adversary attacks the birth of the bit, not its life.
- **Independence is assumed everywhere it is load-bearing** — Data-Credibility (FR-35), Source A/B (FR-24/25), SM-1's denominator (RR-1). A single "independence predicate" (same human? same device? same collusion-flagged supplier? operator-collected?) applied across all three would close a large share of the criticals.
- **Human disposition is the soft underbelly of every hard flag.** FR-41 requires *a* disposition, not a *correct* one; a complicit Manager/Owner dispositions real flags as "false alarm" (F-6, F-8, F-5). The label store (FR-41/NFR-14) then trains v2 on *laundered* labels — a slow-motion poisoning of the future model.
- **Two documented internal contradictions** the PRD should resolve before build: FR-26 ("Source B absence is neutral") vs RR-1 ("Source B is the make-or-break instrument") — see F-11; and NFR-4/NFR-17 (hard-OCR API escalation vs "supplier rates never leave") — see F-20.

## 8. Severity roll-up

| ID | Finding (one line) | Severity | Owning FR/NFR to fix |
|----|--------------------|----------|----------------------|
| F-1 | No consumption/material-OUT capture; stock-on-hand theft invisible | Critical | FR-12/FR-15/FR-36 (new capture) |
| F-5 | Complicit Owner nullifies all client-facing controls | Critical (boundary) | §0/§13 scope; Owner-uncircumventable invariants |
| F-7 | Negative Confirmation reads silent/absent client as consenting; no receipt proof | Critical | FR-31/FR-46/FR-43 |
| F-10 | Audited party can supply/forge the Source B baseline; collusion raises credibility | Critical | FR-24 (independent collection), FR-25 |
| F-11 | "Source B unavailable is graceful" vs "Source B is the SM-1 denominator" — fraudster controls availability | Critical | FR-26 vs RR-1/SM-1 |
| F-13 | Credibility Score rewards agreement, not independence → maximally-credible fraud | Critical | FR-35 (independence predicate) |
| F-2 | Device-supplied geotag/timestamp, no independent anchor; spoof/backdate passes | High | FR-9/FR-11/FR-51 |
| F-3 | "No receipt, no acceptance" doesn't stop forged/borrowed challans | High | FR-6/FR-7/FR-36 |
| F-4 | SoD checks identity≠identity, not human≠human; two logins defeat it | High | FR-19/FR-29/FR-30/NFR-9 |
| F-8 | Cumulative sub-threshold CO value under negative confirmation is uncapped | High | FR-31/FR-34 |
| F-17 | Milestone % still self-reported per line-item; CV check shadow-only/optional | High | FR-14/FR-17 |
| F-19 | Narrative API must see the report's financials; fluent path leaks | High | FR-42/NFR-5/NFR-6 |
| F-6 | No collusion-ring / pairing detection | Medium | FR-34/FR-55 |
| F-9 | "Executed before approval" flag doesn't recover consumed material | Medium | FR-33 (+F-1) |
| F-12 | Amount-based khata can't reconcile quantity diversion | Medium/High | FR-24 |
| F-14 | Rate-Book-hugging padding evades over-threshold + too-clean | Medium/High | FR-36/FR-39/FR-40 |
| F-15 | Consistent day-one fraud invisible to outliers; history import poisons baseline | Medium/High | FR-37/FR-38 |
| F-16 | Append-only stops cover-up not omission; correction versions can restate shown number | Medium | FR-8/FR-23/FR-19 |
| F-18 | Photo integrity likely not cross-project; novel fake + spoofed geotag passes | Medium | FR-51 |
| F-20 | Hard-OCR API escalation sends supplier rates/identity out | Medium/High | NFR-4/NFR-17 |
| F-21 | Crown-jewel residency is a reconfigurable default, not a hard wall | Medium | NFR-12 |

*The four criticals that most directly break the promise: **F-1** (theft moved to the uncaptured consumption side), **F-10/F-11** (the make-or-break instrument is corruptible and opt-out-able by the party it measures), **F-7** (silence-equals-yes against an unverified channel), and **F-13** (redundancy scoring rewards collusion). Close the independence predicate (F-13/F-10) and the consumption capture (F-1) first; they each collapse multiple downstream findings.*
