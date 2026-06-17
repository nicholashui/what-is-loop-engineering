# 08. Building a Loop: Practical Guide

> **In this chapter:** You will move from understanding a Loop to *building* one.
> We start from the simplest thing that could possibly work — a bare bash loop in
> the spirit of the Ralph Loop — then evolve it into a production-grade system
> with isolation, an independent evaluator, persisted state, and explicit stopping
> conditions, and finally distil the common patterns for designing a Loop's goals,
> evaluation criteria, and guardrails. By the end you will be able to construct a
> working Loop yourself and reason about how to harden it.

## From Concept to Construction

The previous chapters gave you the parts (Chapter 06) and showed them in motion
(Chapter 07). This chapter puts a screwdriver in your hand. The fastest way to
understand a Loop is to build the smallest one that runs, see exactly where it is
fragile, and then add structure to fix each weakness in turn.

So we begin deliberately small. The first example below is the *minimal* Loop: a
handful of lines of bash that captures the essential idea — call an agent, feed it
the result, call it again — and nothing more. It is intentionally missing every
production concern. Seeing it bare makes those missing concerns obvious, which is
exactly what sets up the production-grade version that follows.

## A Minimal Loop: The Ralph-Style Bash Example

The Ralph Loop (Chapter 05) is the spiritual ancestor of every Loop in this book:
a simple shell loop that repeatedly hands an agent fresh context plus whatever the
last run produced, and keeps going until the work is done. Here is a minimal Loop
in that spirit.

```bash
#!/usr/bin/env bash
# minimal-loop.sh — a Ralph-style loop: call an agent, recycle its output, repeat.

PROMPT="Implement the tasks in TODO.md. Make the test suite pass."
context=""                                   # carries prior output/errors forward

# Loop condition: keep going until a sentinel signals the work is finished.
# Here, the agent writes the literal text "ALL DONE" when it believes it is done.
while ! grep -q "ALL DONE" last_output.txt 2>/dev/null; do

  # Agent invocation: send the standing prompt plus the recycled context
  # from the previous iteration to the coding agent, and capture everything
  # it prints (its diff summary, command output, and any errors).
  agent --prompt "$PROMPT

Previous attempt output and errors:
$context" > last_output.txt 2>&1

  # Recycle prior output/errors into the next iteration's context.
  # Next time around, the agent sees what it just did and what went wrong,
  # so the following attempt is better-informed than the last.
  context="$(cat last_output.txt)"

done

echo "Loop finished."
```

Three lines do the real work, and each maps to an idea from the earlier chapters:

- **The loop condition** is the `while ! grep -q "ALL DONE" ...` test. This is the
  Loop's *stopping condition* in its crudest form: the cycle repeats until a
  sentinel string appears in the agent's output. It is the same continue-or-stop
  decision from Chapter 07's lifecycle, reduced to a single string match.
- **The agent invocation** is the `agent --prompt "..."` call. This is the
  *Generator Agent* doing the hands-on work — reading the task, editing files,
  running commands — and printing the result. Everything the agent prints
  (`> last_output.txt 2>&1`) is captured, including errors.
- **Context recycling** is the `context="$(cat last_output.txt)"` line combined
  with embedding `$context` into the next prompt. This is the feedback path that
  makes a Loop a *loop*: the previous iteration's output and errors are folded
  into the next iteration's context, so each attempt builds on the last instead of
  starting blind.

### What This Minimal Loop Deliberately Lacks

This example is useful precisely because of what it leaves out. It has, by design:

- **No isolation.** The agent edits files directly in the working directory. There
  is no git worktree and no separate branch, so a bad iteration can corrupt your
  workspace with nothing to roll back to.
- **No independent evaluator.** "Done" is whatever the agent *says* it is — the
  `grep` for `ALL DONE` trusts the agent's own claim. Nothing external runs the
  tests or reviews the diff, so the Loop cannot tell good work from confident
  nonsense.
