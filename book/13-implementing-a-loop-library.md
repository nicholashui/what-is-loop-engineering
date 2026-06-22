# 13. Implementing a Loop Library

> **In this chapter:** You will learn how to *build* a Loop Library of your own —
> a curated, version-controlled collection of reusable loop definitions your team
> can browse, copy, and run. We define a single **loop entry schema** so every loop
> is described the same way, lay out a repository structure to store them, show how
> to make the library searchable, and — most importantly — show how to turn a
> static library entry into a *running* Loop using the machinery from Chapter 08.
> By the end you will be able to take the 50 patterns from Chapter 12 (or your own)
> and turn them into an executable library.

## From a Catalog to a Library

Chapter 12 was a *catalog* — a read-only tour of loops other people wrote. A
**Loop Library** is something stronger: a living artifact in your own repository
where each entry is structured data, not prose, so it can be searched, validated,
versioned, and — the crucial step — *executed*. The difference is the same one this
book keeps returning to: prose describes a loop; a library *defines* it precisely
enough that a machine can run it.

Implementing a library means making three decisions and then wiring them together:

1. **A schema** — one consistent shape for describing every loop, so entries are
   comparable and machine-readable.
2. **A store** — where the entries live and how they are organized, versioned, and
   reviewed.
3. **A runner** — how a library entry becomes a running Loop built from the
   Chapter 06 components.

The rest of this chapter takes them in that order. Everything here builds directly
on the production Loop from Chapter 08; the library is what lets you define many
such Loops without rewriting the orchestration each time.

## The Loop Entry Schema

The foundation of a library is a single, agreed-upon description for what a loop
*is*. Notice that every entry in Chapter 12 implicitly answered the same questions —
who wrote it, what it is for, how it works, when it stops. Make those questions
explicit fields and you have a schema. A good schema mirrors the eight components
(Chapter 06) so that a library entry maps one-to-one onto a runnable Loop.

Here is a schema expressed as a Markdown file with YAML frontmatter — readable by a
human *and* parseable by a machine, which is exactly what a library entry needs:

```markdown
---
# --- Identity & discovery (how humans and search find this loop) ---
id: production-error-sweep            # stable, unique, kebab-case
name: The production error sweep      # human-friendly title
author: Matthew Berman                # attribution
category: engineering                 # engineering|evaluation|design|content|operations
tags: [observability, bugfix, pull-request]
maturity: stable                      # draft|experimental|stable|deprecated
version: 1.2.0                        # semantic version of THIS definition

# --- The loop contract (maps to the Chapter 06 components) ---
purpose: >
  Find, fix, and verify actionable errors in production, then open a PR.

trigger:                              # Automation/Trigger component
  type: schedule                      # schedule|event|manual
  spec: "0 * * * *"                   # hourly; ignored when type is manual

generator:                            # Generator Agent component
  prompt_file: prompts/production-error-sweep.md
  inputs: [log_source, repo]          # parameters the caller must supply

evaluator:                            # Evaluator component (independent of generator)
  objective_gate: "npm test"          # must exit 0
  critic: optional                    # rubric/critic gate, if any
  rubric_file: rubrics/bugfix.md

isolation:                            # Worktrees/Isolation component
  strategy: git-worktree
  base_branch: main

state:                                # State/Memory component
  file: .loop-state/production-error-sweep.md

stopping_conditions:                  # Stopping Condition component
  success: "actionable error fixed AND tests pass AND PR opened"
  no_op: "no actionable errors found"
  max_iterations: 10
  max_cost_usd: 4.00

connectors: [github, log-provider]    # Connectors component
skills: [skills/triage.md]            # Skills/Knowledge component

source: https://signals.forwardfuture.ai/loop-library/
---

## What it does

Prose description for humans browsing the library. Keep this short; the
frontmatter above is the machine-readable contract.

## Guardrails & notes

Anything a human running this loop should know: cost expectations, blast
radius, when NOT to use it, required permissions.
```

Two design principles make this schema work:

- **Every component is a field.** The `trigger`, `generator`, `evaluator`,
  `isolation`, `state`, `stopping_conditions`, `connectors`, and `skills` keys are
  exactly the eight components from Chapter 06. A library entry is therefore a
  *complete, declarative description of a Loop* — nothing about how to run it lives
  outside the file.
- **The prompt is a reference, not inline text.** The actual agent instructions
  live in a separate `prompt_file`. This keeps prompts reusable across loops,
  diffable on their own, and editable without touching the loop's metadata.

