---
title: "Product Requirements Document: Contax"
status: final
created: 2026-07-25
updated: 2026-07-25
review: "Party-mode round-table + Finalize reviewer gate (rubric/anti-fraud/downstream) — see §15 Review Register"
---

# PRD: Contax
*Working title — confirm.*

> A trust machine for Pakistani construction. Contax proves a construction company's honesty to its client every week — and gives the company control over the on-site leakage it cannot otherwise see.

## 0. Document Purpose

This PRD is the requirements contract for Contax, written for the downstream BMad workflows (architecture, UX, epics-and-stories) and for the pilot team who will recruit and run the first customer. It is derived from — and must stay faithful to — the **Product Brief** ([brief.md](../../briefs/brief-contax-2026-07-25/brief.md)) and its **Addendum** ([addendum.md](../../briefs/brief-contax-2026-07-25/addendum.md)); where the brief resolved a decision (the deliberately large v1 MUST set, Source-B reconciliation kept despite collection friction, company-first segment-agnostic beachhead), this PRD carries that decision rather than re-opening it. The one decision revised post-draft is CV progress-% — re-filed by the round-table from critical-path MUST to a shadow research spike pending Owner sign-off (RR-2, §15).

It is structured as a Glossary-anchored vocabulary, features grouped with globally-numbered Functional Requirements (FR-N) nested under each, cross-cutting Non-Functional Requirements in their own section, and assumptions tagged inline (`[ASSUMPTION: ...]`) and collected in §14. Implementation-level detail that does not belong in a capabilities document — AI/ML model selection and execution-location rationale, cost/feasibility numbers, benchmark-data source editions, rejected alternatives, and the six parked monetization plays — lives in the companion **PRD Addendum** ([addendum.md](./addendum.md)) and is referenced from here, not duplicated.

**The value hierarchy is foundational and resolves every trade-off in this document:** the buyer and customer is the construction company; the client's interests matter to the degree they create value for the company; when the two conflict, the company wins — but the product's entire logic is that a client's trust in the company is itself the company's most valuable asset.

## 1. Vision

In Pakistani construction the client's defining experience is **suspicion** — they cannot tell whether they are being cheated, and the person best placed to reassure them (the site engineer) is often the person doing the cheating. Contax inverts the burden of proof: instead of the client hunting for signs of guilt, the construction company proves its innocence, automatically, every week, through a branded evidence-backed report delivered over WhatsApp and mirrored in a live web app.

Underneath that client-facing skin, Contax is an **internal anti-leakage control system**. It captures what happens on site at the three points where money and material move, reconciles those captures against each other and against external benchmarks, and flags the anomalies that signal fraud — most of which today happens below senior management's line of sight, done by the low- and mid-level on-site staff who are always present while management only visits. Contax replaces management's constant *polling* of staff with a system that collects information on its own.

The wedge is a weekly artifact the client keeps and forwards; the moat is anti-fraud **depth** that an incumbent structurally cannot match, because every incumbent ERP depends on the goodwill of the very staff Contax is built to surveil, whereas Contax is mandated top-down by the owner and is adversarial toward its own operators by construction. The long game is bigger than the app: the weekly captures accumulate into the one asset nobody in Pakistani construction currently owns — a cross-company cost-and-productivity dataset that becomes the industry's benchmark for what things should cost and consume.

## 2. Target User

### 2.1 Jobs To Be Done

**The Company / Owner (buyer, champion, beneficiary — one person):**
- *Functional* — "Give me transparency into my own organization so I stop operating on thin confidence in my own numbers."
- *Functional* — "Catch the leakage — cost padding and fake extras directly (challan/rate/CO controls), and material pilferage and ghost labour *indirectly* (BOQ-consumption variance and productivity anomalies) — that happens on site without my knowledge." *(v1 instruments the paper-backed leak points directly; the paperless ones — consumption/material-out and labour — are detected indirectly against BOQ and industry norms, not by a dedicated capture. See R-9 / Non-Goals.)*
- *Functional* — "Stop me having to constantly poll my staff for reports; let the system collect the information."
- *Social / commercial* — "Win the hesitant client who has been burned before — let me say 'with us, you see every rupee and every brick' and prove it."
- *Emotional* — "Let me trust the completion date, the cost, and the record without personally chasing every site."

**The Client (end-user, not buyer):**
- *Emotional* — "Stop me wondering whether I am being cheated." (This is the job — not a dashboard.)
- *Functional* — "Let me see progress and spending without flying home or sending a relative to check."
- *Social* — "Give me something official I can forward to my family watching the site."

**The on-site staff (operators — management, site engineer, accountant, storekeeper):**
- *Contextual* — "Let me record what happened on site in seconds, from my phone, even when I can barely read, even with no signal."
- Note: staff are mandated operators, not voluntary adopters — see the mandate assumption in §8 and §13.

### 2.2 Non-Users (v1)

- **The supplier** — Contax reads the supplier's paper (challan, account statement) but never asks a supplier to adopt anything, install anything, or confirm anything in-app.
- **The client's family / parallel checkers** — receive forwarded reports but get no parallel data-entry surface.
- **Large institutional / public-works contractors** — v1 centers the SMB-residential / absent-owner wedge, not ADB/China-State-scale institutional projects (that is EZYPRO's space).
- **The client as a payer** — v1 bills the company; charging the (often hard-currency) overseas client is a parked option, not built.

### 2.3 Key User Journeys

*Named-persona narratives the product enables, numbered UJ-1..UJ-6. FRs reference these inline ("Realizes UJ-3"). Personas are illustrative; the beachhead does not fork on client segment.*

- **UJ-1. Kamran the storekeeper logs a cement delivery he cannot spell.**
  - **Persona + context:** Kamran runs the store on a DHA Lahore house build; low-literacy, reads numbers better than words, works a dusty site with patchy signal.
  - **Entry state:** authenticated on a shared site phone via the icon/photo/voice-first capture surface; a truck has just arrived.
  - **Path:** taps the big camera icon → photographs the delivery challan → photographs the truck/material → the app reads quantity and rate off the paper (Kamran types nothing) and shows him icon-labelled fields to confirm.
  - **Climax:** a green tick — "delivery recorded" — appears offline; the capture queues for sync. Stock posts automatically against the BOQ.
  - **Resolution:** the material-in event is now a first-class capture with a geotag, a timestamp, and two photos; it will feed the weekly report and the anomaly engine.
  - **Edge case:** no challan paper → the app refuses to record a clean "received" and offers only a flagged "received without challan" state ("no receipt, no acceptance").

- **UJ-2. Ayesha the accountant signs off what the OCR proposed.**
  - **Persona + context:** Ayesha keeps the books for three of the company's projects from the head office.
  - **Entry state:** authenticated on web; a material-in capture from UJ-1 has produced a *recommended* accounts-payable entry.
  - **Path:** opens the AP queue → sees the OCR-derived payable (supplier, quantity, rate, amount) pre-filled as a recommendation → corrects the rate the OCR misread → signs off.
  - **Climax:** only on her sign-off does the payable commit to the ledger; the record stamps that Kamran captured and Ayesha approved — two different people.
  - **Resolution:** money movement is now recorded under separation of duties; the client outstanding balance updates.
  - **Edge case:** she cannot be the person who captured the delivery — the system blocks self-approval.

- **UJ-3. Bilal the site engineer raises a small change the client never has to phone about.**
  - **Persona + context:** Bilal runs day-to-day site work; a minor extra (Rs 40,000 of additional steel) is genuinely needed.
  - **Entry state:** authenticated; mid-project.
  - **Path:** raises a change order → the system classifies it Tier A (≤1% / ≤Rs 50k) → he enters the mandatory cost delta and schedule delta → the CO enters the lifecycle as *Proposed*.
  - **Climax:** the client is informed by **negative confirmation** in that week's report ("Decisions recorded this week"); silence over the stated window records as accepted, one reply disputes.
  - **Resolution:** the change is on an append-only trail; Bilal — who raised it — can never be the one who ratifies or bills it.
  - **Edge case:** Bilal starts the work before approval → it is logged and flagged "executed before approval," and is **not billable** until ratified.

- **UJ-4. Naeem in Dubai reads the proof instead of calling the site.**
  - **Persona + context:** Naeem, an overseas Pakistani five time zones away, is building a house back home and holds the money most worth defending; app-comfortable, genuinely unable to visit.
  - **Entry state:** receives a WhatsApp message on a Sunday — a branded PDF, not a link.
  - **Path:** opens the PDF (works without signal) → sees this week's spend, progress, photos, and a "Decisions recorded this week" section → optionally opens the live in-app view for the evidence timeline.
  - **Climax:** he sees the numbered report is #14 in an unbroken series — no week is missing — and forwards it to his brother watching the site.
  - **Resolution:** his weekly "checking-up" call to the site engineer does not happen; the forward reaches a cousin who is about to hire a builder.
  - **Edge case:** a decision he disputes → one WhatsApp reply flips the negative-confirmation record to disputed and raises an internal item.

- **UJ-5. Rana the owner catches a flag and decides who sees it.**
  - **Persona + context:** Rana owns the company, runs several concurrent projects, visits sites rarely.
  - **Entry state:** authenticated on the admin cross-project rollup.
  - **Path:** sees an anomaly flag — a cement rate 30% above the provincial rate-book on one project — reviews the two captures behind it → confirms it is real.
  - **Climax:** he decides, at his discretion, whether this flag is ever disclosed to that project's client (anomalies are **internal-first**).
  - **Resolution:** he acts on the leakage; the flag and his disposition are logged, feeding the label store that trains later models.
  - **Edge case:** a "suspiciously clean" flag (a rate exactly at benchmark every time) — the two-tailed engine surfaces under-consumption and too-perfect data as well.

- **UJ-6. Sana the manager works a supplier-statement divergence.**
  - **Persona + context:** Sana manages operations across projects from head office.
  - **Entry state:** a supplier's periodic account statement (Source B) has been scanned and reconciled against the accumulated challans (Source A).
  - **Path:** opens the reconciliation → sees Source B shows three deliveries Source A never captured → opens an internal-first investigation.
  - **Climax:** the disagreement between two independent records — not any single trusted entry — is what raised the flag.
  - **Resolution:** the gap is investigated internally before anything reaches the client; agreement on other lines has raised their data-credibility score.
  - **Edge case:** the supplier refuses to hand over a statement → the reconciliation is marked "Source B unavailable" and the affected records simply carry no redundancy bonus (Source B is confidence-enhancing, not a blocker).

## 3. Glossary

*Downstream workflows and readers must use these terms exactly. FRs, UJs, and SMs use Glossary terms verbatim; introducing a synonym anywhere is a discipline violation.*

- **Company** — the construction company; the buyer and customer. Has one Owner, employs Operators, runs Projects.
- **Owner** — the company director who buys, champions, and benefits from Contax; holds Tier-C authority and disclosure discretion. (a.k.a. admin.)
- **Operator** — any company staff member who uses the app to capture or process data: **Manager, Site Engineer, Accountant, Storekeeper**. The app is mandatory for every Operator role.
- **Client** — the end-customer of a Project (e.g. the person whose house is being built). Not a Contax buyer. One Client is a single identity with one or more **Concern-Tagged Contacts**.
- **Concern-Tagged Contact** — a phone number attached to a Client identity, tagged by role/concern (e.g. "primary," "family-watching-site"). One Client, many contacts.
- **Project** — a single construction job for one Client. The unit of pricing and of weekly reporting. A Company runs one or more concurrent Projects.
- **Capture Layer** — the three mandatory atomic capture events (Material-In, Money Movement, Site Photo) from which every other view is computed. Designed once, kept open (may grow past three).
- **Material-In** — a capture event: a delivery arriving on site, recorded by an OCR'd Challan plus a geotagged photo; quantities and rates are read off the paper, not typed.
- **Challan** — the supplier's paper delivery note; the primary material-in evidence (**Source A**).
- **Money Movement** — a capture event: a Ledger Entry recording cash/payment flow.
- **Ledger Entry** — an append-only accounting record of a Money Movement.
- **Site Photo** — a capture event: a geotagged, timestamped photograph serving simultaneously as trust evidence, progress proof, and a fraud datapoint.
- **BOQ (Bill of Quantities)** — the planned schedule of work items and quantities for a Project. Composed of **BOQ Line Items**.
- **BOQ Line Item** — one planned work/material line with quantity and rate; the unit against which consumption, progress, and anomalies are measured.
- **Milestone** — a defined stage of a Project whose **Computed % Completion** is derived from BOQ Line Items marked done — never a typed number.
- **Computed % Completion** — the Project/Milestone progress figure of record, derived from BOQ Line Items. The client-facing number.
- **Drawing** — a versioned project drawing. Carries an **Issued-for-Construction** marker denoting the version approved to build from.
- **Change Order (CO)** — a recorded change to Project scope/cost/schedule, moving through the CO Lifecycle under tiered governance. See §4.6.
- **CO Lifecycle** — Draft → Proposed → Client-Aware → Approved → Executed → Billed (plus terminal Cancelled). Append-only; every transition stamped who/when/evidence.
- **Ratification** — Client confirmation of a CO, via one of three **Ratification Modes**: **Negative Confirmation**, **Positive Evidence**, **Explicit Approval**.
- **Negative Confirmation** — a Ratification Mode: a decision stated in the weekly report; silence over a stated window = accepted, one reply disputes.
- **Positive Evidence** — a Ratification Mode: the Client's own voice note / screenshot / in-app tap attached as the record.
- **Explicit Approval** — a Ratification Mode: the Client's direct, affirmative approval; no silence-equals-yes.
- **Separation of Powers** — the rule that whoever *raises* a CO may never *ratify* or *bill* it.
- **Segregation of Duties** — the rule that whoever logs a purchase/consumption against a CO cannot be the one who proposed it; and (on money) whoever captures cannot self-approve.
- **Source A** — the OCR'd Challan; primary, real-time material record.
- **Source B** — the supplier's periodic **account statement issued to the Company** (the customer khata of deliveries and dues — not the supplier's private profit ledger). Secondary, periodic.
- **Supplier Reconciliation** — comparing accumulated Source A against Source B per supplier; agreement raises Data-Credibility Score, divergence raises an internal investigation.
- **Data-Credibility Score** — a first-class scored attribute on a fact, raised when two or more independent Captures agree. Redundancy, not any single trusted entry, is the fraud control.
- **Anomaly Flag** — a system-raised signal that a Capture or pattern deviates from a rule, a statistical norm, or an external price prior.
- **Anomaly Engine** — the internal-first, two-tailed detection system producing Anomaly Flags. See §4.7.
- **Rate Book** — an external Pakistani government schedule of rates (provincial MRS / CSR, MES Analysis of Rates) providing a day-one price prior.
- **Expected-Value Service** — the internal service answering "what should this cost / consume here?", fed by Rate Books, Company history, and (later) the cross-company pool.
- **Weekly Report** *(a.k.a. Proof-of-Innocence Report)* — the branded English+Urdu PDF (LLM-written narrative) delivered over WhatsApp and mirrored in the Client App, archived as a numbered **Report Series**.
- **Report Series** — the numbered, tamper-evident sequence of a Project's Weekly Reports, such that a missing week is visible.
- **Decision Audit Log** — the append-only record of decisions (including CO ratifications and transcribed voice-note decisions) with who/when/evidence.
- **Client App** — the live, responsive web view of a Project for the Client; mirrors the report and carries the evidence timeline.
- **Admin Rollup** — the Owner's cross-Project view aggregating status, money, and Anomaly Flags.
- **Traceable Payment Rail** — a payment channel that leaves a trace (bank / JazzCash / EasyPaisa), as opposed to untracked cash; cash-heavy sites are flagged.
- **Shadow-Mode** — a capability that runs and records but does not drive the number of record (specifically CV progress-% behind the BOQ-derived Computed % Completion).
- **Flag Disposition** — the human outcome recorded against every Anomaly Flag (confirmed leakage / false alarm / dismissed); the day-one label store that trains v2 models.
- **Report Cadence** — the weekly rhythm on which Captures are expected and Reports are generated.

