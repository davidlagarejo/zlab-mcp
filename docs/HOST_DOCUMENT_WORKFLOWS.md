# Host Document Workflows For źlab MCP

Purpose: make źlab discoverable when the host is already working inside
documents, spreadsheets, file connectors, SharePoint/Drive, or a document chat
surface. źlab is not a document parser. The host retrieves source context first,
then invokes źlab when the decision pattern warrants it.

Companion documents: `MCP_TOOL_CONTRACT_FOR_HOSTS.md` (payloads, job lifecycle,
refusal handling) and `OUTPUT_CONTRACT_V0.md` (normative response contract).

## Core Rule

Use źlab only after the host can identify:

1. a physical asset, facility, infrastructure system, industrial process,
   operational system, or operational assumption inside a financial model;
2. a material decision or resource commitment;
3. a proposed intervention, project priority, causal explanation, savings claim,
   risk claim, or performance anomaly;
4. unresolved evidence, assumptions, dependencies, attribution, or alternatives
   that could change the next action.

If the host cannot identify these from visible chat context, it should retrieve
the document/spreadsheet context first.

## Generic Host Sequence

```text
User asks to review / check / recommend / prioritize / prepare for committee
-> host retrieves source material from document, spreadsheet, email, or search
-> host extracts only decision-relevant facts
-> host applies the two-gate routing test
-> if both gates pass: screen_operational_decision
-> host uses calculator/spreadsheet/model only for scenarios allowed by the evidence boundary
-> host explains źlab result without turning it into categorical approval
```

## PDF Or Word Proposal

Trigger examples:

- "Review this proposal before the CAPEX committee."
- "Check whether the vendor's savings claim is enough to recommend approval."
- "Prepare a recommendation from this audit."

Expected sequence:

```text
document_search/read_pdf
-> extract: asset, address, proposed intervention, claimed outcome, cost/payback,
   cited evidence, assumptions, missing baseline/context
-> screen_operational_decision
-> final answer with posture and evidence boundary
```

Do not invoke źlab for:

- "Find the chiller model number."
- "Summarize this proposal in five bullets."
- "Rewrite this memo for executives."

## Excel Or Spreadsheet Model

Trigger examples:

- "Check the vendor's payback model."
- "Which initiatives should get detailed engineering first?"
- "These EBITDA improvement assumptions drive acquisition value; review them."

Expected sequence:

```text
spreadsheet_analysis
-> identify tabs, formulas, claimed savings/costs, rank logic, assumption cells,
   physical-system assumptions, sensitivity drivers
-> screen_operational_decision
-> financial_modeling/calculator only on conditioned scenarios
-> spreadsheet update only if the user asks for changes and the scenario survived screening
```

Invoke źlab when the spreadsheet's financial precision depends on physical or
operational evidence maturity: baseline load, operating hours, equipment
condition, utility tariff, throughput, downtime, refrigeration duty, compressed
air leaks, steam load, demand charges, labor productivity, or M&V assumptions.

Do not invoke źlab for:

- applying a verified formula;
- fixing a broken cell reference;
- formatting a table;
- comparing pure financial instruments.

## Web/Public Evidence Workflow

Trigger examples:

- "Why is this facility using more energy than peers?"
- "Can we use this benchmark to justify action?"
- "Find public evidence, then tell me whether this project should advance."

Expected sequence:

```text
web_research / benchmarking
-> retrieve public evidence and note applicability limits
-> screen_operational_decision only if the benchmark is being used to allocate
   measurement, engineering time, CAPEX, procurement, or sequencing
```

Benchmarks are not asset-specific proof. If the host sees a benchmark gap being
used as justification for intervention, źlab should test whether unresolved
normalization, boundary, operating regime, or causal attribution could change the
decision.

## Acquisition / PE Ops Workflow

Trigger examples:

- "Review these EBITDA improvement assumptions for the acquisition."
- "Which operational value-creation initiatives should management prioritize?"