This is the same `loop.config.yaml` idea from Chapter 08, generalized: instead of
one config for one loop, you have one schema that every loop in the library
conforms to.

### Required vs. optional fields

Not every loop needs every field, but a small core should be **mandatory** so the
library never contains an unrunnable entry:

| Field | Required? | Why |
|-------|-----------|-----|
| `id`, `name`, `author`, `category` | Required | Identity, attribution, discovery |
| `purpose` | Required | One-line checkable intent |
| `generator.prompt_file` | Required | There is no loop without a generator |
| `stopping_conditions` | Required | An unbounded loop is a defect, not an entry |
| `evaluator` | Strongly recommended | Without it, "done" is the agent's say-so |
| `trigger`, `isolation`, `state` | Optional (defaulted) | Sensible defaults; override per loop |
| `connectors`, `skills`, `tags`, `source` | Optional | Enrichment and integration |

A validator (covered below) should reject any entry missing a required field. The
single most important rule: **no entry may lack a stopping condition.** That one
constraint is what keeps a library of autonomous loops from being a library of
runaway processes.

## Storing the Library

With a schema fixed, the store is straightforward: a directory of entry files under
version control. Plain files in git give you history, review, branching, and diffs
for free — the same properties that make the loops themselves trustworthy.

A workable repository layout:

```
loop-library/
├── README.md                  # generated index (see "Making it searchable")
├── schema/
│   └── loop.schema.json       # JSON Schema the frontmatter must satisfy
├── loops/
│   ├── engineering/
│   │   ├── production-error-sweep.md
│   │   ├── docs-sweep.md
│   │   └── test-stabilizer.md
│   ├── evaluation/
│   │   └── self-improving-champion.md
│   ├── design/
│   │   └── ui-ux-score.md
│   ├── content/
│   └── operations/
├── prompts/                   # the generator prompts referenced by entries
│   └── production-error-sweep.md
├── rubrics/                   # critic/evaluator rubrics referenced by entries
│   └── bugfix.md
└── skills/                    # shared Skills/Knowledge files
    └── triage.md
```

The organizing choices that matter:

- **One file per loop, foldered by category.** This mirrors Chapter 12's five
  categories and keeps the library browsable as a plain directory tree even with no
  tooling at all.
- **Prompts, rubrics, and skills are shared, first-class files.** Because entries
  *reference* them, the same rubric or skill can back many loops, and improving one
  improves every loop that uses it.
- **A machine-checkable schema lives in the repo.** `schema/loop.schema.json` lets
  CI validate every entry on every change — the library's own quality gate, which
  is itself just an evaluator applied to the library.

### Validate entries in CI

The library should hold itself to the standard it preaches. A pre-merge check that
validates every entry against the schema is the minimum bar:

```bash
#!/usr/bin/env bash
# validate-library.sh — reject any loop entry that breaks the contract.
set -euo pipefail

fail=0
for entry in loops/**/*.md; do
  # Extract YAML frontmatter and validate it against the JSON Schema.
  if ! yq --front-matter=extract '.' "$entry" \
        | ajv validate -s schema/loop.schema.json --data - >/dev/null 2>&1; then
    echo "SCHEMA FAIL: $entry"
    fail=1
  fi

  # Hard rule: every entry MUST declare a stopping condition.
  if ! yq -e --front-matter=extract '.stopping_conditions' "$entry" >/dev/null 2>&1; then
    echo "NO STOPPING CONDITION: $entry"
    fail=1
  fi

  # Referenced prompt file must exist, or the entry is unrunnable.
  prompt=$(yq -r --front-matter=extract '.generator.prompt_file' "$entry")
  if [ ! -f "$prompt" ]; then
    echo "MISSING PROMPT: $entry -> $prompt"
    fail=1
  fi
done

exit $fail
```

This script encodes the three non-negotiables from the schema section: valid shape,
a stopping condition, and an existing prompt. Run it in CI and the library can never
merge an entry that would fail to run or run forever.

## Making the Library Searchable

A library only earns its name when people can *find* the right loop quickly. Because
every entry carries structured frontmatter, search is mostly a matter of reading
that metadata and projecting it into something browsable.

Two low-cost mechanisms cover almost every need:

- **A generated index.** A small script reads every entry's frontmatter and writes a
  grouped, linked `README.md` — title, author, purpose, tags — exactly like the
  catalog in Chapter 12. Regenerate it in CI so the index is never stale.
- **Faceted, tag-based filtering.** Because `category`, `tags`, and `maturity` are
  structured fields, you can filter the library without a database: "show stable
  engineering loops tagged `bugfix`" is a one-line query over the frontmatter.

