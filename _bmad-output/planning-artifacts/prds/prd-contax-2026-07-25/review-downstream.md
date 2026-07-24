---
title: "Contax PRD — Downstream Implementability Review"
reviewer: "Claude (downstream-implementability lens)"
date: 2026-07-25
scope: "prd.md + addendum.md — UX/architecture/epics source-extractability only (NOT strategy)"
verdict: "PROCEED WITH FIXES — structurally strong and highly source-extractable, but carries 3 broken cross-refs, one offline-OCR contradiction on the linchpin capture, and a cluster of silently-absent anomaly thresholds that will each force a clarification round-trip."
---

# Downstream Implementability Review — Contax PRD

**What this review is:** an assessment of whether UX, architecture, and epics/story-creation can source-extract from this PRD and build without guessing. Strategy/product direction is assumed fixed and is **not** reviewed. Severity = impact on a downstream team's ability to proceed without a clarification round-trip.

- **Critical** — blocks a core-path build decision; guaranteed round-trip.
- **Major** — forces a clarification round-trip on a real feature or a traceability defect that misroutes a reader.
- **Minor** — cosmetic / low-friction; a careful reader recovers unaided.

**Bottom line:** This is an unusually disciplined PRD — Glossary-anchored, globally-numbered FRs, every FR carrying a "Consequences (testable)" block, and an explicit Open-Questions register that catches most (not all) of its own gaps. The document's *structure* holds. The defects below are localized: a renumbering that left three dangling cross-references, one genuine offline-vs-server OCR contradiction on the make-or-break capture, and a small set of numeric thresholds that are "configurable" with no default and are **not** captured in Open Questions.

---

## 1. FR Testability (all 57)

Every FR carries a "Consequences (testable)" block, and the large majority are genuinely testable as written. The FRs below are **weak** — their consequences hinge on a bound, default, or word that is not pinned. I split them into *acknowledged-open* (the PRD flags the gap in §14, so no surprise — lower risk) and *silently-absent* (no default, no Open Question — these cause round-trips).

### 1a. Weak but ACKNOWLEDGED as open (lower risk — flagged in §14)
These are testable *once the referenced Open Question resolves*; the PRD does not pretend otherwise.

| FR | Weak element | Flagged where |
|----|--------------|---------------|
| FR-17 | "until it demonstrably earns trust" (Shadow-Mode exit) | Open Q12 |
| FR-22 | "cash-heavy when untraceable cash exceeds a **configurable threshold**" — no default | Open Q3 (assumption inline) |
| FR-42 | "Both English and Urdu render **acceptably**" | Open Q8 / NFR-7 (Urdu eval set) |
| FR-46 | "silence over a **stated window**" — dispute-window duration not given | Open Q4 |
| FR-47 | "on a Company-configurable **day**" — default day not given | Open Q4 |
| FR-52 | "**low-confidence** transcripts flagged" — threshold not given | Open Q9 |
| NFR-13 | "retention horizon ≥ project duration + **a defined tail**" — horizon TBD | Open Q10 |

**Note on FR-26 ("graceful state"):** the word "graceful" appears in the title, but the consequences are concrete and testable (does not block capture; records keep Source-A-only score; absence is visible not silently-verified). **Not** a weak FR despite the flagged word.

### 1b. Weak and SILENTLY ABSENT (higher risk — no default, no Open Question)
These will each force a round-trip because the north-star principle (§9: "every default ships as a reconfigurable, pre-filled default") is violated and nothing in §14 catches it.

