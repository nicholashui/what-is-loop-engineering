# 05. Origins & Historical Context

> **In this chapter:** You will learn where the term *Loop Engineering* came
> from and who shaped it — the June 2026 post credited with popularizing the
> name, the practitioner voices that captured the shift in mindset, and the
> minimal *Ralph Loop* technique that came first and pointed the way. Because
> these are recent, attributed claims, you will also learn how this book flags
> them so you can verify them independently.

## A Note on Sources Before We Begin

This chapter is about people, dates, and quotes — the kind of recent history
that is easy to get slightly wrong and hard to check after the fact. This book
was written without live access to the internet, so every dated event, named
attribution, and direct quotation in this chapter is presented as an
**Author-Asserted Claim** *(Author-Asserted; verify independently)*. Wherever you
see an inline marker like `[^aac-osmani-2026]`, it means: *this is drawn from the
supplied source material and you, the reader, should confirm it against the
original source yourself.* Every such marker resolves to a full entry in the
[References](references.md) chapter. Treat the *ideas* in this chapter as solid;
treat the *attributions* as pointers to verify.

With that understood, let us trace how a clever shell trick became a named
discipline.

## The Term Gets a Name: Addy Osmani, June 2026

Ideas often exist in practice long before they have a name. The practice of
running coding agents in a loop was already spreading among early adopters when
the label *Loop Engineering* crystallized and went mainstream.

The popularization of the term "Loop Engineering" is attributed to a post by
Addy Osmani published on June 7, 2026 [^aac-osmani-2026] *(Author-Asserted;
verify independently)*. The significance of that post was less that it invented
something new and more that it *named* something that was already happening: it
gave a crisp label to the emerging practice of designing systems that prompt and
orchestrate AI coding agents, rather than prompting those agents by hand,
turn by turn. A name matters. Once the practice had one, scattered experiments
could be recognized as instances of a single, nameable discipline — and a
discipline with a name is something people can study, compare notes on, and
deliberately improve.

That naming moment is the anchor date for this short history. What gave it
traction, though, were the practitioners who had already felt the shift in their
own daily work and could describe it vividly.

## The Shift in Mindset: Two Practitioner Voices

The clearest evidence that Loop Engineering was a genuine change — and not just a
rebranding — came from engineers describing how their *own* relationship to
coding agents had changed. Two perspectives, both presented here as
Author-Asserted Claims, capture the shift especially well.

### Peter Steinberger: from prompting to designing loops

One supporting perspective is attributed to Peter Steinberger
[^aac-steinberger], who framed the change as a move away from prompting and
toward *building the thing that prompts*:

> "You shouldn't be prompting coding agents anymore. You should be designing
> loops that prompt your agents."

The value of this framing is its bluntness. It does not describe Loop Engineering
as an *optional upgrade* to manual prompting; it describes manual prompting as the
thing you are meant to *stop doing*. The unit of work you design is no longer the
message — it is the loop that produces the messages. That is the entire thesis of
this book compressed into two sentences.

### Boris Cherny: "my job is to write loops"

A second supporting perspective is attributed to Boris Cherny, Head of Claude
Code at Anthropic [^aac-cherny], who described the same shift from the inside of
his own workflow:

> "I don't prompt Claude anymore. I have loops running that prompt Claude and
> figuring out what to do. My job is to write loops."

What makes this perspective striking is *who* is offering it: someone working at
the center of modern coding-agent tooling describing his job not as *using* the
agent but as *writing the loops that use it*. The phrase "my job is to write
loops" is a concise statement of the role change this book keeps returning to —
the practitioner moves from operator to system designer. You are no longer the
one in the chair issuing each instruction; you are the one who built the system
that issues them.

Taken together, these two voices describe the same transition from two angles —
Steinberger as a prescription ("design loops, don't prompt") and Cherny as a
description of lived practice ("I write loops now"). Both point back to a
technique that demonstrated the idea in its rawest form, and which predates the
name itself.

## The Precursor: Geoffrey Huntley's Ralph Loop

