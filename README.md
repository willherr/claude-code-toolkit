# Claude Code toolkit

Six Claude Code skills plus a real example `CLAUDE.md`, all in daily use across my own apps under [will-i-am.dev](https://will-i-am.dev). Not a template or a generic starter kit: these are the real files, shared as-is, because that's more useful than a sanitized rewrite.

## Why these six

Directing an AI agent through real production work needs more than "write this feature." These are the pieces that keep a multi-step build controllable, reviewable, and actually shippable:

- **[`epic-workflow`](skills/epic-workflow/SKILL.md)**: splits a multi-part feature into a shared epic branch plus small, self-contained sub-issues, each one scoped so a fresh Claude session with zero prior context can pick it up and work from it directly. The alternative, one long branch and one exhausting review pass, is exactly the failure mode this exists to prevent.
- **[`pr-pipeline`](skills/pr-pipeline/SKILL.md)**: once work is reviewed and approved, takes it the rest of the way (commit, push, open PR, watch CI, merge on green) without a second round-trip for every step. The approval gate itself never moves: nothing ships without it.
- **[`share-live-build`](skills/share-live-build/SKILL.md)**: builds a real release version and serves it locally with cache-busting headers before anything goes into a PR, because a dev server or a stale cache produces a false "looks fine" review.
- **[`init-tooling`](skills/init-tooling/SKILL.md)**: brings a repo up to a consistent baseline (CLAUDE.md, issue labels, CI) by comparing it against already-mature repos, rather than working from a fixed checklist that goes stale.
- **[`subagent-orientation`](skills/subagent-orientation/SKILL.md)**: orients a spawned subagent to its role and the coordinator relationship, so multi-agent work doesn't collapse into a subagent narrating a plan back instead of executing it, or losing findings that were never actually reported up.
- **[`request-copilot-review`](skills/request-copilot-review/SKILL.md)**: puts GitHub Copilot's review bot into the loop on a PR reliably, the CLI shortcut for this is flaky, this uses the GraphQL call that actually works.

And [`CLAUDE.md`](CLAUDE.md): a genericized version of the "Dev workflow" section from my own global config, the file Claude Code reads at the start of every session. Real rules, each one written down because something went wrong once and got fixed.

## The part that doesn't show up in a demo

The interesting problem with AI-native development isn't getting an agent to write code, it's building the scaffolding that keeps many small, correct changes moving instead of one large, hard-to-review one. These four skills are that scaffolding: how work gets split up, how it gets tested against something real before merge, and how a repo stays consistent as more of it gets touched by an agent instead of by hand.

A couple of these reference personal notes files (an issue-label convention, machine-specific setup) that aren't included in this snapshot; the mechanics and the reasoning behind them are what's meant to be read here, not a drop-in kit.

## More

- [will-i-am.dev](https://will-i-am.dev): the apps these actually run against.
- [linkedin.com/in/will-herrmann-155a09103](https://linkedin.com/in/will-herrmann-155a09103)
