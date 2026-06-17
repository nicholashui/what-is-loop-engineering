# 06. Anatomy of a Loop: The Components

> **In this chapter:** You will learn the eight components that make up a Loop —
> the *Automation/Trigger*, *Worktrees/Isolation*, the *Generator Agent*, the
> *Evaluator*, *State/Memory*, *Skills/Knowledge*, *Connectors*, and the
> *Stopping Condition*. For each one you will learn its purpose — the single job
> it does inside the Loop — and see at least one concrete example of how that job
> is filled in practice. By the end you will be able to identify every building
> block of a Loop and explain why each one earns its place.

## The Parts of the Machine

Chapter 03 gave you the image of a Loop as a circle that turns on its own —
prompt, observe, verify, iterate — and Chapter 04 showed how that circle was
lifted out of a single agent conversation and built into a system. This chapter
opens the machine and names the parts.

A Loop is assembled from **eight components.** Each one corresponds to a judgment
or a job that, in manual prompting, you would do yourself: deciding when to start,
keeping work from colliding, doing the actual coding, checking whether the result
is good, remembering what has happened, knowing the project's rules, talking to
the outside world, and deciding when to stop. Move each of those into an explicit
part of a system, and you have a Loop.

We will take them one at a time, in the order work flows through them. For every
component the structure is deliberately identical — **what it is**, **its
purpose**, and **at least one concrete example** — so you can hold all eight in
your head as variations on a single pattern: *a part that does one job so the Loop
does not need a human to do it.* The terms used here match the
[Glossary](glossary.md) exactly, and several of them — the Generator Agent, the
Evaluator, the Stopping Condition — you have already met in earlier chapters.

## Automation / Trigger

**What it is.** The Automation/Trigger is the component that decides *when and how*
work enters the Loop. It is the starting gun: the mechanism that kicks off an
iteration without a human typing "go."

**Its purpose.** In manual prompting, *you* start every turn — you decide there is
work to do and you begin it. The trigger moves that decision into the system, so
the Loop can begin work on a schedule or in response to an event, rather than
waiting for a person to initiate it. This is what lets a Loop run while you sleep:
the work starts itself.

**Concrete examples.** A trigger can be:

- a **cron schedule** — for example, a job that fires every night at 2 a.m. to
  pick up the next item of work;
- a **webhook on a new issue** — the moment an issue is filed in the tracker, the
  webhook fires and the Loop begins working on it;
- a **slash-style command** such as `/loop` — a human (or another system) types a
  single command to launch the Loop on demand.

```yaml
# Example: a trigger defined as a nightly schedule.
trigger:
  type: cron
  schedule: "0 2 * * *"   # every day at 02:00 — start the Loop unattended
```

The common thread across all three is that *the decision to begin is encoded*, not
performed live by a human.

## Worktrees / Isolation

**What it is.** The Worktrees/Isolation component keeps each agent run in its own
sealed-off workspace, so the changes one run makes cannot trample the changes
another run is making.

**Its purpose.** A Loop may run many iterations — and sometimes several agents at
once. Without isolation, two runs editing the same files at the same time corrupt
each other's work, and a bad run can damage a shared workspace irreversibly.
Isolation prevents agents from stepping on each other and keeps every run *safe and
reversible*: if an iteration goes wrong, you discard its isolated workspace and
nothing else is affected.

**Concrete examples.** The standard tools for this are:

- **git worktrees** — each agent run gets its own working directory checked out
  from the same repository, so its edits are physically separate from every other
  run's;
- **separate branches** — each run commits to its own branch, so its history is
  isolated and can be reviewed, merged, or thrown away independently.

```bash
# Example: give each iteration its own isolated git worktree + branch.
git worktree add ../run-42 -b loop/iteration-42   # separate directory AND branch
# the agent for iteration 42 works only inside ../run-42, untouched by other runs
```

Isolation is what turns "many runs" from a liability into a feature: runs can
proceed in parallel and fail harmlessly.

## Generator Agent

**What it is.** The **Generator Agent** is the AI coding agent that produces the
actual output of the Loop — the code, the edits, the changes. It is the worker
inside the machine.

**Its purpose.** This is the component that does the hands-on work a human would
otherwise do turn by turn. As established in Chapter 03, the Loop is *not* the
agent; the Generator Agent is just *one part* of the Loop — the part responsible
for generating output. Everything else in this chapter exists to decide what the
Generator Agent should do, to keep it safe, to check its work, and to decide
whether to run it again.

