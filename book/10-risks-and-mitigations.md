# 10. Risks, Limitations & Mitigations

> **In this chapter:** You will learn the honest downside of Loop Engineering —
> what goes wrong when a Loop runs unattended, and how to bound each failure mode
> before it costs you. We treat two risks in depth: **runaway token cost**, where
> an autonomous Loop quietly burns a budget, and **"slop"**, where weak
> verification lets a Loop produce a high volume of low-quality output. For each we
> tie the mitigations back to the machinery you have already built — the guardrails
> from Chapter 08 and the Evaluator from Chapters 06–08. Finally, we make the case
> that none of this removes the need for engineering judgment: a Loop relocates your
> judgment rather than retiring it.

## An Honest Accounting

The previous chapter made the case *for* Loops. This one is its necessary
counterweight. Everything that makes a Loop powerful — it runs without you, it
iterates tirelessly, it supplies its own next prompt — is exactly what makes it
dangerous when something is wrong. A human prompting turn-by-turn (Chapter 03)
notices a mistake on the next turn and stops typing. A Loop has no such instinct:
left to itself, it will repeat a mistake as patiently as it repeats a success,
spending real money the whole time.

So this chapter is deliberately not a sales pitch. Loop Engineering is worth
learning, but it is not magic, and overselling it does the reader a disservice. The
two risks below are the ones you will actually meet in practice. Neither is a reason
to avoid Loops; both are reasons to build them with the guardrails and the Evaluator
that earlier chapters described. The good news — and it is genuine — is that the same
structure that makes a Loop production-grade is also what makes these risks
manageable. The mitigations are not new inventions; they are the components you have
already met, pointed at the failure modes they were designed to prevent.

## Risk 1: Runaway Token Cost

The first risk is the most immediate and the easiest to underestimate: **an
autonomous Loop costs money on every iteration, and an unbounded Loop can spend an
unbounded amount.** Each call to a Generator Agent consumes tokens, and a Loop is,
by definition, a machine for making many such calls without asking permission
between them. The very autonomy you built the Loop for is what removes the natural
brake — there is no human reading each turn and deciding whether the next one is
worth it.

This goes wrong in a few characteristic ways:

- **The stuck Loop.** The Loop never reaches its goal but never realizes it. It
  iterates, fails the Evaluator, recycles the feedback, and tries again — forever, if
  nothing stops it. Each lap is cheap; the bill is the sum of a great many cheap
  laps.
- **The oscillating Loop.** A subtler version: the agent fixes problem A, which
  breaks B; the next iteration fixes B, which re-breaks A. The Loop makes confident
  "progress" on every iteration while converging on nothing, paying full price each
  time.
- **The expensive single attempt.** Cost is not only about iteration *count*. One
  attempt that pulls an enormous context into the prompt, or invokes an expensive
  model repeatedly, can blow a budget in a single lap — so capping iterations alone
  is not enough.

The honest framing is that this risk is **structural, not occasional**. It is not a
rare bug that strikes a badly written Loop; it is the default behavior of *any* Loop
that lacks explicit limits. A Loop with no stopping condition does not "usually
finish" — it has no concept of finishing at all.

### Mitigating Runaway Cost

The mitigation is the **Stopping Condition** component (Chapter 06) and the
**guardrails** you designed in Chapter 08, applied with discipline. The runaway risk
is precisely what those guardrails exist to prevent, so the defense is to use them
deliberately rather than as an afterthought:

- **Set a hard iteration cap.** A `max_iterations` limit guarantees the Loop
  terminates even if it never succeeds and never fails outright. This alone converts
  the stuck Loop from an open-ended bill into a bounded one.
- **Set an independent cost cap.** A `max_cost_usd` budget stops the Loop when
  cumulative spend crosses a threshold, regardless of iteration count. This is what
  catches the expensive-single-attempt case that an iteration cap misses. As Chapter
  08 stressed, you want *both* caps, because each covers the other's blind spot.
- **Make the success Stopping Condition real and checkable.** The cleanest way to
  stop spending is to actually finish. A goal the Evaluator can objectively confirm
  (tests pass, rubric threshold met) lets the Loop exit the moment the work is done,
  rather than iterating past completion.
- **Escalate instead of grinding.** When a budget is exhausted without acceptance,
  the Loop should hand off to a human — the escalation-to-human path from Chapter 08 —
  rather than fail silently or, worse, be restarted blindly. Detecting *repeated
  no-progress* (the oscillating Loop) and escalating early saves the budget you would
  otherwise spend confirming the Loop is stuck.

These limits live as data in the Loop's configuration, exactly as in the
production-grade example from Chapter 08:

