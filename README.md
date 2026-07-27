# Implement with Human

`implement-with-human` is an agent skill for implementing a spec or ticket in small, human-reviewed increments.

It is for teams that want AI to write code quickly without asking reviewers to accept one large, opaque diff. The agent plans the work, implements one reviewable slice, validates and commits it, then stops for an explicit human decision before continuing.

## Install

Install the skill for Codex from this repository:

```bash
npx skills add JaeHyunLee123/skills \
  --skill implement-with-human
```

To install it globally instead of into the current project, add `--global`:

```bash
npx skills add JaeHyunLee123/skills \
  --skill implement-with-human \
  --global
```

## Use

Invoke the skill with a spec, ticket, or concrete request:

```text
$implement-with-human Implement issue #123.
```

Or:

```text
$implement-with-human Implement the requirements in docs/my-feature-spec.md.
```

The skill is intentionally user-invoked. It does not begin on its own, because opening a human review loop is a deliberate workflow choice.

## Recommended companion: `grill-with-docs`

Use [`grill-with-docs`](https://github.com/mattpocock/skills) from Matt Pocock's skills repository before `implement-with-human` when the work is not yet fully specified.

`grill-with-docs` conducts a one-question-at-a-time design interview and records resolved domain terminology in `CONTEXT.md`; it can also capture hard-to-reverse architectural decisions as ADRs. That produces a shared understanding before implementation begins, while `implement-with-human` preserves that understanding through small, reviewable code changes.

Install Matt Pocock's skills and select `grill-with-docs` during installation:

```bash
npx skills@latest add mattpocock/skills
```

Then use this flow:

```text
$grill-with-docs Design the change with me.
# Resolve the design, terminology, and relevant decisions.
$implement-with-human Implement the resulting spec or ticket.
```

For a small, already precise ticket, start directly with `implement-with-human`. For an ambiguous feature, workflow, or architectural change, start with `grill-with-docs`.

## How it works

1. **Plan** — The agent explores the work and proposes an ordered Slice Plan. Each slice states its behaviour, test seam, validation method, and estimated size.
2. **Choose the checklist location** — Before creating a progress checklist, the agent asks where it belongs. It recommends adding it to the existing spec; otherwise it recommends a new document in the repository root or `docs/`.
3. **Approve the plan** — The agent waits for explicit approval before changing implementation code.
4. **Build one Review Slice** — The agent implements only one independently verifiable increment. When available, it uses the repository's TDD skill at the agreed seam.
5. **Validate and commit** — The agent runs the relevant tests and static checks, stages only the slice, records the commit, and runs automated code review.
6. **Review Gate** — The agent presents a fixed review package and stops. Only a clear instruction such as `approve` or `next slice` opens the gate.
7. **Repeat** — Feedback remains in the same slice until it is revalidated and approved. The agent never starts the next slice while the current one awaits review.
8. **Finish** — After all slices are approved, the agent runs the full suite and a whole-change review, then waits for one final human approval.

## The idea: AI speed, human control

AI can generate a large amount of code before a reviewer has enough context to judge it safely. This skill changes the unit of collaboration from “one complete implementation” to a **Review Slice**: the smallest independently verifiable, commit-ready increment a human can reasonably inspect.

The default maximum is **300 changed lines** per slice. Changed Lines are additions plus deletions across production code, tests, and manually written documentation. Generated files and lockfiles are excluded from the limit but disclosed in the review package. Feedback commits count toward the same cumulative limit, so splitting a large change across several commits cannot hide its review cost.

A Review Slice is not merely a line-count budget. It has one observable behaviour, its required tests, and only the supporting code needed for that behaviour. Unrelated refactors and formatting changes belong in separate slices. If a change genuinely cannot be split below the limit, the agent must explain why and request an explicit exception before proceeding.

## What the human reviews

At every Review Gate, the agent provides:

- the behaviour added or changed;
- the changed files and Changed Lines, including excluded generated or lockfile counts;
- test, typecheck, and lint results;
- automated code-review findings and unresolved decisions;
- the progress checklist status; and
- the exact approval or decision needed to continue.

Automated review is useful preparation, not approval. A human still decides whether the slice is correct, understandable, and worth building on.

## Progress checklist states

Each slice is tracked as one of:

`planned` → `implementing` → `awaiting-review` → `approved`

Humans can also explicitly mark a slice `deferred` or `dropped`. The agent does not skip planned work by itself.

## Skill location

The installable skill lives at [skills/implement-with-human](./skills/implement-with-human/).
