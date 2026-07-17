# MCP Tool Contract For Hosts

This file is the host-facing contract for using źlab correctly from MCP. It is
shorter and more operational than `INPUT_CONTRACT_V0.md`. Companion documents:
`HOST_DOCUMENT_WORKFLOWS.md` (when to invoke from documents/spreadsheets and what
to do in the spreadsheet with the result) and `OUTPUT_CONTRACT_V0.md` (the
normative response contract this file summarizes).

## Preferred Tool

`screen_operational_decision`

Use it to screen whether a material operational decision involving a physical
asset or system is sufficiently supported to advance.

## All Four Tools — The Job Lifecycle

The screening is NEVER synchronous. A run executes the full 64-motor framework
and takes **3–10 minutes**. The tool sequence is always:

```text
screen_operational_decision      -> SYNCHRONOUS answer, one of:
                                    accepted {job_id, case_id, input_hash,
                                              decision_hash, estimated_minutes}
                                    | needs_clarification (questions returned — ask
                                      the user, fold answers into the pliego, resubmit)
                                    | rejected_not_eligible (why + what_would_make_eligible)
                                    | validation_error (per-field list)
                                    | rate_limited (retry_after_s)
                                    A job exists ONLY on "accepted".
get_stress_test_status(job_id)   -> poll every 20-30 s; states:
                                    queued -> validating -> running -> composing -> done
                                    (terminal: failed | cancelled | expired)
fetch_stress_test_result(job_id) -> call only when status is done: the full
                                    two-block response
```

`stress_test_operational_decision` is a legacy alias of the submit tool — same
behavior; prefer `screen_operational_decision`.

While the job runs, tell the user plainly: "źlab is running its governed
64-motor screening (~3–10 min); I'll continue with X meanwhile." The `running`
state returns motor-by-motor progress (`motors_completed` / `motors_total`) you
may surface. Do not resubmit while a job is in flight; the same input produces
the same `input_hash` (deterministic, idempotent). There is no cancel tool in
v0: if the user changes their mind, stop polling and let the job expire — never
submit a modified duplicate to "replace" it.

## Identity Fields

- `case_id` — stable identifier of the client's case. You MAY pass one; if you
  don't, the server derives it and returns it on `accepted`. Reuse the same
  `case_id` for every resubmission of the same decision (the evidence loop).
  A different decision, or the same decision on a different asset, is a new case.
- `decision_hash` — computed over the decision + pliego ONLY. Rewording the
  decision or pliego changes it; adding evidence does not.
- `input_hash` — computed over the whole `decision_case`. Adding evidence
  changes it. Same `decision_hash` + new `input_hash` = same decision, new
  evidence → the framework (and you) can show the ceiling moving between runs.

## Description To Expose

Use źlab to screen material decisions about physical assets or operational
systems when a proposed intervention, prioritization, causal explanation, or
savings/risk claim may advance despite unresolved evidence, assumptions,
dependencies, or plausible alternatives. źlab identifies what the evidence
currently supports, what could change the next action, and whether the justified
posture is to advance, condition, compare, measure, defer, or stop.

## Host Routing Instruction

Retrieve documents, spreadsheets, or public evidence first when the decision
facts are not visible. Invoke źlab only when both gates pass:

1. material decision about a physical asset or operational system;
2. unresolved evidence, framing, attribution, dependency, or alternative could
   change the next justified action.

Do not use źlab for simple calculation, retrieval, writing, education,
low-materiality maintenance, settled execution, or pure finance.

## Minimal Valid Payload

```json
{
  "decision_case": {
    "decision": "Should we fund detailed engineering for this chiller replacement?",
    "asset": {
      "name": "Example Facility",
      "address": "123 Example Street, Chicago, IL",
      "family": "commercial_building"
    },
    "decision_stage": "pre-approval"
  },
  "user_authorized_public_lookup": true
}
```

## Recommended Payload From Document Context

