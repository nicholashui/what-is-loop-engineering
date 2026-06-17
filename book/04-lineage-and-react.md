# 04. Lineage & the ReAct Connection

> **In this chapter:** You will learn where Loop Engineering came from — the
> four-stage lineage that runs from *Prompt Engineering* to *Context Engineering*
> to *Agent Harness Engineering* to *Loop Engineering*, and what each stage adds
> beyond the one before it. You will also learn how a Loop relates to the **ReAct
> (Reason + Act)** pattern, how a Loop lifts the reason-act cycle out of a single
> agent conversation and into a controllable system you design, and how the
> precursor *Ralph Loop* technique sets the stage for everything that follows.

## A Field That Kept Climbing the Stack

Loop Engineering did not appear from nowhere. It is the latest step in a steady
progression — a field that, year after year, has pushed the boundary of *what the
human designs* up and out, away from individual words and toward whole systems.

Each stage in this lineage solves the bottleneck left by the stage before it. When
one layer becomes reliable enough to take for granted, attention moves to the next
layer up. Understanding this climb is the fastest way to understand *why* Loop
Engineering looks the way it does: it is the natural consequence of everything
underneath it finally working well enough to build on.

The lineage has four stages:

```text
  Prompt          Context          Agent Harness          Loop
  Engineering ──▶ Engineering  ──▶ Engineering       ──▶  Engineering
  (the words)     (the inputs)     (one reliable run)      (many runs,
                                                            orchestrated)
```

Let us walk up the stack one stage at a time.

### Stage 1 — Prompt Engineering: crafting the words