- **No persisted state.** The only memory is a single `last_output.txt` that is
  overwritten every iteration. There is no durable record of progress, no task
  list that survives a crash, and no way to resume where it left off.

It also has no real budget guardrail: if the agent never prints the sentinel, the
loop runs forever. These are not bugs to patch in place — they are the exact gaps
that the production-grade Loop in the next section is built to close. Hold this
minimal version in mind as the baseline we are about to harden.

## A Production-Grade Loop: Closing the Four Gaps

We now evolve the minimal Loop into something you could trust to run unattended.
The structure is the same — call the Generator Agent, fold the result into the next
attempt — but every gap from the previous section is filled by an explicit piece of
machinery. Each addition maps directly to one of the eight components from
Chapter 06 and to a stage of the seven-stage lifecycle from Chapter 07.

It helps to keep both reference frames in view as we build:

| Production concern we add | Chapter 06 component | Chapter 07 lifecycle stage |
|---------------------------|----------------------|----------------------------|
| Run each attempt in a throwaway worktree/branch | Worktrees/Isolation | Stage 2 — spawn agents in an isolated workspace |
| Gate acceptance on an independent check | Evaluator | Stage 4 — Evaluator checks the output |
| Record progress in a durable file | State/Memory | Stage 6 — persist state |
| Terminate on success, threshold, or budget cap | Stopping Condition | Stage 7 — decide continue / spawn / stop |

The Generator Agent (Chapter 06) is still the worker at the center; the trigger that
launches the script is the Automation/Trigger component; and the recycled feedback
from the Evaluator into the next prompt is lifecycle Stage 5 (feed feedback back and
repeat). What changes is that none of these are left implicit anymore.

### Step 1: Externalize the configuration

First we pull the Loop's knobs out of the script and into a config file. This is
where the **Stopping Conditions** live as data rather than buried logic, which makes
budgets auditable and easy to tune.

```yaml
# loop.config.yaml — declarative control surface for the Loop.
goal: "Implement the next unchecked task in TODO.md and make the suite green."

generator:
  command: "agent --prompt-file"   # the Generator Agent (Chapter 06) we invoke
  model: "your-coding-agent"

evaluator:
  # The Evaluator is independent of the generator: it does NOT trust the
  # agent's self-report. It runs real commands and scores the result.
  test_command: "npm test --silent"   # objective gate: must exit 0
  critic_command: "agent --review"     # optional second opinion / rubric scorer
  rubric_threshold: 0.8                # accept only if critic score >= 0.8

stopping_conditions:
  max_iterations: 12        # hard cap: never loop more than this many times
  max_cost_usd: 5.00        # budget guardrail: stop if spend exceeds this
  stop_when_tests_pass: true # success exit: green suite AND rubric met

isolation:
  base_branch: "main"
  worktree_root: ".loop-worktrees"  # each iteration gets its own worktree here

state_file: "TODO.md"        # persisted progress between iterations
```

Every field here is a production concern made explicit. The `stopping_conditions`
block alone closes the "runs forever" gap from the minimal version: the Loop now has
a success exit, a rubric exit, *and* two independent budget caps.

### Step 2: The orchestrator script

The script below reads that config and runs the hardened Loop. Read the inline
comments as a guided tour — each labeled block is one of the four additions.

