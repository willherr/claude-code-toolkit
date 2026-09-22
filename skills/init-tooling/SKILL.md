---
name: init-tooling
description: Bring a repo up to Will's standard Claude Code tooling baseline (CLAUDE.md skeleton, issue label convention, CI if there's something real to check) by comparing it against his established repos. Use when asked to "init tooling", "set up Claude tooling", or bootstrap a repo that doesn't have a CLAUDE.md yet.
---

Global skill (works on any of Will's repos), not project-specific. Takes
one argument: the target repo (a path, or a `willherr/<repo>` GitHub
slug if it's not checked out locally). Investigates what "standard
tooling" actually looks like right now — don't work from a fixed
memorized checklist, since the baseline itself evolves — then closes
the gaps that make sense for *this* repo's actual shape. Don't
transplant sections wholesale from a reference repo; each one is
repo-specific and goes stale immediately if copied blind.

## Step 1: survey the target repo's current state

- `CLAUDE.md` at the repo root — exists? Does it already state the repo's visibility (see the dedicated bullet in Step 3)?
- `gh repo view willherr/<repo> --json visibility,isArchived` — public or private, and is it archived?
- `.github/workflows/*.yml` — any CI at all?
- `.claude/skills/` — any project-specific skills already?
- `gh label list --repo willherr/<repo>` — which labels exist?
- What kind of repo is this? Look for `pubspec.yaml` (Flutter),
  `wrangler.jsonc`/`package.json` (Cloudflare Worker/Node), a plain
  static site (`.nojekyll`, raw `.html`/`.css`, no build manifest), or
  something else. This determines what "Commands"/CI even mean here —
  don't invent a build/test step that doesn't exist.

## Step 2: survey 2-3 of Will's mature repos as the current reference

Don't assume the shape from memory — re-check live, since these
evolve. `Cassandras10Key` is the most mature/lived-in reference
(explicitly called out as such in its own and CassandrasCookbook's
`CLAUDE.md`); pull in `Cardwright` or `CassandrasCookbook` too if the
target repo's shape (static site, Worker, etc.) matches one of those
better than 10-Key's Flutter-app shape. Look at:

- Root `CLAUDE.md`'s actual section structure (Project overview, Dev
  workflow, Commands, Architecture, Notes) and its exact wording for
  the durable, repo-agnostic rules (branch naming, the non-app-change
  commit-to-main exception, the CLAUDE.md-drift-audit reminder,
  pointing at `~/.claude/notes/issue-labels.md` rather than inlining
  the label scheme).
- `.github/workflows/ci.yml` (or a project-specific one, e.g.
  `marketing-site-ci.yml`) — only relevant if the target repo has an
  analogous real check to run. A workflow that runs nothing of
  substance is worse than no workflow.
- `~/.claude/notes/issue-labels.md` for the current label convention
  and its reconciliation rule.

## Step 3: close the gaps, using judgment on what actually applies

- **Repository visibility**: every repo's `CLAUDE.md` states its
  GitHub visibility (`PUBLIC` or `PRIVATE`) as its own short section,
  right after the file's intro paragraph and before Project overview —
  add it if missing, whether or not the rest of the file needs work.
  For `PUBLIC`, note that nothing sensitive (client names, deal terms,
  keys, personal contact info) belongs in an issue/PR/commit here, and
  flag before adding anything that wouldn't be fine for a stranger to
  read. For `PRIVATE`, a one-line flag-before-assuming-otherwise note
  is enough. Standing practice since 2026-09-16, prompted by a real
  incident (`willherr/will-i-am.dev`#30 — a client negotiation posted
  to a public issue got scraped by bounty-bot spam within days).
- **`CLAUDE.md`**: if missing, write one scoped to what's real *today*
  in the target repo — Project overview (what it is, why it exists,
  current architecture), Dev workflow (branch naming
  `issue#<N>/PascalTitle`, PR-per-issue with the non-app-change
  exception, a pointer to the issue-label notes rather than restating
  them), Commands (only if there's something to actually run/build/
  test — omit the section rather than inventing one), Notes (the
  audit-for-drift-on-issue-close reminder). Keep it short — this file
  loads into every session regardless of task, so it covers durable
  invariants only, not narrative. If the repo has open issues
  describing a planned rewrite/restructure, say so explicitly and note
  that this file will need a real pass once that lands, rather than
  writing detailed Architecture for a structure that's about to
  change.
- **Issue labels**: `gh label list --repo willherr/<repo>`. If none of
  the convention's labels exist yet, create the full set
  (`brainstorming`, `needs decision`, `priority: high/medium/low`) —
  see `~/.claude/notes/issue-labels.md` for exact names/colors/
  descriptions, matching what's already live on the reference repos
  exactly (`gh label create`, or edit an existing near-miss rather than
  duplicating). If the repo already has a *different* similar-but-not-
  identical scheme, don't silently layer a second one on top or
  overwrite it — stop and ask Will how he wants it reconciled, per the
  notes file's own rule.
- **CI**: only add a workflow if the repo has a real build/test/lint
  command worth gating on. Don't create a workflow that just checks
  out code and does nothing meaningful.
- **`.claude/skills/`**: only add a project-specific skill if there's
  already a concrete, repeatable manual process to encode (a deploy
  script, a run/screenshot pipeline) — global skills already cover the
  cross-repo stuff (`share-live-build`, `pr-pipeline`, `record-ui-demo`,
  `run`), so don't duplicate those into the project.

## Step 4: commit and report

- Non-application tooling changes (CLAUDE.md, `.github/workflows`,
  nothing touching real app code) commit straight to `main` per every
  repo's own documented exception — no branch/PR needed for this kind
  of change.
- Report back concisely what was added, what was deliberately skipped
  and why (e.g. "no CI — no build/test step exists yet"), and anything
  that needs Will's call before proceeding (a label-scheme conflict,
  an ambiguous repo shape).
