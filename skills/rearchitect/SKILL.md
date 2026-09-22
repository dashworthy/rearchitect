---
name: rearchitect
description: "Turn a described architectural problem into up to three applicable design patterns, each with an interface sketch of the refactor it would produce. Use whenever a developer describes code that is painful to change, extend, or reason about — a class that keeps growing, a switch statement that reopens for every new case, duplicated call-site choreography, a boundary that leaks its internals, a subclass explosion, tangled dependencies — and wants concrete refactoring options rather than a rewrite. Diagnose the smell against SOLID and a smell catalog, name only the Gang-of-Four patterns whose triggers genuinely fire, sketch each candidate's interface, and recommend. Triggers on phrasings like 'this class is doing too much', 'every new type means editing five files', 'I keep copy-pasting this sequence', 'how should I restructure this', 'what pattern fits here', 'this is hard to test/extend/change'. Does not implement the refactor — it proposes options to accept or reject."
---

# rearchitect

A developer hands you an architectural problem — code that is painful to change, extend, test,
or reason about — and wants to know what to do about it. Your job is to turn that pain into a
small set of concrete, comparable options: **up to three applicable design patterns, each with an
interface sketch of the refactor it would produce**, then a recommendation. Not a rewrite, not a
lecture on patterns — options the developer can accept or reject.

You produce a proposal, not a change. You do not implement the refactor; you show what each one
would look like so the developer can choose.

## Why this shape

A pattern proposed for its own sake makes a codebase worse: indirection nobody needed, a factory
for one concrete class, a strategy interface with one implementation. The value is not in knowing
the 23 patterns — you already know them — it is in the **discipline of only reaching for one when
its trigger genuinely fires**, and in showing the developer the *interface* the refactor produces
so they can judge the trade before committing. So the whole method is: diagnose precisely, match
narrowly, sketch concretely, and let "leave it as it is" win by default when nothing fires.

## The method

### 1. Ground the problem in what it costs

Restate the problem as a cost, not a vibe. "This class is ugly" is not actionable; "adding a new
export format means editing `Report`, `ExportMenu`, and three switch statements" is. If the
developer pointed you at code, read the relevant boundary — the class, the call sites, the seam
that hurts — before diagnosing. Name concretely: what change is hard, what has to be edited
together, what a caller must know that it shouldn't. If the problem is too vague to locate, ask
one sharp question rather than guessing.

### 2. Diagnose against the lenses

Run the problem through `references/diagnostic-lenses.md` — the SOLID questions, the smell→remedy
catalog, the leakage tests, and the depth principle. Each lens that fires is a finding: name the
smell, name **where** it shows at this boundary (not in the abstract), and note the remedy the
lens points to. The remedy usually names a pattern; sometimes it names a plain move (a split, a
default, collapsing a pass-through layer) that no pattern owns. Diagnosis comes before pattern
selection — you match patterns to a named defect, never to a hunch.

### 3. Select up to three patterns — only where a trigger fires

Consult `references/pattern-matrix.md`: the 23 Gang-of-Four patterns, each with the one trigger
condition under which it is the right shape. Select a pattern **only when its trigger genuinely
describes this problem** — the trigger fires on its own, not "well, it's kind of like...". Cap the
result at three. Common outcomes, all valid:

- **One pattern fires cleanly.** Present the one, plus the plain-shape default. Don't pad to three.
- **Two or three fire** because the problem has distinct facets, or because there are genuinely
  different ways to cut it (e.g. Decorator vs. Bridge for the same subclass explosion). Present
  each as a real alternative, not a ranked-by-obviousness list.
- **None fires.** Say so plainly. The remedy is a deepening move or a plain split from the lenses,
  or "leave it as it is." A problem that needs no pattern is a common and correct answer — see
  *The plain-shape default* below.

### 4. Sketch each candidate's interface

For every pattern you select, produce an **interface sketch** following
`references/interface-sketches.md`. A sketch is the shape of the refactor's boundary in the
codebase's actual language — the new type(s) or method signatures, what the caller now writes, and
crucially **what moved behind the interface**. It is not the implementation; it is enough for the
developer to see the trade. Each sketch carries, tied to *this* problem:

- **What it buys** — the specific change that becomes easy (adding a case without editing existing
  code, swapping a policy, stacking behaviors).
- **What it costs** — the indirection, the new types, the reader's added hop. Be honest; this is
  how the developer judges whether the pattern earns its keep.

### 5. Recommend, with the plain shape always on the table

Rank the candidates and recommend one, with the reason tied to this problem and this codebase — as
a structured choice, not a decree. Put the recommendation first, keep the alternatives real, and
**always include "leave it as it is / plain shape" as a standing option.** If you present the
choice interactively, follow whatever question convention the host provides; otherwise present it
as plain ranked options. The developer decides.

## The plain-shape default

The default is load-bearing, so protect it. Running the whole catalog against every problem and
offering the closest match is exactly how a codebase fills with patterns nobody needed. A pattern
is worth proposing only when its trigger fires on its own. When none does — or when the honest
remedy is a small split, a default value, or deleting a needless layer — say that, and recommend
the plain move or no change. "You don't need a pattern here" is a first-class answer, not a
failure to find one.

## Output structure

Use this structure so the proposal is scannable:

```
## Problem (as a cost)
<one or two sentences: what change is hard, stated concretely>

## Diagnosis
<the lenses that fired: smell — where it shows here — remedy it points to>

## Candidates (up to 3)

### 1. <Pattern name> — <one-line why it fits>
Interface sketch:
<the refactored boundary in the codebase's language>
- Buys: <the specific change that becomes easy>
- Costs: <the indirection / new types / reader's added hop>

### 2. ...
### 3. ...

## Recommendation
<the one to reach for, why, tied to this problem — with "leave as is / plain shape" on the table>
```

If only one pattern (or none) fires, show only what fired — never pad to three.

## Boundaries — what this does not do

- It does not **implement the refactor.** It sketches interfaces and recommends; writing the
  module and migrating call sites is ordinary implementation work, done after the developer picks.
- It does not **audit a whole codebase.** It works the one problem the developer describes. Sweeping
  every module for smells and drawing an improvement roadmap is a different, larger job.
- It does not **decide what to build.** Product direction and which feature to build are settled
  elsewhere; this skill reshapes structure that already exists (or is about to).
- It does not **force a pattern.** No trigger, no pattern. The plain shape wins by default.

## References

- `references/diagnostic-lenses.md` — SOLID questions, the smell→remedy catalog, the leakage
  tests, and the depth principle. Read in step 2 to name the defect.
- `references/pattern-matrix.md` — the 23 Gang-of-Four patterns and the single trigger for each.
  Read in step 3 to select only patterns whose trigger fires.
- `references/interface-sketches.md` — how to sketch a refactor's interface (the deepening moves
  and the "who decides what" test), so a sketch shows the real trade. Read in step 4.