## 4. Features

*Each subsection is a coherent feature: behavioral description, FRs nested and numbered globally (FR-1..FR-N), optional feature-specific NFRs and notes. Cross-cutting NFRs are in §10.*

### 4.1 Identity, Roles & Access

**Description:** Contax is mandatory for every Operator role and is the client's live window. Identity supports multiple auth methods because site staff, head-office staff, and overseas clients have different realities. A Client is modelled as **one identity with multiple Concern-Tagged Contacts**, so a house build "watched" by three family members on WhatsApp is still a single client of record. The Operator capture surface is **low-literacy-friendly (icon / photo / voice-first)** because the storekeeper — the most fraud-critical capture role — is often the least literate user. Realizes UJ-1, UJ-2, UJ-4.

**Functional Requirements:**

#### FR-1: Multi-method authentication
An Operator or Client can authenticate by phone, email, or username/password.
**Consequences (testable):**
- A user with only a phone number can complete signup and login without an email address.
- The same human account is reachable by any of its configured methods; methods can be added/removed without creating a new identity.

#### FR-2: Role-based access with mandatory-app roles
The Company can assign each Operator exactly one role (Manager, Site Engineer, Accountant, Storekeeper) that governs what they may capture, view, and approve.
**Consequences (testable):**
- The role set includes Storekeeper as a first-class role with capture rights; there is no "no-app" capture path (SMS/IVR capture is a non-goal — §5).
- An Operator cannot perform an action outside their role's grant (e.g. a Storekeeper cannot sign off an AP entry).

#### FR-3: Client identity with concern-tagged contacts
The Company can create a Client as one identity and attach one or more Concern-Tagged Contacts to it.
**Consequences (testable):**
- A Client with three contact numbers appears as a single Client on the Project, not three clients.
- Each contact carries an editable concern tag; report delivery and dispute handling resolve to the correct contact(s).

#### FR-4: Low-literacy capture surface
An Operator can complete every mandatory Capture through an icon / photo / voice-first interface without reading or typing prose.
**Consequences (testable):**
- Each of the three Capture events (§4.2) is reachable and completable using icons, camera, and voice input alone.
- Field confirmation uses icons/values, not free-text labels only. `[ASSUMPTION: the confirmation UI uses picture+number chips; exact iconography is a UX-doc decision.]`
**Notes:** `[BUILD-GATE — RR-3, Accepted-with-condition]` The storekeeper capture flow is the product's true linchpin (its abandonment is the failure mode behind Risk R-1). A dedicated **storekeeper-flow UX specification** — real screens, thumb-reach, offline states, and the "truck idling" time pressure — is a **precondition on building FR-6** (and the capture layer). Handoff: `bmad-ux`. *Registered dissent (John/PM): a rougher flow could ship and iterate on-site; revisit trigger — if the UX spec threatens the pilot timeline, the iterate-in-place path returns to the table (§15/RR-3).*

#### FR-5: Client-segment configuration (not a product fork)
The Company can configure Client-segment handling (e.g. async-approval emphasis, contact handling) per Project without changing product surface.
**Consequences (testable):**
- Overseas vs domestic is a configuration on the Project, not a separate build/flow.
- No feature is gated behind a "segment type"; the same capture and report pipeline serves both. Realizes the company-first, segment-agnostic beachhead.

### 4.2 The Capture Layer (three atomic events)

**Description:** The entire mandatory capture surface is **three atomic events**, and everything else in Contax is a computed view projected from them. The discipline is deliberate: the Capture Layer is designed once and kept open (it may grow past three), and this is the architectural spine that keeps the product coherent. Quantities and rates come **off the paper**, not from a clerk who could lie. Realizes UJ-1, UJ-2, UJ-5.

**Functional Requirements:**

#### FR-6: Material-In capture via OCR'd Challan
A Storekeeper (or authorized Operator) can record a Material-In event by photographing the delivery Challan, from which the system reads quantities and rates.
**Consequences (testable):**
- Quantity and rate fields are populated by OCR from the Challan image; the Operator confirms/corrects but is not required to type them from scratch.
- **OCR timing (resolves the offline/online contradiction):** authoritative OCR runs **server-side on sync** (FR-10), not necessarily at the moment of an offline capture. Offline, the capture always succeeds by storing the Challan photo (and any minimal Operator-entered fields); on sync the server OCRs it and posts the parsed quantities/rates back for confirmation. An optional lightweight **on-device draft OCR** may pre-fill fields for immediate confirmation where the device supports it, but is never the system of record. UJ-1's "app reads the paper" is satisfied by the on-device draft where available and by the on-sync result otherwise; the Operator is never blocked offline. `[ASSUMPTION: on-device draft OCR is best-effort and optional; the server OCR is authoritative — an architecture/UX decision on the on-device model footprint, tied to Addendum §A.]`
- The captured Challan image is retained as Source A evidence and linked to the event.
- OCR handles Urdu (Nastaʿlīq) + English + numeric fields; illegible scans escalate per the AI/ML pipeline (see Addendum) rather than silently failing.

#### FR-7: "No receipt, no acceptance" enforcement
The system can record a delivery arriving without a Challan only as an explicitly flagged state, never as a clean acceptance.
**Consequences (testable):**
- A Material-In event without a Challan image is recorded as "received without challan" and carries a flag; it does not post as a normal, credibility-scored receipt.
- The flag is visible to Management and feeds the Anomaly Engine.

