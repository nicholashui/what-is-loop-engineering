# Implementation Plan

This plan authors the Loop Engineering book as Markdown files under `book/`,
following the chapter breakdown, conventions, and correctness properties in
design.md. Tasks are ordered so foundational content and shared conventions come
first, then chapters in reading order, then supporting files, then the
content-quality verification pass.

- [x] 1. Set up the book skeleton and shared conventions
  - Create the `book/` directory and placeholder files for all chapters, the
    glossary, and references using the file layout from the design.
  - Establish the reusable chapter template (single H1, "In this chapter" intro,
    "Key Takeaways" summary, prev/next navigation footer) as the pattern every
    chapter will follow.
  - Update the repository `README.md` with a short book description linking into
    `book/README.md`.
  - _Requirements: 10.1, 10.4_

- [x] 2. Author the table of contents
  - Write `book/README.md` listing all chapters in reading order with links, plus
    links to the glossary and references.
  - Ensure ordering places foundational chapters before practical ones.
  - _Requirements: 10.2, 10.3_

- [x] 3. Author Chapter 01 — Introduction & Learning Objectives
  - State the explicit learning objectives list.
  - Set up the reader-outcome framing (what a Loop is, how it differs, core
    components) that later chapters deliver.
  - Apply the chapter template (intro callout, Key Takeaways, navigation footer).
  - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5_

- [x] 4. Author Chapter 02 — Audience & Prerequisites
  - State the target audience (engineers using AI agents, new to orchestration).
  - State prerequisites (version control, command-line usage, interacting with AI
    coding agents).
  - Note that concepts beyond prerequisites will be defined before use.
  - _Requirements: 2.1, 2.2, 2.3_

- [x] 5. Author Chapter 03 — What Is Loop Engineering
  - Define Loop Engineering and explicitly contrast it with manual, turn-by-turn
    prompting.
  - Reinforce the core "what is a Loop" learning objective.
  - _Requirements: 1.3, 3.1_

- [x] 6. Author Chapter 04 — Lineage & the ReAct Connection
  - Explain the lineage: Prompt Engineering -> Context Engineering -> Agent Harness
    Engineering -> Loop Engineering, describing what each stage adds.
  - Explain the Loop/ReAct relationship and how a Loop lifts the reason-act cycle
    out of a single conversation into a controllable system.
  - Introduce the Ralph Loop as the precursor that Loop Engineering evolves.
  - Define the acronym "ReAct" at first use.
  - _Requirements: 3.2, 3.3, 3.4, 11.4_

- [x] 7. Author Chapter 05 — Origins & Historical Context
  - Describe the popularization of "Loop Engineering" attributed to Addy Osmani's
    June 7, 2026 post, flagged as an Author-Asserted Claim.
  - Present the supporting perspectives attributed to Peter Steinberger and Boris
    Cherny, each flagged as Author-Asserted.
  - Describe the Ralph Loop attributed to Geoffrey Huntley in 2025, flagged as
    Author-Asserted.
  - Apply the inline `[^aac-...]` marker convention to every dated/named/quoted
    statement.
  - _Requirements: 4.1, 4.2, 4.3, 4.4_

- [x] 8. Author Chapter 06 — Anatomy of a Loop: The Components
  - Write a dedicated subsection for each of the eight components
    (Automation/Trigger, Worktrees/Isolation, Generator Agent, Evaluator,
    State/Memory, Skills/Knowledge, Connectors, Stopping Condition).
  - For each component, state its purpose and give at least one concrete example as
    listed in the design's component table.
  - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5, 5.6, 5.7, 5.8, 5.9_

- [x] 9. Author Chapter 07 — The Loop Lifecycle
  - Describe the lifecycle as the ordered seven-stage sequence from the design.
  - Explain the continue / spawn-sub-agent / stop decision point and the criteria
    driving each choice.
  - Include at least one structured representation (mermaid diagram, numbered flow,
    or fenced pseudocode block).
  - _Requirements: 6.1, 6.2, 6.3_

