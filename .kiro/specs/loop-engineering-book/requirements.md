# Requirements Document

## Introduction

This document defines the requirements for an educational book, authored as a
collection of Markdown files, that teaches the emerging discipline of **Loop
Engineering** — the practice of designing systems that prompt and orchestrate AI
coding agents autonomously, rather than prompting them manually turn-by-turn.

The deliverable is a **content/learning artifact**, not software. Accordingly,
these requirements describe the book's learning objectives, target audience,
scope and coverage, depth, structural expectations, and content-quality
criteria. The "system" in each acceptance criterion below refers to the Book or
one of its structural components (a Chapter, the Glossary, etc.), and "SHALL"
statements describe properties the finished content must exhibit.

Because live internet and web-search tooling is **not available** in this
authoring environment, all factual coverage requirements are scoped to the
source material supplied by the user plus general background knowledge. Any
attributed quote, date, or named-person claim is treated as **author-asserted**
and MUST be flagged in the book as independently verifiable by the reader (see
Requirement 11).

## Glossary

- **Book**: The complete educational deliverable, composed of ordered Markdown
  chapter files plus supporting files (table of contents, glossary, references).
- **Chapter**: A single top-level Markdown file covering one major topic area of
  the Book.
- **Reader**: The target learner — a software engineer or technical practitioner
  who is familiar with AI coding agents at a user level but new to orchestrating
  them autonomously.
- **Loop Engineering**: The practice of designing a control system that prompts,
  observes, verifies, and iterates AI coding agents autonomously.
- **Loop**: The orchestration system built by a loop engineer; it discovers
  work, spawns agents, verifies output, persists state, and decides whether to
  continue or stop.
- **Generator Agent**: An AI coding agent (e.g., Claude Code, a Grok agent) that
  produces output within a Loop.
- **Evaluator**: A verification component (a critic/verifier agent, a test
  suite, or a rubric scorer) that checks Generator Agent output.
- **Harness**: The reliable execution environment for a single agent run, over
  which the Loop orchestration layer is built.
- **Ralph Loop**: The precursor technique (the "Ralph Wiggum technique")
  attributed to Geoffrey Huntley (2025): a simple bash loop repeatedly feeding an
  agent fresh context plus previous output/errors until the task is done.
- **ReAct**: The "Reason + Act" agent pattern that interleaves reasoning steps
  with actions; a conceptual ancestor of the Loop.
- **Stopping Condition**: A rule that terminates a Loop (tests pass, rubric score
  threshold reached, maximum iterations or budget exhausted).
- **Author-Asserted Claim**: A factual statement (quote, date, attribution) drawn
  from supplied source material that the Book presents transparently as
  requiring independent verification by the Reader.
- **Worked Example**: A concrete, end-to-end illustration (code, configuration,
  or annotated walkthrough) that demonstrates a concept in practice.

## Requirements

### Requirement 1: Learning Objectives and Outcomes

**User Story:** As a Reader, I want the Book to state clear learning objectives
and deliver on them, so that I know what I will be able to do after reading and
can verify I have learned it.

#### Acceptance Criteria

1. THE Book SHALL state, in its introductory material, a list of explicit
   learning objectives describing what the Reader will understand and be able to
   do after completing the Book.
2. THE Book SHALL ensure that each stated learning objective is addressed by at
   least one Chapter.
3. THE Book SHALL enable a Reader with no prior knowledge of Loop Engineering to
   explain what a Loop is, why it differs from manual prompting, and the core
   components of a Loop after reading.
4. WHERE a Chapter introduces a major concept, THE Chapter SHALL begin with a
   short statement of what the Reader will learn in that Chapter.
5. WHERE a Chapter introduces a major concept, THE Chapter SHALL end with a
   summary of the key takeaways from that Chapter.

### Requirement 2: Target Audience and Prerequisites

**User Story:** As a Reader, I want the Book to identify its intended audience
and assumed background, so that I can confirm the material is pitched at my
level.

#### Acceptance Criteria

1. THE Book SHALL state its target audience as software engineers and technical
   practitioners who use AI coding agents but are new to orchestrating them
   autonomously.
