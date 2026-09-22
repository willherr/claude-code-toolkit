---
name: pr-pipeline
description: Take already-approved work all the way from working tree to a merged PR — commit, push, open the PR, watch CI, and merge on green — in one go. Use when Will says "pr pipeline," "run the pipeline," or similar, after he has already approved the actual work (e.g. from a shared live build).
---

Global skill (works in any of Will's repos), not project-specific.

## What this phrase means

Per `~/.claude/CLAUDE.md`'s "No PR gets created, and no PR gets merged,
without Will's explicit confirmation first" rule: Will saying **"pr
pipeline"** (or invoking this skill) *is* that confirmation, for the
whole rest of the flow through merge. Confirmed 2026-09-06
(CassandrasCookbook #28/PR #37) — he does not want to separately say
"merge" after CI goes green; "pr pipeline" already covers it.

This does **not** waive the step *before* it: the work itself (the
actual feature, fix, or change) still needs to have been shared and
approved first — a live build per "Sharing work for live testing" in
the global CLAUDE.md, or otherwise clearly OK'd — before this skill
runs. "Pr pipeline" is the go-ahead to *ship* already-approved work,
not permission to skip getting the work itself approved.

The CI-green gate is also not waived. If CI fails, stop and report why
— don't merge anyway, and don't loop retrying a failing check.

## Steps

1. **Loose-ends check** before touching git — per the global CLAUDE.md's
   "run a loose-ends check before considering the work done" convention:
   - Stop any stray background processes started this session (dev
     servers, duplicate port bindings), using the project's documented
     safe-restart method if one exists.
   - `git status` in every repo actually touched this session (not just
     the primary one) — nothing stray, nothing left uncommitted that
     should be.
   - Check whether any other surface needs the same update (other
     screenshot sets, a persistent demo server, a marketing page) —
     don't stop at the issue's own literal checklist.

2. **Commit.** Stage only the files that belong to this change (never
   a blanket `git add -A`/`.`). Message follows this repo's own commit
   conventions if documented, otherwise a normal concise subject +
   body. End with the attribution block the session's system reminder
   specifies (commit trailer + session link) if one is given.

3. **Push** the feature branch (`git push -u origin <branch>` if it has
   no upstream yet). Branch naming and workflow otherwise follow
   whatever this repo's own CLAUDE.md documents (e.g. `issue#<N>/PascalTitle`).

4. **Open the PR** (`gh pr create`). Body needs a real GitHub closing
   keyword for any issue(s) it closes (`Closes #N`, not just a bare
   reference) — and for **multiple** issues, repeat the keyword before
   every single one (`Closes #82, closes #87, closes #88`, never a
   keyword followed by a comma list) since GitHub only reliably parses
   the first issue in a comma list as a closing reference. Include a
   Summary and Test plan, and the PR-description attribution footer the
   session's system reminder specifies, if one is given.

5. **Watch CI**: `gh pr checks <N> --watch`.
   - **Pass** → proceed to merge (next step) without asking again.
   - **Fail** → stop. Report which check failed and why (pull the
     actual failure output, don't just say "CI failed"). Do not merge.
     Fix and re-push if the fix is clear, or ask Will how he wants to
     proceed if it isn't.

6. **Merge on green**: `gh pr merge <N> --squash --delete-branch`
   (squash is this workflow's default — match the repo's own history if
   it clearly prefers merge commits instead). This routes through
   whatever `ask`-permission prompt `~/.claude/settings.json` has
   configured for `gh pr merge` — that's an intentional tooling
   backstop per the global CLAUDE.md, not something to route around;
   let it prompt.

7. **Post-merge loose-ends check** — same convention, now that the
   change is actually shipped:
   - Confirm the issue(s) the PR referenced actually closed (a closing
     keyword usually does this automatically; verify rather than assume,
     especially for a multi-issue PR).
   - `git fetch --prune` and confirm local tracking branches/refs are
     clean (no dangling `origin/HEAD`, no stale local copy of the now-
     deleted remote branch).
   - Re-check whether anything from step 1's wider-surface scan still
     needs attention now that the change is live (e.g. a persistent
     demo server that needs the new build pushed to it).
   - Report a short summary: what merged, what's still open/deferred
     (filed issues, flagged-but-not-fixed items), and any loose ends
     that remain outside this skill's scope (e.g. a sibling repo with
     the same gap, per the cross-repo-consistency convention — flag it,
     don't silently fix it there too).