```yaml
# Cost guardrails: layered Stopping Conditions that bound spend two ways.
stopping_conditions:
  max_iterations: 12          # bounds the stuck/oscillating Loop's lap count
  max_cost_usd: 5.00          # bounds spend independently of iteration count
  stop_when_tests_pass: true  # success exit: stop the moment the goal is verified

escalation:
  on_budget_exhausted: notify-human   # hand off rather than restart blindly
  on_repeated_no_progress: 3          # same failure 3x in a row -> escalate early
```

The limitation to be honest about: tighter budgets trade away some of the autonomy
that made the Loop attractive. A cap set too low stops the Loop before it can finish
genuinely hard work; a cap set too high defeats its own purpose. There is no
universally correct number — choosing it is an act of engineering judgment, which is
the theme this chapter builds toward.

## Risk 2: "Slop" — High-Volume, Low-Quality Output

The second risk is quieter and, in the long run, more corrosive. **Slop** is what a
Loop produces when it iterates quickly but verifies weakly: a high volume of output
that *looks* like progress but does not hold up. Where runaway cost announces itself
on an invoice, slop hides inside plausible-looking diffs, passing review by sheer
volume and confidence until someone discovers that much of it is subtly wrong,
redundant, or off-target.

Slop is the direct consequence of letting autonomy outrun verification. Recall the
argument from Chapter 09: autonomy *without* an Evaluator merely produces mistakes
faster. A Loop is a multiplier — it amplifies whatever process you point it at. Point
it at a strong verification process and it multiplies good work; point it at a weak
one and it multiplies slop just as efficiently. The danger is that the *form* of the
output stays convincing even as the *substance* degrades:

- **Confident-but-wrong output.** The Generator Agent reports success, the diff
  reads cleanly, and nothing objective contradicts it — because nothing objective was
  ever consulted. The minimal Loop from Chapter 08, which trusted the agent's own
  `ALL DONE`, is the canonical slop generator.
- **Volume mistaken for value.** A Loop can generate far more code than a human
  would in the same time. Without a quality gate, that throughput becomes a liability:
  more surface area to review, more places for defects to hide, more plausible noise
  drowning the signal.
- **Goodhart drift.** When a Loop optimizes against a weak proxy — a shallow test, a
  vague rubric — it learns to satisfy the proxy rather than the intent. The metric
  goes green while the real goal quietly recedes.

### Mitigating Slop

The mitigation for slop is the **Evaluator** (Chapters 06–08), and specifically the
principle that the Evaluator must be *independent* of the Generator Agent. Slop is
what fills the vacuum when verification is weak; a strong, independent Evaluator is
how you remove the vacuum. The patterns are the ones Chapter 08 laid out for
designing evaluation criteria, now understood as your primary defense against
low-quality output:

- **Lead with strong, objective tests.** An objective gate the agent cannot talk its
  way past — a real test suite, a type checker, a build, a schema validation — is the
  backbone of slop resistance. It returns an unambiguous pass/fail that no amount of
  confident prose can fake. The stronger and more meaningful the tests, the less room
  there is for slop to pass.
- **Add a critic/rubric agent for what tests miss.** Objective checks confirm
  behavior but say little about readability, security, or whether the change matches
  intent. A second agent scoring the diff against an explicit, versioned rubric — a
  critic acting as Evaluator — covers that gap. Hold the rubric threshold deliberately,
  as Chapter 08 advised.
- **Keep evaluation independent of generation.** The single most important property:
  the thing that *judges* the work must not be the thing that *produced* it, and must
  not simply take the producer's word. Independent evaluation is what turns
  "the agent says it's done" into "an external check confirms it's done" — the exact
  difference between slop and verified progress.
- **Make verification a hard gate, not a suggestion.** Accept an attempt only when it
  passes the objective gate *and* clears the rubric threshold. A check that merely
  warns, and lets unverified work through anyway, is not a defense against slop — it
  is slop with extra logging.

That acceptance rule is the same independent two-gate check from Chapter 08, and it
is worth seeing again specifically as anti-slop machinery:

```python
# Anti-slop acceptance: independent verification the agent cannot self-certify.
def is_acceptable(result, rubric_score, threshold=0.8):
    if not result.tests_passed:   # objective gate: unfakeable pass/fail
        return False
    if rubric_score < threshold:  # independent critic clears an explicit rubric
        return False
    return True                   # only verified work is promoted; the rest is discarded
```

The honest limitation here is that **an Evaluator is only as good as the criteria you
give it.** A Loop cannot verify quality you never specified. If your tests are
shallow or your rubric is vague, the Evaluator will faithfully wave slop through, and
the Loop will faithfully produce more of it. This is not a flaw you can automate away
by adding more agents — it is a demand on the human to define what "good" means well
enough that a machine can enforce it. Which brings us to the limitation that underlies
both risks.