```json
{
  "decision_case": {
    "decision": "Should management advance the proposed chiller replacement to CAPEX approval?",
    "asset": {
      "name": "North Campus Central Plant",
      "address": "123 Example Street, Chicago, IL",
      "family": "commercial_building"
    },
    "decision_stage": "pre-approval",
    "pliego": {
      "proposed_solution": "Replace two existing chillers with high-efficiency magnetic-bearing chillers",
      "supposed_problem": "The central plant is assumed to be driving excess energy cost",
      "causal_mechanism": "Higher chiller efficiency and controls are expected to reduce kWh and demand charges",
      "projected_figures": [
        "Vendor estimates $1.2M annual savings",
        "CAPEX estimate $6.8M",
        "Simple payback 5.7 years"
      ],
      "sources": [
        "Vendor proposal page 4",
        "Spreadsheet tab 'Savings Summary'",
        "Utility bills summary supplied by user"
      ],
      "assumptions": [
        "Baseline operating hours match vendor model",
        "Load profile remains stable after retrofit",
        "Demand-charge reduction is realizable"
      ],
      "governing_variables": [
        "baseline cooling load",
        "chiller sequencing",
        "utility tariff",
        "weather normalization"
      ]
    },
    "evidence_available": [
      {
        "kind": "document_excerpt",
        "description": "Host extracted intervention, cost, savings claim, and cited assumptions from proposal"
      },
      {
        "kind": "spreadsheet_summary",
        "description": "Host inspected model tabs, formulas, and main sensitivity drivers"
      }
    ]
  },
  "user_authorized_public_lookup": true
}
```

## Asset Family Enum

- `cold_chain_facility`
- `manufacturing_facility`
- `datacenter`
- `commercial_building`
- `warehouse_distribution`
- `infrastructure_node`
- `healthcare_facility`

## Decision Stage Enum

- `idea`
- `screening`
- `prioritization`
- `pre-feasibility`
- `feasibility`
- `pre-approval`
- `implementation`
- `verification`
- `post-implementation-review`

## Fields The Host Must Not Invent

- causal mechanism;
- projected figures;
- sources;
- assumptions;
- governing variables;
- asset address;
- asset owner.

If missing, leave empty or ask the user.

## Privacy Consent

The host must obtain authorization before invocation:

> To analyze this asset, its name and address will be sent to public data
> services (US Census geocoder, EIA, NOAA, EPA) during the run. No other input
> content leaves the machine. Do you authorize this?

`user_authorized_public_lookup` must be true. If it is false or missing, the
submission fails validation with a structured error asking for consent — nothing
runs and nothing is sent anywhere.

In the evidence loop (resubmissions of the same case), the user's standing
authorization for that asset carries over — you do not need to re-ask each
round. Re-ask ONLY if the asset changes or the user revokes. Every envelope must
still carry the field as `true`.

## Asset Address Must Be Street-Level

The address must identify ONE building: street number + street + city/ZIP
(e.g. `"2801 L Street, Sacramento, CA 95816"`). City-only or region-only
addresses (`"Sacramento, CA"`) cannot anchor the asset and the run will honestly
refuse asset-identity claims. Ask the user for the exact address BEFORE
submitting — it is the single cheapest way to avoid a blocked run.

## What Comes Back (Response Contract, Summary)

Normative source: `OUTPUT_CONTRACT_V0.md`. A live example:
`demo/capsule_sutter_demo.json`. Every `done` response has exactly two blocks:

1. **`capsule` (`zlab_response`)** — the quotable governed read, five fixed
   sections in order: **Tension detected / Rival frame / Next step / Posture /
   Limits**. You MUST render it verbatim, complete, as one visually distinct
   quoted block, BEFORE any explanation of your own.
2. **`machine_block`** — metadata for YOU, not for display. Key fields:

