# 01. Introduction & Learning Objectives

> **In this chapter:** You will learn what this book sets out to teach, who it is
> for, and the explicit learning objectives you will achieve by the end. You will
> also get a first, plain-language picture of what a *Loop* is, how it differs
> from prompting an AI coding agent by hand, and the core components later
> chapters will unpack in depth.

## Why This Book Exists

Most engineers meet AI coding agents the same way: a chat box, a prompt, a
response, and then another prompt. You stay in the driver's seat, steering the
agent one turn at a time. That works, but it puts a hard ceiling on what you can
accomplish — your attention becomes the bottleneck. Every step forward needs you
to read the output, decide what is wrong, and type the next instruction.

**Loop Engineering** is the discipline that removes that bottleneck. Instead of
prompting an agent manually, turn by turn, you design a small control system — a
*Loop* — that prompts the agent for you. The Loop discovers work, runs the agent
in an isolated workspace, checks the result, feeds problems back in, remembers
what has happened, and decides on its own whether to keep going or stop. Your job
shifts from typing prompts to designing the system that does the prompting.

This book teaches you how to think about, reason through, and build that system.
It assumes you already use AI coding agents at a user level, and it takes you from
there to designing autonomous Loops you can trust with long, iterative work.

## What You Will Be Able to Do

After completing this book, you will be able to do the following. Each objective
is delivered by one or more specific chapters, so you can always trace a goal to
the material that fulfills it.

1. **Explain what a Loop is and how it differs from manual prompting.**
   _Delivered by Chapter 03 (and previewed below)._
2. **Place Loop Engineering in its lineage and relate it to the ReAct (Reason +
   Act) pattern.** Understand how the field grew from prompt engineering, through
   context engineering, to agent harness engineering, and finally to Loop
   Engineering — and how a Loop relates to that ReAct pattern. _Delivered by
   Chapter 04._
3. **Recount the origins of the term and the precursor Ralph Loop.** Know where
   the term came from, who shaped it, and how the simple "Ralph Loop" technique
   evolved into production-grade Loop Engineering. _Delivered by Chapter 05._
4. **Identify and explain the purpose of each Loop component.** Describe all eight
   building blocks — automation/trigger, isolation, generator agent, evaluator,
   state/memory, skills/knowledge, connectors, and stopping condition — and give a
   concrete example of each. _Delivered by Chapter 06._
5. **Trace the end-to-end Loop lifecycle and its decision points.** Walk the
   ordered flow from triggering work to deciding whether to continue, spawn a
   sub-agent, or stop. _Delivered by Chapter 07._
6. **Build a minimal Loop and evolve it toward a production-grade Loop.** Start
   from a simple bash-style loop and grow it into one with isolation, a separate
   evaluator, persisted state, and explicit stopping conditions. _Delivered by
   Chapter 08._
7. **Judge when to apply Loops and weigh their benefits.** Recognize the
   categories of work — refactoring, feature implementation, debugging — where
   Loops pay off, and articulate why. _Delivered by Chapter 09._
8. **Anticipate the risks of Loops and apply concrete mitigations.** Guard against
   runaway costs and low-quality output, and understand where human engineering
   judgment remains essential. _Delivered by Chapter 10, with the closing
   synthesis in Chapter 11._

If you can do all eight, you will have moved from *operating* an AI coding agent
to *engineering the system* that operates it for you.

## A First Picture: What Is a Loop?

The chapters ahead develop each idea carefully, but it helps to carry a working
mental model from the start. Three questions frame the whole book, and this
section answers each one just enough to orient you.

### What a Loop is

A **Loop** is an orchestration system — a small amount of software and
configuration — that drives an AI coding agent through repeated cycles of work
without a human prompting each step. It is not the agent itself. The agent is one
*part* of the Loop. The Loop is the surrounding machinery that decides what the
agent should do, runs it, judges the result, and repeats until the work meets a
defined goal or hits a defined limit.

The name is literal: the system *loops*. It produces output, evaluates that
output, and uses the evaluation to drive the next iteration — over and over — the
way you would if you were supervising the agent yourself, except that the Loop
does the supervising.

### How it differs from manual prompting

With **manual, turn-by-turn prompting**, a human is the control system. You read
each response, hold the goal in your head, judge whether the output is good
enough, and craft the next prompt. Progress stops the moment you step away.

With a **Loop**, that control system is designed once and then runs on its own.
The decisions you used to make in your head — *Is this good enough? What should
happen next? Are we done?* — become explicit, encoded rules: an evaluator that
checks the output, a state file that remembers progress, and a stopping condition
that ends the run. The practitioner's role shifts from operator to system
architect. You design the loop; the loop does the prompting.

### The core components

Every Loop, from the simplest to the most elaborate, is assembled from the same
small set of building blocks. You will study each one in depth in Chapter 06, but
here is the cast of characters so the rest of the book reads as a coherent whole:

- **Automation / Trigger** — decides *when* and *how* work enters the Loop (for
  example, a schedule, an event such as a new issue, or a command you run).
- **Worktrees / Isolation** — keeps each agent run separate and reversible so
  mistakes do not contaminate your main workspace.
- **Generator Agent** — the AI coding agent that actually produces the output.
- **Evaluator** — verifies that output, by running tests, reviewing code, or
  scoring against a rubric.
- **State / Memory** — persists progress across iterations so the Loop remembers
  what has been done.
- **Skills / Knowledge** — supplies reusable guidance and standards the agent
  should follow.
- **Connectors** — integrate the Loop with external systems such as source
  control, issue trackers, and chat platforms.
- **Stopping Condition** — the rule that ends the Loop, such as tests passing, a
  quality threshold, or a budget limit.

Hold this picture loosely for now. The point is simply that a Loop is *built from
parts*, that those parts replace the judgments you would otherwise make by hand,
and that designing them well is the craft this book teaches.

## How the Rest of the Book Is Organized

The chapters follow a deliberate arc. The early chapters establish *what* and
*why*: who the book is for (Chapter 02), a precise definition of the discipline
(Chapter 03), its lineage and intellectual roots (Chapter 04), and its origins
(Chapter 05). The middle chapters dissect the *parts* and how they interact:
the eight components (Chapter 06) and the end-to-end lifecycle (Chapter 07). The
practical chapter shows you how to *build* one (Chapter 08). The closing chapters
cover *when* to use Loops, their *benefits* and *risks*, and a final synthesis
(Chapters 09–11). A glossary and a references section round out the book.

You can read straight through for a complete course, or use the table of contents
to jump to a specific topic.

## Key Takeaways

- **Loop Engineering** is the practice of designing a control system — a *Loop* —
  that prompts and orchestrates AI coding agents autonomously, instead of
  prompting them manually one turn at a time.
- A **Loop** is the orchestration system, not the agent; the agent is just one
  component within it.
- The defining difference from manual prompting is *who controls the cycle*: with
  a Loop, the judgments a human would make by hand become explicit, encoded rules,
  shifting your role from operator to system architect.
- Every Loop is assembled from the same core components: automation/trigger,
  isolation, a generator agent, an evaluator, state/memory, skills/knowledge,
  connectors, and a stopping condition.
- This book has eight explicit learning objectives, each delivered by specific
  chapters, taking you from understanding a Loop to building and judging one.

---
[Table of Contents](README.md) | [Next: Audience & Prerequisites >](02-audience-and-prerequisites.md)
