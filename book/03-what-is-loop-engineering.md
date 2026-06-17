# 03. What Is Loop Engineering

> **In this chapter:** You will learn a precise definition of *Loop Engineering*
> and of the *Loop* it produces, see exactly how that practice differs from
> prompting an AI coding agent manually, one turn at a time, and build a durable
> intuition for what a Loop is and why it earns its name. This is the chapter that
> delivers the book's first and most foundational learning objective: being able
> to explain what a Loop is and how it differs from manual prompting.

## A Precise Definition

**Loop Engineering** is the practice of designing a control system that prompts,
observes, verifies, and iterates AI coding agents autonomously — rather than
prompting them manually, turn by turn.

That single sentence carries a lot of weight, so it is worth unpacking each part:

- **A control system.** The thing you build is a system, not a single clever
  prompt. It is a small amount of software and configuration whose job is to keep
  an agent productively at work toward a goal.
- **That prompts, observes, verifies, and iterates.** These four verbs are the
  cycle. The system issues instructions to an agent (*prompts*), watches what the
  agent produces (*observes*), checks whether that output is actually good
  (*verifies*), and uses the result to drive the next attempt (*iterates*).
- **AI coding agents.** The system orchestrates one or more AI coding agents — the
  same kind of assistant you already prompt by hand today.
- **Autonomously.** Once you have designed it, the system runs the cycle on its
  own, without a human composing each instruction.

The artifact this practice produces has its own name. A **Loop** is the
orchestration system built by a loop engineer: it discovers work, spawns agents,
verifies their output, persists state, and decides whether to continue or stop.
Throughout this book, *Loop Engineering* names the **discipline** — the way of
thinking and designing — and *Loop* names the **thing you build**. Keep the two
straight and the rest of the book stays clear: you do Loop Engineering; you end up
with a Loop.

One clarification that prevents a common confusion: the Loop is **not** the agent.
The AI coding agent — what this book calls a **Generator Agent**, the component
that actually produces code and edits — is just *one part* of the Loop. The Loop
is the surrounding machinery that decides what the agent should do, runs it, judges
the result, and repeats. When this book says "the Loop decides" or "the Loop
verifies," it means that surrounding machinery, not the agent inside it.

## The Contrast: Manual Prompting vs. a Loop

The fastest way to understand Loop Engineering is to set it directly against the
thing it replaces.

### How manual, turn-by-turn prompting works

When you prompt an AI coding agent by hand, **you are the control system.** Consider
what you actually do across a session:

1. You hold the goal in your head — what "done" looks like.
2. You type a prompt describing the next step.
3. You read the agent's output and decide whether it is any good.
4. If it is wrong, you compose a correction and prompt again.
5. You remember what has already been tried so you do not go in circles.
6. You decide when the work is finished — or when it is time to give up.

Every one of those steps runs on *your* attention. The agent generates; you supply
all the judgment. This is powerful for short, exploratory work, but it has a hard
ceiling: **progress stops the moment you step away.** You are, in the most literal
sense, babysitting the agent — present for every turn, because every turn needs a
decision only you are making.

### How a Loop works

Loop Engineering takes that list of things you do in your head and makes each one
*explicit and encoded*, so a system can do them instead. The judgments do not
disappear; they move out of your attention and into the design of the Loop:

| In manual prompting, *you*… | In a Loop, this becomes… |
|------------------------------|---------------------------|
| hold the goal in your head | a written goal and evaluation criteria |
| type each next prompt | an automated trigger and prompting step |
| judge whether output is good | an **Evaluator** (tests, a critic agent, a rubric) |
| remember what's been tried | persisted **state / memory** on disk |
| decide when to stop | an explicit **stopping condition** |

This is the heart of the matter, and it is worth stating as plainly as possible:
**you stop being the babysitter, and you build the loop that prompts, observes,
verifies, and iterates.** Your role shifts from *operator* — the person typing the
next instruction — to *system architect* — the person who designs the system that
types the instructions. You spend your judgment once, up front, in the design,
rather than continuously, turn after turn.

The difference is not that a Loop is "smarter" than you prompting carefully. The
difference is *who runs the cycle*. A Loop runs it without you, which means it can
run while you sleep, run across hundreds of iterations, and run on the long,
grinding, iterative tasks that are too tedious to babysit by hand.

## Building the Intuition: Why "Loop"?

The name is not a metaphor; it is a description of the mechanism. The system
**loops** — it repeats a cycle — and that repetition is exactly what makes it
valuable.

Picture the cycle as a circle that turns again and again:

```text
        ┌─────────────────────────────────────────┐
        │                                           │
        ▼                                           │
   ┌─────────┐   ┌─────────┐   ┌──────────┐         │
   │ prompt  │──▶│ observe │──▶│  verify  │─────────┘
   │ (agent  │   │ (read   │   │ (is it   │   iterate:
   │  acts)  │   │ output) │   │  good?)  │   feed result
   └─────────┘   └─────────┘   └──────────┘   back in
                                     │
                                     ▼
                              ┌────────────┐
                              │   done?    │  ── yes ──▶ stop
                              └────────────┘
```

Each time around the circle, the Loop prompts the agent, observes what comes back,
verifies it against the goal, and then makes a decision: if the work is not yet
good enough, it feeds the result — including any errors or failed checks — back in
and goes around again; if the work meets the goal (or hits a limit), it stops.

This is precisely the loop you run in your own head during a manual session. You
prompt, you read, you judge, you correct, you repeat until you are satisfied. Loop
Engineering simply lifts that cycle out of your head and turns it into a system
that can run it on its own. Once you see the cycle clearly, the whole discipline
follows: everything in the chapters ahead is about designing each part of this
circle well — how work enters it, how the agent runs safely inside it, how the
verification step decides "good enough," how progress is remembered between turns,
and how the Loop knows when to stop.

Hold onto this image. A Loop is a circle that turns on its own, and Loop
Engineering is the craft of designing a circle worth letting turn.

## Key Takeaways

- **Loop Engineering** is the practice of designing a control system that prompts,
  observes, verifies, and iterates AI coding agents *autonomously*, instead of
  prompting them manually one turn at a time.
- A **Loop** is what that practice produces: the orchestration system that
  discovers work, spawns agents, verifies output, persists state, and decides
  whether to continue or stop. The Loop is *not* the agent — the **Generator
  Agent** is just one component inside it.
- The defining contrast with manual prompting is *who runs the cycle*. In manual
  prompting **you** are the control system, supplying all the judgment turn by
  turn; in a Loop those judgments become explicit, encoded parts — a goal, an
  **Evaluator**, persisted state, a stopping condition — so the system runs the
  cycle without you.
- Put plainly: **you stop being the babysitter and build the loop that prompts,
  observes, verifies, and iterates.** Your role shifts from *operator* to *system
  architect*.
- The name is literal: a Loop *loops*, repeating a prompt → observe → verify →
  iterate cycle until the work meets its goal or hits a limit. That repeating
  circle is the mental model to carry through the rest of the book.

---
[< Previous: Audience & Prerequisites](02-audience-and-prerequisites.md) | [Table of Contents](README.md) | [Next: Lineage & the ReAct Connection >](04-lineage-and-react.md)
