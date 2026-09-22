---
name: request-copilot-review
description: >-
  Request a GitHub Copilot code review on a pull request via the GitHub API.
  Use whenever a PR pipeline calls for requesting a Copilot review (see each
  project's own policy on WHEN to request one -- this skill only covers HOW).
  The obvious `gh pr edit <n> --add-reviewer @copilot` CLI shortcut is
  unreliable for this bot reviewer -- use this skill's GraphQL method instead
  of re-deriving it from scratch.
---

# Requesting a Copilot review (the reliable way)

## The wrong way (unreliable)

`gh pr edit <n> --add-reviewer @copilot` (or `@Copilot`) uses the REST
`requested_reviewers` endpoint, which is built for `User`/`Team` reviewers.
Copilot's reviewer identity (`copilot-pull-request-reviewer`) is a `Bot` node
in GitHub's GraphQL schema, not a `User`. In practice this CLI call has
returned success (prints the PR URL, no error) while silently no-op'ing --
confirmed failing on `scottgroupstudio2/jt_pgs` PR #1708 (2026-09-14), despite
an earlier session logging it as "confirmed working" on a different PR five
days prior. Don't trust it either way; use the method below and verify.

## The right way: GraphQL `requestReviews` with `botIds`

The `requestReviews` mutation has separate `userIds`/`botIds`/`teamIds`
fields on its input type -- Copilot must go through `botIds`. Steps:

1. **Get Copilot's bot node ID for this repo.** It's per-repo-installation,
   not a global constant -- re-derive it each time you work in a new repo
   (or cache it, but verify before trusting a stale value). Easiest source:
   any PR in the repo that already has a Copilot review or request on it.
   ```
   gh api graphql -f query='
   {
     repository(owner: "<owner>", name: "<repo>") {
       pullRequest(number: <any PR with a prior Copilot review>) {
         reviews(first: 5) { nodes { author { __typename login ... on Bot { id } } } }
       }
     }
   }'
   ```
   Look for `login: "copilot-pull-request-reviewer"` and grab its `id`
   (looks like `BOT_kgDOCnlnWA`). If no PR in the repo has ever had a Copilot
   review, `reviewRequests` on any PR where a *request* was made (not
   necessarily completed) also carries `requestedReviewer { ... on Bot { id
   login } }`.

2. **Get the target PR's node ID:**
   ```
   gh api graphql -f query='{ repository(owner: "<owner>", name: "<repo>") { pullRequest(number: <n>) { id } } }'
   ```

3. **Request the review:**
   ```
   gh api graphql -f query='
   mutation {
     requestReviews(input: {pullRequestId: "<PR node id>", botIds: ["<bot node id>"], union: true}) {
       pullRequest { number }
     }
   }'
   ```

4. **Verify the request landed** (this DOES show up immediately, unlike the
   `--add-reviewer` CLI path):
   ```
   gh api graphql -f query='
   {
     repository(owner: "<owner>", name: "<repo>") {
       pullRequest(number: <n>) {
         reviewRequests(first: 5) { nodes { requestedReviewer { __typename ... on Bot { login } ... on User { login } } } }
       }
     }
   }'
   ```

5. **Wait for the actual review**, then check via
   `gh pr view <n> --json reviews` (look for `author.login ==
   "copilot-pull-request-reviewer"`). Turnaround has varied from under a
   minute to ~10 minutes in practice -- don't conclude "it didn't work" from
   an empty result after only a minute or two; recheck a few minutes later
   before troubleshooting further.

## If `RequestReviewsInput` ever seems to have changed

Confirm the field name via introspection rather than guessing:
```
gh api graphql -f query='{ __type(name: "RequestReviewsInput") { inputFields { name } } }'
```

## Scope

This skill only covers the "how" of the request. Whether to request a
Copilot review at all (quota limits, org rulesets, "skip for mechanical
changes" policies, etc.) is project-specific -- check that project's own
saved PR-pipeline definition first.
