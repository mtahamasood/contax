# Brief → PRD Reconciliation — Contax

**Inputs reconciled:** SOURCE brief (`briefs/brief-contax-2026-07-25/brief.md`) against PRD (`prds/prd-contax-2026-07-25/prd.md`) + PRD Addendum (`prds/prd-contax-2026-07-25/addendum.md`).

**Overall:** Fidelity is high. The PRD Vision (§1), value hierarchy (§0/§1), JTBD (§2), moat framing (§12), and invariants (§9) carry the brief's voice and emotional core unusually well for an FR document — including verbatim sales lines ("with us, you see every rupee and every brick"), the "suspicion / the person best placed to reassure is the cheater" premise, the "artifact the client keeps and forwards" wedge, and the "table-stakes mechanics; positioning-then-data moat" humility. The WON'T list, COULD-v2 list, business model, why-now, and every Open-Question/Risk from the brief are all carried. The gaps below are the residue.

---

## Gaps

### 1. [weakened] Labour fraud / ghost workers has no capture or requirement
- **Brief:** The Problem enumerates four structural leak points; one is "**Labour fraud — ghost workers and inflated headcounts on the muster roll**" (brief §Problem). PM addendum H also names "labour & consumption fraud (the paperless leak points) caught against industry-average labour productivity."
- **PRD:** JTBD §2.1 lists "ghost labour" in the owner's pain, and Addendum §B/§H mention labour-productivity comparison in passing — but **no FR captures labour/attendance/headcount**, and the Anomaly Engine FRs (FR-35..41) list no labour-productivity signal among their features. The three-capture spine (material-in, money, photo) has no labour datapoint.
- **Why it matters:** A named quarter of the stated problem has no mechanism in the requirements. Either it should be explicitly deferred (like a WON'T/v2) or an FR should exist. Right now it is silently under-served — the PRD implies it's solved (JTBD promise) without a requirement that solves it.

### 2. [weakened] Schedule drift is a named problem with thin coverage
- **Brief:** Fourth leak point: "**Schedule drift — completion dates slip with no accountable cause**" (brief §Problem); emotional payoff "trust the completion date."
- **PRD:** Emotional JTBD "let me trust the completion date" is present, and CO schedule-deltas (FR-28) + Computed % Completion (FR-14) exist — but there is **no timeline/schedule-baseline-vs-actual tracking FR** that surfaces drift with an "accountable cause." Progress % ≠ schedule slippage against a plan.
- **Why it matters:** Of the four leak points, schedule drift is the least instrumented. The brief frames drift as a first-class trust failure; the PRD reduces it to a byproduct of milestone completion.

### 3. [added] Source B promoted from "redundancy nicety" to SM-1's primary measuring instrument (RR-1)
- **Brief:** Source B (supplier statement) is explicitly framed as "**confidence-through-redundancy (owner's call)**," carrying "collection friction," kept in v1 as an enhancement, not a blocker (brief §Scope MUST; Open Q on Source-B friction).
- **PRD:** RR-1 (§15) and SM-1 (§8) and §4.5 **re-cast Source B as the denominator of the make-or-break metric** — "the primary instrument of the make-or-break metric," "load-bearing for the pilot's primary success criterion, not optional."
- **Why it matters:** This elevation is a PRD-originated analytical move, not in the brief. It is defensible (a missed delivery is invisible to the system that missed it), but it makes an **undefined workflow** — who obtains/scans the statement, cadence, supplier-refusal path (still open as R-7/Open-Q6) — suddenly *pilot-critical*. The brief's stance (Source B is enhancing, absence is graceful, FR-26) now sits in tension with SM-1 depending on it. Flag for the owner: the brief did not treat Source B as make-or-break.

### 4. [weakened] CV progress-% relaxed from a v1 MUST to a non-build-blocking research spike (RR-2)
- **Brief:** CV progress-% is "**kept as a v1 MUST (owner's call)**," against the AI/ML research's advice, made safe by BOQ fallback + shadow-mode (brief §Scope "MUST — but flagged as known-risk"; Open Questions).
- **PRD:** §0 claims fidelity ("CV progress-% kept in shadow-mode"), but §7.1 + RR-2 (§15) actually **re-file it from "critical-path v1 MUST" to "v1 Shadow research spike, not a build-blocking MUST,"** pending Owner sign-off (Open Q15).
- **Why it matters:** This is a genuine relaxation of a decision the brief records as the owner's deliberate call. It is transparently flagged (Accepted-with-condition, owner sign-off required) rather than silently dropped — but a reader trusting §0's "carries that decision" language would miss that the status was downgraded. The gap is between what §0 asserts and what RR-2 does.

### 5. [weakened] "Trust engineered through structure, not forced behavior change" not elevated to a named invariant
- **Brief:** A headline differentiator bullet: "**Trust is engineered through structure, not forced behavior change.** Users stay on WhatsApp and phone calls; approvals happen outside the app... Nothing asks the client — or the suppliers — to adopt anything" (brief §What Makes This Different).
- **PRD:** The mechanics exist (FR-52 records decisions after the fact; §4.11 "does not force that behavior change"; non-goals reject in-app chat / supplier app). But this "**no adoption ask / structure-not-behavior**" principle is **not stated as a §9 invariant**, unlike the other differentiator bullets which all became invariants (internal-first, credibility-scored, adversarial-by-construction).
- **Why it matters:** It is one of the brief's five load-bearing positioning claims and the answer to "why won't users reject this." Scattering it across FRs instead of naming it as an invariant risks a downstream designer adding an adoption ask (an in-app approval step, a supplier confirmation) that violates the moat without tripping any stated principle.

### 6. [added] Storekeeper-flow UX spec as a hard build-gate on FR-6 (RR-3)
- **Brief:** Requires low-literacy icon/photo/voice-first UX and app-mandatory-for-storekeeper — but imposes **no process precondition** on building the capture layer.
- **PRD:** RR-3 + FR-4 note make "a dedicated storekeeper-flow UX specification" a **precondition on building FR-6**, with registered PM dissent that a rougher flow could ship and iterate.
- **Why it matters:** Reasonable and low-risk, but it is a PRD-added gate (with acknowledged dissent) not present in the brief; it could pressure the pilot timeline. Noted as scope the brief did not mandate.

### 7. [added] v1 elements sourced from research, beyond the brief's explicit MUST list
- **Company-history bootstrap at onboarding (FR-38)** — importing past closed projects is called "the single highest-leverage cold-start move" (Addendum §C), but the brief's v1 MUST list does not name it; it derives from `ai-ml-research`, not the brief. The brief only cites "company's own project history (month one)" as a moat stage, not a v1 onboarding import feature.
- **Expected-Value Service as a first-class v1 FR (FR-56)** — the brief frames the "expected value" service as part of the *year-one+* dataset moat; the PRD makes a unified internal service a v1 requirement. Defensible (rate-book prior is day-one) but a mild forward-pull of moat machinery into v1.
- **Counter-metrics SM-C1/C2/C3 (§8)** — sensible guardrails, but not in the brief's Success Criteria.
- **Why it matters:** None contradict the brief and all are reasonable, but each adds v1 surface/obligation the brief did not explicitly authorize — relevant given the brief's own flagged "MVP scope weight" risk and the round-table's over-scope dissent (D-1, RR-4).

### 8. [weakened] Minor framing losses
- **"The ROI is self-funding"** (brief §Business Model, pricing power) — the vivid self-funding framing is dropped; PRD §11 keeps only "frame ROI on the documented ~35% overrun."
- **"Opens the owner's wallet three ways"** — the brief's persuasion framing of the value prop is flattened into the functional JTBD list (§2.1). Content preserved; the sales-narrative punch is gone.
- **Overseas persona as "the story that dramatizes the value in sales"** — the brief assigns the overseas client an explicit *sales-narrative* role even though v1 doesn't fork on segment; the PRD keeps the persona (UJ-4) and the config (FR-5) but drops the "this is the story sales tells" positioning note.
- **Why it matters:** Individually small; collectively these are the tone/positioning nuances an FR structure tends to shed. They matter most to whoever writes GTM/sales collateral downstream, who will read the PRD and not the brief.

---

*No material brief content was found silently deleted; the significant items above are either under-instrumented problems (labour, schedule), PRD-originated re-weightings that should get owner eyes (Source B, CV re-file), or added v1/process scope. The brief's core thesis, voice, and moat are substantially preserved.*