#### FR-8: Money Movement capture (Ledger Entry)
An authorized Operator can record a Money Movement as an append-only Ledger Entry.
**Consequences (testable):**
- Ledger Entries are append-only; corrections create a new versioned entry, never an in-place edit or delete (see FR-23).
- Each entry records who/when and links to related Captures where applicable.

#### FR-9: Geotagged Site Photo capture
An Operator can capture a Site Photo that is automatically geotagged and timestamped.
**Consequences (testable):**
- A captured Site Photo carries device geolocation and capture timestamp as metadata.
- The photo is a first-class Capture usable simultaneously as trust evidence, progress proof, and a fraud datapoint.

#### FR-10: Offline capture with deferred sync
An Operator can complete any of the three Capture events with no network connection; captures queue locally and sync when connectivity returns.
**Consequences (testable):**
- A Capture completed offline produces a local confirmation and is not lost on app close.
- On reconnect, queued Captures sync; server-side processing (OCR, anomaly checks) runs on the synced copy.
- Sync preserves original capture timestamp/geotag, not the sync time.

#### FR-11: Capture provenance stamping
Every Capture records the capturing identity, role, device time, and (for photos/material-in) geolocation, immutably.
**Consequences (testable):**
- Provenance fields cannot be edited after capture; a later correction is a new versioned record referencing the original.
- Provenance is available to the Anomaly Engine and the Decision Audit Log.

#### FR-12: Extensible capture registry
The Company/system can add new Capture event types over time without redesigning existing captures or downstream computed views.
**Consequences (testable):**
- Adding a fourth Capture type does not require schema changes to the three existing types.
- Downstream computed views (report, rollup, anomaly engine) consume Captures through a common contract. `[ASSUMPTION: v1 ships the three canonical captures; the registry is the seam, not additional v1 capture types.]`

### 4.3 Project Structure: BOQ, Milestones, Drawings & Computed Progress

**Description:** Progress is **computed, never typed**. Milestone completion is derived from BOQ Line Items marked done; the BOQ is also the yardstick for material-vs-plan consumption and the anchor the Anomaly Engine measures deviation against. Drawings are versioned with an Issued-for-Construction marker so "which drawing were we building from" is never ambiguous. Realizes UJ-1, UJ-3, UJ-5.

**Functional Requirements:**

#### FR-13: BOQ definition
The Company can define a Project's BOQ as a set of BOQ Line Items, each with planned quantity and rate.
**Consequences (testable):**
- Each Line Item has a stable identifier used by consumption tracking, progress, and anomaly features.
- The BOQ is the reference against which material consumption and progress are measured.

#### FR-14: Computed % Completion from BOQ
The system computes Milestone and Project % Completion from BOQ Line Items marked done; a % completion cannot be directly typed as the number of record.
**Consequences (testable):**
- Marking Line Items done changes Computed % Completion; there is no free-text "percent complete" input that overrides it.
- Computed % Completion is the client-facing figure in the Weekly Report and Client App.

#### FR-15: Material-vs-BOQ tracking
The system tracks consumed/received material against BOQ-planned quantities per Line Item.
**Consequences (testable):**
- For any material Line Item, the system shows planned vs received/consumed and the variance.
- Consumption exceeding BOQ-remaining raises an Anomaly Flag (see FR-36).

#### FR-16: Versioned Drawings with Issued-for-Construction marker
The Company can upload Drawings as versions and mark a specific version Issued-for-Construction.
**Consequences (testable):**
- Uploading a new Drawing version preserves prior versions; history is retained.
- Exactly one version per Drawing is the current Issued-for-Construction version; the marker is auditable.

#### FR-17: CV progress-% in Shadow-Mode (known-risk, guarded)
The system may estimate progress-% from Site Photos, but this estimate runs in **Shadow-Mode** and never becomes the client-facing number until it demonstrably earns trust.
**Consequences (testable):**
- The BOQ-derived Computed % Completion (FR-14) is the sole client-facing progress figure of record in v1.
- CV progress estimates are recorded and comparable against BOQ-derived progress but are not shown to the Client as the number of record.
- The v1 CV capability is coarse **stage classification** (e.g. grey-structure / plaster / finishing) rather than a precise percentage. `[ASSUMPTION: shadow-mode surfaces CV output to Management only, as a discrepancy signal against staff-reported progress.]`
**Notes:** `[NOTE FOR PM] The AI/ML research judged single-photo CV progress-% unreliable and advised deferring precise estimation to a v2/v3 research spike; it is retained as a v1 MUST by owner's call, made safe by the BOQ fallback and shadow-mode. See Addendum §CV and Risk R-3.`

### 4.4 Accounts, Money & Traceable Payments

**Description:** Money is where Segregation of Duties matters most. OCR posts material stock automatically, but the **accounts-payable entry is a recommendation the Accountant edits and signs off** before it commits — human-in-the-loop plus Segregation of Duties on money. Payments prefer **Traceable Payment Rails**; cash-heavy sites are flagged as a risk signal, not blocked. Realizes UJ-2, UJ-5.

**Functional Requirements:**

#### FR-18: Auto-posted material stock from OCR
On a confirmed Material-In capture, the system posts material stock automatically against the Project/BOQ.
**Consequences (testable):**
- Stock levels update from the confirmed Capture without a separate manual stock entry.
- The stock posting links back to the originating Challan and capturing Operator.

#### FR-19: Accountant-signed AP entry (human-in-loop, Segregation of Duties)
The system generates the accounts-payable entry for a Material-In as a **recommendation** that an Accountant must edit and sign off before it commits to the ledger.
**Consequences (testable):**
- No AP entry commits without an Accountant sign-off; until then it is a pending recommendation.
- The Operator who captured the Material-In cannot be the one who signs off its AP entry (self-approval blocked). Realizes UJ-2.
- The committed entry stamps both the capturing and the approving identity.

#### FR-20: Client outstanding balance
The system maintains and displays each Client's outstanding balance for their Project.
**Consequences (testable):**
- Outstanding balance updates from committed Ledger Entries and billed items.
- The balance shown to the Client in the Client App matches the internally computed figure (subject to disclosure rules — no supplier rates/margins, §4.8).

#### FR-21: Accounts core
The Company can maintain project accounts (payables, receipts against the Client, project cost roll-up).
**Consequences (testable):**
- Project cost roll-up reflects committed Ledger Entries and billed COs.
- Accounts feed the Admin Rollup and the Weekly Report's financial section.

#### FR-22: Traceable Payment Rails and cash-heavy flag
The system can record payments against a Traceable Payment Rail (bank / JazzCash / EasyPaisa) and flags a Project as cash-heavy when untraceable cash exceeds a configurable threshold.
**Consequences (testable):**
- A payment can be tagged with its rail; rail is retained on the Ledger Entry.
- When the cash share crosses the configurable threshold, the Project raises a cash-heavy risk flag visible to Management. `[ASSUMPTION: default cash-heavy threshold is a configurable % of project outflow; exact default TBD with pilot — see Open Questions.]`

#### FR-23: Append-only ledger with versioned corrections
The ledger is append-only; a correction creates a new version referencing the original, and backdating is auto-flagged.
**Consequences (testable):**
- No Ledger Entry can be deleted or edited in place; corrections are new versioned entries.
- An entry whose effective date precedes its capture time is flagged as backdated to the Anomaly Engine.

### 4.5 Supplier Reconciliation (Source A / Source B)

**Description:** Confidence through redundancy on materials — *and* the measuring instrument for the make-or-break metric. Source A (the OCR'd Challan, primary and real-time) is reconciled against Source B (the supplier's periodic account statement issued to the Company). Agreement raises the Data-Credibility Score; divergence raises an internal-first investigation. **Beyond credibility, Source B is the only independent count of deliveries the system has, and is therefore the denominator of SM-1's capture-completeness ratio (RR-1): the gap between captured Challans and the supplier statement is the invisible-miss rate that the capture layer cannot measure about itself.** Source B carries collection friction and is confidence-enhancing, but it is load-bearing for the pilot's primary success criterion, not optional. Realizes UJ-6. `[Kept as a v1 MUST by owner's call; the round-table (§15) upgraded its rationale from "redundancy nicety" to "SM-1 instrument."]`

**Functional Requirements:**

#### FR-24: Source B capture and per-supplier reconciliation
An authorized Operator can scan a supplier's periodic account statement (Source B), which the system reconciles against accumulated Source A challans for that supplier.
**Consequences (testable):**
- Source B is OCR'd and matched per supplier against the accumulated Source A record for the reconciliation period.
- Reconciliation output lists agreements and divergences (missing/extra deliveries, quantity/amount mismatches).
- **Collection independence (RR-7, anti-fraud):** because Source B is the SM-1 denominator (RR-1), it must be obtainable through a channel **not controlled by the surveilled site Operator** — the intended path is head-office ↔ supplier directly (the supplier already issues the statement to the Company). A Source B supplied *only* by the same site Operator whose captures it is meant to check carries reduced independence weight and does not fully validate SM-1. The Source B collection workflow (owner, cadence, refusal path) is Open Q6 and is pilot-critical, not deferrable.

#### FR-25: Credibility on agreement, investigation on divergence
On Source A/Source B agreement the system raises the Data-Credibility Score of the reconciled facts; on divergence it raises an internal-first investigation item.
**Consequences (testable):**
- Agreement measurably increases the Data-Credibility Score of the affected records.
- Divergence creates an internal investigation item (not disclosed to the Client by default — internal-first).

#### FR-26: Source B unavailable is a graceful — but *flagged* — state
The system records "Source B unavailable" without blocking capture, but the absence is a tracked risk state, not a silent neutral.
**Consequences (testable):**
- A supplier refusing/withholding a statement does not block Material-In capture or reporting.
- Records lacking Source B keep their Source-A-only Data-Credibility Score; the absence is visible, not silently treated as verified. Realizes UJ-6 edge case.
- **Un-measurability is itself surfaced (RR-7, anti-fraud):** because Source B is the SM-1 denominator (RR-1), a supplier or Project where Source B is *persistently* unavailable means SM-1's miss-rate cannot be measured there — a state the Anomaly Engine and Admin Rollup flag (an Operator who can make themselves un-measurable is a fraud signal, not a neutral gap). Contrast with FR-25's positive-agreement bonus: absence ≠ verified, and persistent absence ≠ harmless.

### 4.6 Change-Order Governance

**Description:** This governance stops an unguarded "the client approved X" from becoming a fraud instrument — a fabricated change order is otherwise perfect cover for pilferage or an inflated bill that lands on the client while management sees a normal record. Governance is **tiered and company-configurable**: every threshold and tier→mode mapping ships as a recommended default the Company can reconfigure (the north-star principle, §9). Two independent axes — **internal approval** and **client Ratification** — are configured separately. Realizes UJ-3.

**Functional Requirements:**