- [x] 10. Author Chapter 08 — Building a Loop: Practical Guide
  - [x] 10.1 Write the minimal Ralph-style worked example
    - Present a simple bash-style loop in a language-hinted fenced block.
    - Annotate the loop condition, agent invocation, and context recycling.
    - _Requirements: 7.2, 7.4, 7.5_
  - [x] 10.2 Write the production-grade worked example
    - Evolve the minimal example to add isolation (git worktrees/branches), a
      separate Evaluator, persisted state, and explicit Stopping Conditions.
    - Use language-hinted fenced blocks with annotations; connect back to the
      Chapter 06 components and Chapter 07 lifecycle.
    - _Requirements: 7.3, 7.4, 7.5_
  - [x] 10.3 Write the practical guidance and patterns section
    - Guide the reader through building a Loop from its components.
    - Describe common patterns for designing goals, evaluation criteria, and
      guardrails.
    - _Requirements: 7.1, 7.6_

- [x] 11. Author Chapter 09 — Benefits & When to Use Loops
  - Explain the benefits, including autonomous progress without continuous
    prompting and the operator-to-architect role shift.
  - Identify suitable categories of work (refactoring, feature implementation,
    debugging).
  - _Requirements: 8.1, 8.2_

- [x] 12. Author Chapter 10 — Risks, Limitations & Mitigations
  - Describe the token-cost growth risk and mitigations (tight budgets, stopping
    conditions).
  - Describe the "slop" risk and mitigations (tests, critic agents).
  - State that engineering judgment remains necessary for goals, criteria, and
    guardrails.
  - _Requirements: 9.1, 9.2, 9.3_

- [x] 13. Author Chapter 11 — Synthesis & Conclusion
  - Write the concluding synthesis framing Loop Engineering as replacing manual
    prompt engineering with a small, well-designed control system that manages
    agent work autonomously.
  - _Requirements: 9.4_

- [x] 14. Author the Glossary
  - Write `book/glossary.md` defining all key terms, mirroring the requirements
    glossary, with one definition per term.
  - _Requirements: 10.6, 11.1_

- [x] 15. Author the References & Author-Asserted Claims chapter
  - Write `book/references.md` listing every Author-Asserted Claim
    (`aac-osmani-2026`, `aac-steinberger`, `aac-cherny`, `aac-huntley-2025`, plus
    any others introduced) with attribution and a note to verify independently.
  - Include the requirement-to-chapter traceability note.
  - _Requirements: 11.2, 11.3, 11.5_

- [x] 16. Add and wire navigation footers across all chapters
  - Add prev/next/TOC footers to every chapter consistent with the table-of-contents
    order; first chapter omits Previous, last content chapter's Next points to the
    Glossary.
  - _Requirements: 10.5_

- [x] 17. Run the content-quality verification pass
  - Verify each correctness property from the design against the authored files:
  - [x] 17.1 Verify learning-objective coverage (every objective maps to a present
    chapter). _Property 1; Requirements 1.2_
  - [x] 17.2 Verify chapter framing and heading structure (single H1, intro,
    Key Takeaways, no skipped levels) across all chapters. _Property 2; Requirements 1.4, 1.5, 10.4_
  - [x] 17.3 Verify all eight Loop components are documented with purpose + at least
    one example. _Property 3; Requirements 5.1-5.9_
  - [x] 17.4 Verify navigation integrity (TOC links all chapters in order; prev/next
    footers agree). _Property 4; Requirements 10.2, 10.5_
  - [x] 17.5 Verify terminology consistency with the glossary across all chapters.
    _Property 5; Requirements 3.5, 11.1_
  - [x] 17.6 Verify Author-Asserted Claim integrity (inline markers ↔ references
    entries, with verification notes). _Property 6; Requirements 4.1-4.4, 11.2, 11.3, 11.5_
  - [x] 17.7 Verify worked-example formatting (language-hinted fenced blocks +
    annotations). _Property 7; Requirements 7.4, 7.5_
  - [x] 17.8 Editorial review for active-voice prose and acronym-at-first-use, and
    confirm the example-classified criteria (audience, lineage, lifecycle, benefits,
    risks, synthesis) are all present. _Requirements 2.1-2.3, 3.1-3.4, 6.1-6.3, 7.1, 7.6, 8.1, 8.2, 9.1-9.4, 11.4_