A minimal index generator:

```python
# build_index.py — regenerate README.md from loop frontmatter.
import pathlib, frontmatter   # `python-frontmatter`
from collections import defaultdict

by_category = defaultdict(list)
for path in pathlib.Path("loops").rglob("*.md"):
    post = frontmatter.load(path)
    by_category[post["category"]].append((post, path))

lines = ["# Loop Library\n", "_Generated index — do not edit by hand._\n"]
for category in sorted(by_category):
    lines.append(f"\n## {category.title()} Loops\n")
    for post, path in sorted(by_category[category], key=lambda p: p[0]["name"]):
        # Each row: name (linked), author, one-line purpose, tags.
        tags = ", ".join(post.get("tags", []))
        lines.append(
            f"- [{post['name']}]({path}) — *{post['author']}* — "
            f"{post['purpose'].strip()}  `{tags}`"
        )

pathlib.Path("README.md").write_text("\n".join(lines))
```

For a small team this is enough; the structured frontmatter means you can graduate
to full-text search or a web UI later without changing the entries themselves. The
metadata *is* the search index.

## Turning a Library Entry into a Running Loop

This is the step that separates a real library from a fancy notes folder. A library
entry is a *declaration*; running it means handing that declaration to a **runner**
that assembles the Chapter 06 components and drives the Chapter 07 lifecycle — the
exact production Loop from Chapter 08, now parameterized by the entry instead of
hard-coded.

The runner's job is a faithful translation, field by field:

| Schema field | What the runner does with it | Chapter 07 stage |
|--------------|------------------------------|------------------|
| `trigger` | Decides when to start (cron, webhook, or manual invoke) | Stage 1 — discover/trigger work |
| `isolation` | Creates a git worktree on a throwaway branch per attempt | Stage 2 — spawn in isolation |
| `generator.prompt_file` + `inputs` | Renders the prompt and invokes the agent | Stage 3 — agent acts |
| `evaluator` | Runs the objective gate, then the critic/rubric gate | Stage 4 — evaluate |
| `state.file` | Persists progress and recycles it into the next prompt | Stages 5–6 — feed back, persist |
| `stopping_conditions` | Decides continue / merge-and-stop / escalate | Stage 7 — decide |

A runner that consumes a library entry looks like this — note that it is the
Chapter 08 orchestrator with its constants replaced by reads from the entry:

```bash
#!/usr/bin/env bash
# run-loop.sh <entry.md> [key=value ...] — run any library entry as a Loop.
set -euo pipefail

ENTRY="$1"; shift
fm() { yq -r --front-matter=extract "$1" "$ENTRY"; }   # read a frontmatter field

# --- Resolve the loop contract from the entry (no hard-coded loop logic) ---
GOAL=$(fm '.purpose')
PROMPT_FILE=$(fm '.generator.prompt_file')
TEST_CMD=$(fm '.evaluator.objective_gate')
RUBRIC=$(fm '.evaluator.rubric_file // ""')
BASE=$(fm '.isolation.base_branch // "main"')
STATE=$(fm '.state.file')
MAX_ITERS=$(fm '.stopping_conditions.max_iterations')
MAX_COST=$(fm '.stopping_conditions.max_cost_usd')

iteration=0; spend=0; context=""
mkdir -p "$(dirname "$STATE")"; touch "$STATE"

# --- Same heartbeat as Chapter 08, now driven entirely by the entry ---
while (( iteration < MAX_ITERS )) && (( $(echo "$spend < $MAX_COST" | bc -l) )); do
  iteration=$((iteration + 1))
  branch="loop/$(fm '.id')-attempt-$iteration"
  worktree=".loop-worktrees/$branch"
  git worktree add -b "$branch" "$worktree" "$BASE"          # ISOLATION (Stage 2)
  pushd "$worktree" >/dev/null

  # GENERATOR (Stage 3): render the referenced prompt + state + feedback + caller inputs.
  { cat "../../$PROMPT_FILE";
    echo; echo "Goal: $GOAL";
    echo "Caller inputs: $*";
    echo "Progress so far:"; cat "../../$STATE";
    echo "Previous evaluation feedback: $context";
  } > .loop-prompt.txt
  agent --prompt-file .loop-prompt.txt

  # EVALUATOR (Stage 4): independent objective gate, then optional rubric gate.
  if eval "$TEST_CMD"; then tests_pass=true; else tests_pass=false; fi
  score=1.0
  [ -n "$RUBRIC" ] && score=$(agent --review --rubric "../../$RUBRIC" --format score)
  spend=$(echo "$spend + $(agent --last-cost)" | bc -l)

  # DECISION (Stage 7): accept and stop, or persist feedback and continue.
  if [ "$tests_pass" = true ] && (( $(echo "$score >= 0.8" | bc -l) )); then
    popd >/dev/null
    git merge --no-ff "$branch" -m "Loop $(fm '.id'): accepted attempt $iteration"
    echo "Accepted on iteration $iteration."; break
  fi
  context="tests_pass=$tests_pass score=$score"
  printf '\n## Iteration %s\n- tests: %s\n- score: %s\n' \
    "$iteration" "$tests_pass" "$score" >> "../../$STATE"   # PERSIST (Stages 5-6)
  popd >/dev/null
  git worktree remove --force "$worktree"
done
echo "Loop $(fm '.id') ended after $iteration iteration(s); spend \$$spend."
```

