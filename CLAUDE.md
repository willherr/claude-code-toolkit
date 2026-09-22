# Example CLAUDE.md (genericized)

This is a trimmed, de-identified version of the "Dev workflow" section from my own global `CLAUDE.md`, the file Claude Code reads at the start of every session. It's not a template to copy wholesale, every project's rules should come from its own real incidents, but it's a real example of what turns "an agent that writes code" into "an agent I trust to run a repo's workflow largely unsupervised."

The pattern behind all of it: most of these rules exist because something went wrong once, got fixed, and got written down so it wouldn't happen again. A CLAUDE.md that just states good intentions ("write clean code", "test your changes") does nothing; one built from actual incidents, with the specific failure named, is what an agent can actually act on.

## Branching and PRs

- Feature work branches off `main` (pull latest first), gets a real PR even solo, never committed straight to `main`.
- For a group of dependent issues that are really sub-parts of one larger piece of work: branch a shared epic branch off `main`, branch each individual issue off that epic branch, merge each issue branch into the epic branch as it completes, then merge the epic branch to `main` once the whole group is done. See the `epic-workflow` skill for the full recipe.
- PR bodies must use a real closing keyword ("Closes #N", not "Implements #N"), and when closing multiple issues in one PR, repeat the keyword before every issue ("Closes #82, closes #87", not "Closes #82, #87"), a single keyword followed by a list isn't reliably parsed past the first issue.

## No PR gets created, and no PR gets merged, without explicit confirmation first

CI passing and automated screenshot verification are necessary, not sufficient. Neither substitutes for a human's own hands on the build, especially for anything touching layout or UX. The flow: finish the work, share a live build, wait for a "looks good" before even opening the PR. Once open, watching CI and reporting the result is fine to do proactively, but merging still needs a separate go-ahead.

**Exception**: a single standing phrase ("pr pipeline") is a deliberate, explicit override of the "merge needs a separate go-ahead" half of this rule, agreed on in advance, not a blanket default. Once work is already approved, that one phrase covers the entire rest of the flow (commit, push, open PR, watch CI, merge on green) without a second round-trip per step. The approval gate itself never moves, only what happens after it's already been given. See the `pr-pipeline` skill.

## Keep CLAUDE.md itself honest

Audit a project's own CLAUDE.md for drift every time an issue closes, not via periodic bulk syncs, small incremental fixes beat letting staleness compound. Keep the architectural "why", the stuff load-bearing for writing correct code later, a design decision's non-obvious rationale, an invariant a future change must respect. Don't let blow-by-blow incident narrative accumulate once a fix has landed and isn't likely to recur, that's what git history and PR descriptions already carry. For pure bookkeeping, a terse one-liner with the issue/PR number beats restating the whole story.

## Tests are a hard constraint, not a nice-to-have

Every new module a feature adds (a screen, an API client, a set of parsers, a new endpoint) needs its own real tests covering actual edge cases (null/missing fields, error paths, loading/error/success states), not just whatever incidental smoke test happened to already exist. Before considering a feature done or opening its PR, check every new class/function/endpoint for at least one test exercising its real behavior. For a layer that calls out to a service, make that dependency injectable so tests don't need a real network call to exercise it.

## Match code-review effort to actual risk

Don't run a full dedicated review pass reflexively on every change, and don't default to asking which level either, pick it yourself based on what changed:

- Skip a dedicated review and self-review inline for small, mechanical, or low-blast-radius changes, a copy tweak, a config value, a straightforward fix with obvious test coverage.
- Reach for a real review pass on anything touching money, auth or trust boundaries, data integrity (migrations, anti-replay/idempotency logic), security-relevant client/server contracts, or a diff too large to hold fully in your own head.
- Bias slightly toward running it when genuinely unsure, a skipped review that would have caught something costs far more than an unnecessary low-effort one.

## After shipping, check the loose ends, not just the issue's own checklist

A PR merge or equivalent "shipped" moment isn't the end of the task. Check: every surface that shows the same feature the change touched, not just the one the issue happened to name; any persistent/standing deployment that needs the new build pushed to it; stray background processes started during the session; and that git status is clean and pushed in every repo actually touched, not just the primary one.