**Concrete examples.** A Generator Agent is filled by an off-the-shelf AI coding
agent, such as:

- **Claude Code** — running as the agent that reads the task and edits the
  codebase;
- **a Grok agent** — playing the same generating role within the Loop.

```bash
# Example: invoke the Generator Agent for one iteration.
claude-code --task "$(cat current-task.md)" --workdir ../run-42
# the agent reads the task and produces edits inside its isolated worktree
```

The Loop does not care *which* agent fills this slot — only that something does the
generating. The agent is interchangeable; the role is fixed.

## Evaluator / Critic / Verifier

**What it is.** The **Evaluator** — also called a critic or verifier — is the
component that checks the Generator Agent's output against some criteria and decides
whether it is good enough. It is the independent judge.

**Its purpose.** This is one of the most important additions a Loop makes over a
bare agent run. In a single agent conversation, the agent is both the worker *and*
the only judge of its own work; a Loop adds an **external** Evaluator so that "done"
is *verified* rather than merely *assumed*. The Evaluator is what protects a Loop
from confidently producing wrong or low-quality output, because nothing is accepted
until something other than the generator has checked it.

**Concrete examples.** Evaluation can take several forms, often in combination:

- a **separate agent that runs the tests** — executing the project's test suite and
  treating a failure as a rejection;
- a **code-review / critic agent** — a second agent that reads the diff and reviews
  it for quality, correctness, and style;
- a **rubric scorer** — scoring the output against an explicit rubric and comparing
  the score to a threshold.

```bash
# Example: a test-based Evaluator gates acceptance of the generator's output.
if npm test; then
  echo "PASS — accept this iteration"
else
  echo "FAIL — feed the errors back and iterate"
fi
```

Whatever its form, the Evaluator's job is the same: *judge the work from the
outside* and hand back a verdict the Loop can act on.

## State / Memory

**What it is.** The State/Memory component persists the Loop's progress *outside*
any single agent's context, so information survives from one iteration to the next.

**Its purpose.** A single agent run remembers only what is in its context window,
and that memory vanishes when the run ends. For a Loop to make real, cumulative
progress across many runs, it needs memory that *outlives* any one run — a record
of what has been done, what is left, and what was learned. State/Memory is what lets
iteration 42 build on what iteration 41 accomplished instead of starting from
scratch.

**Concrete examples.** State is typically kept in durable, inspectable places:

- a **TODO file** (for example, `TODO.md`) that records the remaining work and gets
  updated each iteration;
- a **scratchpad** where the Loop notes findings and decisions between runs;
- a **project board** in a tool like Linear or Notion that tracks the status of each
  unit of work;
- plain **files on disk** that capture intermediate results.

```markdown
<!-- Example: TODO.md as persisted state, updated between iterations -->
# TODO
- [x] Add input validation to the parser   <- done in iteration 41
- [ ] Handle empty-file edge case           <- next up for iteration 42
- [ ] Update the docs
```

Because state lives outside the agent, it is also something *you* can read to see
exactly where the Loop stands.

## Skills / Knowledge

**What it is.** The Skills/Knowledge component is the codified, reusable guidance a
Loop supplies to its agents — the project's rules, conventions, and patterns,
written down where an agent can use them.

**Its purpose.** An agent that does not know your project's standards will reinvent
them inconsistently, run after run. Skills/Knowledge captures that guidance *once*
so every iteration follows the same rules, instead of relying on a human to repeat
"use our naming convention" or "always write tests" in each prompt. It is how a Loop
stays aligned with how *your* team actually works.

**Concrete examples.** This guidance is usually stored as files the agent reads:

- a **skills file** (for example, `SKILL.md`) describing reusable procedures the
  agent should follow;
- an **agents-guidance file** (for example, `AGENTS.md`) telling agents how to
  operate in this repository;
- written **coding standards** the agent must adhere to.

```markdown
<!-- Example: AGENTS.md — codified knowledge every iteration consumes -->
# Agent Guidance
- Follow the existing module layout in `src/`.
- Every new function needs a unit test.
- Prefer the project's logging helper over `print`.
```

Skills/Knowledge is the difference between an agent that guesses your conventions
and one that has them in front of it.

## Connectors

**What it is.** The Connectors component is the set of integrations that let the Loop
talk to external systems — the tools your team already uses to track and ship work.

**Its purpose.** A Loop does not operate in a vacuum; it has to read work from
somewhere, write results somewhere, and tell people what is happening. Connectors
give the Loop hands and a voice in the outside world: they let it pull a task from a
tracker, open a pull request, or post an update, so the Loop is part of the real
workflow rather than an island.