- **FR-34** — "A CO rate exceeding the Rate Book prior **by a configurable margin**" — no default margin, not in Open Questions.
- **FR-36** — Layer-A day-one rules ship "payment with no Challan/photo **within N days**" and "purchase rate exceeds Rate Book **by > X%**". **N and X are undefined and unflagged.** This is the *day-one, zero-data* rule engine — the one anomaly layer that must ship first — and its rule thresholds have no recommended defaults. **Major.**
- **FR-39** — "comparable against the provincial prior **with a configurable deviation margin**" — no default; same undefined X% as FR-36, not reconciled with it.
- **FR-25 / FR-35** — the **Data-Credibility Score** "measurably increases" / "raises" on agreement, but there is **no scale, no scoring model, no bounds, and no Open Question**. A story writer implementing the score has nothing to build to. The score is load-bearing (it is a "first-class, queryable attribute" consumed by report, rollup, and anomaly features, and it is the mechanism behind SM-C1). **Major — silent gap.**
- **FR-31** — Tier thresholds "**≤1% or Rs 50k**" / "≤5% or Rs 500k": the percentage base is unstated (1% of the project contract value? BOQ total? the CO's own amount?), and the "or" (whichever-is-lower vs whichever-is-greater) is ambiguous. Numbers are present but the rule is not fully determined. **Minor–Major.**
- **FR-51** — near-duplicate photo detection "matching perceptual hash" with no distance threshold/default (mitigated: explicitly "best-effort", authoritative check server-side). **Minor.**

---

## 2. Traceability

### 2a. FR / NFR ID integrity — PASS
- **FR-1 … FR-57 are contiguous and unique.** No gaps, no duplicates. The "57 FRs" claim holds.
- **NFR-1 … NFR-20 are contiguous and unique.**
- **UJ-1 … UJ-6** all defined (§2.3) and every "Realizes UJ-N" reference resolves to a defined journey. Each of the six is referenced by at least one feature/FR.
- **RR-1 … RR-4** defined in §15.2; references from FR-4 note, §7.1, SM-1, and D-1..D-3 all resolve.
- **Open Q1 … Q15** defined in §14; inline references (FR-22→Q3, FR-31→Q5, FR-39→Q7, FR-42→Q8, FR-47→Q4, FR-52→Q9, §7.1→Q12, NFR-13→Q10, RR-2→Q12/Q15) and Addendum references (Open Q5/Q6/Q7) all resolve.
- **SM→FR:** SM-1→FR-6..12 + FR-24..26; SM-2→FR-42..50; SM-3→FR-34..41 + FR-54..55; SM-C1/C2/C3 counterbalance SM-1/3/2. All targets exist.

### 2b. BROKEN FR cross-references — MAJOR (3 found)
There is a clear pattern of **stale cross-references left behind by a renumbering** — each points at a real FR that is the *wrong* FR. A story writer or epic planner following the reference lands on an unrelated requirement.

1. **FR-8 → "(see FR-24)"** is wrong. FR-8 (Money Movement) says corrections create a new versioned entry "never an in-place edit or delete **(see FR-24)**." FR-24 is *Source B reconciliation*. The append-only-ledger-with-versioned-corrections requirement is **FR-23**. Should read **(see FR-23)**.
2. **FR-15 → "(see FR-42)"** is wrong. FR-15 (Material-vs-BOQ) says "Consumption exceeding BOQ-remaining raises an Anomaly Flag **(see FR-42)**." FR-42 is the *Weekly Report PDF*. The rule "Challan quantity > BOQ-remaining raises a flag" lives in the Layer-A rule engine, **FR-36** (also relates to FR-35/FR-40). Should point at **FR-36**, not the report.
3. **FR-40 → "(see FR-52)"** is wrong. FR-40 (two-tailed, internal-first) says "disclosure requires a Management action **(see FR-52)**." FR-52 is *voice-note ASR*. Management disclosure control is **FR-53** (and FR-53 correctly back-references FR-40). Should read **(see FR-53)**.

**Impact:** each misroutes a downstream reader to the wrong requirement; #2 in particular sends an epic planner tracing the anomaly path into the reporting feature. Cheap to fix, but the pattern warrants a full cross-reference sweep before handoff (there may be others in prose I did not exhaustively diff).

---

## 3. Glossary Discipline

Sampled nouns and checked for synonym drift: **Challan, Concern-Tagged Contact, Operator, Data-Credibility Score, Site Photo, Rate Book, Separation of Powers vs Segregation of Duties, Computed % Completion.**

- **Mostly disciplined.** Challan, Concern-Tagged Contact, Operator, Data-Credibility Score, Ledger Entry, Anomaly Flag, Source A/Source B are used verbatim throughout.
- **DRIFT — "Separation of Powers" vs "Segregation of Duties" (Minor–Major).** The Glossary defines two *distinct* terms and correctly keeps them apart in FR-29 (Separation of Powers: raise≠ratify≠bill) and FR-30 (Segregation of Duties: log-purchase≠propose, and *on money* capture≠self-approve). But the **money self-approval rule** — which the Glossary explicitly files under **Segregation of Duties** — is repeatedly called lowercase **"separation of duties"** in FR-19's title/description, §4.4's description, and §7.1. NFR-9 then lumps FR-19 under "separation of powers and segregation of duties" without saying which. Because these two terms are a load-bearing product invariant (§9), the blur is worth tightening: the AP self-approval block should be named **Segregation of Duties** consistently.
- **Minor — "Computed % Completion":** rendered as "Computed % Completion", "% Completion", and "progress-%" in different places (e.g. FR-14 body: "Milestone and Project % Completion"). Recoverable, but the canonical term should be used verbatim.
- **Acceptable a.k.a.:** "Weekly Report / Proof-of-Innocence Report / Weekly Proof-of-Innocence Report" is a Glossary-sanctioned a.k.a., not drift. "Rate Book" umbrella over MRS/CSR/MES is consistent.

---

## 4. Build-Sequencing Hazards (epic planner + architect)

### 4a. BUILD-GATE cascade on FR-6 — MAJOR (unresolved)
FR-4's note and RR-3 make a **storekeeper-flow UX specification a hard precondition on building FR-6**. FR-6 (Material-In OCR capture) is the root of the largest dependency subtree in the product:
- FR-7 (no-receipt-no-acceptance), FR-10/11/12 (offline, provenance, registry) sit on the same capture layer;
- FR-18 (auto-post stock), FR-19 (AP sign-off), FR-15 (material-vs-BOQ), FR-24–26 (Source B reconciliation) all consume Material-In;
- **SM-1**, the make-or-break metric, is measured on it.

So an epic planner cannot schedule roughly a third of v1 until a UX artifact that **does not yet exist** is produced — and Open Q14 records that its *owner and timeline are unconfirmed*, while D-2 (John/PM registered dissent) argues the gate should be relaxed. **The planner inherits an unresolved gate, not a decision.** Recommend: resolve Open Q14 (commission or waive the gate) *before* epic sequencing, or the FR-6 subtree has no defensible start date.

### 4b. Offline OCR timing — CONTRADICTION on the linchpin capture — CRITICAL
There is a genuine contradiction across three sources about *where and when* challan OCR runs:
- **UJ-1** narrates the storekeeper photographing the challan, the app "reads quantity and rate off the paper" and showing "icon-labelled fields to confirm," then a green tick appears **offline** and the capture queues for sync — i.e. OCR happens **on-device, before confirmation, offline.**
- **FR-6** says the Operator "confirms/corrects" OCR-populated quantity/rate fields at capture.
- **FR-10** says "server-side processing (**OCR**, anomaly checks) runs on the synced copy" — i.e. OCR happens **after sync, server-side.**
- **Addendum §A/§B** puts challan OCR (PaddleOCR + Qwen2.5-VL Urdu fine-tune) on the **self-hosted server**; the on-device tier carries only a small plausibility VLM and EXIF/pHash — **not** challan field extraction.

**These cannot all be true.** If OCR is server-side on sync (FR-10 + Addendum), then offline the storekeeper has **nothing to confirm** — contradicting UJ-1 and FR-6. If OCR is on-device (UJ-1), that contradicts FR-10 and the Addendum's model-placement. This is not a nuance: it determines the entire offline storekeeper flow (the product's stated linchpin), the on-device model footprint the architect must budget for, and the sync contract. **A UX designer and an architect both stall here.** Must be resolved explicitly (recommended framing: on-device *best-effort* extraction for offline confirm, with authoritative server OCR re-run on sync — but the PRD must say so; today it says both).

