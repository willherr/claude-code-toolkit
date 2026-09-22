---
name: epic-workflow
description: Split a multi-part, dependent piece of work into an epic issue + small self-contained sub-issues, each on its own branch off a shared epic branch. Recognize and proactively recommend this whenever a task looks like it would otherwise become one large bulk push — don't wait for Will to ask for it.
---

Global skill (works in any of Will's repos), not project-specific. This is
the concrete recipe behind the epic-branch rule already stated briefly in
`~/.claude/CLAUDE.md`'s "Dev workflow" section — that file has the one-line
mechanical rule; this skill has the recognition trigger and the full
GitHub-issue/branch recipe.

## When to recommend this — proactively, not just when asked

Confirmed 2026-09-20 (Cassandra's 10-Key #313/#324): the referral program's
v1 was scoped and built as one large, multi-day push across backend + three
client platforms in a single continuous session/branch. Will's own
retrospective, right after it shipped: it was genuinely hard to manage as
one bulk task, and he explicitly asked that the *next* multi-part feature
(a v2 pass on the same program) be split into an epic + smaller sub-issues
from the start, specifically so each piece is small enough to hand to a
fresh Claude session with no other context.

**Recognize this pattern and suggest it, unprompted, whenever a task you're
about to scope or start has more than ~2-3 genuinely distinct pieces that
depend on each other or share a common goal** — e.g. a feature that touches
a backend plus multiple client platforms, a migration that has to happen in
stages, a research-then-build pair where the build depends on the research
landing first, or any request that's naturally phrased as "and also X, and
we should probably Y too" stacking up mid-conversation (exactly how #324
arose — three separate follow-on ideas surfaced in one conversation right
after v1 shipped). Don't wait for Will to name the pattern himself; offer
it as soon as you notice the shape.

The alternative — one long branch, one huge PR, one exhausting review pass
— is the failure mode this skill exists to prevent. If a task is genuinely
small (one file, one clear change, no real sub-parts), this is overkill;
use the repo's normal single-issue/single-branch flow instead.

## The recipe

### 1. File the epic issue first

One GitHub issue describing the overall goal, the decisions already made
(if any were reached in conversation before filing — write them down so a
sub-issue doesn't re-litigate something already settled), and a checklist
of sub-issues **using their real issue numbers**, filled in after step 2
below (a placeholder like `#TBD` is fine for the very first draft, but
don't leave it unresolved — edit the epic issue once the real numbers
exist, in the same pass).

Include, explicitly, in the epic issue body:
- The workflow itself (branch naming, sub-issues branch off the epic
  branch not `main`, PRs target the epic branch not `main`, final PR from
  the epic branch to `main` once everything's done) — so anyone (or any
  fresh session) reading the epic issue knows the mechanics without
  needing this skill loaded.
- A rough dependency order for the sub-issues if one exists (e.g. "do the
  research sub-issue before the sub-issue whose design depends on its
  answer").
- What's still genuinely undecided, called out separately from what's
  already been decided — so a sub-issue doesn't have to guess which open
  questions are load-bearing.

### 2. File each sub-issue as real, standalone working context

Per the existing "write issues as your own working context" convention —
each sub-issue needs enough detail that a **fresh Claude session with zero
conversation history** can pick it up and work from it directly, not just
enough to jog the memory of whoever was in the room when it was scoped.
Concretely, each sub-issue should include:
- `Part of epic #<N>.` as its opening line, linking back.
- Enough background that the "why" doesn't require reading the epic issue
  or any other sub-issue first (a short recap is fine — full re-derivation
  isn't necessary, just enough that the task makes sense standalone).
- A concrete implementation shape: which files/modules are likely touched,
  what the real design questions are, what to check against existing
  conventions in the repo (point at the specific doc-comment or
  `docs/notes/*.md` file that has the relevant precedent, don't restate it
  wholesale).
- Explicit open product/design questions the sub-issue shouldn't silently
  resolve on its own — flag them to surface to Will, not guess.
- A real test plan section, matching this project's existing
  automated-test-coverage standards.
- The same attribution footer any other issue in this repo uses.

Keep each one small — a single cohesive change, not another bundle. If a
sub-issue itself starts accumulating multiple distinct pieces while
drafting it, that's a sign it should split further, not a sign to stop
splitting.

### 3. Update the epic issue with real sub-issue numbers

`gh issue edit <epic>` once all sub-issues exist, replacing any placeholder
numbers with the real ones as a checklist (`- [ ] #<N> — <title>`) — this
is the one place a future session (or Will, scanning the issue list) sees
the whole group's status at a glance.

### 4. Create the epic branch off `main`

```
git checkout main && git pull
git checkout -b "issue#<epic>/PascalTitle"
git push -u origin "issue#<epic>/PascalTitle"
```

Do this as part of filing the epic, not later — it costs nothing (an empty
branch), and it means the very first sub-issue worked on already has
somewhere real to branch from.

### 5. Each sub-issue's own branch and PR

- Branch off the **epic branch**, not `main`: `git checkout "issue#<epic>/PascalTitle" && git pull && git checkout -b "issue#<sub>/PascalTitle"`.
- Work the sub-issue normally — same testing/review bar as any other
  issue in the repo.
- Open the PR **against the epic branch** (`gh pr create --base
  "issue#<epic>/PascalTitle"`), not against `main`. Everything else about
  the standing PR rules still applies unchanged: CI green, Will's explicit
  go-ahead before merge (or "pr pipeline" if he's said that), a real
  closing keyword in the PR body (`Closes #<sub>`) so the sub-issue closes
  on merge.
- Merging a sub-issue's PR into the epic branch does **not** touch `main`
  at all — `main` stays exactly as it was until the final epic PR in the
  last step.

### 6. Final epic PR to `main`

Once every sub-issue's checklist item in the epic issue is checked off:
`gh pr create --base main` from the epic branch into `main`, `Closes
#<epic>` in the body. Same CI-green-and-go-ahead gate as any other merge —
this is often the highest-stakes single PR in the whole group (it's
everything at once from `main`'s perspective), so don't treat it as a
rubber-stamp just because each individual piece was already reviewed going
into the epic branch.

## What this doesn't change

Every other standing rule still applies unchanged inside this workflow:
no PR opens or merges without Will's go-ahead (at both the sub-issue level
and the final epic level), CI must be green, tests must cover real edge
cases, live builds get shared before a sub-issue's work is considered
approved, and so on. This skill only adds structure around *how the
branches and issues are organized*, not a shortcut through anything else.