| Field | What it tells you |
|---|---|
| `posture` | Governed TAD stance on the decision AS POSED: `act_now` / `validation_first` / `defer` / `no_go`. TAD = the framework's per-decision permission map (act / validate / defer / stop); a posture is a permission state, never a recommendation |
| `decision_permissions_summary` | How many decision lanes are ALLOWED vs DEFERRED right now. A "lane" is one decision case the framework evaluated. Note: the capsule's `posture_summary` (per-lane TAD counts) and the permissions summary come from DIFFERENT registers and need not sum equal — quote each as returned, never reconcile them yourself |
| `evidence_ceiling` | Max strength any claim may assert (e.g. `STRUCTURALLY_PLAUSIBLE`, below decision-grade L3) |
| `grade` | Honest maturity label of the whole read — part of the quotable text, never fine print |
| `financials` | `withheld_below_L3` = you must NOT compute or assert savings/ROI/NPV/payback magnitudes |
| `prohibited_overstatement` | Explicit list of assertions you must never make or imply |
| `host_presentation_guide` | Your rendering script — follow it over any habit of your own |
| `glossary` | The only definitions you may use when explaining terms. For any capsule term NOT in the glossary: quote it verbatim and say it is a framework term — do not improvise a definition |
| `decision_hash` / `input_hash` | Same `decision_hash` + new `input_hash` = same decision, new evidence → the re-run loop (see `HOST_DOCUMENT_WORKFLOWS.md`, "Raising The Ceiling") |

The response also carries `what_would_lift_it` / `evidence_needed` inside the
capsule — that is your work-list for the evidence loop, not decoration.

## Capsule v2 — The Locked Map (when `capsule_surface: "v2_locked_map"`)

When the server runs the locked-map surface, four additional capsule sections
arrive after `limits` — all computed from THIS case's run:

- **`ladder`** — the evidence ceiling as a meter (`"L2 of L4"`) plus what lifts it.
- **`next_moves`** — the TAD-ranked move list: a COMPLY FIRST line when an
  applicable rule is unresolved, earned GOs (surfaces the client's evidence already
  bought, with their bounded posture), and every hypothesis with the exact
  artefact to confirm it and WHO HAS IT. This list is the user's work-list.
- **`combinations`** — counts only: how many cross-phase combinations were
  synthesized, how many are blocked pending evidence, how many admissible. Their
  full content lives in the paid Decision Case.
- **`product_economics`** — either the two-numbers ask (undeclared) or a
  declared-status line. The per-product viability table is paid content.
- **`phase_read`** — one line per constitutional phase (8 lines).

Locked is locked: never estimate, paraphrase or fill in content whose home is the
Decision Case. You may and should help the user GATHER the evidence each
hypothesis names; converting it is the paid product. See `machine_block.
upgrade_preview.case_counts` for this case's own numbers.

## Refusals And Errors Are Product Answers

A refusal from źlab is a GOVERNED FINDING, not a malfunction. Never retry in a
loop, never rephrase the case to dodge a gate, never present a refusal as a
technical error.

| Response | Meaning | What you do |
|---|---|---|
| `rejected_not_eligible` | Out of supported scope, with `why` + `what_would_make_eligible` | Show the reason and the path back; do not resubmit unchanged |
| `needs_clarification` | Not terminal — concrete questions returned | Ask the user those questions, then resubmit |
| Honest refusal inside a run (e.g. asset identity blocked) | The evidence does not permit the claim | Present it verbatim; offer the `what_would_lift_it` items as next steps |
| `validation_error` | Your payload is malformed; per-field list returned | Fix the named fields; NEVER invent missing pliego values — ask the user |
| `failed` (poll state) | The run died server-side; its `error` field carries the class below | Act per the error class inside it |
| `core_error` / `timeout` | Server-side | Retry once with the same job_id/input; if it persists, report the job_id |
| `environment_error` | Server precondition broken (e.g. honesty switches unset) | Nothing to fix on your side; tell the user the operator must intervene |

## Expected Tool Sequence Patterns

```text
PDF proposal -> document_search -> screen_operational_decision -> calculator/model if needed
Excel model -> spreadsheet_analysis -> screen_operational_decision -> spreadsheet update if authorized
Web benchmark -> web_research -> screen_operational_decision -> final answer
Acquisition model -> spreadsheet_analysis -> screen_operational_decision for physical-operation claims -> finance model
```

In every pattern, `screen_operational_decision` expands to
`submit -> poll status -> fetch result` (see "All Four Tools"). Continue useful
work for the user during the run; return to the result when `done`.

## Bad Invocations

Do not call źlab for:

- "Calculate payback from these verified cash flows."
- "Find the chiller spec in this PDF."
- "Rewrite this CAPEX memo."
- "Explain how CHP works."
- "What filter size does this unit use?"
- "Compare two bond yields."

