# 12. The Loop Library: A Catalog of Real-World Loops

> **In this chapter:** You will move from building a single Loop (Chapter 08) to
> studying a *library* of them. We survey a public collection of 45 production
> loops — the [Forward Future Loop Library](https://signals.forwardfuture.ai/loop-library/)
> — organized by the same five categories the library uses: Engineering,
> Evaluation, Design, Content, and Operations. For each loop you get its author,
> its purpose, and a plain-language summary of how it works, mapped back to the
> eight components (Chapter 06) and the lifecycle (Chapter 07). By the end you
> will have a reference shelf of proven loop *patterns* to adapt — and the raw
> material for the implementation chapters that follow.

## Why a Library, Not Just a Loop

Chapter 08 taught you to build *one* Loop and harden it. But teams rarely run a
single Loop forever. The same handful of shapes — *audit-and-fix*, *generate-and-score*,
*build-and-review*, *clean-up-safely* — recur across projects with different
goals plugged in. A **Loop Library** is the natural response: a curated, reusable
collection of loop definitions that a team can browse, copy, and adapt instead of
re-deriving each one from scratch.

This chapter is a guided tour of one such library so you have concrete patterns in
hand before Chapter 13 shows you how to *implement* a library of your own. Each
entry below is a real, named loop with an attributed author. Read it as a pattern
catalog: notice how every loop, however different its domain, is still the
generate → evaluate → feed-back → stop heartbeat from this book, with a
domain-specific goal and a domain-specific check bolted on.

> **Source and attribution.** Every loop in this chapter is drawn from the
> publicly published [Forward Future Loop Library](https://signals.forwardfuture.ai/loop-library/)
> (45 loops, last updated June 20, 2026), where each loop is credited to its
> contributor. The names and authors below are reproduced from that source; the
> "how it works" summaries are **rephrased in our own words for compliance with
> licensing restrictions** and to map each loop onto this book's vocabulary. For
> the exact, copy-ready prompt wording, consult the original library entry. If any
> attribution here disagrees with the source, trust the source.

## How to Read the Catalog

The library sorts loops into five categories. We keep that grouping so you can
cross-reference easily, and within each category every entry follows the same
shape:

- **Loop name** — *by author* — its category on the source site.
- **Purpose** — the one-line outcome the loop is meant to achieve.
- **How it works** — a paraphrased summary of the loop's prompt, written to
  highlight its stopping condition, its evaluator, and any guardrails.

As you read, keep asking the three diagnostic questions from earlier chapters:
*What is the checkable goal? What independent check decides "done"? What bounds the
run?* Almost every loop below answers all three — which is exactly what makes them
worth cataloging.

## Engineering Loops

The largest category. These loops operate directly on code, repositories, and
running systems, and they lean heavily on objective evaluators — test suites,
builds, benchmarks, and production telemetry.

### The docs sweep — *by Matthew Berman*

- **Purpose:** Keep documentation aligned with the current codebase and open a
  reviewable pull request.
- **How it works:** When a documentation pass is triggered, the agent reviews the
  whole codebase, finds documentation that no longer matches the implementation,
  updates it, verifies the changes, and opens a PR. The PR is the reviewable
  stopping artifact.

### The architecture satisfaction loop — *by Peter Steinberger*

- **Purpose:** Refactor architecture in small, independently reviewed, tested
  checkpoints.
- **How it works:** The agent refactors until the architecture is satisfactory.
  After each meaningful step it live-tests the system, runs an automated review,
  and commits, tracking progress in a scratch file (e.g. `/tmp/refactor-{project}.md`)
  so work is resumable. The "happy with the architecture" judgment is the stop.

### The sub-50 ms page-load loop — *by Matthew Berman*

- **Purpose:** Optimize every page until it consistently loads in under 50 ms.
- **How it works:** The agent keeps optimizing for speed; after each significant
  change it measures page-load performance across every page under identical,
  repeatable test conditions. The numeric threshold — every page under 50 ms — is
  the stopping condition.

### The production error sweep — *by Matthew Berman*

- **Purpose:** Find, fix, and verify actionable errors in production.
- **How it works:** The agent reviews production logs; for each actionable issue it
  traces the root cause, fixes it, verifies the fix, and opens a PR. If no
  actionable errors exist, it stops without changing anything — a clean no-op
  stopping condition.

### The 100% test coverage loop — *by Matthew Berman*

- **Purpose:** Add meaningful tests until the full suite reaches 100% coverage.
- **How it works:** The agent adds tests, iterating until coverage hits 100%. The
  coverage metric is the objective evaluator and the stop.

### The logging coverage loop — *by Matthew Berman*

- **Purpose:** Add useful, tested logs to every important system path.
- **How it works:** The agent reviews the system's logging and fills gaps until
  every important path emits useful, tested log output.

### The nightly changelog loop — *by Matthew Berman*

- **Purpose:** Keep the changelog current with meaningful changes.
- **How it works:** On a nightly schedule (the trigger), the agent reviews the
  previous day's changes and updates the changelog with anything users should know.

### The test-suite speed loop — *by Matthew Berman*

- **Purpose:** Speed up the test suite without weakening coverage, assertions, or
  isolation.
- **How it works:** The agent optimizes the suite to run as fast as possible while
  preserving coverage and behavior — the unchanged-behavior constraint is the
  guardrail.

### The repository cleanup loop — *by Matthew Berman*

- **Purpose:** Recover valuable work and safely remove proven-stale state.
- **How it works:** The agent inspects local and remote branches, PRs, commits, and
  worktrees, recovering anything valuable and clearing what is verifiably stale
  until the repository is current and organized.

### The ticket-to-PR-ready loop — *by Hiten Shah*

- **Purpose:** Turn a ticket, bug report, or complaint into a verified,
  reviewer-ready pull request.
- **How it works:** The agent reproduces the failure in the smallest representative
  environment, proves the root cause, makes the smallest credible fix, and reruns
  the original reproduction plus regression tests. If it cannot reproduce after two
  serious attempts, it says so. It avoids unrelated refactors and finishes with the
  cause, changed files, before/after proof, risks, and a PR summary.

### The Clodex adversarial-review loop — *by Lukas Kucinski*

- **Purpose:** Use one model to adversarially review another's pull request until
  blocking findings are resolved.
- **How it works:** Claude plans and implements a task and opens a PR; Codex
  performs an adversarial review; Claude fixes findings above an accepted severity
  and repeats. State (branch, PR, findings, verdict, iteration) stays resumable.
  It stops on approval, only-accepted-findings-remaining, stalled progress, or an
  iteration cap — and never reports an errored or exhausted run as approved.

### The Loop Harness verification loop — *by Istasha*

- **Purpose:** Ship scheduled agent work only after an independent verification
  pass.
- **How it works:** For scheduled repo work (CI triage, issue grooming, dependency
  updates, docs sync), one agent session stages a patch in an isolated worktree and
  a second session verifies it against explicit criteria. Work ships only on a
  pass; otherwise findings are preserved and the run retries within a set limit.

### The fresh-clone loop — *by 0xUmbra*

- **Purpose:** Repeat clean onboarding from the README until no hidden setup
  assumptions remain.
- **How it works:** The agent clones the repo into a disposable environment and
  follows *only* the README to the documented ready state. When a step fails or
  assumes missing knowledge, it records the gap, fixes the setup/docs, discards the
  environment, and starts over — carrying nothing between attempts. It stops when
  one uninterrupted fresh clone succeeds, progress stalls, or a budget ends.

### The autonomy-loop builder-reviewer loop — *by @inferencegod*

- **Purpose:** Pass code between a builder and a reviewer until tests prove each
  accepted fix.
- **How it works:** After test/build/lint gates pass, a builder and reviewer run in
  separate worktrees. The builder makes one bounded change with a red-before /
  green-after test; the reviewer reruns gates and proves the test by reverting or
  mutating the fix. A change is accepted only on both passes; protected or
  repeatedly failing work is parked for a human.

### The Codex completion-contract loop — *by 3goblack (@Dis_Trackted)*

- **Purpose:** Define "done" up front and require evidence for every reported
  result.
- **How it works:** Before acting, the agent defines every required outcome and the
  evidence that proves it. After each bounded action it marks each requirement
  proved, weak, missing, or contradicted. The goal completes only when all are
  proved; otherwise it stops as blocked, stalled, or exhausted — and asks before
  creating goal state.

### The five-minute repository maintainer loop — *by Peter Steinberger*

- **Purpose:** Keep repository work moving through dedicated threads without
  interrupting active agents.
- **How it works:** While maintenance is active, the loop wakes every five minutes,
  triages a set of repositories, reuses one thread per repo, and assigns each
  repo's highest-value bounded task within granted permissions — without
  interrupting coherent active work. It requires tests, live proof, autoreview, and
  green CI before anything lands, and escalates product/access/security/irreversible
  decisions.

### The recent-feedback sweep — *by Matthew Berman*

- **Purpose:** Turn recent user corrections into a project-wide audit and verified
  fixes.
- **How it works:** The agent reviews threads from a lookback window where the user
  reported something wrong, builds a deduplicated issue list grouped into failure
  patterns, audits the whole project for each pattern, fixes every confirmed
  instance, and adds regression coverage. It repeats the full audit until none
  remain or an iteration budget ends.

### The propagation compliance loop — *by @iamTristan*

- **Purpose:** After one value changes, find every other place still showing the
  old value.
- **How it works:** After a version, count, rule, name, or config changes, the agent
  lists where the new value belongs, searches for the old value and related forms,
  and updates real stale matches while preserving intentional history, examples,
  migrations, and compatibility rules. It repeats until zero stale values remain;
  if one keeps returning, it stops and identifies what regenerates it.

### The Goal Forge loop — *by michael Guo (@michaelzsguo)*

- **Purpose:** Turn a rough coding idea into measurable planning files before a
  long autonomous run starts.
- **How it works:** The agent interviews the user, then writes a spec file (what to
  build, exclude, and consider, plus measurable completion checks) and a goal file
  (work plan, scorecard, quick and final checks, memory files, evidence, and
  approval boundaries). If any key decision, permission, tool, or test is missing,
  it stops as "not ready" and does not start implementation without approval.

### The cold-load trimmer loop — *by Christian Katzmann*

- **Purpose:** Reduce data downloaded before a web app's first screen without
  changing behavior or appearance.
- **How it works:** The agent records passing tests, mobile/desktop screenshots, and
  compressed transferred bytes as a baseline, then defers, compresses, or removes
  one item at a time. It keeps a change only if tests pass, screenshots stay
  pixel-identical, and bytes decrease; otherwise it reverts. It stops when no safe
  candidate remains, progress stalls, or approval is needed.

### The housekeeper loop — *by Eric Lott*

- **Purpose:** Clean a code project one proven, low-risk change at a time.
- **How it works:** The agent reviews for dead code, stale files/comments, unused
  dependencies, duplication, broken links, inconsistent names, and confusing
  structure — while protecting unrelated, active, uncommitted, generated, and
  uncertain work. It proves one low-risk cleanup, makes the smallest coherent
  change, reruns build/tests/runtime/diff checks, and keeps only verified
  improvements.

### The prepare-a-new-project loop — *by Brad Shannon (@bradshannon)*

- **Purpose:** Strengthen project documents until independent engineers would build
  substantially the same system.
- **How it works:** The agent ensures the documents cover requirements, technical
  design, tasks with acceptance criteria, and test strategy. Each round it fixes the
  largest gap or contradiction that could make two competent engineers diverge, then
  has two independent reviewers describe the components, data model, dependencies,
  and definition of done. It stops when they materially agree and every artifact is
  testable.

### The test stabilizer loop — *by hungtv27 (@hungtv27)*

- **Purpose:** Find flaky tests, fix their root causes, and prove stability with
  repeated full-suite runs.
- **How it works:** The agent runs the suite N times under identical conditions and
  lists tests whose result changes, then fixes the most frequent flake at its root
  cause (shared state, timing, ordering, external dependency) — never with a blind
  sleep or retry. It reruns that test N times, then the full suite, repeating until
  N consecutive full-suite runs pass.

## Evaluation Loops

These loops exist to *judge* — products, prompts, policies, claims, and even other
agents. Their defining trait is a deliberately designed evaluator: holdout sets,
rubrics, must-pass checks, and independent reviewers.

### The quality streak loop — *by Matthew Berman*

- **Purpose:** Fix product failures until a defined streak of realistic tests
  passes.
- **How it works:** The agent tests realistic scenarios; when one fails it
  documents the failure, adds regression and benchmark coverage, fixes it, and
  restarts the streak. It stops after N successful cases in a row — the streak
  length is the stopping condition.

### The full product evaluation loop — *by Matthew Berman*

- **Purpose:** Test every major product capability and fix outcomes below the
  quality bar.
- **How it works:** The agent creates N realistic scenarios covering every major
  capability, defines success criteria and a consistent evaluation method
  (pass/fail or a scoring rubric) *before* testing, runs each scenario under
  identical conditions with recorded evidence, fixes root causes of anything below
  the bar, reruns affected scenarios, then reruns the complete set — continuing
  until every scenario meets the original bar.

### The self-improving champion loop — *by Jose C. Munoz*

- **Purpose:** Promote prompt or policy changes only when they win on fresh holdout
  cases.
- **How it works:** The agent saves a champion plus its score, a working set,
  untouched holdout cases, must-pass checks, and a budget. Each round it changes one
  thing based on a recorded failure and promotes the challenger only if it beats the
  champion on holdouts by a margin without weakening a must-pass check; otherwise the
  champion stays. It stops at a target, a budget limit, or no progress.

### The devil's-advocate loop — *by Anonymous contributor*

- **Purpose:** Challenge a design until every high-impact objection is resolved or
  explicitly accepted.
- **How it works:** Before committing to an architecture, interface, or rollout
  plan, a critic argues it is wrong; each objection, its impact, and its status are
  logged (e.g. `.agent-reviews/redteam.md`). The builder must fix and verify each
  high-impact weakness or document why it is accepted, and the critic may reopen
  weak answers. It stops when no high-impact objection remains or the same issues
  repeat for two rounds without new evidence.

### The Revolve versioned-experiment loop — *by Agent Zero*

- **Purpose:** Improve prompts, code, or configs through comparable, checkpointed
  experiments.
- **How it works:** In a dedicated directory the agent defines the goal and budget,
  freezes the tests and scoring, checkpoints the current version, and records a
  baseline. Each round it tests one hypothesis and keeps only a clear,
  regression-free win; if the evaluation changes it opens a new revision and reruns
  the baseline. It asks before changing live files and stops on success, no
  progress, a blocker, or exhausted budget.

### The promise-to-proof loop — *by Felix Haeberle (@felixhaberle)*

- **Purpose:** Check whether every customer-facing claim is true, then fix the
  riskiest mismatch first.
- **How it works:** The agent lists every customer-facing promise in marketing,
  docs, demos, and AI answers, compares each with actual product behavior and
  evidence, and labels it proven, partly proven, misleading, unsupported, outdated,
  or missing evidence. It fixes or narrows the riskiest mismatch, reruns the
  affected check, and repeats until no high-risk unsupported promise remains —
  asking before changing production or public copy.

### The multi-LLM convergence loop — *by Donn Felker (@donnfelker)*

- **Purpose:** Have two different AI systems review the same work until both approve
  one unchanged version.
- **How it works:** Within a pass limit, a reviewer from one model family reviews
  the work against a quality bar; verified findings are fixed and the revised
  version is handed to a reviewer from a *genuinely different* provider. Success
  requires both to approve the same unchanged version. It stops at the limit, on
  oscillating disagreement, on unavailable review, or when approval is required.

### The easy onboarding loop — *by Eric Lott*

- **Purpose:** Act like a first-time user, fix one obstacle, and retry from a
  completely clean session.
- **How it works:** Starting at the real entry point in a clean session (no saved
  login, data, or hidden setup), the agent completes onboarding using only visible
  guidance and records obstacles. It fixes the worst one with the smallest change
  that preserves every security/access/product requirement, discards the session,
  and retries — stopping after one uninterrupted success, no safe fix, blocked
  access, or required approval.

### The Axelrod subagent arena loop — *by Kan Yuenyong (@sikkha)*

- **Purpose:** Test whether AI agents learn to cooperate, retaliate, or forgive in a
  repeated two-choice game.
- **How it works:** Two reasoning agents play a fixed Axelrod tournament; each round
  both privately choose cooperate or defect, code records simultaneous moves and
  applies fixed scoring, with always-defect and always-cooperate comparison players.
  It runs a fixed number of cycles, pairings, and rounds, hides opponent type and
  private reasoning, validates every move and total, and reports rankings, reasoning
  summaries, and violations; a partial tournament counts as incomplete.

### The artifact-to-skill loop — *by Hiten Shah (@hnshah)*

- **Purpose:** Extract the method behind a strong artifact and prove it works on a
  fresh case.
- **How it works:** The agent records evidence that an artifact succeeded, defines
  success criteria, and extracts the decisions, sequence, checks, and
  failure-avoidance patterns (not surface style), removing sensitive material. An
  independent reviewer applies the method to a fresh real case; it revises at most
  twice and stops when the method meets the bar without the original artifact — or
  reports it as not generalizable.

## Design Loops

These loops target visual, interaction, and front-end fidelity. Their evaluators
are unusual: repeatable screenshots, pixel comparisons, scoring rubrics, and
accessibility scans stand in for a test suite.

### The Boeing 747 benchmark — *by @victormustar*

- **Purpose:** Build and improve a Three.js Boeing 747 across nine repeatable views.
- **How it works:** Before building, the agent chooses reference images, a scoring
  rubric, a visual threshold, and a budget. It builds the model from Three.js
  primitives, then a rig that screenshots nine repeatable angles. After each change
  it renders and scores the same views, has a critic name the weakest feature, and
  fixes it without regressing stronger views — keeping the best version and stopping
  at the threshold, stalled progress, or budget.

### War Loops: frontend reconstruction — *by Swayam*

- **Purpose:** Reconstruct a real interface and repair its weakest visual and motion
  mismatches.
- **How it works:** Pointed at an authorized URL or image, the agent captures it
  with a real browser and records layout, styles, content, motion, and responsive
  behavior. It builds a static mirror and a moving version, compares both with the
  source at desktop/tablet/mobile sizes, and repairs only the weakest fidelity
  signals — stopping when every gate passes, progress stalls, or capture is blocked.

### The Infinite Clickbait thumbnail loop — *by @Alex_FF*

- **Purpose:** Iterate thumbnail concepts until one clears the quality bar without
  misleading viewers.
- **How it works:** For a given video and approved assets the agent makes ten
  thumbnail concepts, scores each at real YouTube sizes against an inspiration
  channel for clarity, curiosity, emotional pull, contrast, and accuracy, then
  improves the weakest dimension of the top three and rescores. It keeps iterating
  the strongest concept until it clears a quality threshold or budget — rejecting
  anything the video cannot deliver.

### The UI/UX Score Loop — *by Hayden Cassar (@hcassar93)*

- **Purpose:** Walk a real user task, score each screen, improve weak spots, and
  retest.
- **How it works:** In a real browser the agent runs a user flow (e.g. signup) from
  fresh state on each pass — no saved login, cookies, or site data — capturing
  meaningful screens at agreed sizes and modes, scoring them with one checklist, and
  improving the weakest safe area. It reruns the whole flow, keeps only
  regression-free changes, and stops on success, two passes with no gain, blocked
  access, or required approval.

### The pixel-safe CSS trim loop — *by Christian Katzmann*

- **Purpose:** Shrink the CSS sent to users while keeping every tested screen
  visually identical.
- **How it works:** The agent captures representative pages, sizes, themes, and
  interactions and records the built CSS size, treating coverage reports only as
  suggestions. It removes one declaration or rule, rebuilds, and reruns screenshots
  and project checks, keeping the change only if every screenshot is pixel-identical
  and the built CSS is smaller; otherwise it reverts.

### The accessibility repair loop — *by Eric Lott*

- **Purpose:** Find barriers for keyboard, screen-reader, low-vision, and other
  users, then fix the most harmful first.
- **How it works:** The agent checks a scope against an accessibility standard (e.g.
  WCAG 2.2 AA) with automated scans plus available keyboard, screen-reader, and
  other manual tests, confirms each issue, ranks it by harm, and fixes the
  highest-impact blocker. It reruns the same checks plus the affected task and
  regression tests, keeps only verified fixes, and never silences a check or weakens
  the target.

## Content Loops

A small but distinct category: loops that produce or maintain published content,
gated on accuracy and source-grounding rather than tests.

### The SEO/GEO visibility loop — *by Matthew Berman*

- **Purpose:** Fix the highest-impact gaps in search and AI-answer visibility.
- **How it works:** The agent runs an SEO/GEO audit across crawlability,
  indexation, page intent, titles, internal links, structured data, source
  citations, and answer-first content, ranks gaps by expected impact, fixes the
  highest-leverage one, then reruns the same crawl and target-query benchmark across
  search and AI answer engines. It repeats until no critical technical issues remain
  and no high-impact gap is left.

### The product update podcast loop — *by Pierson Marks*

- **Purpose:** Turn meaningful product updates into a short, source-grounded podcast
  episode.
- **How it works:** Each night the agent reviews publicly released product changes,
  selects only what users need to know, and verifies each against the product, docs,
  or release notes. It uses a podcast-generation integration to produce a three-to-
  five-minute episode explaining what changed, why it matters, and how to try it,
  checks the script and audio for accuracy and pronunciation, makes no episode if
  nothing meaningful shipped, and asks before publishing.

## Operations Loops

These loops manage releases, production data, and customer-facing AI workflows.
They emphasize controlled rollout, monitoring, and excluding stale or unfinished
work.

### The stale-safe batch release loop — *by Matthew Berman*

- **Purpose:** Batch valid changes and release complete artifacts from the latest
  integrated main.
- **How it works:** The agent reviews pending changes and pull requests, excludes
  stale or unfinished work, combines the valid changes, and releases them together —
  the "exclude stale/unfinished" rule is the guardrail against shipping
  half-done work.

### The production data cleanup loop — *by Matthew Berman*

- **Purpose:** Remove disallowed production data and prevent the same classification
  errors from returning.
- **How it works:** The agent reviews production records, removes anything outside
  the allowed definition, improves the classification logic so the error class does
  not recur, and verifies the remaining data.

### The post-release baseline loop — *by Matthew Berman*

- **Purpose:** Benchmark each completed release and record a reproducible baseline.
- **How it works:** After current releases finish (the trigger), the agent runs the
  standard benchmarks and records the results as the new baseline for future
  comparison.

### The customer AI deployment loop — *by AgentLed.ai Agent*

- **Purpose:** Move one customer AI priority through validation, controlled rollout,
  and monitoring.
- **How it works:** Triggered when a customer requests an AI workflow, reports a
  failure, or reaches an operations review, the agent picks one priority (enriching
  leads, drafting emails, summarizing meetings, updating a CRM), defines the owner,
  inputs, approvals, success metric, and ROI hypothesis, dry-runs it on realistic
  customer data, fixes the smallest verified problem, then releases through approved
  stages and monitors production.

## Reading the Catalog as Patterns

Step back from the 45 individual entries and the same handful of *shapes* appear
again and again. Recognizing the shape is what lets you reuse a loop you have never
seen for a problem it was never written for:

- **Audit-rank-fix-reverify.** Scan a scope, rank findings by impact, fix the
  highest-leverage one, then rerun the same scan. *(production error sweep, SEO/GEO
  visibility, accessibility repair, promise-to-proof, housekeeper.)*
- **Generate-score-improve-the-weakest.** Produce several candidates, score them on
  a fixed rubric, and improve the weakest dimension of the best ones until a
  threshold is met. *(Boeing 747 benchmark, Infinite Clickbait thumbnail, UI/UX
  Score, self-improving champion.)*
- **Build-then-independently-review.** One agent produces, a *different* agent or
  model verifies, and work is accepted only on an independent pass. *(Clodex,
  autonomy-loop builder-reviewer, multi-LLM convergence, Loop Harness, devil's-
  advocate.)*
- **Reproduce-from-clean-state.** Start from a pristine environment every attempt,
  fix one obstacle, and retry from scratch — carrying nothing between runs.
  *(fresh-clone, easy onboarding.)*
- **Streak / threshold convergence.** Keep iterating until N successes in a row, a
  numeric target, or a coverage figure is reached. *(quality streak, 100% test
  coverage, sub-50 ms page-load, test stabilizer.)*
- **Plan-before-you-build.** Refuse to start implementation until measurable,
  reviewed planning artifacts exist. *(Goal Forge, prepare-a-new-project, Codex
  completion-contract.)*

Each shape is a reusable template waiting for a goal, an evaluator, and a stopping
condition. That observation is the bridge to the next chapter: if loops cluster
into a small number of reusable shapes, you can *catalog* them deliberately — which
is exactly what implementing a loop library means.

## Key Takeaways

- A **Loop Library** is a curated, reusable collection of loop definitions; this
  chapter surveys one public example — the
  [Forward Future Loop Library](https://signals.forwardfuture.ai/loop-library/) of
  45 loops — as a pattern catalog. *(Summaries here are rephrased for licensing
  compliance; see the source for copy-ready prompt wording.)*
- The library sorts loops into five categories: **Engineering** (the largest,
  gated on tests/builds/benchmarks/telemetry), **Evaluation** (gated on rubrics,
  holdouts, and independent reviewers), **Design** (gated on screenshots, pixel
  comparisons, and accessibility scans), **Content** (gated on accuracy and
  source-grounding), and **Operations** (gated on controlled rollout and
  monitoring).
- However different their domains, every loop is still the book's generate →
  evaluate → feed-back → stop heartbeat with a domain-specific **goal**,
  **evaluator**, and **stopping condition** plugged in.
- The 45 loops collapse into a few reusable **shapes** —
  audit-rank-fix-reverify, generate-score-improve-the-weakest,
  build-then-independently-review, reproduce-from-clean-state, streak/threshold
  convergence, and plan-before-you-build — and recognizing the shape is what makes a
  loop reusable beyond its original purpose.
- Because loops cluster into a small number of shapes, they can be cataloged
  deliberately — which is what **implementing a loop library** (Chapter 13) is all
  about.

---
[< Previous: Synthesis & Conclusion](11-synthesis-and-conclusion.md) | [Table of Contents](README.md) | [Next: Implementing a Loop Library >](13-implementing-a-loop-library.md)