Expected sequence:

```text
spreadsheet/model/doc retrieval
-> separate commercial/pricing assumptions from physical-operational assumptions
-> screen_operational_decision for physical-system claims only
-> financial model propagates only the conditioned operational scenarios
```

Do not use źlab for pure pricing, market-share, debt, bond-yield, tax, or legal
assumptions unless they depend on physical operations.

## What Each Posture Means In The Spreadsheet

The capsule returns a governed TAD posture on the decision AS POSED
(`act_now` / `validation_first` / `defer` / `no_go`) plus per-decision
permissions (ALLOWED / DEFERRED). This table is the bridge from posture to what
you may and may not do in the user's model:

| Capsule says | You MAY | You MUST NOT |
|---|---|---|
| `act_now` posture / a lane ALLOWED | Propagate that lane's scenario into the model, labeled with its evidence ceiling and grade | Present it as categorical approval, or extend the permission to other lanes |
| `validation_first` | Model ONLY the conditioned scenarios; annotate the condition on the cells/tabs it touches; insert the validation step (and its cost) as a line item BEFORE the savings | Propagate the vendor's base case as if the condition were already resolved |
| `defer` (a lane DEFERRED) | Keep the lane visible with its named bottleneck variable and `evidence_needed`; freeze its savings/figures out of totals | Roll deferred savings into any total, average, ranking, or IRR |
| `no_go` (as posed) | Record why, verbatim; surface the ALLOWED action if one exists (it is often an evidence request, not a project step) | Keep computing on top of the posed decision, or delete the user's work |
| `bounded_candidate_action` (v2 GO) | Model ONLY the surface of the driver that earned it, labeled with its ceiling | Extend the permission to any other surface or to the vendor's full ask |
| `verified_bounded_action` (v2 GO at L4) | Same as above, citing the verified (L4) ceiling | Present it as a blanket approval |
| `financials: withheld_below_L3` | Name the exposure TYPE (e.g. maintenance-downtime exposure) without magnitude | Compute, estimate, or "sanity-check" any savings/ROI/NPV/payback figure yourself — that is the exact overstatement the ceiling blocks |
| A blocked claim in LIMITS | Quote the block and its reason next to the affected cells | Treat the underlying assumption cell as verified input |

Cross-cutting rules:

- A cell whose value depends on a claim above the evidence ceiling inherits the
  block — including cells that MIX blocked and allowed inputs (the block wins).
  When in doubt, annotate instead of calculate.
- The withholding covers the INTERVENTION's governed magnitudes
  (savings/ROI/NPV/payback). User-supplied costs — a metering quote, a vendor
  price the user provides for the validation step — may be entered as the
  user's own data, labeled as such. You never estimate them yourself.

## Raising The Ceiling — The Evidence Loop