## The Limitation That Does Not Go Away: Engineering Judgment

It would be dishonest to end a risks chapter by implying that the right configuration
makes Loops safe automatically. It does not. The deepest limitation of Loop
Engineering is also the most easily overlooked: **a Loop does not remove the need for
engineering judgment — it relocates it.**

Every mitigation in this chapter ultimately rests on a human decision that no Loop can
make for you:

- **The goals.** A Loop pursues the goal you give it, exactly as you phrased it. Deciding
  *what* is worth building, and stating it as a clear, checkable outcome (Chapter 08), is
  human judgment. A Loop will chase a badly chosen goal with the same tireless energy as
  a good one.
- **The evaluation criteria.** The Evaluator enforces the standard you define — no more,
  no less. Deciding what to test, how strong the tests must be, what the rubric rewards,
  and where to set its threshold is human judgment. Slop is, at root, the symptom of
  evaluation criteria that were too weak to catch it.
- **The guardrails.** The budgets, the iteration caps, the isolation boundaries, and the
  escalation path are all values *you* choose. How much money is this task worth? How
  many attempts before a human should look? What must never reach `main`? These are
  judgment calls a Loop cannot originate.

Seen this way, the two risks in this chapter are really the same lesson from two
directions. Runaway cost is what happens when the *guardrails* embody too little
judgment; slop is what happens when the *evaluation criteria* embody too little
judgment. In both cases the Loop did precisely what it was told — the gap was in the
telling.

This is why the operator-to-architect shift from Chapter 09 is a *relocation* of
judgment, not an elimination of it. As an operator you applied judgment a hundred
times, one prompt at a time, and could catch a problem mid-stream. As an architect you
apply judgment fewer times but with far more leverage — and crucially, *up front*,
before the Loop runs unattended. The stakes on each decision rise precisely because
you are no longer there to correct it turn by turn. A Loop concentrates your judgment
into a handful of high-leverage choices and then executes them faithfully and at scale.
That is its power and its risk in a single sentence: it does exactly what you designed
it to do, whether or not that is what you meant.

The practical posture that follows is neither hype nor fear. Build Loops for the work
that suits them (Chapter 09), bound them with the guardrails and the Evaluator this book
has described, and treat the time you spend designing goals, criteria, and guardrails as
the real engineering — because that is where your judgment now lives.

## Key Takeaways

- Loop Engineering is powerful but not magic; an honest account names its risks
  plainly rather than overselling. The same structure that makes a Loop
  production-grade is what makes its risks manageable — the mitigations are
  components you have already met, pointed at the failures they prevent.
- **Risk 1 — runaway token cost:** because a Loop makes many autonomous agent calls
  with no human brake between them, an unbounded Loop can spend without limit, whether
  through a *stuck* Loop, an *oscillating* one, or a single expensive attempt. This is
  the default behavior of any Loop that lacks explicit limits, not a rare bug.
- **Mitigate cost with Stopping Conditions and guardrails (Chapter 08):** set *both* a
  hard iteration cap and an independent cost cap, make the success condition real and
  checkable so the Loop can actually finish, and escalate to a human on budget
  exhaustion or repeated no-progress instead of grinding.
- **Risk 2 — slop:** when iteration outruns verification, a Loop produces a high volume
  of plausible-looking but low-quality output — confident-but-wrong diffs, volume
  mistaken for value, and Goodhart drift against weak proxies. A Loop multiplies
  whatever process you point it at, good or bad.
- **Mitigate slop with a strong, independent Evaluator (Chapters 06–08):** lead with
  objective tests the agent cannot fake, add a critic/rubric agent for what tests miss,
  keep the evaluator independent of the generator, and make verification a hard
  acceptance gate rather than a warning. An Evaluator is only as good as the criteria
  you give it.
- **Engineering judgment does not disappear — it relocates.** A Loop faithfully pursues
  the goals, enforces the evaluation criteria, and respects the guardrails *you* design;
  it cannot originate those decisions. Runaway cost and slop are both symptoms of too
  little human judgment in the guardrails and the criteria, respectively.
- The responsible posture is neither hype nor fear: build Loops for suitable work, bound
  them with guardrails and an Evaluator, and treat designing goals, criteria, and
  guardrails as the real engineering — because that is where your judgment now lives, at
  higher stakes and greater leverage.

---
[< Previous: Benefits & When to Use Loops](09-benefits.md) | [Table of Contents](README.md) | [Next: Synthesis & Conclusion >](11-synthesis-and-conclusion.md)
