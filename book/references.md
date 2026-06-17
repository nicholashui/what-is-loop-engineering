# References & Author-Asserted Claims

> **In this chapter:** This chapter collects every **Author-Asserted Claim** made
> in the book — each dated event, named attribution, and direct quote — in one
> place, alongside an explicit note that you should verify each of them
> independently. It also provides a requirement-to-chapter traceability table so
> you (or a reviewer) can confirm at a glance that every requirement is covered by
> a chapter.

## Why This Chapter Exists

This book was authored **without live access to the internet or web-search
tooling**. That constraint does not affect the *ideas* in the book — the
definition of Loop Engineering, the anatomy of a loop, the lifecycle, the worked
examples, and the risks and mitigations are all explained from general background
knowledge and the supplied source material. It does, however, affect the
*attributions*: any statement that names a specific person as the source of an
idea or quote, asserts a specific date, or reproduces a quotation could **not be
checked against its original source** while the book was being written.

Rather than silently drop those facts or present them as settled history, the book
treats every one of them as an **Author-Asserted Claim**. Each such claim is:

1. **Flagged inline** in the prose with a footnote-style marker of the form
   `[^aac-<slug>]`, plus a parenthetical *"(Author-Asserted; verify
   independently)"* on first use in a chapter; and
2. **Collected here** with its attribution and an explicit verification note.

**Please verify these claims independently before relying on them.** Treat the
attributions below as *pointers to sources you should confirm yourself*, not as
established fact. If your own check of a primary source disagrees with what is
recorded here, trust the source.

## Author-Asserted Claims

All Author-Asserted Claims in this book originate in
[Chapter 05 — Origins & Historical Context](05-origins-and-history.md). Each entry
below corresponds exactly to an inline `[^aac-...]` marker in that chapter.

### `aac-osmani-2026`

- **Asserted claim:** The popularization of the term *"Loop Engineering"* is
  attributed to a post by Addy Osmani published on **June 7, 2026**. The post is
  credited with *naming* an already-emerging practice rather than inventing it.
- **Attributed to:** Addy Osmani (post dated June 7, 2026).
- **Appears in:** Chapter 05, section *"The Term Gets a Name: Addy Osmani, June
  2026"* (and the Key Takeaways).
- **Verification note:** Author-asserted; the existence, authorship, date, and
  content of this post could not be verified in the authoring environment. The
  reader should confirm it against the original post independently.

### `aac-steinberger`

- **Asserted claim:** A supporting perspective is attributed to Peter Steinberger,
  framing the shift as moving from prompting agents to designing the loops that
  prompt them — captured in the quotation: *"You shouldn't be prompting coding
  agents anymore. You should be designing loops that prompt your agents."*
- **Attributed to:** Peter Steinberger.
- **Appears in:** Chapter 05, section *"Peter Steinberger: from prompting to
  designing loops"* (and the Key Takeaways).
- **Verification note:** Author-asserted; the attribution and the exact wording of
  the quotation could not be verified in the authoring environment. The reader
  should confirm both the source and the wording independently.

### `aac-cherny`

- **Asserted claim:** A supporting perspective is attributed to Boris Cherny,
  described as Head of Claude Code at Anthropic, capturing the same shift from the
  inside of his own workflow — in the quotation: *"I don't prompt Claude anymore.
  I have loops running that prompt Claude and figuring out what to do. My job is
  to write loops."*
- **Attributed to:** Boris Cherny (described as Head of Claude Code at Anthropic).
- **Appears in:** Chapter 05, section *"Boris Cherny: 'my job is to write loops'"*
  (and the Key Takeaways).
- **Verification note:** Author-asserted; the attribution, the stated role/title,
  and the exact wording of the quotation could not be verified in the authoring
  environment. The reader should confirm all three independently.

### `aac-huntley-2025`

- **Asserted claim:** The precursor **Ralph Loop** — also called the *"Ralph
  Wiggum technique"* — is attributed to Geoffrey Huntley in **2025**. It is
  described as a simple bash loop that repeatedly feeds an agent fresh context plus
  the previous run's output and errors until the task is done.
- **Attributed to:** Geoffrey Huntley (2025).
- **Appears in:** Chapter 05, section *"The Precursor: Geoffrey Huntley's Ralph
  Loop"* (and the Key Takeaways); the technique is also referenced conceptually in
  the Glossary and in Chapters 03, 04, and 08.
- **Verification note:** Author-asserted; the attribution, the name of the
  technique, and the 2025 date could not be verified in the authoring environment.
  The reader should confirm them against the original source independently.

## Claim-to-Marker Integrity