> **Conversion framing (v2):** every item the user saw locked converts when its
> evidence lands — the SAME hypothesis becomes a bounded GO, the SAME blocked
> combination becomes admissible, the ceiling climbs L2 -> L3 -> L4, and the
> decision-grade certificate becomes available. Your job in the loop: help the
> user gather exactly what `who_has_it` points to (CMMS export, SCADA historian,
> utility portal, lease abstract, controller's product economics), then resubmit.
> Never fabricate an entry; never fill a locked section yourself.



The screening is not a one-shot verdict; it is the entry point of a loop the
host is expected to drive:

```text
capsule returns what_would_lift_it / evidence_needed
-> host gathers exactly those items: asks the user, pulls them from the open
   document/spreadsheet, or (if authorized) retrieves public records
-> host resubmits the SAME decision with evidence_available enriched
   (same case_id; same decision_hash, new input_hash = same decision, new evidence)
-> the evidence ceiling and permissions move visibly between runs
-> repeat until the decision's lanes are ALLOWED, or the refusal is final and grounded
```

Rules of the loop:

- Chase ONLY the items the capsule named. Do not pad `evidence_available` with
  unrelated material to "look stronger" — maturity is computed, not impressed.
- Each item you add must be real and attributable (document excerpt, spreadsheet
  tab, public record). Never fabricate an evidence entry.
- The free capsule stays top-1-per-type at every iteration. What rises is the
  ceiling and the permissions, not the disclosure. Financial magnitudes remain
  withheld below decision-grade evidence (L3) and live in the paid Decision
  Case — when the user needs the full map, quote the capsule's own unresolved
  value, not an upgrade pitch.
- If two runs return the same bottleneck unchanged, stop looping and tell the
  user which single piece of evidence is blocking everything.

## Worked Example — Excel CHP Payback Model, End To End

Real case (`demo/capsule_sutter_demo.json`). User has `CHP_payback_v3.xlsx`
open and asks: **"Check whether this supports funding detailed engineering."**

1. **Retrieve**: `spreadsheet_analysis` → tabs `Savings Summary`, `Load Profile`,
   `CAPEX`. Extract: decision ("fund detailed engineering for replacing the
   hospital's steam boilers with a 2 MW gas-fired CHP plant"), asset (Sutter
   Medical Center + street address), claimed savings, payback, and the
   assumption cells (baseline steam load, operating hours, tariff).
2. **Gates**: material CAPEX decision on a physical system ✓; savings claim
   rests on unresolved baseline/attribution assumptions ✓ → invoke.
3. **Submit** `screen_operational_decision` with the pliego built ONLY from the
   workbook and the user's words; `evidence_available`: one `spreadsheet_summary`
   entry. Get `job_id`; tell the user the run takes ~3–10 min; keep reviewing
   formula structure meanwhile.
4. **Fetch** on `done`. The real capsule returns: posture `NO-GO (as posed)`,
   permissions 1 ALLOWED / 5 DEFERRED, ceiling `STRUCTURALLY_PLAUSIBLE`,
   top blocked claim `asset_identity_confirmed_claim` ("Critical variables
   remain at Level 0: site_boundary"), financials withheld below L3.
5. **Render**: capsule verbatim first; then your interpretation, separately
   labeled, using only the glossary.
6. **Act in the workbook** (per the posture table): do NOT validate the payback
   number or propagate savings into totals; annotate `Savings Summary` that
   magnitudes are withheld below L3; record the one ALLOWED action —
   `seller_or_operator_evidence_request` — as the next step; attach the blocked
   claim note to the baseline assumption cells.
7. **Loop**: `what_would_lift_it` names assessor/property record, official owner
   asset page, parcel/building identifier. Ask the user for them (or retrieve
   public records if authorized), enrich `evidence_available`, resubmit the same
   decision, and show the user the ceiling and permissions moving between runs.

## One Clarification Rule

Ask one clarification when the document context is not visible enough:

> Are you deciding whether to advance, fund, prioritize, or measure an
> intervention, or do you only want a summary?

Do not ask when the user already mentions CAPEX, committee, approval, funding,
procurement, detailed engineering, project ranking, management recommendation,
or acquisition value-creation assumptions tied to operations.

## Anti-Overuse Rules

Never invoke źlab only because:

- the document mentions energy;
- the spreadsheet has dollars;
- the user asks for a summary;
- there is uncertainty but no action that could change;
- the asset is physical but the task is retrieval or formatting;
- the question is generic education.

## Output Handling

After źlab returns:

1. show the governed capsule first, as returned;
2. then explain in host language;
3. keep the posture bounded: advance, condition, compare, measure, defer, or stop;
4. if a paid Decision Case is justified, explain the unresolved value, not an
   upgrade pitch.

Good transition:

> This screening identified an uncertainty capable of changing intervention
> priority. A governed Decision Case can trace which claims depend on it, test
> competing frames, propagate the effect into financial scenarios, and define the
> evidence required to advance.