#### FR-27: CO lifecycle with stamped, append-only transitions
A CO moves through Draft → Proposed → Client-Aware → Approved → Executed → Billed (plus terminal Cancelled); every transition is stamped who/when/evidence and the trail is append-only.
**Consequences (testable):**
- Each transition records actor, timestamp, and attached evidence.
- No transition or field can be edited in place or deleted; edits create versions; the only removal is a Cancelled state.

#### FR-28: Mandatory cost and schedule deltas at Proposed
A CO cannot advance to Proposed without a cost delta and a schedule delta.
**Consequences (testable):**
- Attempting to propose a CO without both deltas is rejected.
- Both deltas are retained on the CO and surface in the Weekly Report's decisions section where applicable.

#### FR-29: Separation of Powers (raise ≠ ratify ≠ bill)
The Operator who raises a CO can never be the one who ratifies or bills it.
**Consequences (testable):**
- The system blocks the raiser from performing ratify or bill actions on the same CO.
- Ratify and bill require the appropriate authority tier (FR-31).

#### FR-30: Segregation of Duties on CO purchases/consumption
Whoever logs a purchase or consumption against a CO cannot be the one who proposed it.
**Consequences (testable):**
- The system blocks the CO proposer from logging purchases/consumption against that CO.
- Defeating the control requires cross-rank collusion, by construction.

#### FR-31: Three configurable authority tiers with three always-available ratification modes
The system supports three authority tiers scaling by rupee amount, each mapped to a Ratification Mode by a configurable default, with all three Ratification Modes always available per tier.
**Consequences (testable):**
- Defaults ship as: **Tier A** (≤1% or Rs 50k) — Site Engineer proposes, Client Negative Confirmation; **Tier B** (≤5% or Rs 500k) — Manager sign-off, Client Positive Evidence; **Tier C** (above) — Owner sign-off, Explicit Approval (no negative confirmation).
- The Company can override any tier threshold and any tier→mode cell.
- Tier C never permits Negative Confirmation even by override. `[ASSUMPTION: the "no silent yes above Tier C" floor is a hard guardrail, not a configurable cell; confirm.]`

#### FR-32: Two independent governance axes
The Company can configure internal approval (senior-management sign-off) and client Ratification independently.
**Consequences (testable):**
- Changing the internal-approval requirement does not change the client-Ratification requirement, and vice versa.
- The "executed before approval" gate is treated as an internal concern, separate from client Ratification.

#### FR-33: No retroactive billing
A CO's costs are not billable until it reaches Approved; pre-approval work is logged and flagged "executed before approval" and is not billable until ratified.
**Consequences (testable):**
- Billing actions are blocked for any CO not in Approved (or later) state.
- Work executed before approval is recorded and flagged, and cannot appear on a Client bill until ratified. Kills the end-of-project surprise bill. Realizes UJ-3 edge case.

#### FR-34: CO fraud surveillance
The system flags CO patterns indicative of fraud: rate-vs-benchmark, CO velocity per engineer (cross-project), round-number bias, and just-under-threshold clustering.
**Consequences (testable):**
- A CO rate exceeding the Rate Book prior by a configurable margin raises a flag.
- Repeated COs clustering just under an authority threshold raise a flag; CO velocity is evaluated per engineer across Projects.

### 4.7 Anomaly Engine & Data Credibility

**Description:** Fraud is caught not by trusting any single entry but by the **disagreement between entries** and by deviation from external priors. The engine is **internal-first** (Management audits before any client disclosure) and **two-tailed** (it flags suspiciously-clean and under-consumption as well as over-consumption). It ships in v1 without a training-data baseline via a three-layer cold-start: deterministic rules + unsupervised statistical outlier detection + a government Rate Book price prior. Every flag's human outcome is logged from day one as the **Flag Disposition** label store. Realizes UJ-5, UJ-6.

**Functional Requirements:**

#### FR-35: Data-Credibility Score driven by *independent* source agreement
The system assigns and updates a Data-Credibility Score on facts, raising it when two or more **independent** Captures agree — where independence means the sources are not under the control of a single actor.
**Consequences (testable):**
- A fact confirmed by two independent Captures (e.g. Challan + supplier statement, photo + ledger, milestone + consumption) carries a higher score than a single-source fact.
- **Independence is a precondition, not just agreement (RR-5, anti-fraud):** agreement between two captures produced or controlled by the *same* actor (same Operator, or an Operator plus a colluding supplier on the same transaction) does **not** raise the score. The credibility bonus requires distinct, non-colluding actors/channels; otherwise a single actor controlling both "sources" could manufacture maximally-credible fraud — inverting the control into an amplifier.
- The score is a first-class, queryable attribute usable by the report, rollup, and anomaly features.
- Low-independence agreement (same actor on both sides) is itself a signal the Anomaly Engine may flag.

#### FR-36: Layer A — deterministic rule engine (day one, zero data)
The system runs explainable deterministic rules from day one, requiring no training data.
**Consequences (testable):**
- Ships at least these rules: Challan quantity > BOQ-remaining for that material; Milestone % up with zero corresponding consumption; payment with no Challan/photo within N days; purchase rate exceeds Rate Book by > X%; duplicate Challan number or reused photo hash; cash clustered before report generation.
- Each rule flag carries a human-readable reason (required for the trust product — a flag must be explainable).

#### FR-37: Layer B — unsupervised statistical outlier detection (v1)
The system runs unsupervised outlier detection over engineered tabular features without labels, primarily intra-Project (cross-line-item) at cold start.
**Consequences (testable):**
- Outlier scoring operates on features such as unit rate, quantity-per-milestone, consumption-vs-BOQ ratio, cost-per-sqft, vendor concentration, transaction cadence.
- On a Project's first weeks (few rows), detection is cross-line-item within the Project (e.g. one vendor's cement rate vs all cement rates), not dependent on cross-project data.

#### FR-38: Company-history bootstrap at onboarding
The Company can import past closed projects at onboarding to seed a per-Company baseline of normal rates, consumption ratios, and cost-per-sqft.
**Consequences (testable):**
- Imported history produces a per-Company baseline used by Layers B and C before the first live Project accumulates data.
- Import accepts pre-existing formats (including spreadsheets). `[ASSUMPTION: import is best-effort/normalized, not a strict schema; exact import UX is an architecture/UX decision.]`

#### FR-39: Layer C — external Rate Book price prior
The system ingests Pakistani government Rate Books into a rates reference table providing a per-province expected-value prior, feeding both Layer A rules and Layer B features.
**Consequences (testable):**
- Each reference rate carries its source edition/date and province.
- A purchase/CO rate is comparable against the provincial prior with a configurable deviation margin. (Source editions/verification — see Addendum and Open Questions.)

#### FR-40: Two-tailed, internal-first flagging
Anomaly Flags surface deviation in either direction and are internal-first; disclosure to the Client is at Management discretion.
**Consequences (testable):**
- The engine flags below-expected/too-clean patterns (e.g. rate exactly at benchmark every time, under-consumption) as well as above-expected.
- No Anomaly Flag is disclosed to a Client automatically; disclosure requires a Management action (see FR-53).

#### FR-41: Flag Disposition label store (day one)
Every Anomaly Flag records its human Flag Disposition (confirmed leakage / false alarm / dismissed) from day one.
**Consequences (testable):**
- No flag can be closed without a recorded disposition and actor.
- The disposition store is retained as the labeled dataset that trains v2 learned models (see MVP Out-of-Scope §7.2).

### 4.8 The Weekly Proof-of-Innocence Report

**Description:** The wedge. An automatically-generated, branded PDF (English + Urdu, LLM-written narrative) delivered over WhatsApp and mirrored by a live Client App view. It reverses the burden of proof and is an artifact the client **keeps**: it survives without signal, feels official, and gets forwarded — to family watching the site, into WhatsApp groups, and eventually in front of the next person about to hire a builder. Because that same forwarding reaches competitors, the report **must never leak supplier rates or margins**. Reports are archived as a numbered, tamper-evident **Report Series** so a missing week is visible — disciplining the Company's own staff as much as it reassures the Client. Realizes UJ-4.

**Functional Requirements:**

#### FR-42: Automated weekly branded PDF, English + Urdu, LLM narrative
The system generates a branded Weekly Report PDF per Project on the Report Cadence, in English and Urdu, with an LLM-written narrative wrapped around DB-sourced numbers.
**Consequences (testable):**
- The PDF is branded to the Company, and a no-context forward recipient can identify the Company, Project, reporting week, and headline figures from the PDF alone.
- Narrative numbers come from the database; the LLM writes prose around them and cannot alter the figures (templated numeric guardrail).
- The report renders in both English and Urdu; Urdu quality is gated by passing an internal Urdu eval set (see Open Questions and Addendum).

#### FR-43: WhatsApp delivery as a PDF artifact
The system delivers the Weekly Report over WhatsApp as a PDF file (not a link into the app) to the Project's Concern-Tagged Contacts.
**Consequences (testable):**
- The delivered artifact is a self-contained PDF that opens and reads without signal after receipt.
- Delivery targets the correct Client contacts per Concern tag.

#### FR-44: No rate/margin leakage
The Weekly Report never exposes supplier rates or Company margins.
**Consequences (testable):**
- Supplier unit rates and margin figures are absent from the client-facing PDF and Client App by construction, not by manual redaction each week.
- A forwarded report reveals nothing a competitor could use on pricing. `[Cross-references the field-level redaction NFR, §10.]`

#### FR-45: Numbered tamper-evident Report Series
Reports are archived server-side as a numbered sequence such that a missing week is detectable.
**Consequences (testable):**
- Each report has a sequence number within its Project; a gap in the sequence is visible to Client and Company.
- Archived reports are immutable; regeneration produces a new versioned artifact, not an in-place overwrite. `[ASSUMPTION: "tamper-evident" = immutable archive + verifiable sequence/hash chain; cryptographic mechanism is an architecture decision.]`

#### FR-46: Negative-confirmation decisions section
Each Weekly Report carries a "Decisions recorded this week" section stating decisions under Negative Confirmation; silence over a stated window records as accepted, one reply disputes; a week with none states "decisions recorded this week: none."
**Consequences (testable):**
- The section lists each stated decision with its dispute window; the window is explicit in the report.
- No decisions → the report explicitly says "none," making unrecorded verbal decisions visible by their absence.
- A single Client reply within the window flips the record to disputed and raises an internal item. Realizes UJ-3, UJ-4.
- Where possible the Client's **own** evidence (voice note / screenshot) is attached so the record is the Client's voice, not the Company's paraphrase.
- **Dead-channel guard (RR-6, anti-fraud):** Negative Confirmation may only auto-accept on a channel with evidence of liveness (e.g. WhatsApp delivery/read signal, or a recent Client interaction). Sustained silence on a channel showing *no* liveness does **not** launder into "accepted" — it escalates the decision to a higher Ratification Mode (Positive Evidence / Explicit Approval) and flags the Client channel as unconfirmed. This closes the "overseas Client never opens WhatsApp, so silence = yes" loophole. `[ASSUMPTION: liveness signal availability depends on the WhatsApp delivery path; if read receipts are unavailable, a periodic positive check-in substitutes — Open Q16.]`

