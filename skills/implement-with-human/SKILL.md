---
name: implement-with-human
description: Implement a spec or ticket in human-reviewed slices.
disable-model-invocation: true
---

Implement the user's work one **Review Slice** at a time. A Review Gate is a hard boundary: do not start a later slice until the human explicitly approves the current one.

## 1. Establish the plan

1. Read the relevant `CONTEXT.md` and ADRs, the supplied spec or ticket, and the code needed to plan the work.
2. Inspect `git status --short`. If unrelated changes exist, report them and wait until the human says whether they belong to this work.
3. Propose a **Slice Plan** in dependency order. For every slice, state its observable behaviour, scope, test seam and validation, and estimated Changed Lines. Keep independent refactors or formatting changes in their own slice.
4. Ask where to keep the **Progress Checklist**; never create one without this answer. Recommend adding it to an existing requirement or spec document. If none exists, recommend a new file at the repository root or under `docs/`.
5. Once the human chooses a location, create or update the checklist with each slice's scope, validation, Changed Lines estimate, and `planned` state. Its permitted states are `planned`, `implementing`, `awaiting-review`, `approved`, `deferred`, and `dropped`.
6. Present the plan and wait for explicit approval before editing implementation code.

If a later discovery changes a slice's scope, order, or boundary, present a Plan Revision and wait for approval before starting its affected slice. Only the human may mark a slice `deferred` or `dropped`.

## 2. Build one slice

1. Mark the approved slice `implementing` in the Progress Checklist and record its base as the current `HEAD` SHA.
2. If `/tdd` is available, use it at the plan's agreed seam. Otherwise, follow the agreed validation method.
3. Build only the slice's behaviour and required supporting change. Run relevant tests and typecheck or lint regularly.
4. Run the slice's validation. Run the full suite as well for the final slice and any high-impact shared, security, or migration change.
5. Stage only the slice files. Measure Changed Lines as additions plus deletions from `git diff --cached --numstat <slice-base>`; include production code, tests, and manual documentation. Exclude generated files and lockfiles from the 300-line limit, but list their counts separately.
6. Keep the cumulative staged slice diff, including feedback commits, at or below 300 Changed Lines. If that cannot be done, pause before proceeding past the limit and request an explicit exception with the reason, expected size, and review strategy.
7. Commit the slice to the current branch, following the repository's normal commit-message convention.

## 3. Prepare the Review Gate

1. Run `/code-review` from the recorded slice base to `HEAD`, supplying the original spec or ticket.
2. Correct clear, in-scope findings, then repeat validation and automated review. Bring findings that need a scope or requirement decision to the human instead.
3. Update the checklist with validation results and every slice commit SHA. Mark it `awaiting-review`.
4. Present a **Review Package** containing:
   - purpose and observable behaviour;
   - changed files and Changed Lines, including excluded generated/lockfile counts;
   - validation results;
   - `/code-review` results and unresolved decisions;
   - current Progress Checklist state; and
   - the exact approval or decision requested.
5. Stop. Treat only an unambiguous instruction such as `approve` or `next slice` as approval. Questions, comments, and informal praise leave the Review Gate closed.

## 4. Handle the human response

- On feedback, keep work within the same Review Slice: make the requested correction, revalidate, rerun `/code-review`, update the checklist, and return to its Review Gate.
- On explicit approval, mark the slice `approved`, then begin only the next approved slice.
- Never merge feedback with a later slice or begin a later slice while the current one awaits review.

## 5. Finish the work

After every planned slice is approved, run the full test suite and `/code-review` across the whole implementation range. Present a final integrated Review Package and wait for explicit human approval before declaring the work complete.