### 4c. Three-execution-location + API-first + offline-sync architect decisions
- **3 execution locations — ENOUGH.** NFR-4/NFR-5 plus Addendum §A give the architect a clear, per-task placement policy (on-device / self-hosted / redacted third-party API), the redaction-proxy rule, and the provider-router abstraction. This decision is well-sourced. **No gap** (apart from 4b's OCR-placement contradiction).
- **API-first — ENOUGH.** NFR-1 is explicit; §7.2 confirms native is a later client on the same API.
- **Offline-sync — MOSTLY ENOUGH, one gap.** NFR-2/NFR-19/FR-10/FR-11 specify local-queue, durability-across-restart, and original-timestamp/geotag preservation. **Gap:** conflict/ordering semantics for append-only records created offline are not stated (largely mitigated because append-only creates versions rather than conflicting edits), and 4b's OCR-timing contradiction is the real blocker embedded here.

### 4d. External-data dependencies feeding v1 MUSTs
- **FR-39 / FR-56 (Rate Book / Expected-Value Service)** depend on obtaining provincial MRS/CSR and MES Analysis-of-Rates — which R-6 / Open Q7 record as **not yet obtained/verified**, and the Addendum flags MES as "[VERIFY: no current public doc retrieved]." Layer C is called "the centerpiece of a credible v1 engine," so a centerpiece v1 capability depends on unacquired external data. **Acknowledged (Q7/R-6)** but pilot-scheduling-relevant.
- **FR-24–26 Source B workflow.** RR-1 promotes Source B to the **denominator of SM-1** (the make-or-break metric), yet the *collection workflow* (who obtains/scans the statement, cadence, supplier-refusal path) is explicitly **undefined** (R-7 / Open Q6). §15.5 item 3 already flags "design this now, not later." A story for FR-24 cannot be written past "an authorized Operator scans it" without Q6. **Acknowledged**, but the single most pilot-critical undefined workflow.

### 4e. RR-4 heavy-stack dissent — noted, not a blocker
Winston's registered dissent (self-hosted GPU + offline sync + 3-layer engine + redaction proxy is heavy for a pilot) is a *scope* observation with a post-pilot revisit trigger, not an implementability gap. The architect has what they need to build the heavy stack; the dissent does not block.

---

## 5. Missing Acceptance Detail (would story-creation stall?)

The task named four specifics; here is their status:

| Detail | Status | Where |
|--------|--------|-------|
| Negative-confirmation **window duration** | **Flagged open** (but absent from FR-46 body) | Open Q4 |
| Report **cadence day** | **Flagged open** | Open Q4 |
| **Cash-heavy threshold** default | **Flagged open** | Open Q3 |
| **Retention horizon** | **Flagged open** | Open Q10 / NFR-13 |

These four are **not silent** — the PRD catches them. Good hygiene. Story creation for the affected FRs still cannot complete until Q3/Q4/Q10 resolve, so they should be closed before those epics are pulled, but no team is *surprised*.

**The genuinely SILENT acceptance gaps (not in §14) that will surprise a story writer:**
1. **Anomaly rule thresholds** — FR-36's "within N days" and "> X%", FR-34's / FR-39's "configurable margin" (§1b above). The day-one rule engine has undefined rule constants. **Round-trip.**
2. **Data-Credibility Score model/scale** — FR-25/FR-35 (§1b above). No scoring scheme at all. **Round-trip.**
3. **CO lifecycle "Approved" vs two-axis model** — FR-33 blocks billing "for any CO not in **Approved** state" *and* says work "cannot appear on a Client bill until **ratified**." The lifecycle (FR-27) is a single linear chain Draft→Proposed→Client-Aware→Approved→Executed→Billed, but FR-32 insists internal-approval and client-Ratification are **two independent axes**. It is not stated whether the lifecycle's "Approved" state = internal sign-off, client ratification, or both — nor how the two independent axes collapse into one linear "Approved" node, nor where "Client-Aware" ends and ratification begins. A story writer modeling the CO state machine cannot tell whether billing is gated on internal-Approved, client-ratified, or the conjunction. **Major — modeling ambiguity, not flagged in §14.**
4. **Tier percentage base** — FR-31's "≤1%" of *what* (§1b).

---

## 6. Summary Table of Findings

| # | Finding | Severity | Fix |
|---|---------|----------|-----|
| 1 | Offline OCR timing contradiction (UJ-1/FR-6 on-device-offline vs FR-10/Addendum server-on-sync) on the linchpin storekeeper capture | **Critical** | State the on-device best-effort + server-authoritative split explicitly; reconcile FR-6/FR-10/UJ-1/Addendum §A |
| 2 | Three broken FR cross-refs: FR-8→24 (want 23), FR-15→42 (want 36), FR-40→52 (want 53); renumbering residue | **Major** | Fix the three; run a full cross-ref sweep |
| 3 | FR-6 BUILD-GATE cascade — ~1/3 of v1 blocked on a not-yet-authored UX spec with unconfirmed owner/timeline (Q14) + live dissent (D-2) | **Major** | Close Open Q14 (commission or waive) before epic sequencing |
| 4 | Silently-absent anomaly thresholds: FR-36 N-days/X%, FR-34/FR-39 margin — day-one rule engine constants undefined, not in §14 | **Major** | Add recommended defaults per §9 north-star; add to Open Questions |
| 5 | Data-Credibility Score has no scale/model/bounds and no Open Question (FR-25/FR-35) | **Major** | Specify a scoring scheme or file an Open Question |
| 6 | CO "Approved" state vs two-independent-axes model ambiguous; billing gate (FR-33) references both "Approved" and "ratified" | **Major** | Clarify whether lifecycle "Approved" is internal, client, or both; map the two axes onto the linear states |
| 7 | Source B collection workflow undefined but is now SM-1 denominator (R-7/Q6) | **Major (acknowledged)** | Design workflow now (already §15.5 item 3) |
| 8 | Glossary blur: "Segregation of Duties" (money self-approval) written as lowercase "separation of duties" in FR-19/§4.4/§7.1 | **Minor–Major** | Normalize to the Glossary term |
| 9 | FR-31 tier percentage base unstated; "or" ambiguity | **Minor–Major** | State the base and the whichever-is rule |
| 10 | Layer-C Rate Book data unacquired (R-6/Q7); MES edition unverified | **Minor (acknowledged)** | Track as data-acquisition dependency for the engine epic |
| 11 | "Computed % Completion" rendered inconsistently | **Minor** | Use canonical term verbatim |

---

## 7. What is GOOD (so downstream teams trust the rest)

- FR/NFR/UJ/RR/Open-Q ID spaces are all contiguous, unique, and (apart from the 3 broken FR refs) resolve.
- Every FR has a testable-consequences block; the majority are genuinely testable.
- The three-execution-location, API-first, and redaction-proxy architecture decisions are well-sourced (NFR-4/5 + Addendum §A) — an architect can act.
- The PRD catches most of its own acceptance gaps in §14 (cadence, window, cash threshold, retention, Urdu bar, Punjabi threshold, CV exit) — these are *tickets, not surprises*.
- §15 preserves dissent honestly (build-gate, CV re-file, heavy-stack) rather than laundering it into false consensus — downstream teams can see which decisions are still soft.

**Recommended gate before handoff to epics/UX/architecture:** fix findings #1, #2, #4, #5, #6 (each is a guaranteed round-trip), and close Open Q14 (#3). #7–#11 can be resolved in-flight.
