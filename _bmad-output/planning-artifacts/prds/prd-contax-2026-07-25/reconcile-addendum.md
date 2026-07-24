---
title: "Reconciliation — Brief Addendum → PRD + PRD Addendum"
status: review
created: 2026-07-25
reconciler: automated source-vs-PRD pass
---

# Reconciliation: Brief Addendum vs PRD / PRD Addendum

**Source:** `briefs/brief-contax-2026-07-25/addendum.md` (downstream depth: full CO governance, data-entry-paradox mechanisms, supplier reconciliation A/B, AI/ML strategy, moat + 6 parked plays, client delivery, benchmark sourcing, competitive landscape).
**Checked against:** `prds/prd-contax-2026-07-25/prd.md` and `prds/prd-contax-2026-07-25/addendum.md`.

**Headline:** Coverage is strong. Every governance mechanic, the fraud-surveillance detail set, and every named constraint (WhatsApp Business Platform rules, field-level redaction, no-rate-leak) made it through intact. Below are the exceptions — one substantive dropped requirement, one weakened strategic caveat, and a few minor/tracked items.

---

## Gaps

### 1. [dropped] Labour-productivity fraud detection — a named leak point with no FR

- **Source says:** Brief addendum §2 (Data-Entry Paradox) names **"Labour & consumption fraud (the two paperless leak points)"** and states both are "caught against industry-average **labour productivity** / material consumption and against the planned BOQ." Brief addendum §7 reinforces it: the *Analysis of Rates* behind MRS/CSR "breaks each work item into **labour** and material coefficients (mason-days per unit of brickwork...)."
- **PRD coverage:** **Material** consumption-vs-BOQ is a first-class requirement (FR-15, FR-37 features include "consumption-vs-BOQ ratio"). **Labour productivity has no FR, no capture event, and no anomaly feature.** It survives only as prose rationale in PRD addendum §H, and the labour coefficients in §K have no consuming FR. Meanwhile the PRD promises to catch "**ghost labour**" in the Owner's JTBD (§2.1) — a promise with no detection mechanism behind it. The three Capture events (Material-In, Money Movement, Site Photo) capture no labour/wage-productivity data that could be benchmarked.
- **Why it matters:** This is one of the two paperless leak points the anti-fraud premise rests on and is explicitly promised to the buyer. It has been silently reduced from a stated fraud control to unrequirement'd rationale. Either an FR should benchmark labour productivity (against the Analysis-of-Rates labour coefficients) or the "ghost labour" claim in §2.1 should be scoped down to match. Left as-is, the PRD over-promises relative to what its FRs deliver.

### 2. [weakened] The competitive self-collapse caveat softened into pure advantage

- **Source says:** Brief addendum §8 (Honest whitespace) carries a self-directed warning: "**if the contractor is the voluntary buyer/adopter, the adversarial positioning can collapse into 'just a nicer client portal' — i.e. what Buildertrend already is.** Novelty lives in *who pays and who it's pointed at* = a GTM moat, copyable by Smart Construction."
- **PRD coverage:** PRD §12 and R-4 capture the *copyable-by-Smart-Construction* half and frame buyer-conflict as Contax's **moat/advantage** ("adversarial toward its own operators by construction," "owner-mandated"). But the source's *self-critique* — that Contax's own positioning risks collapsing into a mere client portal because its buyer is the contractor — is not carried; the risk is presented only as a strength.
- **Why it matters:** The source deliberately flags this as a risk the pitch must not gloss over. The PRD's one-sided framing removes the guardrail. Worth restoring as an explicit GTM risk (the moat is *who it's pointed at*, and that is thin if the buyer ever de-emphasizes the adversarial stance).

### 3. [weakened] "Random spot-check photo prompts" demoted from paradox mechanism to deferred v2

- **Source says:** Brief addendum §2 lists "random spot-check photo prompts" among the mechanisms answering the data-entry paradox (the make-or-break problem).
- **PRD coverage:** Moved to Out-of-Scope v2 (§7.2) with a "cheap and adversarially valuable; revisit for late-v1" note; also flagged v2 in addendum §H.
- **Why it matters:** Low severity — it is *documented*, not silently dropped. Noted only because it is one of the paradox mechanisms and the deferral slightly thins the day-one adversarial toolset against the very risk (R-1) it addresses.

### 4. [note — documented change, not a silent gap] CV progress-% re-filed from v1 MUST to shadow research spike

- **Source says:** Brief addendum §4 carries single-photo CV progress-% "as a v1 MUST with manual / BOQ-derived fallback."
- **PRD coverage:** The round-table (RR-2, §15) **re-filed** it to a v1 Shadow research spike, off the critical path, gated on Owner sign-off (Open Q15). PRD addendum §D still records the "v1 MUST by owner's call" origin.
- **Why it matters:** Not a silent distortion — it is explicitly registered as a decision change requiring Owner sign-off. Flagged here only so the source↔PRD divergence is on the record; the reconciliation does not treat it as a lost item.

---

## Confirmed fully carried (spot-check of the emphasis areas)

- **CO governance mechanics** — separation of powers (FR-29), lifecycle + stamped append-only transitions (FR-27), mandatory cost+schedule deltas (FR-28), segregation of duties / cross-rank collusion (FR-30), three configurable tiers × three always-available ratification modes with defaults (FR-31), two independent axes (FR-32), no retroactive billing / "executed before approval" (FR-33), immutable-trail-only-Cancelled (FR-27). All intact, including the proposed Tier-C "no silent yes" hard floor (FR-31 assumption / Open Q5).
- **CO fraud surveillance** — all four signals preserved: rate-vs-benchmark, CO velocity per engineer cross-project, round-number bias, just-under-threshold clustering (FR-34, FR-55).
- **Negative-confirmation mechanics** — WhatsApp decision, "Decisions recorded this week," silence=accepted / one reply disputes, "decisions recorded this week: none," attach client's *own* evidence (FR-46).
- **WhatsApp Business Platform constraint** — pre-approved templates, 24-hour session window, metering, BSP requirement, unofficial-libraries-get-banned, "Meta changes rules — re-verify" (PRD §5 + addendum §J; the "libraries banned" clause lives in addendum §J).
- **Field-level redaction + location-is-config** (NFR-4/NFR-5, addendum §A) and **no supplier-rate/margin leak** (FR-44, NFR-17).
- **Supplier reconciliation A/B**, incl. Source B = statement issued *to the company*, not the private profit ledger; collection friction non-blocking; rejected supplier one-tap (FR-24..26, §5, addendum §I).
- **AI/ML three execution locations, 3-layer cold-start, model picks, two-tailed/internal-first** (NFR-4, FR-35..41, addendum §A/B/C).
- **Moat + six parked monetization plays** (FR-56/57, §11, addendum §L) and **benchmark sourcing** MRS/CSR/MES + labour/material coefficients (FR-39, addendum §K).