```bash
#!/usr/bin/env bash
# production-loop.sh — a hardened Loop with isolation, an independent
# evaluator, persisted state, and explicit stopping conditions.
set -euo pipefail

# --- Load configuration (the declarative control surface from Step 1) ---
GOAL=$(yq '.goal' loop.config.yaml)
TEST_CMD=$(yq -r '.evaluator.test_command' loop.config.yaml)
CRITIC_CMD=$(yq -r '.evaluator.critic_command' loop.config.yaml)
THRESHOLD=$(yq -r '.evaluator.rubric_threshold' loop.config.yaml)
MAX_ITERS=$(yq -r '.stopping_conditions.max_iterations' loop.config.yaml)
MAX_COST=$(yq -r '.stopping_conditions.max_cost_usd' loop.config.yaml)
BASE_BRANCH=$(yq -r '.isolation.base_branch' loop.config.yaml)
STATE_FILE=$(yq -r '.state_file' loop.config.yaml)

iteration=0
spend=0
context=""

# === STOPPING CONDITION (Chapter 06 component / lifecycle Stage 7) ===
# The loop header encodes the budget guardrails as data-driven tests, not a
# blind sentinel. We stop when we hit the iteration cap OR the cost cap.
while (( iteration < MAX_ITERS )) && (( $(echo "$spend < $MAX_COST" | bc -l) )); do
  iteration=$((iteration + 1))
  echo "=== Iteration $iteration (spend so far: \$$spend) ==="

  # === ISOLATION (Worktrees/Isolation component / lifecycle Stage 2) ===
  # Each attempt runs in its OWN git worktree on its OWN throwaway branch.
  # A bad iteration can never corrupt main or a sibling attempt; we just
  # delete the worktree. This is the reversibility the minimal Loop lacked.
  branch="loop/attempt-$iteration"
  worktree=".loop-worktrees/$branch"
  git worktree add -b "$branch" "$worktree" "$BASE_BRANCH"

  pushd "$worktree" >/dev/null

  # === GENERATOR AGENT (lifecycle Stage 3: agent acts to produce output) ===
  # We hand the agent the standing goal, the PERSISTED state file, and the
  # recycled feedback from the previous Evaluator run (lifecycle Stage 5).
  cat > .loop-prompt.txt <<EOF
Goal: $GOAL

Current progress (persisted state):
$(cat "../../$STATE_FILE")

Feedback from the previous attempt's evaluation:
$context
EOF
  agent --prompt-file .loop-prompt.txt

  # === EVALUATOR (Evaluator component / lifecycle Stage 4) ===
  # Independent verification. The agent's opinion does not count here.
  # Gate 1 is objective (tests must pass); Gate 2 is a critic/rubric score.
  if $TEST_CMD; then
    tests_pass=true
  else
    tests_pass=false
  fi
  score=$($CRITIC_CMD --rubric ../../rubric.md --format score)  # e.g. 0.0–1.0

  # Accumulate spend for the budget stopping condition.
  spend=$(echo "$spend + $(agent --last-cost)" | bc -l)

  # === DECISION POINT (lifecycle Stage 7: continue / spawn / stop) ===
  if [ "$tests_pass" = true ] && (( $(echo "$score >= $THRESHOLD" | bc -l) )); then
    # SUCCESS exit: both gates satisfied. Promote this attempt and stop.
    popd >/dev/null
    git merge --no-ff "$branch" -m "Loop: accepted attempt $iteration"
    echo "Accepted on iteration $iteration (score $score). Stopping."
    break
  fi

  # === PERSIST STATE (State/Memory component / lifecycle Stage 6) ===
  # Record what was learned so the NEXT iteration resumes instead of
  # starting blind, and so progress survives a crash. The feedback is also
  # recycled into the next prompt (lifecycle Stage 5).
  context="tests_pass=$tests_pass; critic_score=$score; see notes below"
  {
    echo ""
    echo "## Iteration $iteration result"
    echo "- tests passed: $tests_pass"
    echo "- critic score: $score (threshold $THRESHOLD)"
  } >> "../../$STATE_FILE"

  popd >/dev/null
  # Discard the rejected attempt's worktree; the branch is kept for forensics.
  git worktree remove --force "$worktree"
done

# If we fell out of the loop without a SUCCESS break, a stopping condition
# (max iterations or budget) halted us — a controlled stop, not a runaway.
echo "Loop ended after $iteration iteration(s); total spend \$$spend."
```

### How each gap was closed

