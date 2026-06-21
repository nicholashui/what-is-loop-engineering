# Glossary

> **In this chapter:** This glossary is the single source of truth for the key
> terms used throughout the book. Each term is defined exactly once here; every
> chapter uses these terms with the meaning given below. Entries are ordered
> alphabetically, and acronyms are spelled out on first mention.

**Agent Harness Engineering**
: The third stage in the lineage that leads to Loop Engineering: the practice of
building a reliable execution environment — a Harness — for a single agent run.
It adds action and feedback within one session (tools, observed results,
multi-step control) on top of well-curated context.

**AI Coding Agent**
: A large-language-model-based system that can reason about a software task and
take actions on it — running commands, reading and editing files, and checking
results — rather than only producing text. Within a Loop, an AI coding agent
usually plays the role of Generator Agent.

**Author-Asserted Claim**
: A factual statement — a dated event, a named attribution, or a direct quote —
drawn from supplied source material that the book presents transparently as
requiring independent verification by the Reader, because it could not be checked
against live sources in the authoring environment. Such claims carry an inline
marker and are collected in the References file.

**Automation/Trigger**
: The Loop component that decides when and how work enters the Loop. Examples
include a cron schedule, a webhook that fires on a new issue, and a slash-style
command invoked on demand.

**Book**
: The complete educational deliverable, composed of ordered Markdown chapter
files plus supporting files (the table of contents, this glossary, and the
references).

**Chapter**
: A single top-level Markdown file covering one major topic area of the Book.

**Connectors**
: The Loop component that integrates the Loop with external systems. Examples
include source-control hosts, issue trackers, and chat platforms.

**Context Engineering**
: The second stage in the lineage that leads to Loop Engineering: the practice of
deliberately assembling the information placed in the model's context window —
relevant files, retrieved documents, prior conversation, examples, and
instructions — so the model reasons over the right material. It adds the right
information to the well-chosen words of Prompt Engineering.

**Evaluator (Critic / Verifier)**
: The Loop component that independently checks the Generator Agent's output from
outside the agent. An Evaluator may be a test suite, a code-review or "critic"
agent, or a Rubric scorer. It is what lets a Loop judge whether work is actually
good rather than trusting the agent's own assessment.

**Generator Agent**
: The Loop component that produces the actual output — code, edits, or other
artifacts — within a Loop. It is typically an AI coding agent such as Claude Code
or a Grok agent.

**Harness**
: The reliable execution environment for a single agent run: it supplies the
agent with tools, feeds tool results back into its context, manages the
back-and-forth of a multi-step session, and keeps that run stable. The Loop's
orchestration layer is built on top of the Harness.

**Iteration**
: One pass through the Loop — discovering or receiving work, running an agent,
evaluating the output, persisting state, and deciding what to do next. A Loop
drives a goal forward across many iterations rather than in a single run.

**Loop**
: The orchestration system built by a loop engineer. It discovers or triggers
work, spawns agents in isolation, verifies their output, persists state across
runs, and decides whether to continue, spawn a sub-agent, or stop.

**Loop Engineering**
: The practice of designing a control system that prompts, observes, verifies,
and iterates AI coding agents autonomously, rather than prompting them manually
turn-by-turn. It is the top stage of the lineage that runs from Prompt
Engineering through Context Engineering and Agent Harness Engineering.

**Manual / Turn-by-turn Prompting**
: The conventional way of working with an AI coding agent, in which a human issues
one instruction, reads the result, and issues the next instruction — supervising
every step. Loop Engineering replaces this hands-on supervision with a system that
prompts and iterates the agent on its own.

**Prompt Engineering**
: The first stage in the lineage that leads to Loop Engineering: the practice of
crafting the wording of a single instruction — phrasing, examples, output format,
role framing — to steer a model toward a good answer for one prompt and one
response.

**Ralph Loop (Ralph Wiggum technique)**
: The precursor technique that Loop Engineering evolves: a simple shell loop that
repeatedly feeds an agent fresh context plus the previous run's output and errors
and re-runs it until the task is done. It demonstrates the core idea of running an
agent in a loop with results fed back in, without isolation, an independent
Evaluator, or persisted state.

**ReAct (Reason + Act)**
: The agent pattern that interleaves reasoning steps with actions: the agent
thinks about what to do next, takes a concrete action through a tool, observes the
result, reasons again, and continues until the task is done. It is the conceptual
ancestor of the Loop, which lifts this reason-act cycle out of a single agent
conversation and into an external, controllable system.

**Reader**
: The target learner — a software engineer or technical practitioner who is
familiar with AI coding agents at a user level but new to orchestrating them
autonomously.

**Rubric**
: An explicit set of scored evaluation criteria against which a Generator Agent's
output is judged. An Evaluator can use a rubric to assign a score, and a rubric
score threshold can serve as a Stopping Condition.

**Skills/Knowledge**
: The Loop component that supplies reusable guidance and standards to agents.
Examples include a skills file, an agents-guidance file, and coding standards.

**Slop**
: High-volume, low-quality output produced when agents run without strong
verification. It is a primary risk of Loops, mitigated by tests and critic agents
that gate what the Loop accepts.

**State/Memory**
: The Loop component that persists progress across iterations so work is not lost
when any single run ends. Examples include a TODO file, a scratchpad, a project
board, and files on disk.

**Stopping Condition**
: A rule that terminates a Loop. Examples include tests passing, a Rubric score
reaching a threshold, and a maximum number of iterations or an exhausted budget.

**Sub-agent**
: A secondary agent that a Loop spawns to handle a focused portion of the work —
for example, a specialized task branched off from the main run. Spawning a
sub-agent is one of the three outcomes (alongside continue and stop) at the Loop's
decision point.

**Token Budget**
: A cap on the number of tokens (and therefore the cost) a Loop may consume.
Tight budgets, paired with Stopping Conditions, mitigate the risk of uncontrolled
token-cost growth.

**Worked Example**
: A concrete, end-to-end illustration — code, configuration, or an annotated
walkthrough — that demonstrates a concept in practice. The book presents a minimal
Ralph-style example and a production-grade example.

**Worktrees/Isolation**
: The Loop component that keeps each agent run isolated and reversible. Examples
include git worktrees and separate branches, which let agent work proceed without
disturbing the main line of development.

---
[< Previous: Curating, Operating & Scaling a Loop Library](14-operating-a-loop-library.md) | [Table of Contents](README.md) | [Next: References & Author-Asserted Claims >](references.md)
