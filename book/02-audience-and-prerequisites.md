# 02. Audience & Prerequisites

> **In this chapter:** You will learn who this book is written for, the
> background knowledge it assumes you already have, and how it handles any
> concept that goes beyond that background. By the end you will be able to
> confirm that the material is pitched at your level before you invest in it.

## Who This Book Is For

This book is written for **software engineers and technical practitioners who
already use AI coding agents but are new to orchestrating them autonomously.**

If you have opened an AI coding assistant, described a change, read its response,
and typed a follow-up prompt to steer it, you are exactly the reader this book
has in mind. You are comfortable being the human in the loop — reading each
result, deciding what is wrong, and prompting the next step by hand. What you
have not yet done is hand that supervising role to a system you designed.

That is the leap this book is built to support. It does not teach you what an AI
coding agent is or how to prompt one well; it assumes you can already do that. It
takes you from *operating* an agent one turn at a time to *engineering the Loop*
that operates it for you. The center of gravity is the orchestration system
around the agent — discovering work, running the agent in isolation, verifying
the result, remembering progress, and deciding whether to continue or stop — not
the craft of writing a single good prompt.

A few readers who will feel at home here:

- **Working engineers** who lean on AI coding agents day to day and want to
  reclaim their attention from turn-by-turn babysitting.
- **Tech leads and platform engineers** evaluating whether autonomous agent
  workflows are worth adopting on their team, and wanting an honest picture of
  the benefits *and* the risks.
- **Tinkerers and early adopters** who have run a quick agent loop in a shell
  script and want to understand how to evolve it into something dependable.

If you have never used an AI coding agent at all, you can still read along, but a
short period of hands-on experience with one will make every chapter land more
firmly.

## What You Should Already Know

To keep the book focused on Loop Engineering itself, it assumes a modest amount of
prior knowledge. None of it is advanced; all of it is the kind of background a
practicing engineer typically already has.

### Version control (Git)

You should be comfortable with the basics of version control, and with **Git** in
particular: commits, branches, and the idea that work can be isolated on a branch
and merged back later. Loops lean heavily on isolation — running each agent
attempt somewhere separate so a bad result never contaminates your main code — and
the book uses Git concepts such as branches and worktrees to explain how that
isolation is achieved. (A *worktree* is a Git feature for checking out more than
one branch into separate working directories at the same time; the book reminds
you of this where it matters, so a passing familiarity with branches is enough to
start.)

### Command-line usage

You should be able to read and run commands in a terminal. The worked examples
begin with a simple shell loop and build up from there, so being able to follow a
short `bash` script — variables, a loop, invoking a command — is assumed. You do
not need to be a shell-scripting expert; you need to be unintimidated by a
terminal and able to recognize what a handful of lines are doing.

### Interacting with AI coding agents

You should have hands-on experience using at least one AI coding agent — for
example, prompting it to write or modify code and reading back what it produced.
This is the most important prerequisite, because the whole book reframes that
familiar experience: the thing you do by hand today is the thing a Loop will do
on your behalf. You need a felt sense of what an agent does well, where it goes
wrong, and why a human currently has to stay in the loop — so that you can
appreciate what it takes to remove that human.

You do **not** need any of the following: prior experience building agent
frameworks, knowledge of a specific orchestration tool or vendor, machine-learning
or model-training background, or any particular programming language beyond the
ability to read the short, commented examples in the practical chapters.

## How This Book Handles New Concepts

Loop Engineering sits on top of several ideas — some you will know, some you may
not. This book follows one firm rule: **any concept that goes beyond the
prerequisites above is defined before it is relied upon.**

In practice, that means:

- **Terms are introduced before they are used.** When a chapter needs a concept
  such as *Generator Agent*, *Evaluator*, *Harness*, or *Stopping Condition*, it
  explains the term in plain language at the point of first use rather than
  assuming you already carry the definition.
- **Acronyms are spelled out the first time they appear.** For instance, the
  *ReAct* (Reason + Act) pattern is named in full when it is first introduced in
  Chapter 04, before the book uses the short form.
- **A glossary backs the whole book.** Every key term has a single, consistent
  definition collected in the [Glossary](glossary.md), so if you jump into a later
  chapter you can always look a term up rather than hunting for where it was first
  explained.

The aim is that you never hit a wall of unexplained jargon. If you meet all three
prerequisites — version control, the command line, and hands-on experience with an
AI coding agent — everything else the book needs, it will teach you along the way.

## Key Takeaways

- This book is for **software engineers and technical practitioners who already
  use AI coding agents but have not yet built systems to orchestrate them
  autonomously.**
- It assumes three prerequisites: familiarity with **version control (Git)**,
  comfort with **command-line usage**, and hands-on experience **interacting with
  AI coding agents**.
- It does **not** assume prior experience building agent frameworks, knowledge of
  any specific tool or vendor, or a machine-learning background.
- Any concept that goes beyond those prerequisites is **defined before it is
  used**, acronyms are spelled out at first use, and a [Glossary](glossary.md)
  serves as the single source of truth for terminology.

---
[< Previous: Introduction & Learning Objectives](01-introduction.md) | [Table of Contents](README.md) | [Next: What Is Loop Engineering >](03-what-is-loop-engineering.md)