2. THE Book SHALL state the assumed prerequisite knowledge, including familiarity
   with version control, command-line usage, and interacting with AI coding
   agents.
3. WHERE a concept requires background beyond the stated prerequisites, THE Book
   SHALL define that concept before relying on it.

### Requirement 3: Background and Conceptual Foundation

**User Story:** As a Reader, I want thorough background and conceptual grounding,
so that I genuinely understand the topic rather than memorizing terminology.

#### Acceptance Criteria

1. THE Book SHALL define Loop Engineering and contrast it with manual,
   turn-by-turn prompting of AI coding agents.
2. THE Book SHALL explain the evolution lineage from Prompt Engineering, to
   Context Engineering, to Agent Harness Engineering, to Loop Engineering, and
   SHALL describe what each stage adds beyond the previous stage.
3. THE Book SHALL explain the relationship between a Loop and the ReAct
   (Reason + Act) pattern, including how a Loop lifts the reason-act cycle out of
   a single agent conversation into a controllable system.
4. THE Book SHALL describe the precursor Ralph Loop technique and explain how
   Loop Engineering is its structured, production-grade evolution.
5. WHERE the Book introduces a term defined in the Glossary, THE Book SHALL use
   that term consistently with its Glossary definition.

### Requirement 4: Origin and Historical Context

**User Story:** As a Reader, I want the origin and recent history of the term, so
that I understand where the idea came from and who shaped it.

#### Acceptance Criteria

1. THE Book SHALL describe the popularization of the term "Loop Engineering"
   attributed to Addy Osmani's June 7, 2026 post, presented as an
   Author-Asserted Claim.
2. THE Book SHALL present the supporting perspectives attributed to Peter
   Steinberger and to Boris Cherny, each presented as an Author-Asserted Claim.
3. THE Book SHALL describe the Ralph Loop technique attributed to Geoffrey
   Huntley in 2025, presented as an Author-Asserted Claim.
4. WHEN the Book presents a dated event, a named attribution, or a direct quote,
   THE Book SHALL mark the statement as independently verifiable by the Reader.

### Requirement 5: Anatomy of a Loop — Components

**User Story:** As a Reader, I want each component of a Loop explained in depth,
so that I understand the building blocks before assembling them.

#### Acceptance Criteria

1. THE Book SHALL describe the Automation/Trigger component, including examples
   such as cron schedules, webhooks on new issues, and a slash-style command.
2. THE Book SHALL describe the Worktrees/Isolation component, including the use
   of git worktrees and separate branches to isolate agent work.
3. THE Book SHALL describe the Generator Agent component, including examples of
   AI coding agents that fill this role.
4. THE Book SHALL describe the Evaluator component, including running tests,
   reviewing code, and scoring output against a rubric.
5. THE Book SHALL describe the State/Memory component, including examples such as
   a TODO file, a scratchpad, a project board, and files on disk.
6. THE Book SHALL describe the Skills/Knowledge component, including examples
   such as a skills file, an agents-guidance file, and coding standards.
7. THE Book SHALL describe the Connectors component, including examples such as
   source-control hosts, issue trackers, and chat platforms.
8. THE Book SHALL describe the Stopping Condition component, including tests
   passing, a rubric score threshold, and a maximum iteration or budget limit.
9. FOR EACH Loop component listed above, THE Book SHALL state the component's
   purpose and at least one concrete example of its use.

### Requirement 6: The Loop Flow / Lifecycle

**User Story:** As a Reader, I want the end-to-end flow of a Loop explained step
by step, so that I understand how the components interact over time.

#### Acceptance Criteria

1. THE Book SHALL describe the Loop lifecycle as an ordered sequence: discover or
   trigger work, spawn agents in an isolated workspace, agent acts to produce
   output, Evaluator checks the output, feed feedback back and repeat if the
   output is insufficient, persist state, and decide whether to continue, spawn a
   sub-agent, or stop.
2. THE Book SHALL explain the decision point at which a Loop continues, spawns a
   sub-agent, or stops, including the criteria that drive each choice.