The point is not the specific script — it is the *separation*. The runner is written
**once** and contains all the orchestration; the library entries contain only
*declarations*. Add a hundred loops and the runner does not change. This is the
payoff of the schema: a loop becomes data, and running any loop is the same code
path.

### Parameterized loops

Many catalog loops in Chapter 12 had blanks — "stop after [N] successes,"
"[repository task]," "[quality threshold]." A library makes those blanks first-class
**inputs** (the `generator.inputs` field), supplied at run time:

```bash
# Run the same library entry against different targets, no edits to the entry.
./run-loop.sh loops/engineering/production-error-sweep.md \
    log_source=prod-us-east repo=checkout-service

./run-loop.sh loops/evaluation/quality-streak.md  streak_n=20
```

A single well-parameterized entry replaces a dozen near-duplicate copies. This is
how a library stays small as it grows: you add *parameters*, not forks.

## A Practical Path to Your First Library

You do not build the whole library up front, any more than you built the whole Loop
up front in Chapter 08. Grow it the same subtractive way:

1. **Promote one working loop.** Take a Loop you already run from Chapter 08, write
   it up in the schema, and commit it as the first entry. One real entry beats ten
   speculative ones.
2. **Write the runner once.** Generalize your Chapter 08 orchestrator into a runner
   that reads an entry. Prove it by running the first entry through it.
3. **Add the schema validator to CI.** Now the library cannot accept a broken or
   unbounded entry — the quality bar is in place before the second entry arrives.
4. **Promote loops only as you actually reuse them.** When you find yourself
   rebuilding a loop shape you have built before — one of the Chapter 12 patterns —
   that is the signal to add it to the library, parameterized.
5. **Generate the index.** Once you have a handful of entries, wire up the index
   generator so the library is browsable and the team can discover what exists.

The discipline is the same as everywhere in this book: **add a loop to the library
when *not* having it there causes real duplication — and not before.** A library of
three loops your team actually runs is worth more than fifty you copied and
never executed.

## Key Takeaways

- A **Loop Library** turns the read-only catalog of Chapter 12 into a living,
  executable artifact: each loop becomes structured, version-controlled data rather
  than prose.
- Implementing one means three decisions wired together — a **schema** (one shape
  for every loop), a **store** (version-controlled files), and a **runner** (that
  executes any entry).
- The **schema** should make every Chapter 06 component an explicit field
  (`trigger`, `generator`, `evaluator`, `isolation`, `state`,
  `stopping_conditions`, `connectors`, `skills`), with the prompt stored as a
  referenced file; a small core of fields is mandatory, and **no entry may lack a
  stopping condition.**
- The **store** is a git directory of one-file-per-loop entries foldered by
  category, with shared prompts/rubrics/skills and a JSON Schema validated in CI —
  the library applying an evaluator to itself.
- Search comes almost for free from the structured frontmatter: a generated index
  plus tag/category/maturity filtering, with no database required.
- The **runner** is written once and translates an entry's fields onto the Chapter
  07 lifecycle — it is the Chapter 08 production Loop with its constants replaced by
  reads from the entry, so adding loops never changes the orchestration.
- Parameterized **inputs** turn the catalog's `[blanks]` into run-time arguments, so
  one entry replaces many near-duplicates and the library stays small as it grows.
- Build the library **subtractively**: promote one working loop, write the runner,
  add CI validation, then add entries only when reuse actually demands them.

---
[< Previous: The Loop Library](12-the-loop-library.md) | [Table of Contents](README.md) | [Next: Curating, Operating & Scaling a Loop Library >](14-operating-a-loop-library.md)