Walking the four concerns back against the minimal baseline:

- **Isolation.** The minimal Loop edited the working directory in place. Here, every
  iteration gets a fresh `git worktree` on a throwaway `loop/attempt-N` branch
  (Worktrees/Isolation, lifecycle Stage 2). A failed attempt is simply removed; only
  an *accepted* attempt is merged back into the base branch. Nothing the agent does
  can corrupt `main` or a sibling attempt.
- **Independent evaluator.** "Done" is no longer the agent's say-so. An objective
  gate (`npm test` must exit 0) and a critic/rubric gate (`score >= 0.8`) must *both*
  pass before the work is accepted (Evaluator, lifecycle Stage 4). This is the
  primary defense against confident-but-wrong output.
- **Persisted state.** Progress is appended to a durable `TODO.md` after each
  iteration, and that file is fed back into the next prompt (State/Memory, lifecycle
  Stage 6). The Loop can resume after a crash and each attempt builds on a recorded
  history rather than a single overwritten scratch file.
- **Stopping conditions.** Termination is explicit and multi-pronged: success (both
  gates green), exhausting `max_iterations`, or exceeding `max_cost_usd`
  (Stopping Condition, lifecycle Stage 7). The runaway-forever failure mode of the
  minimal Loop is structurally impossible here.

Notice what did *not* change: the heartbeat is still generate → evaluate → feed
back → repeat. The production Loop is not a different idea from the Ralph-style
script — it is the same idea with each implicit assumption replaced by an explicit,
inspectable component. That is the whole move of Loop Engineering: take the parts
from Chapter 06, arrange them along the lifecycle from Chapter 07, and make every
control surface something you can see, tune, and trust.

## Building a Loop from Its Components: A Practical Path

The two examples above show the destination. This section is the route. You do not
sit down and write the production Loop in one pass — you grow it, one component at a
time, in an order that keeps you in control at every step. The guiding principle is
simple: **start with the smallest Loop that runs against your real task, then add
each component only when the absence of it actually hurts.**

A reliable order for assembling a Loop from the eight Chapter 06 components is:

1. **Start with the Generator Agent and a stopping condition.** This is the minimal
   Loop from the first section: a `while` loop, one agent invocation, and a crude
   stop test. Run it against a real task immediately. You are not trying to be
   correct yet — you are trying to *see the heartbeat* (generate → feed back →
   repeat) work end to end on your machine.
2. **Add isolation next, before you trust the Loop with anything real.** The moment
   the agent edits files unattended, a bad iteration can corrupt your workspace.
   Wrap each attempt in a git worktree on a throwaway branch (Worktrees/Isolation)
   so every iteration is reversible. This is the cheapest insurance in the whole
   system; add it early.
3. **Add the Evaluator.** This is the single highest-leverage component, because it
   is what separates a Loop from a random-output generator. Begin with the
   strongest objective signal you already have — usually your existing test suite —
   and only later layer in a critic/rubric gate (covered under *Evaluation
   Criteria* below).
4. **Add persisted state (State/Memory).** Once iterations are isolated and gated,
   give the Loop a durable memory — a `TODO.md`, scratchpad, or project board — so
   progress survives a crash and each attempt builds on a recorded history rather
   than starting blind.
5. **Tighten the stopping conditions into explicit guardrails.** Replace the crude
   sentinel with data-driven caps: success exit, iteration cap, and a budget cap
   (see *Guardrails* below).
6. **Add the remaining components only when your task demands them.** The
   Automation/Trigger (cron, webhook, slash-command), Connectors (issue tracker,
   chat, source-control host), and Skills/Knowledge (a skills file, coding
   standards) components are what turn a Loop you run by hand into one that runs
   itself. They are essential for an unattended Loop, but they add nothing to
   correctness — so defer them until the generate-evaluate core is trustworthy.