3. THE Book SHALL include at least one visual or structured representation (for
   example, a numbered flow, a diagram described in text, or a fenced
   pseudocode block) of the Loop lifecycle.

### Requirement 7: Practical Guidance — How to Build a Loop

**User Story:** As a Reader, I want a practical, hands-on section on building a
Loop, so that I can construct a working Loop myself.

#### Acceptance Criteria

1. THE Book SHALL include a practical section that guides the Reader through
   building a Loop from its components.
2. THE Book SHALL present at least one minimal Worked Example of a Loop,
   beginning from a simple bash-style loop comparable to the Ralph Loop.
3. THE Book SHALL present at least one production-grade Worked Example of a Loop
   that includes isolation, a separate Evaluator, persisted state, and explicit
   Stopping Conditions.
4. WHERE the Book presents a Worked Example containing code or configuration, THE
   Book SHALL present that code or configuration in a fenced code block with a
   language hint.
5. WHERE the Book presents a Worked Example, THE Book SHALL annotate the example
   with explanations of what each significant part does.
6. THE Book SHALL describe common patterns for designing goals, evaluation
   criteria, and guardrails for a Loop.

### Requirement 8: Benefits and Motivation

**User Story:** As a Reader, I want to understand why Loop Engineering matters,
so that I can judge when to apply it.

#### Acceptance Criteria

1. THE Book SHALL explain the benefits of Loop Engineering, including the ability
   to progress work autonomously without continuous human prompting and the
   shift of the practitioner's role from operator to system architect.
2. THE Book SHALL identify the categories of work for which Loops are well
   suited, including long, iterative tasks such as refactoring, feature
   implementation, and debugging.

### Requirement 9: Risks, Limitations, and Mitigations

**User Story:** As a Reader, I want an honest treatment of risks and their
mitigations, so that I can apply Loop Engineering responsibly.

#### Acceptance Criteria

1. THE Book SHALL describe the risk of uncontrolled token-cost growth and SHALL
   describe mitigations using tight budgets and Stopping Conditions.
2. THE Book SHALL describe the risk of high-volume low-quality output ("slop")
   that arises without strong verification, and SHALL describe mitigations using
   tests and critic agents.
3. THE Book SHALL state that engineering judgment remains necessary for designing
   goals, criteria, and guardrails.
4. THE Book SHALL include a concluding synthesis that frames Loop Engineering as
   replacing manual prompt engineering with a small, well-designed control system
   that manages agent work autonomously.

### Requirement 10: Structure, Navigation, and Format

**User Story:** As a Reader, I want the Book to be well structured and easy to
navigate as Markdown, so that I can read it linearly or jump to topics.

#### Acceptance Criteria

1. THE Book SHALL be authored as Markdown files located within the spec's
   project repository.
2. THE Book SHALL include a table of contents that lists the Chapters in reading
   order with links to each Chapter.
3. THE Book SHALL order its Chapters so that background and foundational concepts
   precede practical, build-oriented material.
4. THE Book SHALL use Markdown heading levels consistently, with one top-level
   title per Chapter and nested headings for sub-sections.
5. WHERE consecutive Chapters exist, THE Chapter SHALL provide navigation to the
   previous and next Chapter.
6. THE Book SHALL include a Glossary file that defines the key terms used
   throughout the Book.

### Requirement 11: Content Quality and Source Integrity

**User Story:** As a Reader, I want the content to be accurate, internally
consistent, and transparent about its sources, so that I can trust and verify
what I learn.

#### Acceptance Criteria

1. THE Book SHALL maintain consistent terminology, using each Glossary term with
   a single meaning throughout.
2. THE Book SHALL include a references or sources section that lists the
   Author-Asserted Claims and notes that the Reader should verify them
   independently.
3. WHERE the Book states a claim that originates from supplied source material
   rather than established general knowledge, THE Book SHALL attribute the claim
   to its source.
4. THE Book SHALL write explanatory prose in clear, active-voice language and
   SHALL define each acronym at first use.
5. IF a factual detail cannot be verified within the authoring environment, THEN
   THE Book SHALL present the detail as Author-Asserted rather than as
   established fact.