#### FR-47: Report generation is visible by its absence
A Project that misses its scheduled report generation surfaces the gap to Management (and the gap is visible to the Client via the sequence).
**Consequences (testable):**
- A missed Report Cadence raises an internal alert to Management.
- The missing sequence number is detectable in the Client App / Series view. `[ASSUMPTION: default cadence is weekly on a Company-configurable day; see Report Cadence in Glossary.]`

### 4.9 Client App (Live View)

**Description:** The live, responsive web mirror of the Project for the Client — the primary client surface, with the PDF as the forwardable artifact and the app as the deeper live view. It carries the evidence timeline and the decisions record, and it obeys the same no-rate-leak rule as the report. Realizes UJ-4.

**Functional Requirements:**

#### FR-48: Live Project view for the Client
A Client can view their Project's current progress (Computed % Completion), spend against them, photo evidence timeline, and decisions record in a responsive web app.
**Consequences (testable):**
- The Client App shows the same figures of record as the latest Weekly Report (progress, outstanding balance) subject to disclosure rules.
- The view is responsive (usable on a phone browser) — the API-first architecture allows a later native app against the same API (§10).

#### FR-49: Client evidence timeline
A Client can browse the chronological Site Photo and milestone evidence for their Project.
**Consequences (testable):**
- Photos appear in capture-time order with milestone context.
- Only client-appropriate evidence is shown (no internal-only anomaly detail, no supplier rates).

#### FR-50: Client dispute entry
A Client can dispute a stated decision or raise a concern from the Client App or via WhatsApp reply, landing it in the internal queue.
**Consequences (testable):**
- A dispute from either channel creates an internal item linked to the relevant decision/CO.
- The dispute is recorded in the Decision Audit Log with the Client's own words/evidence.

### 4.10 Photo Evidence & Integrity

**Description:** Site Photos are trust evidence, progress proof, and fraud datapoints at once. Integrity is largely classical — geotag/timestamp validation and duplicate/stale-photo detection — with on-device cheap checks so recycled progress photos are caught before they pollute the record. CV stage-classification runs in Shadow-Mode (FR-17). Realizes UJ-1, UJ-5.

**Functional Requirements:**

#### FR-51: Photo integrity checks
The system validates Site Photo geotag/timestamp and detects duplicate/near-duplicate ("recycled") photos.
**Consequences (testable):**
- A photo reused across milestones/weeks (matching perceptual hash) is flagged.
- Geotag/timestamp inconsistent with the Project location/time raises a flag.
- Cheap integrity checks can run on-device at capture (see AI/ML NFRs, §10). `[ASSUMPTION: on-device checks are best-effort; authoritative checks re-run server-side on sync.]`

### 4.11 Voice-Note Capture & Decision Audit Log

**Description:** Decisions in Pakistani construction happen on WhatsApp and phone calls, in Urdu and Punjabi — outside any app. Contax does not force that behavior change; it **records the decision after the fact**. Urdu/Punjabi voice notes are transcribed (ASR) into a searchable, append-only Decision Audit Log, so a verbal "haan, kar do" becomes an evidenced record. Realizes UJ-3, UJ-4.

**Functional Requirements:**

#### FR-52: Urdu/Punjabi voice-note ASR into the Decision Audit Log
An Operator or Client can submit an Urdu/Punjabi voice note that the system transcribes into a decision record in the append-only Decision Audit Log.
**Consequences (testable):**
- The voice note is transcribed and stored with its audio, transcript, speaker/source, and timestamp.
- The record is append-only and links to the CO/decision it ratifies where applicable (Positive Evidence mode). `[ASSUMPTION: ASR quality for Punjabi is lower than Urdu; low-confidence transcripts are flagged for human confirmation — see Open Questions.]`

#### FR-53: Management disclosure control for anomalies
A Manager or Owner (the disclosure-authority role is company-configurable, default Manager-and-above) can decide, per Anomaly Flag, whether it is disclosed to the Client; nothing is disclosed by default.
**Consequences (testable):**
- Disclosure is an explicit, logged action by an Operator holding the configured disclosure authority.
- Absent that action, no anomaly detail reaches the Client App or Weekly Report (internal-first — cross-refs FR-40).
- The disclosure-authority role sits in the role→permission matrix (Open Q17); the Glossary term for the deciding party is **Management** (Manager or Owner), used consistently across FR-40, FR-53, §9, and UJ-5.

### 4.12 Admin Cross-Project Rollup

**Description:** The Owner's single pane across all concurrent Projects — status, money, and Anomaly Flags aggregated — replacing the polling that Contax exists to eliminate. Realizes UJ-5.

**Functional Requirements:**

#### FR-54: Cross-Project rollup
The Owner can view an aggregated cross-Project dashboard of status, spend, outstanding balances, and open Anomaly Flags.
**Consequences (testable):**
- The rollup spans all of a Company's Projects and drills into any one.
- Open Anomaly Flags and their Dispositions are visible and actionable from the rollup. Realizes UJ-5.