The discipline here is **subtractive, not additive**: every component you add is a
control surface you now have to understand and maintain, so the right question at
each step is "does omitting this cause a concrete failure on my task?" If a Loop
that runs locally on a green test suite already does the job, you do not need a
webhook trigger or a chat connector. Grow toward the production example only as far
as your task actually pushes you.

## Patterns for Goals, Evaluation Criteria, and Guardrails

Three design decisions determine a Loop's success more than any other: the **goal**
you give it, the **evaluation criteria** that decide when its work is acceptable,
and the **guardrails** that bound its behavior. The components are just plumbing;
these three are where engineering judgment lives. The patterns below are the ones
worth reaching for first.

### Designing the goal

A Loop runs unattended, so its goal must be **clear and checkable** — phrased so
that something other than the Generator Agent's own optimism can tell whether it has
been met.

- **Make the goal externally verifiable.** Prefer a goal whose completion a machine
  can confirm. *"Make `npm test` exit 0 with no skipped tests"* is checkable;
  *"improve the code"* is not. A goal the Evaluator cannot check is a goal the Loop
  cannot finish.
- **Bound the scope to one coherent unit of work.** A Loop steered at *"refactor the
  payments module so all existing tests still pass"* converges; one aimed at
  *"modernize the codebase"* wanders. Narrow goals give the feedback path something
  concrete to push against each iteration.
- **State the goal as an outcome, not a procedure.** Describe the end state you want
  ("the suite is green and the new endpoint returns 201 on success"), not the steps
  to get there. The agent chooses the steps; the Loop verifies the outcome.

A goal written this way drops straight into the `goal` field of the configuration
from the production example:

```yaml
# A clear, checkable, outcome-phrased goal — the kind a Loop can actually finish.
goal: >
  Add pagination to GET /orders so it accepts ?page and ?limit query params,
  returns at most `limit` items, and keeps the existing /orders tests green.
# Checkable: the existing + new tests are the objective signal of "done".
# Bounded: one endpoint, one behavior — not "improve the orders API".
```

### Designing the evaluation criteria

The Evaluator (Chapter 06) is the Loop's defense against confident-but-wrong output.
A trustworthy evaluator shares one defining trait: **it is independent of the
Generator Agent** — it never accepts the agent's self-report as proof of success.
Good evaluation usually layers two kinds of check:

- **An objective gate that is hard to fake.** Tests, type checks, linters, a build
  step, or a schema validation — anything that returns an unambiguous pass/fail the
  agent cannot talk its way past. This is the backbone of trustworthy evaluation;
  start here.
- **A critic/rubric gate for what objective checks miss.** A test suite confirms
  behavior but says little about readability, security, or whether the change
  matches the intent. A second agent scoring the diff against a written rubric
  covers that gap. Keep the rubric explicit and version it, so the standard the Loop
  is held to is visible and tunable.

The two gates compose into an acceptance rule — *accept only when the objective gate
passes AND the rubric clears its threshold* — exactly the decision point from the
production example:

```python
# The acceptance rule: an independent two-gate check the agent cannot self-certify.
def is_acceptable(result, rubric_score, threshold=0.8):
    # Gate 1 — objective and unfakeable: the real test suite must pass.
    if not result.tests_passed:
        return False
    # Gate 2 — qualitative: an independent critic must clear the rubric threshold.
    if rubric_score < threshold:
        return False
    return True  # Both gates satisfied -> promote this attempt and stop.
```

Two patterns make an evaluator trustworthy in practice. **Run the objective gate in
the same isolated worktree the agent worked in**, so you are scoring exactly what it
produced. And **set the rubric threshold deliberately**: too low and slop slips
through; too high and the Loop burns its whole budget chasing a score it can never
reach. A threshold you can defend — and adjust as you learn — beats an arbitrary one.

### Designing the guardrails

Guardrails are the Stopping Conditions (Chapter 06) plus the isolation and
escalation rules that keep an unattended Loop from doing damage. Where evaluation
decides *whether work is good*, guardrails decide *how far the Loop is allowed to go*
before something stops it. Reach for these patterns:

- **Always set more than one budget.** Cap iterations *and* spend. Either alone has a
  blind spot — a cheap-but-stuck Loop can spin through its iteration count; an
  expensive single attempt can blow the dollar budget in one shot. Together they box
  in both runaway modes.
- **Isolate every attempt and merge only on success.** Reversibility *is* a
  guardrail: if a rejected attempt lives in a throwaway worktree, a bad iteration
  costs you a `git worktree remove`, not a corrupted `main`.
- **Define an explicit escalation-to-human path.** A Loop should know when to stop
  and *ask*. When it exhausts its budget without acceptance, oscillates without
  improving, or trips a high-risk tripwire, it must hand off to a human rather than
  fail silently or keep burning tokens.

These rules live in the config's `stopping_conditions` block alongside an escalation
hook:

```yaml
# Guardrails: layered budgets plus an explicit hand-off to a human.
stopping_conditions:
  max_iterations: 12          # caps a stuck-but-cheap Loop that spins in place
  max_cost_usd: 5.00          # caps an expensive Loop independently of iterations
  stop_when_tests_pass: true  # success exit: objective + rubric gates both clear

escalation:
  # When a budget is hit WITHOUT acceptance, don't fail silently — hand off.
  on_budget_exhausted: notify-human   # e.g. open an issue / post to chat (Connectors)
  on_repeated_no_progress: 3          # same failure N times in a row -> escalate
```

Notice that every pattern in this section ties back to a component from Chapter 06
and a stage of the lifecycle from Chapter 07: goals shape the work that is
*discovered* (Stage 1), evaluation criteria are the *Evaluator check* (Stage 4), and
guardrails are the *continue/spawn/stop decision* (Stage 7). Designing a Loop well
is, in the end, designing these three things well — the components simply give each
decision a place to live.

## Key Takeaways

- The fastest way to understand a Loop is to build the smallest one that runs,
  observe where it breaks, and add structure to fix each weakness.
- A **minimal Ralph-style Loop** is just a bash `while` loop that calls a Generator
  Agent, captures its output and errors, and recycles them into the next
  iteration's context until a sentinel stopping condition is met.
- That minimal Loop deliberately has **no isolation, no independent evaluator, and
  no persisted state** — making those missing concerns the motivation for the
  production-grade Loop that follows.
- The **production-grade Loop** keeps the same generate → evaluate → feed-back
  heartbeat but fills each gap with an explicit component: a **git worktree** per
  attempt (isolation), an **independent evaluator** that gates on tests *and* a
  rubric score, a **persisted `TODO.md`** that survives crashes and informs the next
  attempt, and **explicit stopping conditions** (success, max iterations, budget
  cap) that make runaway loops impossible.
- Each production addition maps one-to-one onto a **Chapter 06 component** and a
  **Chapter 07 lifecycle stage** — building a Loop is mostly making those parts
  explicit and inspectable.
- **Build a Loop subtractively, one component at a time:** start with a Generator
  Agent plus a stopping condition, add **isolation** before trusting it, then an
  **Evaluator**, then **persisted state**, then explicit budget guardrails — adding
  the Automation/Trigger, Connectors, and Skills/Knowledge components only when the
  task actually demands them.
- A Loop's success hinges on three design decisions more than any component choice:
  a **goal** that is clear, checkable, and outcome-phrased; **evaluation criteria**
  built on an *independent* objective gate (tests) plus a critic/rubric gate; and
  **guardrails** — layered iteration *and* budget caps, per-attempt isolation, and an
  explicit escalation-to-human path — so an unattended Loop knows when to stop and ask.

---
[< Previous: The Loop Lifecycle](07-loop-lifecycle.md) | [Table of Contents](README.md) | [Next: Benefits & When to Use Loops >](09-benefits.md)
