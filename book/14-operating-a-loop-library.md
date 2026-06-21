# 14. Curating, Operating & Scaling a Loop Library

> **In this chapter:** You have a library that can describe and run loops
> (Chapter 13). This chapter is about keeping it *good* as it grows and as more
> people use it. We cover the contribution workflow that lets a team add loops
> safely, the quality bar and review standards that keep slop out, how to version
> and deprecate entries, the metrics that tell you whether a loop is worth keeping,
> and the governance and security concerns that come with autonomous loops running
> across a team. By the end you will be able to operate a Loop Library as shared
> infrastructure, not just a personal folder of scripts.

## A Library Is a Product, Not a Folder

A handful of loops in a directory is a convenience. A library that a whole team
trusts and runs against real systems is *infrastructure* — and infrastructure needs
an owner, a quality bar, a way to change safely, and a way to measure whether it is
earning its keep. The failure mode of a successful library is not too few loops; it
is too many low-quality ones, so that nobody can tell which entry to trust. Curation
is the work that prevents that.

This chapter assumes the mechanics from Chapter 13 — the schema, the store, the
runner, the CI validator — and adds the *human* layer around them: how loops get in,
how they stay good, and how the whole thing scales past one author.

## The Contribution Workflow

The same discipline you apply *inside* a loop — propose a change, verify it
independently, accept only on a pass — applies to changes *to the library itself*.
Treat a new or modified loop entry like any other code change: a pull request, a
review, and an automated gate.

A workflow that scales from one contributor to many:

1. **Propose via pull request.** A contributor adds or edits an entry file (and any
   referenced prompt, rubric, or skill) on a branch and opens a PR. Nothing merges
   directly to the main library.
2. **Automated gates run first.** CI runs the Chapter 13 validator: schema
   conformance, a present stopping condition, and an existing prompt file. A PR that
   fails these never reaches a human reviewer — the cheap checks go first.
3. **A maintainer reviews against the quality bar.** A human checks what automation
   cannot: is the goal genuinely checkable, is the evaluator independent of the
   generator, are the guardrails honest, is the attribution correct? (The quality
   bar is detailed in the next section.)
4. **Dry-run before acceptance.** For any non-trivial loop, the reviewer runs it
   once through the runner against a safe target and confirms it behaves as
   described — especially that it *stops*. A loop that has never been executed is a
   hypothesis, not a library entry.
5. **Merge promotes it to `stable`.** On acceptance, the entry's `maturity` is set
   to `stable` and the generated index is rebuilt. The loop is now discoverable and
   runnable by the team.

The principle is recursive and worth saying plainly: **a loop library is itself
maintained by a loop** — propose, verify independently, accept only on a pass. The
devil's-advocate and builder-reviewer patterns from Chapter 12 apply as much to
library entries as to the code those entries operate on.

## The Quality Bar

Automated validation proves an entry is *well-formed*. The quality bar proves it is
*good*. Every entry admitted to the library should clear a written, visible
checklist — the standard the library holds itself to, kept in the repo so
contributors can self-check before opening a PR:

- **The goal is externally checkable.** Phrased as an outcome a machine or an
  independent reviewer can confirm — not "improve X." (Chapter 08's goal rule.)
- **The evaluator is independent of the generator.** "Done" never rests on the
  agent's own report; there is a real objective gate, and a rubric gate where
  judgment is needed.
- **Stopping conditions are explicit and layered.** A success exit *and* at least
  one budget cap (iterations and/or cost). An unbounded entry fails the bar
  outright.
- **Isolation and a human-escalation path are defined.** The loop is reversible, and
  it knows when to stop and ask rather than fail silently or burn budget.
- **The blast radius is documented.** The entry states what the loop can touch, what
  permissions it needs, and when *not* to use it.
- **Attribution and source are correct.** Authorship and any upstream source are
  recorded honestly (as in the Chapter 12 catalog).

Make this checklist a PR template so every contribution is reviewed against the same
criteria. A loop that cannot clear the bar is not rejected forever — it is admitted
as `experimental` or `draft` (see versioning below), clearly marked so users know it
has not earned full trust yet.

## Versioning and Deprecation

Loops change: a prompt improves, an evaluator tightens, a better approach
supersedes an old one. Because each entry carries a semantic `version` and a
`maturity` field (Chapter 13), the library can evolve without breaking the people
who depend on it.

- **Version every meaningful change.** Bump the entry's `version` when the prompt,
  evaluator, or stopping conditions change in a way that alters behavior. Callers
  that pin a version get reproducibility; callers that track latest get
  improvements.
- **Use the maturity ladder.** New entries enter as `draft` or `experimental`,
  graduate to `stable` once proven, and end as `deprecated` when superseded.
  `maturity` is a first-class search facet, so users can choose to see only `stable`
  loops by default.
- **Deprecate, don't delete.** When a loop is replaced, mark it `deprecated`, point
  to its successor in the entry, and keep it for one release cycle. Deleting an entry
  silently breaks anything that referenced it; deprecating it gives users a migration
  path. (This is the *propagation compliance* discipline from Chapter 12 applied to
  the library itself.)

The maturity ladder is what lets a library be *welcoming to new loops* without being
*reckless with trust*: anyone can contribute a `draft`, but only proven loops carry
the `stable` badge that signals "run this against production."

## Measuring Whether a Loop Earns Its Place

A loop in a library is a claim that it is worth running. Once loops run regularly,
you can replace that claim with evidence. Because the runner already drives every
loop, instrument it once and you get telemetry for the whole library for free.

The metrics worth tracking per loop:

- **Acceptance rate.** Of runs that reached a stopping condition, how many ended in
  *success* versus hitting a budget cap or escalating? A loop that almost never
  reaches its success exit has a goal or evaluator problem.
- **Iterations and cost to success.** The average budget a successful run consumes.
  Rising cost over time is an early warning that the loop or the codebase it targets
  has drifted.
- **Escalation rate.** How often the loop hands off to a human. Some escalation is
  healthy; constant escalation means the loop is mis-scoped.
- **Usage.** How often the loop is run at all. An entry nobody runs is
  documentation, not infrastructure — a candidate for deprecation.

These tie directly back to the stopping conditions from Chapter 08: the same exits
that *bound* a loop's behavior are the events you *count* to judge it. A loop whose
acceptance rate is high, cost is stable, and escalations are rare has earned its
`stable` badge; one that thrashes against its budget is telling you to fix it or
retire it.

## Governance and Security at Scale

The moment a library is shared and its loops run autonomously across many
repositories, the risks from Chapter 10 stop being one engineer's problem and become
an organizational one. A personal loop that misbehaves costs *you* a `git worktree
remove`; a shared loop with broad permissions that misbehaves can affect everyone.
Operating a library responsibly means putting limits around it.

The controls that matter most:

- **Least-privilege connectors.** A loop's `connectors` should grant only the access
  it needs — read-only where possible, scoped tokens, no standing production
  credentials baked into entries. The schema records *which* connectors a loop uses;
  governance ensures each is minimally scoped.
- **Central budget ceilings.** Beyond per-entry `max_cost_usd`, enforce an
  organization-wide cap so the *aggregate* of many loops cannot run away even if each
  individual loop is well-behaved. Layered budgets, applied at the fleet level.
- **An audit trail.** Log every run — which entry, which version, which inputs, who
  triggered it, what it changed, how it stopped. When an autonomous loop touches a
  shared system, you must be able to answer "what ran, and why?" after the fact.
- **Human approval gates for high blast-radius loops.** Loops that touch production
  data, public copy, or irreversible state should require explicit human sign-off
  before their results land — exactly the approval boundaries several Chapter 12
  loops (the customer AI deployment loop, the production data cleanup loop) build in
  by design.
- **A clear owner.** Someone is accountable for the library: the quality bar, the
  deprecation decisions, the security posture. Shared infrastructure without an owner
  decays into the low-quality sprawl this chapter exists to prevent.

None of these are new ideas — they are the Chapter 10 mitigations (bounded cost,
guarded against slop, human judgment retained) lifted from a single loop to a fleet
of them. Scaling a library is largely the work of making those per-loop safeguards
hold at the level of the whole organization.

## Scaling the Library Itself

As the library grows past a few dozen entries, two pressures appear: discovery gets
harder, and duplication creeps in. Both are manageable with the structure you
already have.

- **Lean on shapes, not entries.** Chapter 12 showed that loops collapse into a
  small set of shapes (audit-rank-fix, generate-score-improve, build-then-review,
  and so on). When two entries are the same shape with different goals, prefer one
  *parameterized* entry over two near-duplicates. The library should grow in
  *parameters and prompts*, not in copies.
- **Curate against sprawl.** Periodically run a housekeeping pass over the library —
  itself a loop, fittingly (the *housekeeper* pattern from Chapter 12): find entries
  nobody runs, duplicates, broken references, and stale prompts, and deprecate or
  consolidate them. A library that only ever grows eventually becomes unsearchable.
- **Federate by domain, share the runner.** Large organizations can let teams own
  their own category folders (engineering, design, operations) while sharing one
  schema and one runner. Local ownership keeps entries relevant; the shared runner
  keeps execution consistent.

The endgame is a library that stays *small relative to its usefulness*: a curated set
of trusted, parameterized, measured loops that the whole team reaches for — not an
ever-growing pile of one-off scripts. That is the same subtractive discipline this
book has argued for throughout, applied one level up: add a loop to the library when
not having it causes real duplication, keep it while it earns its place, and retire
it when it stops.

## Key Takeaways

- A shared **Loop Library is infrastructure, not a folder**: its failure mode is too
  many low-quality entries, so **curation** — controlling what gets in and what
  stays — is the central job.
- Changes to the library follow the **same loop discipline as the loops themselves**:
  propose via PR, run automated schema/stopping-condition gates, review against a
  written **quality bar**, and dry-run to confirm the loop actually stops before
  merging to `stable`.
- The **quality bar** requires a checkable goal, an independent evaluator, explicit
  layered stopping conditions, defined isolation and human-escalation, a documented
  blast radius, and correct attribution — kept as a PR template so contributors
  self-check.
- **Version and deprecate** rather than break: semantic `version` bumps for
  behavior changes, a `draft → experimental → stable → deprecated` maturity ladder
  exposed as a search facet, and successors pointed to instead of silent deletion.
- **Measure each loop** with the events its own stopping conditions already
  produce — acceptance rate, iterations/cost to success, escalation rate, and usage
  — and retire loops that thrash or that nobody runs.
- **Governance scales the Chapter 10 mitigations to a fleet**: least-privilege
  connectors, organization-wide budget ceilings, an audit trail of every run, human
  approval gates for high-blast-radius loops, and a clearly accountable owner.
- **Scale by shapes, not copies**: prefer one parameterized entry over
  near-duplicates, run periodic housekeeping passes against sprawl, and optionally
  federate category ownership while sharing one schema and runner — keeping the
  library small relative to its usefulness.

---
[< Previous: Implementing a Loop Library](13-implementing-a-loop-library.md) | [Table of Contents](README.md) | [Next: Glossary >](glossary.md)