Before the term existed, the technique did. The most direct ancestor of Loop
Engineering is the **Ralph Loop** — also called the "Ralph Wiggum technique" —
attributed to Geoffrey Huntley in 2025 [^aac-huntley-2025].

The Ralph Loop is disarmingly simple. In its original form it is little more than
a basic bash loop: it repeatedly feeds an AI coding agent fresh context plus the
previous run's output and errors, and runs the agent again and again until the
task is done. There is no orchestration framework, no elaborate machinery — just
the core insight that *if you keep restarting an agent with what it produced last
time, it can grind its way toward a goal across many iterations* instead of giving
up at the end of a single run. The playful "Ralph Wiggum" name fits the spirit of
the trick: it is almost naively straightforward, and yet it works.

Conceptually, the bare loop looks like this:

```bash
# The Ralph Loop, in spirit: re-run the agent until the task is done,
# feeding the previous output and errors back in as fresh context.
while ! task_is_done; do
  agent --context "$(cat prompt.md previous_output.txt errors.txt)" \
        > previous_output.txt 2> errors.txt
done
```

That tiny loop is the seed of everything in this book. It established the
foundational move — *run an agent in a loop, feeding results back in* — in the
most minimal way imaginable, with nothing else attached.

### From Ralph Loop to Loop Engineering

Loop Engineering is what you get when you take that seed seriously and grow it
into something production-grade. The Ralph Loop is the proof of concept; Loop
Engineering is the engineering discipline built on top of it. The evolution adds
the parts the bare loop deliberately leaves out:

- **Isolation**, so each run is safe and reversible rather than scribbling over a
  shared workspace.
- **An independent Evaluator**, so "done" is *judged* by something other than the
  agent itself rather than simply assumed.
- **Persisted state**, so progress is real and durable instead of being whatever
  happens to survive in a context window.
- **Explicit stopping conditions**, so the loop knows when to quit — on passing
  tests, a quality threshold, or a budget cap — instead of spinning indefinitely.

In other words, the Ralph Loop supplies the *intuition* (keep re-running the agent
with feedback) and Loop Engineering supplies the *structure* (do it safely,
verifiably, durably, and with a defined exit). The rest of this book is, in large
part, the story of adding that structure — component by component in the chapters
that follow.

This is why the historical thread matters: the technique came first and proved the
idea would work; the practitioner voices articulated the mindset shift; and the
naming moment gave the whole practice an identity. Together they explain not just
*what* Loop Engineering is, but *how* it arrived.

## Key Takeaways

- The popularization of the term **"Loop Engineering"** is attributed to Addy
  Osmani's post of June 7, 2026 [^aac-osmani-2026]; its importance was in
  *naming* an already-emerging practice so it could be studied and improved as a
  discipline.
- Two practitioner perspectives capture the mindset shift. **Peter Steinberger**
  [^aac-steinberger] urged designing loops that prompt your agents rather than
  prompting agents directly, and **Boris Cherny**, Head of Claude Code at
  Anthropic [^aac-cherny], described his own job as writing the loops that prompt
  the agent — both framing the move from operator to system designer.
- The direct precursor is the **Ralph Loop** ("Ralph Wiggum technique"),
  attributed to Geoffrey Huntley in 2025 [^aac-huntley-2025]: a simple bash loop
  that re-runs an agent with fresh context plus prior output and errors until the
  task is done.
- **Loop Engineering evolves the Ralph Loop** into a production-grade discipline
  by adding isolation, an independent evaluator, persisted state, and explicit
  stopping conditions — the bare loop is the proof of concept; the engineering is
  the structure built around it.
- Every dated event, named attribution, and direct quote in this chapter is an
  **Author-Asserted Claim** marked with an inline `[^aac-...]` reference and
  collected in the [References](references.md) chapter; the reader should verify
  these claims independently.

---
[< Previous: Lineage & the ReAct Connection](04-lineage-and-react.md) | [Table of Contents](README.md) | [Next: Anatomy of a Loop: The Components >](06-anatomy-of-a-loop.md)