#### FR-55: Cross-Project fraud signals
The rollup surfaces cross-Project signals (e.g. CO velocity per engineer across Projects, repeated supplier divergences).
**Consequences (testable):**
- A pattern spanning Projects (one engineer's CO behavior across sites) is visible at the rollup level, not only per-Project.

### 4.13 Benchmark Data & the Expected-Value Service

**Description:** The moat's foundation and a first-class internal service. Contax ingests government Rate Books on day one, accumulates the Company's own history in month one, and (post-v1) the cross-company pool. A single **Expected-Value Service** — "what should this cost / consume here?" — feeds BOQ estimation, the Anomaly Engine, and CO pricing alike. The architecture keeps a **pluggable supplier/financier/insurer seam** and the pooled dataset as a first-class internal service so parked platform plays can switch on later without a rebuild (extensibility NFR, §10). Realizes UJ-5.

**Functional Requirements:**

#### FR-56: Expected-Value Service
The system exposes an internal Expected-Value Service answering "what should item X cost/consume here?" fed by Rate Books and Company history.
**Consequences (testable):**
- The Anomaly Engine (Layer C), CO fraud surveillance (FR-34), and rate comparisons consume expected values from one service, not duplicated logic.
- Each expected value is traceable to its source (Rate Book edition, Company history, or — later — pool).

#### FR-57: Flag-disposition and history feed the moat
The system retains anonymizable Company history and Flag Dispositions structured so a future cross-company benchmarking tier and learned models can be built without re-architecture.
**Consequences (testable):**
- Company history and dispositions are stored in a form the Expected-Value Service can later aggregate across companies.
- No v1 feature exposes another company's data; cross-company benchmarking is out of scope for v1 (§7.2). `[Third-party sale of market-index data is a non-goal — §5.]`

## 5. Non-Goals (Explicit)

- **Not a contractor-productivity / general PM-ERP tool.** Contax is an anti-fraud client-transparency control system; it does not compete on scheduling/Gantt/HR-payroll breadth (that is Smart Construction / EZYPRO territory).
- **No client-family parallel data entry.** Family members receive forwarded reports; they get no capture surface.
- **No in-app client chat.** Communication stays on WhatsApp/phone; Contax records decisions, it does not become a messenger.
- **No supplier one-tap confirmation / supplier app.** Suppliers won't engage; Contax reads supplier paper, it does not ask suppliers to adopt anything.
- **No no-app SMS/IVR capture.** The app is mandatory for all Operator roles including Storekeeper; there is no lower-friction non-app capture path.
- **No direct consumption/material-out or labour capture in v1** `[NON-GOAL for MVP — R-9]`. v1 keeps the three-capture spine and detects consumption pilferage and ghost-labour *indirectly* (BOQ-consumption variance + rate/productivity anomalies, FR-15/FR-37/FR-39). A dedicated material-out/stock-take capture and a labour/muster capture are **v2** candidates (the registry seam FR-12 anticipates them); v1 does not claim direct instrumentation of the two paperless leak points.
- **No direct third-party sale of market-index data** in v1. The dataset is an internal moat asset; monetizing it externally is a parked, post-v1 play (see §11 and Addendum).
- **No two-way Company-side WhatsApp Business integration in v1.** Designed for in the architecture, deferred past MVP (Meta's WhatsApp Business Platform constraints — pre-approved templates, 24-hour session window, metering, BSP requirement — must be re-verified before building).
- **Not forking on client segment.** Overseas vs domestic is configuration, not two products.
- **Anomalies are never auto-disclosed to clients.** Disclosure is always a Management decision.

## 6. Why Now

Timing is load-bearing; the enabling conditions are verified in the brief's market research:
- **WhatsApp is ubiquitous in Pakistan** (Pakistan #3 globally in WhatsApp downloads) — the delivery channel is where users already are, so no adoption ask for the client.
- **Overseas remittances hit a record ~$38.3B in FY2025 (+27% YoY)** — the money pool behind remote home-building. *(Caveat: overseas-funded-construction PKR volume is unverified; do not cite a specific figure — Open Question.)*
- **Digital payments dominate** (JazzCash/EasyPaisa, ~88% of retail transactions digital) — Traceable Payment Rails are viable in-market.
- **OCR/LLM costs have collapsed** (self-hosted OCR ~16× cheaper than cloud) — document-capture of every Challan is economically trivial, which is what makes the three-capture spine affordable at scale.
- **Documented pain:** one study found ~100% of surveyed Pakistani projects overran, averaging ~34.6%. Lead ROI messaging on **cost overruns**, not on unquantified theft (Open Question / Risk).

## 7. MVP Scope

### 7.1 In Scope (v1 MUST)

The MVP is defined by one question: *what is the minimum that produces a credible Weekly Proof-of-Innocence Report and the internal control beneath it?* The v1 MUST set is **deliberately large** — an examined, owner's-call decision that the trust proposition requires the full set. In scope:

- Weekly Proof-of-Innocence Report (FR-42..FR-47): branded PDF over WhatsApp + live Client App, English + Urdu, LLM narrative, numbered tamper-evident Series, negative-confirmation decisions loop.
- The three Capture events (FR-6..FR-12): OCR'd Challan Material-In, Money Movement, geotagged Site Photo — offline-capable, provenance-stamped, extensible registry.
- BOQ, Milestones with Computed % Completion, Material-vs-BOQ, versioned Drawings with Issued-for-Construction (FR-13..FR-16).
- Accounts + Client outstanding balance; auto-posted stock; **Accountant-signed AP recommendation** (human-in-loop + separation of duties); Traceable Payment Rails + cash-heavy flag; append-only ledger (FR-18..FR-23).
- **Supplier Reconciliation Source A / Source B** (FR-24..FR-26) — kept despite collection friction.
- Change-Order governance: full lifecycle, mandatory deltas, separation of powers, segregation of duties, three configurable tiers × three ratification modes, no retroactive billing, CO fraud surveillance (FR-27..FR-34).
- Anomaly Engine v1: three-layer cold-start (rules + unsupervised + Rate Book prior), Data-Credibility Score, two-tailed, internal-first, Flag Disposition label store (FR-35..FR-41).
- Client App live view, evidence timeline, dispute entry (FR-48..FR-50).
- Photo integrity checks (FR-51).
- Urdu/Punjabi voice-note ASR into the Decision Audit Log; Management disclosure control (FR-52..FR-53).
- Admin cross-Project rollup + cross-Project fraud signals (FR-54..FR-55).
- Multi-auth identity; Client = one identity, many Concern-Tagged Contacts; low-literacy capture surface; segment as configuration (FR-1..FR-5).
- Expected-Value Service + extensibility seam (FR-56..FR-57).

**Re-filed by the round-table — v1 Shadow research spike, not a critical-path MUST (RR-2, Accepted-with-condition):**
- **CV progress-% estimation from photos (FR-17)** — originally a v1 MUST by owner's call, against the AI/ML research's judgment that single-photo CV progress-% is unreliable. The round-table (§15/RR-2) re-filed it: it remains in v1 as a **Shadow-Mode research spike** (coarse stage classification, Management-only, ships nothing client-facing) so it is off the pilot's critical path, but is **not** a build-blocking MUST. BOQ-derived % remains the sole client-facing number of record, and defined exit criteria (Open Q12) must be met before CV output could ever become client-facing. **Condition: this re-file requires Owner sign-off, because it revises a decision the Owner made deliberately — it is re-filed, not deleted.** See Risk R-3.

### 7.2 Out of Scope for MVP

- **Learned ML anomaly models + cross-company benchmarking** — deferred to v2; needs the accumulated Flag Disposition label store and multi-tenant volume. *(The v1 engine's most important output is the labeled dataset that unlocks this — FR-41.)*
- **Precise CV progress-% as a client-facing number** — deferred to a dedicated v2/v3 research spike; v1 is shadow-mode stage classification only.
- **Quality checklists / inspections / snag lists / test reports** — v2.
- **Deep document vault** — v2.
- **Quiet-mode (exceptions-only) client view** — v2.
- **Random spot-check photo prompts** — v2. `[NOTE FOR PM] spot-check prompts are cheap and adversarially valuable; revisit for late-v1 if timeline permits.`
- **Two-way Company-side WhatsApp Business integration** — architected-for, deferred (see §5).
- **Native mobile apps** — v1 is a responsive web app on an API-first backend; native follows against the same API.
- **Charging the Client directly / overseas-client-pays** — parked pricing option, not built (see §11).

## 8. Success Metrics

*Each SM cross-references the FR(s) it validates. Targets are first-pass and must be calibrated with the pilot Company; the point is that each is measurable.*

**Primary**
- **SM-1 — The mandate test (make-or-break):** on a pilot site, the three Capture events actually happen, driven by the Owner's mandate without Management chasing, at high completeness. Target: **≥90% of deliveries produce an OCR'd Challan**; Ledger and Site Photo capture on the same cadence. Validates FR-6..FR-12 and the single assumption the whole model rests on (§13). *Everything below is moot if this fails.*
  - **Denominator definition (load-bearing — review-registered, RR-1):** capture-completeness is a ratio whose denominator — *deliveries that actually happened* — cannot be supplied by the capture system itself, because a missed delivery is invisible to the very system that missed it. The denominator of record is therefore the **Supplier Reconciliation (Source B, FR-24..FR-26)**: accumulated captured Challans (Source A) reconciled against the supplier's independent account statement, where the gap **is** the miss rate. SM-1 is measured against Source B, not against a count the capture layer invents. This reframes Source B from a credibility bonus to the primary instrument of the make-or-break metric — see §4.5 and §15/RR-1.

**Secondary**
- **SM-2 — Client trust:** measurable drop in Client "checking-up" contacts to the Site Engineer; Weekly Report open and forward rates; Client-reported confidence (before/after). Validates FR-42..FR-50.
- **SM-3 — Owner value / leakage caught:** Anomaly Flags surfaced *and acted on* (documented leakage or overrun caught, per Flag Disposition); management hours reclaimed from polling. Validates FR-34..FR-41, FR-54..FR-55.
- **SM-4 — Commercial:** at least one sale the Owner attributes to the transparency pitch; first pilot converted to paid; logo/project retention across renewal. Validates the value hierarchy and §11.

**Counter-metrics (do not optimize)**
- **SM-C1 — Capture completeness at the cost of honesty.** Do not optimize SM-1's ≥90% by accepting low-credibility or "received without challan" captures as if clean. Watch the share of captures with low Data-Credibility Score; a rising capture rate with falling credibility is a fraud-laundering signal, not success. Counterbalances SM-1.
- **SM-C2 — Flag volume.** Do not optimize the Anomaly Engine for more flags; a flood of low-value flags trains Management to ignore them. Watch confirmed-leakage precision (Flag Disposition), not raw flag count. Counterbalances SM-3.
- **SM-C3 — Report cadence at the cost of report truth.** Do not optimize for unbroken Report Series by auto-generating hollow reports; a report must reflect real captures. Counterbalances SM-2.

## 9. Key Product Principles (Invariants)

These are invariants the architecture and downstream work must preserve:
- **North star:** prefer a *configurable choice with a recommended, pre-filled default* over a hard-coded single answer. Opinionated, not rigid — every governance threshold ships as a reconfigurable default.
- **Two separate axes, never conflated:** internal-facing control vs. client-facing transparency; internal approval vs. client Ratification. Both configurable, independently.
- **Anomalies are internal-first.** Management has discretion over whether any flag is ever disclosed to the Client.
- **Data credibility is scored, driven by source-agreement.** Redundancy, not any single trusted entry, is the fraud control.
- **Adversarial toward its own operators by construction.** Append-only ledgers, separation of powers, segregation of duties, evidence-mandatory entries — the design does not depend on the goodwill of the surveilled.
- **Trust is engineered through structure, not forced behavior change.** Users stay on WhatsApp and phone; approvals happen *outside* the app and are recorded after the fact; nothing asks the Client or the suppliers to adopt anything. A downstream design that introduces an adoption ask on the Client or supplier side violates this invariant and erodes the moat.
- **AI/ML is a standing evaluation lens on every feature,** across three execution locations, behind field-level redaction (see §10 and Addendum).
- **Capture Layer is small and closed-by-discipline; everything else is computed.** Keep the three-capture spine; resist adding capture surface.

## 10. Cross-Cutting Non-Functional Requirements

### 10.1 Platform & Architecture
- **NFR-1 (API-first, web-first):** the backend is API-first; v1 client surfaces are responsive web apps; native mobile is a later client against the same API. No web-only shortcut may foreclose the native path.
- **NFR-2 (Offline-tolerant capture):** capture must work on intermittent/low-bandwidth sites; captures queue locally and sync, preserving original timestamps/geotags; server processing tolerates retries and queueing (FR-10).
- **NFR-3 (Extensibility seam):** the architecture keeps a pluggable supplier/financier/insurer seam and the pooled dataset as a first-class internal service, so parked platform plays switch on without a rebuild (FR-56, FR-57, §11).

### 10.2 AI/ML Governance (three execution locations)
- **NFR-4 (Execution-location policy):** AI/ML runs in one of three locations by task — **on-device** (cheap integrity checks, offline), **self-hosted open-source models** (all sensitive/fraud data and the anomaly/ledger math — *never* a third-party API), **third-party LLM API** (only Urdu narrative and hard-OCR escalation). This is config, not code — a provider/router abstraction can swap or disable the API tier. *(Model picks per feature: Addendum.)*
- **NFR-5 (Field-level redaction):** a field-level redaction layer sits in front of every third-party API call — account balances, client identities, and totals are stripped; only what the task needs is sent. Ledger, transactions, and fraud flags are crown jewels and default OFF third-party APIs. **Two named tensions to resolve in architecture:** (a) *hard-OCR escalation* of an illegible Challan to a third-party vision API necessarily exposes that document's supplier line-rates to the API vendor — mitigation is crop-to-document + strip surrounding account context, prefer self-hosted 72B before any external escalation, and treat external escalation as an opt-in per-Company setting; supplier unit-rates on a single challan are lower-sensitivity than margins/balances but are not zero-sensitivity. (b) The *Urdu narrative* API sees the report's figures by construction — the numeric guardrail (NFR-6) means it cannot alter them, and identities/balances are redacted, but the Company must accept that redacted report numbers transit the narrative provider (or run Urdu self-hosted once NFR-7's gate is met).
- **NFR-6 (Templated numeric guardrails):** every LLM-generated report derives all numbers from the database; the model writes prose around fixed figures and cannot alter them (FR-42).
- **NFR-7 (Urdu quality gate):** self-hosted Urdu long-form narrative must pass an internal Urdu eval set before it replaces the third-party API for Urdu; at launch Urdu narrative may use the redacted API tier. An internal Urdu eval set (narrative + ASR) is a v1 deliverable. *(Research: no small open model is reliably fluent in long-form Urdu yet — Open Question.)*

### 10.3 Anti-Fraud & Integrity (system-wide)
- **NFR-8 (Immutability / append-only):** ledgers, CO trails, the Report Series, and the Decision Audit Log are append-only; no in-place edit or delete; corrections are versioned; backdating is auto-flagged.
- **NFR-9 (Separation & segregation enforced by the system):** separation of powers and segregation of duties (FR-29, FR-30, FR-19) are enforced by access control, not policy alone; self-approval and raise-then-ratify are blocked.
- **NFR-10 (Explainable flags):** every Anomaly Flag carries a human-readable reason — a trust product must be able to tell the Client *why* something was flagged (FR-36).
- **NFR-11 (Provenance everywhere):** every Capture, transition, and decision is stamped who/when and (where relevant) where/evidence, immutably.

### 10.4 Data Governance
- **NFR-12 (Data residency of financial data):** sensitive financial data and fraud math never leave the Company's/Contax's controlled environment (self-hosted); external processing is limited to redacted, structured inputs (NFR-4, NFR-5).
- **NFR-13 (Tamper-evident retention):** the Report Series and audit logs are retained immutably and verifiably (sequence + integrity check) so a missing or altered artifact is detectable (FR-45). `[ASSUMPTION: retention horizon ≥ project duration + a defined tail; exact horizon TBD — Open Question.]`
- **NFR-14 (Label store):** every Anomaly Flag's disposition is retained as a labeled record for v2 model training (FR-41).

### 10.5 Localization, Accessibility & Delivery
- **NFR-15 (Bilingual):** client-facing artifacts (report, Client App) render in English and Urdu; Operator capture is low-literacy-friendly (icon/photo/voice-first) (FR-4, FR-42).
- **NFR-16 (WhatsApp as delivery channel):** the Weekly Report is delivered as a self-contained PDF over WhatsApp that works without signal after receipt (FR-43). *(Two-way WhatsApp Business integration is out of scope — §5.)*
- **NFR-17 (No competitive leakage):** client-facing surfaces never expose supplier rates or Company margins, by construction (FR-44, NFR-5).

### 10.6 Security & Reliability
- **NFR-18 (Least privilege):** access is role-scoped (FR-2); actions outside a role's grant are impossible, not merely discouraged.
- **NFR-19 (Capture durability):** an offline capture is never silently lost; local persistence survives app restart and syncs on reconnect (FR-10).
- **NFR-20 (Auditability):** any figure of record (progress, balance, a billed CO) is traceable to the Captures and approvals that produced it.

## 11. Business Model

*(Working hypothesis carried from the brief — to be validated against market/pricing research and the pilot. Tagged `[ASSUMPTION]` where unvalidated.)*

- **Pricing model:** a **per-active-Project monthly subscription**, tiered at the Company level by number of concurrent Projects. Value scales with Projects (each running Project generates the weekly reports and the leakage-catching), and per-Project deliberately **avoids per-seat pricing** — which would fight the "app mandatory for every role including Storekeeper" requirement (FR-2, FR-4).
- **Who pays:** the Company Owner/Director — the same person the internal-control pitch targets. **Buyer, champion, and beneficiary are one person.**
- **Pricing power / ROI framing:** frame ROI on the **documented ~35% average cost overrun**, not on "pilferage" (theft is not quantified in the literature — Risk). "Reduce a 35% overrun" is defensible; "catch pilferage" is a hypothesis. Plus the softer "win one hesitant client."
- **Pricing anchor:** the in-market reference is **Smart Construction at ~PKR 3,000/mo** — Contax prices into an existing cheap anchor, not a blank market. `[ASSUMPTION: exact PKR price points, Pakistani SMB-contractor willingness-to-pay, and construction-company margins are unvalidated — Open Question.]`
- **Model shape:** emerging-market SMBs show low SaaS willingness-to-pay (they can revert to running without software), which pushes toward a **flat low-PKR subscription possibly paired with a transaction/commission hook** rather than pure per-seat SaaS. `[ASSUMPTION]`
- **Parked option — client-pays:** because the Client (often an overseas Pakistani with hard-currency, higher WTP) is a distinct payer from the local Contractor, Contax **could** charge the Client for the transparency guarantee. Worth testing against the company-first buyer model; **not built in v1** (§5, §7.2).
- **Revenue evolution:** subscription is the **wedge** revenue; the parked platform plays (procurement brokerage take-rate, finance-referral revenue-share, verified-supplier marketplace, premium benchmarking tier, AI BOQ auto-estimation, verified-contractor certification — detailed in the Addendum) are the **scale** revenue, enabled by the extensibility seam (NFR-3). Pricing shape mirrors the moat: SaaS entry, data/platform economics later.

## 12. Competitive Position

*(Grounded in the brief's July-2026 market research; sourced detail in the Addendum.)* The mechanics are table-stakes; the differentiation is **positioning and, later, data.**
- **Smart Construction (Pakistan, ~PKR 3,000/mo)** — closest and most dangerous: contractor ERP with WhatsApp automation, an AI Copilot, and branded client PDFs. Sold *to the contractor* as operations software; one product decision from copying the visible transparency skin. Contax's bet: they **won't follow into adversarial anti-fraud depth** because it conflicts with their whole-company buyer.
- **EZYPRO / EzyPMP (Lahore)** — a second capable local incumbent; contractor mid-market/enterprise PM/ERP serving institutional projects (ADB, China State Construction). Adjacent, not direct — no client-facing anti-fraud layer, wrong wedge.
- **Buildertrend / CoConstruct / Procore / Powerplay / Onsite** — global and Indian contractor-productivity plays with builder-controlled client portals.
- **The primary moat is anti-fraud depth via buyer conflict** (NFR-8..NFR-11, §9): an incumbent ERP adopted by the whole company cannot build genuine surveillance into a tool whose adoption depends on the surveilled; Contax's owner-mandated, adversarial-by-construction design can. **The durable second moat is data** (§13), which compounds only at scale — a year-two+ reinforcement, not a day-one defense. **The live threat this creates is the single assumption in §13.**
- **The self-collapse risk (honest caveat):** the positioning moat is real only while Contax stays pointed at the Owner's control over the surveilled. If it is sold and adopted as a *voluntary contractor convenience*, the adversarial framing collapses into "just a nicer client portal" — exactly what Buildertrend already is. The owner-mandate (R-1) and the adversarial-by-construction NFRs are what keep it from degrading into that; a GTM that soft-pedals the mandate is the way this moat is lost from the inside.

## 13. Assumptions Index & Core Risks

### 13.1 Assumptions Index
*(Every inline `[ASSUMPTION]`, surfaced for confirmation.)*
- §4.1 FR-4 — the low-literacy confirmation UI uses picture+number chips (exact iconography is a UX decision).
- §4.2 FR-12 — v1 ships the three canonical captures; the registry is the extensibility seam, not additional v1 capture types.
- §4.3 FR-17 — CV Shadow-Mode surfaces output to Management only, as a discrepancy signal.
- §4.4 FR-22 — default cash-heavy threshold is a configurable % of project outflow (default TBD).
- §4.6 FR-31 — the "no silent yes above Tier C" floor is a hard guardrail, not a configurable cell (confirm).
- §4.7 FR-38 — Company-history import is best-effort/normalized, not a strict schema.
- §4.8 FR-45 — "tamper-evident" = immutable archive + verifiable sequence/hash chain (mechanism is an architecture decision).
- §4.8 FR-47 — default cadence is weekly on a Company-configurable day.
- §4.2 FR-6 — on-device draft OCR is best-effort/optional; server-side OCR on sync is authoritative (offline/online timing resolution).
- §4.8 FR-46 — negative-confirmation liveness signal depends on the WhatsApp delivery path; a periodic positive check-in substitutes where read receipts are unavailable (Open Q16).
- §4.10 FR-51 — on-device integrity checks are best-effort; authoritative checks re-run server-side.
- §4.11 FR-52 — low-confidence (esp. Punjabi) ASR transcripts are flagged for human confirmation.
- §10.4 NFR-13 — retention horizon ≥ project duration + a defined tail (exact horizon TBD).
- §11 — exact PKR pricing, SMB WTP, and Company margins unvalidated; flat-subscription-plus-commission shape is hypothesis; client-pays is a parked option.

### 13.2 Core Risks
- **R-1 — The mandate / data-entry paradox (the make-or-break):** the people who must capture on site are the same people doing the cheating. The whole model rests on the **Owner's top-down mandate carrying honest capture** where an incumbent's voluntary buy-in cannot. If mandate does not drive honest capture, the moat and the product fail together. *This is the single assumption most worth testing first — the pilot's primary job (SM-1).* Mitigations already built into the design: anchor on paper that doesn't lie (challan/supplier statement/photos/client payments), role-separated double entry, append-only + backdating flags, even-fraudulent-data-has-value, the client as free auditor.
- **R-2 — No committed pilot customer yet.** The concept is well-reasoned but only lightly validated (some real owner/site signal, no signed pilot). Securing a first pilot Company is the highest-leverage next step; targets in §8 must be calibrated with them.
- **R-3 — CV progress-% reliability.** Kept as a v1 MUST against the research's advice; hard dependency on the BOQ-derived fallback as the number of record and Shadow-Mode until earned (FR-17). *(Owner's examined call.)*
- **R-4 — Competitive fast-follow.** Smart Construction is one product decision from a client-transparency layer; defensibility rests on speed, anti-fraud depth, and dataset head start — not features (§12).
- **R-5 — Urdu quality.** No internal Urdu eval set exists yet for narrative and ASR; no small open model is reliably fluent in long-form Urdu (NFR-7). Launch may lean on the redacted API tier for Urdu.
- **R-6 — Benchmark data sourcing.** The moat rests on obtaining MES Analysis of Rates and provincial MRS/CSR coefficients; current editions and per-province granularity are unverified (FR-39, Addendum).
- **R-7 — Source B collection friction.** Source B is a v1 MUST but the collection workflow (who obtains/scans the periodic statement, how often, supplier refusal) is undefined and must be designed (FR-24..FR-26).
- **R-8 — Two premise claims not to overstate:** material *theft* as a quantified problem, and overseas-funded home-construction PKR volume. Both unverified; lead with documented cost-overrun pain (§6, §11).
- **R-9 — Uninstrumented leak points: consumption/material-OUT and labour (finalize reviewer, CRITICAL).** The Capture Layer records material-*in*, money, and photos, but there is no material-*out* / physical stock-take capture and no labour capture. Material diverted *between* a clean receipt and its use, and ghost-labour, are not directly instrumented — yet the Owner JTBD (§2.1) promises catching "ghost labour" and pilferage. v1 detection relies on BOQ-consumption *variance* (FR-15) and rate/consumption anomalies (FR-37, FR-39) as an *indirect* proxy, not a direct control. **RESOLVED (Owner, finalize): v1 stays indirect-only and keeps the three-capture spine; the Owner JTBD (§2.1) and Non-Goals (§5) are trued-up to say v1 detects these leak points *indirectly*, and a direct material-out/stock-take capture and labour capture are v2 candidates. This closes the scope-honesty gap; the residual book-vs-physical-stock hole is an accepted, documented v1 limitation.**
- **R-10 — Segregation of Duties is enforced identity-to-identity, not human-to-human (anti-fraud, HIGH).** One person holding two logins, or a small firm without enough distinct staff to separate roles, can collapse FR-19/FR-29/FR-30/NFR-9. Mitigation is partial (device/identity signals, the append-only trail still records *which login* acted); the residual is a known limitation of any SoD control at small scale. Surface to the pilot Company; do not claim SoD defeats a determined solo Owner-operator.

## 14. Open Questions

*(Numbered — future tickets / follow-up research, not silent gaps.)*
1. **Pilot company** — who is the first pilot, and what are their concurrent-project count and client mix (informs pricing tier and segment config)?
2. **Pricing** — exact PKR price points and tier boundaries; validate SMB WTP and Company margins; decide flat-vs-commission shape; decide whether to test client-pays (§11).
3. **Cash-heavy threshold default** — the default % of outflow that trips the cash-heavy flag (FR-22).
4. **Report cadence** — confirm weekly + the configurable default day; define the negative-confirmation dispute-window default duration (FR-46, FR-47).
5. **Tier-C floor** — confirm "no negative confirmation above Tier C" is a hard, non-configurable guardrail (FR-31).
6. **Source B workflow** — define who obtains/scans the supplier statement, cadence, and the supplier-refusal path (R-7, FR-24).
7. **Benchmark data** — obtain and verify current MRS/CSR editions and MES Analysis of Rates, with per-province granularity (R-6, FR-39).
8. **Urdu eval set** — build the internal Urdu eval set for narrative and ASR; set the quality bar that gates self-hosted Urdu (NFR-7, R-5).
9. **Punjabi ASR** — set the low-confidence threshold and human-confirmation path for Punjabi transcripts (FR-52).
10. **Retention horizon** — define the immutable-retention horizon for the Report Series and audit logs (NFR-13).
11. **Tamper-evidence mechanism** — decide the sequence/hash-chain approach for the Report Series (FR-45) — architecture handoff.
12. **CV shadow-mode exit criteria** — define what "demonstrably earns trust" means before CV progress could ever become client-facing (FR-17, RR-2).
13. **Overseas-construction volume** — verify (or continue not to cite) the PKR volume of overseas-funded home construction (R-8, §6).
14. **Storekeeper-flow UX spec** — owner and timeline for the capture-flow specification that gates FR-6 build; confirm it fits (or how it fits) the pilot schedule (RR-3, R-1). Handoff: `bmad-ux`.
15. **Owner sign-off on the CV re-file** — confirm the Owner accepts moving CV progress-% from critical-path MUST to shadow research spike (re-filed, not deleted) (RR-2).
16. **Negative-confirmation liveness signal** — define the channel-liveness signal that gates auto-accept (WhatsApp delivery/read receipt vs a periodic positive check-in), given two-way WhatsApp Business is out of v1 scope (FR-46/RR-6).
17. **Role → permission / authority matrix** — the PRD states role rules in prose (capture rights, sign-off, disclosure authority, tier authority) but has no consolidated matrix; produce one for architecture/UX (FR-2, FR-19, FR-31, FR-53). Handoff: `bmad-architecture` / `bmad-ux`.
18. **Data-Credibility Score model** — define the score's scale, inputs, weighting (esp. the independence weighting from RR-5), and bounds; it is a first-class queryable attribute with no defined model yet (FR-25, FR-35).
19. **Anomaly-rule threshold defaults** — set the day-one constants the rule engine needs: "payment with no Challan/photo within *N* days," "rate exceeds Rate Book by *X*%," and the CO/rate "configurable margin" (FR-34, FR-36, FR-39). These are pilot-calibratable but must ship with defaults.
20. **Consumption / labour instrumentation decision (R-9) — RESOLVED (finalize):** v1 stays indirect-only, three-capture spine kept; JTBD (§2.1) and Non-Goals (§5) trued-up; direct stock-take + labour capture are v2. Residual book-vs-physical-stock hole accepted as a documented v1 limitation.

## 15. Review Register (Party-Mode Round-Table)

*A structured record of a multi-perspective review of this PRD by the software-development team (PM, Architect, UX, Analyst, Dev, Test Architect, Tech Writer). Its purpose is to move the PRD forward **without laundering disagreement into false consensus**: every contested call carries an explicit state and a paper trail, mirroring the append-only, dissent-preserving discipline of the product itself. This register is for later analysis and feedback; conditional items are not yet closed.*

### 15.1 Acceptance mechanism
Every contested decision is assigned one of three states, never a silent verdict:
- **Accepted** — folded into the PRD as-is; no outstanding condition.
- **Accepted-with-condition** — folded in, but gated on a named precondition with an owner; not "done" until the condition is met.
- **Registered dissent** — a reviewer still disagrees; recorded with who, the rationale, and the **revisit trigger** that would re-open it. The PRD proceeds anyway.

### 15.2 Decision register

| ID | Decision | State | Condition / owner | Where applied |
|----|----------|-------|-------------------|---------------|
| **RR-1** | Source B (supplier statement) is **promoted** from a credibility bonus to the **measuring instrument for SM-1's denominator**: capture-completeness cannot be self-measured by the capture layer (a missed delivery is invisible to the system that missed it), so SM-1 is measured as the gap between captured Challans and the supplier's independent count. | **Accepted** (unanimous) | — | SM-1 (§8), §4.5, FR-24..FR-26 |
| **RR-2** | CV progress-% (FR-17) **re-filed** from critical-path v1 MUST → **v1 Shadow research spike** (Management-only, ships nothing client-facing, BOQ-% remains the number of record). Off the pilot's critical path; not build-blocking. | **Accepted-with-condition** | **Owner sign-off** — this revises a deliberate owner decision; it is *re-filed, not deleted*. Exit criteria required before CV is ever client-facing (Open Q12). Owner: Owner/PM. | §7.1, FR-17, Open Q12/Q15 |
| **RR-3** | A dedicated **storekeeper-flow UX specification** is a **precondition on building FR-6** (the capture layer), because the storekeeper's abandonment is the failure mode behind the make-or-break Risk R-1. | **Accepted-with-condition** + **Registered dissent** | Condition: UX spec exists before FR-6 build (owner: UX / `bmad-ux`). Dissent (John/PM): a rougher flow could ship and iterate on-site. | FR-4 build-gate note, Open Q14 |
| **RR-4** | The v1 pilot **infrastructure is heavy** relative to what the pilot is testing (self-hosted GPU stack, offline sync, 3-layer anomaly engine, redaction proxy). Not blocked — the Owner made the scope call knowingly. | **Registered dissent** (Winston/Architect) | Revisit trigger: **post-pilot, before scaling to customer #2**, run a "what did we actually need" stack audit. | §7.1 scope, NFR-1..NFR-7 |

*The entries below (RR-5..RR-7) came from the **Finalize reviewer gate** (rubric walker + anti-fraud + downstream reviewers, 2026-07-25). They close loopholes the PRD's own principles (§9) require closed, so they were applied directly as hardening rather than re-opened as decisions — full reviews in `review-antifraud.md`, `review-downstream.md`, `review-rubric.md`.*

| ID | Decision | State | Condition / owner | Where applied |
|----|----------|-------|-------------------|---------------|
| **RR-5** | Data-Credibility Score requires **source independence**, not mere agreement — two captures controlled by the same actor do not raise the score (else a single fraudster manufactures maximally-credible fraud). | **Accepted** (hardening) | Score model incl. independence weighting to be defined — Open Q18. | FR-35 |
| **RR-6** | Negative Confirmation may not auto-accept on a **dead channel**; sustained silence with no liveness signal escalates rather than defaulting to "accepted." | **Accepted** (hardening) | Liveness-signal definition — Open Q16. | FR-46 |
| **RR-7** | Because Source B is the SM-1 denominator (RR-1), it must be **collected through a channel the surveilled Operator does not control**, and *persistent* Source B absence is a flagged un-measurability state, not a neutral gap. | **Accepted** (hardening) | Collection workflow — Open Q6 (now pilot-critical). | FR-24, FR-26 |

### 15.3 Registered dissent (standing)
- **D-1 (John / PM) — scope weight.** 57 FRs against zero signed pilots reads as over-scoped; John accepts the Owner's examined big-scope call but keeps a standing reservation. *Revisit trigger:* post-pilot scope retrospective — which MUSTs actually earned their place.
- **D-2 (John / PM) — storekeeper flow could iterate.** Believes RR-3's build-gate is stronger than necessary and a rough flow could ship. *Revisit trigger:* if the UX spec threatens the pilot timeline, the iterate-in-place path returns to the table.
- **D-3 (Winston / Architect) — pilot infrastructure weight** (see RR-4). *Revisit trigger:* pre-customer-#2 stack audit.

### 15.4 Points of consensus (no dissent)
- The **Glossary and global FR numbering hold** — every decision above can be renegotiated without the document's structure failing (Paige/Tech Writer). The panic is about *decisions*, not the *document*.
- **Source B must stay in v1** — the one point the room reversed itself on and then agreed unanimously (RR-1).

### 15.5 Plan of action (owners + triggers)
1. **Obtain Owner sign-off on RR-2** (CV re-file). Blocks nothing to build; needed to close the condition. → Open Q15.
2. **Commission the storekeeper-flow UX spec** before FR-6 enters build. → `bmad-ux`, Open Q14, RR-3.
3. **Design the Source B collection workflow now, not later** — RR-1 makes it pilot-critical (who obtains/scans the statement, cadence, supplier-refusal path). This was Q6; RR-1 raises its priority. → Open Q6, R-7.
4. **Rewrite/keep SM-1's denominator against Source B** in all downstream measurement plans and the test strategy. → `bmad-tea` / test design, RR-1.
5. **Schedule the post-pilot stack audit** as a named milestone (customer-#2 gate). → RR-4, D-3.
6. **Carry D-1 into the post-pilot scope retrospective** as an explicit agenda item. → D-1.
7. **Resolve the consumption/labour instrumentation decision** before UX/architecture — it changes scope and the honesty of the Owner JTBD. → Open Q20, R-9. *(Surfaced to the Owner at finalize; see §14 Q20.)*
8. **Produce the role→permission/authority matrix**, the Data-Credibility Score model, and the day-one anomaly-rule threshold defaults as the first downstream artifacts. → Open Q17/Q18/Q19; `bmad-architecture`.

### 15.6 Finalize reviewer gate — summary
- **Rubric walker:** Good/Strong (0 critical, 0 high, 5 medium, 4 low). Decision-readiness / substance / strategic coherence / scope honesty / shape fit **strong**; done-ness & downstream usability **adequate**. All medium/low findings resolved above (cross-refs, glossary casing, FR-17 status, thresholds→Open Qs, permission matrix→Open Q17).
- **Input reconciliation (brief + addendum):** high fidelity; residual gaps folded in — dropped "trust-through-structure" invariant restored (§9), self-collapse caveat restored (§12), labour/consumption gap raised to R-9/Q20.
- **Anti-fraud reviewer:** 4 critical + 1 high exploit paths; loophole-class findings hardened directly (RR-5/6/7, R-9, R-10) since they enforce §9's stated principles.
- **Downstream reviewer:** PROCEED-WITH-FIXES; offline-OCR contradiction resolved (FR-6), broken cross-refs fixed, undefined constants → Open Qs.
- Reviews preserved at `review-rubric.md`, `review-antifraud.md`, `review-downstream.md`, `reconcile-brief.md`, `reconcile-addendum.md`.

*Reviewers (round-table): 📋 John (PM) · 🏗️ Winston (Architect) · 🎨 Sally (UX) · 📊 Mary (Analyst) · 💻 Amelia (Dev) · 🧪 Murat (Test Architect) · 📚 Paige (Tech Writer). Finalize gate: rubric walker + anti-fraud + downstream subagents. Review date: 2026-07-25.*
