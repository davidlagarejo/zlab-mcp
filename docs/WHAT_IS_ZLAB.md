# What źlab Operational Decision Stress Test does

A plain-language explanation for people — not for the AI host. (If you are an AI
assistant, read [`MCP_TOOL_CONTRACT_FOR_HOSTS.md`](MCP_TOOL_CONTRACT_FOR_HOSTS.md)
and [`HOST_DOCUMENT_WORKFLOWS.md`](HOST_DOCUMENT_WORKFLOWS.md) instead.)

## The problem it addresses

Decisions about physical assets — a facility retrofit, an equipment
replacement, an energy project, a vendor's savings claim, a value-creation plan
for an acquired company — usually arrive as a confident number. A proposal says
"$1.2M annual savings, 5.7-year payback." A model has a tidy IRR. A committee is
asked to approve.

But that number is only as good as the evidence under it: the baseline load, the
operating hours, the equipment condition, the utility tariff, the assumption that
the vendor's model matches your building. When those are unverified, the
precision is an illusion. Money and engineering time get committed to the wrong
thing — not because anyone lied, but because a plausible framing was treated as a
settled fact too early.

**źlab screens that decision before it advances.** It does not tell you the
answer. It tells you whether the evidence actually supports advancing — and if
not, exactly what is missing and what it would take to know.

## What it actually does

You (through your AI assistant) hand it a decision: the decision in your own
words, the asset (name + street address), the stage you're at, and optionally the
proposed solution and the evidence you already have. It runs a deterministic,
evidence-governed analysis and returns a **governed read** with five parts:

1. **Tension detected** — the assumption or framing that, if wrong, would change
   what you should do next.
2. **Rival frame** — an alternative explanation of the problem that the evidence
   has not yet ruled out. It is always labeled as a *challenger*; it never
   silently replaces the way you framed the decision.
3. **Next step** — the single measurement or piece of evidence that would
   discriminate the most, ranked by value.
4. **Posture** — the bounded stance the evidence supports: **advance, condition,
   compare, measure, defer, or stop.** This is a permission state, not a
   recommendation.
5. **Limits** — the evidence ceiling (how strong any claim in the case is allowed
   to be), the specific claim that is currently blocked, and the honest maturity
   grade of the whole read.

Three things it deliberately will **not** do:

- It never issues a categorical GO / NO-GO. The result is evidence-bounded.
- It withholds financial magnitudes (savings, ROI, NPV, payback) until the
  evidence reaches decision grade. It will name the *type* of exposure, never a
  fabricated number.
- It never invents the facts you didn't give it. Missing evidence is reported as
  missing.

## A concrete example

A hospital is weighing a **$6.8M CHP (combined heat and power) retrofit**. The
vendor's model shows $1.2M/year savings and a 5.7-year payback. The proposal goes
to the CAPEX committee.

Your assistant sends the decision to źlab. The governed read comes back roughly
like this:

> **Posture:** NO-GO *as posed* — 1 decision lane allowed, 5 deferred.
> **Tension:** the visible "high energy cost" framing may be premature; dominant
> variables (baseline steam load, operating regime) are not yet bounded.
> **Next step:** confirm the asset-identity and baseline record — it is the
> bottleneck blocking five downstream decisions.
> **Limits:** evidence ceiling is below decision grade; the $1.2M savings figure
> is **withheld** pending site-level evidence. Grade: preliminary.

What your assistant does with that: it does **not** propagate the $1.2M into the
model as if it were real. It flags the payback cell as unverified, records the one
allowed action (gather the baseline evidence), and tells you precisely what to
collect. That is the point — you learn the number isn't trustworthy *yet*, and
you learn exactly how to make it trustworthy, before the committee commits.

## The evidence loop (why it gets more useful, not less)

A "no" from źlab is not a dead end — it's a work list. Each result names
`what_would_lift_it`: the evidence that would raise the ceiling. Your assistant
gathers exactly those items (asks you, reads the open document, or pulls public
records) and resubmits the *same* decision with the evidence attached. The
ceiling and the permissions move visibly between runs. You watch a decision
mature from "structurally plausible" toward "decision grade" — with a record of
what changed and why.

## When to use it — and when not to

**Use it** when a material decision about a physical asset or operational system
could advance despite unresolved evidence, assumptions, dependencies, or a
plausible alternative explanation: retrofit and equipment-replacement proposals,
energy and utility projects competing for a budget, vendor/audit savings claims,
operational value-creation assumptions inside an acquisition model.

**Don't use it** for simple arithmetic, finding a value in a document, rewriting
a memo, general education, low-stakes maintenance, settled execution, or purely
financial questions with no physical system underneath. It governs *operational
evidence*; it does not replace search, spreadsheets, calculators, engineering
simulation, measurement, or human approval.

## Free read vs. paid Decision Case

The **free read** (what this connector returns) tells you *what* could change the
decision: the top tension, the rival frame, the limiting evidence, the highest-
value next step, and the bounded posture — under full governance.

The **paid Decision Case** demonstrates *why*, *how much* it could change the
decision, *what evidence* would resolve it, and *what action* becomes
legitimately enabled: the full claim/source/dependency map, the maturity of every
piece of evidence, governed financial scenarios once the ceiling permits, the
structured comparison against rival interventions, and an auditable history that
updates as new evidence arrives. Same governance in both — paying buys
completeness and evolution, never a different truth.

## What it is not

Not engineering advice. Not a professional-engineer certification. Not
investment, legal, or financial advice. It records what the submitted evidence
permits and prohibits asserting; it does not recommend actions, and no third
party should rely on it as a substitute for site-specific professional judgment.

## How it works, and where the line is

The analysis is produced by a deterministic, evidence-governed framework running
server-side. This repository distributes the connector and its host-facing
documentation; the analysis engine itself is not open source. What every result
*must* and *must not* say is fixed by contract — see
[`MCP_TOOL_CONTRACT_FOR_HOSTS.md`](MCP_TOOL_CONTRACT_FOR_HOSTS.md) for the response
contract and [privacy](https://zircular.io/privacy) for what leaves the machine
and what is stored.

---

Built by [źlab](https://zircular.io) (Zircular LLC) — decision governance for
physical assets. Questions: davidl@zircular.io