Every inline `[^aac-...]` marker used in the book resolves to exactly one entry
above, and every entry above is referenced by at least one inline marker — a
complete two-way correspondence. The table below records that mapping.

| Claim ID | Inline marker | Originating chapter | Entry present above |
|----------|---------------|---------------------|---------------------|
| `aac-osmani-2026` | `[^aac-osmani-2026]` | Chapter 05 | Yes |
| `aac-steinberger` | `[^aac-steinberger]` | Chapter 05 | Yes |
| `aac-cherny` | `[^aac-cherny]` | Chapter 05 | Yes |
| `aac-huntley-2025` | `[^aac-huntley-2025]` | Chapter 05 | Yes |

No dated event, named attribution, or direct quote appears anywhere else in the
book without a corresponding inline marker. All four Author-Asserted Claims are
concentrated in Chapter 05, which is the book's dedicated history chapter; the
remaining chapters rely on general background knowledge and therefore introduce no
further author-asserted facts.

## Requirement-to-Chapter Traceability

The table below maps each acceptance criterion from the requirements document to
the chapter (or supporting file) that primarily owns it. Where a requirement is a
cross-cutting structural or quality property, the owner is the convention or
supporting file that realizes it across the whole book. Some criteria are
reinforced by additional chapters beyond the primary owner noted here.

| Requirement | Acceptance criteria | Primary owner |
|-------------|---------------------|---------------|
| 1. Learning Objectives & Outcomes | 1.1, 1.2, 1.3 | Chapter 01 (objectives stated; delivered across Ch 03–10) |
| | 1.4, 1.5 | Every chapter (the "In this chapter" intro and "Key Takeaways" convention) |
| 2. Target Audience & Prerequisites | 2.1, 2.2, 2.3 | Chapter 02 |
| 3. Background & Conceptual Foundation | 3.1 | Chapter 03 |
| | 3.2, 3.3, 3.4 | Chapter 04 |
| | 3.5 | Glossary (terminology source of truth; honored by every chapter) |
| 4. Origin & Historical Context | 4.1, 4.2, 4.3, 4.4 | Chapter 05 (claims flagged here in References) |
| 5. Anatomy of a Loop — Components | 5.1–5.9 | Chapter 06 |
| 6. The Loop Flow / Lifecycle | 6.1, 6.2, 6.3 | Chapter 07 |
| 7. Practical Guidance — How to Build a Loop | 7.1–7.6 | Chapter 08 |
| 8. Benefits & Motivation | 8.1, 8.2 | Chapter 09 |
| 9. Risks, Limitations & Mitigations | 9.1, 9.2, 9.3 | Chapter 10 |
| | 9.4 | Chapter 11 (concluding synthesis) |
| 10. Structure, Navigation & Format | 10.1 | Repository layout (`book/` directory of Markdown files) |
| | 10.2 | Table of Contents ([README.md](README.md)) |
| | 10.3 | Reading order (TOC + chapter sequence: foundations before practice) |
| | 10.4 | Every chapter (single-H1 / nested-heading convention) |
| | 10.5 | Every chapter (prev/next navigation footers) |
| | 10.6 | [Glossary](glossary.md) |
| 11. Content Quality & Source Integrity | 11.1 | Glossary (single-meaning terminology; honored by every chapter) |
| | 11.2, 11.3, 11.5 | This References chapter |
| | 11.4 | Every chapter (clear, active-voice prose; acronyms defined at first use) |

A reviewer can use this table together with the
[Table of Contents](README.md) to confirm that every requirement has an owning
chapter and that no requirement is left uncovered.

## Key Takeaways

- The book was written **without live internet access**, so every dated event,
  named attribution, and direct quote is an **Author-Asserted Claim** that the
  reader should **verify independently**.
- There are four such claims, all originating in
  [Chapter 05](05-origins-and-history.md): `aac-osmani-2026` (the term's
  popularization, June 7, 2026, Addy Osmani), `aac-steinberger` (Peter
  Steinberger's "design loops that prompt your agents" perspective), `aac-cherny`
  (Boris Cherny, Head of Claude Code at Anthropic, "my job is to write loops"),
  and `aac-huntley-2025` (the Ralph Loop / "Ralph Wiggum technique," Geoffrey
  Huntley, 2025).
- Every inline `[^aac-...]` marker resolves to an entry here, and every entry here
  is referenced by at least one marker — a complete two-way correspondence.
- The requirement-to-chapter traceability table maps each acceptance criterion to
  its owning chapter, so full coverage is auditable at a glance.

---
[< Previous: Glossary](glossary.md) | [Table of Contents](README.md)