**Concrete examples.** Common connectors include:

- a **source-control host** such as GitHub — to read code and open pull requests;
- an **issue tracker** — to pull the next unit of work and update its status;
- a **chat platform** such as Slack — to announce progress or escalate when human
  attention is needed.

```yaml
# Example: connectors wiring the Loop into existing tools.
connectors:
  source_control: github      # read code, open pull requests
  issue_tracker: github-issues # pull tasks, update status
  chat: slack                  # post progress, escalate to humans
```

Connectors are what make a Loop a participant in your existing systems rather than a
sealed experiment.

## Stopping Condition

**What it is.** The **Stopping Condition** is the rule that tells the Loop when to
stop iterating — or when to escalate to a human instead of continuing.

**Its purpose.** A Loop repeats, and repetition without a defined exit is dangerous:
it can spin forever, burn budget, and produce nothing useful. The Stopping Condition
gives the Loop a clear definition of "finished" (or "give up and ask for help") so it
terminates deliberately rather than running indefinitely. It is the component that
keeps a Loop from becoming a runaway process.

**Concrete examples.** A stopping condition is usually one or more explicit rules:

- **tests pass** — once the Evaluator's test suite is green, the goal is met and the
  Loop stops;
- **a rubric score reaches a threshold** — for example, stop when the rubric score is
  at least 9 out of 10;
- **a maximum number of iterations or a budget is reached** — for example, stop after
  20 iterations or once a token/cost budget is exhausted, escalating to a human
  rather than continuing.

```bash
# Example: composite stopping condition checked each iteration.
if tests_pass; then            stop "goal met"
elif [ "$iteration" -ge 20 ]; then  stop "max iterations — escalate to a human"
elif over_budget; then         stop "budget exhausted — escalate to a human"
fi
```

The first rule defines *success*; the others define *safety*. Together they
guarantee the circle stops turning.

## How the Eight Fit Together

Read in order, the eight components trace the path of a single unit of work through
the Loop. The **Automation/Trigger** starts an iteration. **Worktrees/Isolation**
gives that iteration a safe place to run. The **Generator Agent** does the work
there, guided by **Skills/Knowledge** and reaching the outside world through
**Connectors**. The **Evaluator** judges the result; **State/Memory** records what
happened so the next iteration can build on it; and the **Stopping Condition**
decides whether the Loop goes around again or finishes.

You do not need every component in every Loop — Chapter 08 will show a minimal Loop
that has only a few of them. But the eight together form the complete vocabulary of
parts. Knowing what each one is *for* is what lets you decide which to include, which
to combine, and which to leave out when you design a Loop of your own. The next
chapter puts these parts in motion, tracing how they interact across the Loop's
lifecycle.

## Key Takeaways

- A Loop is assembled from **eight components**, each one taking over a judgment or
  job that a human performs during manual prompting.
- **Automation/Trigger** decides when and how work enters the Loop — e.g., a cron
  schedule, a webhook on a new issue, or a `/loop` command.
- **Worktrees/Isolation** keeps each agent run separate and reversible so runs do
  not step on each other — e.g., git worktrees and separate branches.
- The **Generator Agent** does the actual work of producing code and edits — e.g.,
  Claude Code or a Grok agent. The Loop is *not* the agent; the agent is one part of
  it.
- The **Evaluator** (critic/verifier) checks the output from the outside so "done" is
  verified, not assumed — e.g., a separate agent running tests, a code-review agent,
  or a rubric scorer.
- **State/Memory** persists progress outside any agent's context so iterations
  accumulate — e.g., a `TODO.md`, a scratchpad, a Linear/Notion board, or files on
  disk.
- **Skills/Knowledge** codifies the project's rules and patterns so every iteration
  follows them — e.g., a `SKILL.md`, an `AGENTS.md`, or written coding standards.
- **Connectors** integrate the Loop with external tools — e.g., GitHub, an issue
  tracker, or Slack.
- The **Stopping Condition** tells the Loop when to stop or escalate — e.g., tests
  pass, a rubric score reaches a threshold, or a maximum iteration/budget limit is
  hit.
- Together the eight trace a unit of work from trigger to termination; not every Loop
  needs all eight, but knowing each one's purpose is what lets you design a Loop
  deliberately.

---
[< Previous: Origins & Historical Context](05-origins-and-history.md) | [Table of Contents](README.md) | [Next: The Loop Lifecycle >](07-loop-lifecycle.md)