The first thing anyone learns when working with a language model is that *how you
ask* changes *what you get*. **Prompt Engineering** is the practice of crafting the
wording of a single instruction to steer the model toward a good answer — choosing
phrasing, giving examples, specifying a format, assigning a role ("you are an
expert reviewer").

The unit of work here is **one prompt and one response.** The engineer's craft is
verbal: find the words that make this one turn succeed.

- **What it adds:** the recognition that the model's output is controllable through
  deliberate input, and a vocabulary of techniques (few-shot examples, step-by-step
  instructions, role framing) for that control.
- **What it leaves unsolved:** a single well-worded prompt still operates on
  whatever information happens to be in the message. The moment a task needs facts
  the model was never given — your codebase, your docs, yesterday's decision — clever
  wording is not enough.

### Stage 2 — Context Engineering: curating the inputs

The next bottleneck is *what the model knows when it answers.* **Context
Engineering** is the practice of deliberately assembling the information placed in
the model's context window: the relevant files, retrieved documents, prior
conversation, examples, and instructions — so the model reasons over the right
material, not just well-chosen words.

The unit of work expands from a single prompt to **the whole input the model sees.**
The craft shifts from phrasing to *selection and assembly*: what to retrieve, what
to include, what to leave out, and how to fit it into a limited window.

- **What it adds beyond Prompt Engineering:** it treats the entire context window as
  the design surface, not just the instruction sentence. Good wording over the wrong
  information still fails; Context Engineering makes sure the right information is
  present.
- **What it leaves unsolved:** even a perfectly worded prompt over perfectly curated
  context is still *one turn*. Real work requires the model to *act* — run a command,
  read a file, edit code, check a result — and then act again based on what happened.
  A static input cannot do that.

### Stage 3 — Agent Harness Engineering: making one run reliable

To let a model *act*, you wrap it in a **harness**: the reliable execution
environment for a single agent run. The harness gives the model tools (a shell, a
file editor, the ability to run tests), feeds tool results back into its context,
manages the back-and-forth of a multi-step session, and keeps that whole run stable
and reproducible. **Agent Harness Engineering** is the practice of building this
environment well.

The unit of work expands again — from a single input to **one complete agent run**:
a session in which the model reasons, takes actions through tools, observes the
results, and continues until it believes the task is done.

- **What it adds beyond Context Engineering:** *action and feedback within a single
  session.* The model is no longer limited to producing text over fixed inputs; it
  can change the world (edit files, run commands) and respond to what it observes.
  The harness makes that loop of acting-and-observing dependable for one run.
- **What it leaves unsolved:** a harness makes *one* run reliable, but a single run
  is still bounded by one conversation. It ends. Its context fills up. Nobody outside
  the agent decides whether the result was actually good, whether to try again from a
  clean slate, or whether to keep going for another hundred iterations. The judgment
  and the persistence still live with the human watching the session.

### Stage 4 — Loop Engineering: orchestrating many runs

This is where this book begins. **Loop Engineering** is the practice of designing a
control system that prompts, observes, verifies, and iterates AI coding agents
autonomously. It sits *on top of* the harness: where Agent Harness Engineering makes
a single agent run reliable, Loop Engineering builds the orchestration layer that
runs agents over and over, judges their output from the outside, remembers progress
between runs, and decides whether to continue, branch off a sub-agent, or stop.

The unit of work expands one final time — from a single run to **many runs driven
toward a goal.** The craft is no longer verbal or even informational; it is
*systemic*. You are designing the machine that operates the agent.

- **What it adds beyond Agent Harness Engineering:** *external control and
  iteration across runs.* A harness trusts the agent to decide when it is finished;
  a Loop adds an independent **Evaluator** that checks the work, **state** that
  outlives any single run, and an explicit **stopping condition**. It turns "one good
  run" into "keep producing good runs until the goal is met."

Notice the through-line. At every stage, the human's design target moves up: from
the *words*, to the *inputs*, to the *single run*, to the *system that runs runs*.
Loop Engineering is simply the top of that climb so far — the point at which the
thing you engineer is no longer a message to an agent, but a system that manages the
agent for you.

## The ReAct Connection

To understand the *mechanism* a Loop externalizes, it helps to name the pattern at
the core of every modern coding agent: **ReAct**, which stands for **Reason +
Act**.

The ReAct pattern interleaves two kinds of steps. The agent *reasons* — thinks
through what to do next in natural language — and then *acts* — takes a concrete
action through a tool, such as running a command or editing a file. It then observes
the result of that action, reasons again in light of what it learned, acts again,
and continues interleaving thought and action until it concludes the task is done.
This reason-act-observe rhythm is what makes an agent feel less like an autocomplete
and more like a worker: it does not just answer, it *works the problem* step by step.

ReAct is a conceptual ancestor of the Loop. Both are built on the same fundamental
shape — a cycle of thinking, doing, and checking the result. But there is a crucial
difference in *where that cycle lives* and *who controls it.*

### Inside a single conversation vs. an external system

In a plain ReAct agent, the reason-act cycle lives **inside one conversation.** The
agent carries it out within a single session, in its own context window, judging for
itself when to stop. That is powerful, but it inherits all the limits of a single
run: the conversation eventually ends, the context window fills, and the agent is
both the worker *and* the only judge of its own work. There is no independent voice
asking "is this actually correct?" and no memory that survives once the session
closes.

A Loop **lifts that reason-act cycle out of the single agent conversation and turns
it into an external, controllable system.** The same think → do → check rhythm
still runs — but now it runs *around* the agent rather than only inside it:

| In a single ReAct conversation… | In a Loop, the cycle is lifted so that… |
|---------------------------------|------------------------------------------|
| the agent reasons internally | the Loop frames the goal and prompts the agent |
| the agent acts via its tools | the agent run (the harness) performs the action |
| the agent judges its own result | an external **Evaluator** verifies the output |
| context is the only memory | **state** is persisted outside any one run |
| the agent decides it is "done" | the Loop applies an explicit **stopping condition** |

The point is not that the inner ReAct cycle disappears — each agent run still
reasons and acts internally. The point is that a Loop wraps an *outer* reason-act
cycle around the whole run, and crucially, **moves the control out of the agent and
into a system you design.** The reasoning about "what next" and the acting of
"run the agent again" are no longer trapped in one conversation that you have to
supervise; they become parts of a system that can run on its own, judge from the
outside, and persist across as many runs as the goal requires.

If Chapter 03 gave you the image of a circle that turns on its own, ReAct is the
shape of that circle — reason, act, observe — and a Loop is what you get when you
take that circle out of a single chat window and build a machine to turn it.

## The Ralph Loop: The Precursor That Pointed the Way

There is one more piece of lineage to introduce, and it is the most direct ancestor
of all: the **Ralph Loop**.

The Ralph Loop is the precursor technique that Loop Engineering evolves. In its
original form it is strikingly simple: a basic shell loop that repeatedly feeds an
agent fresh context plus the previous run's output and errors, and runs it again and
again until the task is done. There is no elaborate framework — just the insight that
if you keep restarting an agent with what it produced last time, it can grind its way
toward a goal across many iterations rather than giving up at the end of one.

That simple bash loop is the seed of everything in this book. It demonstrated the
core idea — *run an agent in a loop, feeding results back in* — in the most minimal
way imaginable. Loop Engineering is what you get when you take that seed seriously
and grow it into a production-grade system: you add isolation so each run is safe,
an independent Evaluator so "done" is judged rather than assumed, persisted state so
progress is real and not just whatever survives in a context window, and explicit
stopping conditions so the Loop knows when to quit. The Ralph Loop is the proof of
concept; Loop Engineering is the engineering.

We are introducing the Ralph Loop only briefly here, as the conceptual bridge from
"a clever shell trick" to "a designed system." Its fuller story — who originated it,
when, and how the term *Loop Engineering* itself came to be named and popularized —
is the subject of the next chapter.

## Key Takeaways

- Loop Engineering is the top of a four-stage lineage, where each stage moves the
  human's design target further up the stack: **Prompt Engineering** (craft the
  *words* of one instruction) → **Context Engineering** (curate the *inputs* in the
  context window) → **Agent Harness Engineering** (make *one agent run* reliable
  through tools and feedback) → **Loop Engineering** (orchestrate *many runs* with
  external verification, persisted state, and stopping conditions).
- Each stage adds what the previous one lacked: Context Engineering adds the right
  information to good wording; Agent Harness Engineering adds action-and-feedback to
  static inputs; Loop Engineering adds external control and iteration across runs to
  a single reliable run.
- **ReAct** stands for **Reason + Act** — the pattern in which an agent interleaves
  reasoning steps with tool actions, observing results and continuing until the task
  is done. It is the conceptual ancestor of the Loop.
- A Loop **lifts the reason-act cycle out of a single agent conversation** and turns
  it into an external, controllable system: the inner agent still reasons and acts,
  but the Loop wraps an outer cycle around it that prompts, verifies from the
  outside, persists state, and decides when to stop — moving control out of the agent
  and into a system you design.
- The **Ralph Loop** — a minimal shell loop that repeatedly re-runs an agent with
  fresh context plus prior output and errors — is the direct precursor that Loop
  Engineering evolves into a production-grade system. Its full origin story comes
  next.

---
[< Previous: What Is Loop Engineering](03-what-is-loop-engineering.md) | [Table of Contents](README.md) | [Next: Origins & Historical Context >](05-origins-and-history.md)
