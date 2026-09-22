---
name: share-live-build
description: Build a release version of a web app and serve it locally so Will can actually test it before a PR opens. Use whenever real feature work is ready to test, not just a one-off verification screenshot — proactively, don't wait to be asked.
---

Global skill (works in any of Will's repos), not project-specific.
Whenever real feature work is in progress, send a live URL to test
before opening a PR — don't wait to be asked, this is standing
behavior. Include a short numbered "what to test" list alongside the
link every time, not just the bare URL.

## Steps

1. **Build a release version, never the debug dev server.** For
   Flutter web: `flutter build web --release`. Debug/DDC is slow to
   first-load and isn't representative, and Flutter's injected DWDS
   hot-reload client has a real bug that can leave a real person's
   browser on a permanently blank white page with no visible error
   (confirmed reproducible on Will's actual machine, unrelated to his
   browser/profile). Check for a project's own web-run skill first —
   use its exact build/serve commands if one exists.

2. **Serve with `Cache-Control: no-store` headers — never a bare
   `python -m http.server`.** Flutter web's own PWA service worker
   caches the whole app in Cache Storage independent of HTTP headers on
   a first pass, and a plain `http.server` sends no cache-busting
   headers at all — the two compound so a real rebuild can sit
   correctly on disk while a browser (even a fresh one hitting the same
   URL) keeps serving the stale pre-rebuild version. Use (or create, if
   the project doesn't have one) a small server that sets
   `Cache-Control: no-store` on every response — see
   `Cassandras10Key` or `CassandrasCookbook`'s own
   `tool/serve_no_cache.py` for the pattern to copy into a new project.

3. **Default to a local URL (`localhost:<port>`), not a devtunnel.**
   Will is normally at this PC when testing a live build, so a public
   tunnel is the exception, not the default — "if I need a devtunnel
   I'll ask for it." Only stand up a devtunnel when he explicitly asks
   for one (e.g. to test from his phone, or send a link to someone
   else). See `~/.claude/notes/machine-quirks.md` for devtunnel
   setup/gotchas and this machine's shared-port ownership table if a
   tunnel is actually needed.

4. **Never `Start-Process` the link on his behalf.** Always include the
   actual clickable URL as literal text near the end of the response —
   not a description like "the homepage" that makes him guess or
   re-derive it. This applies to reference/comparison links too (e.g.
   opening a competitor site for him to look at), not just this
   project's own live build. Claude in Chrome (or Playwright) remains
   fine to use when *you* need to click around, read the rendered page,
   or interact with it yourself — only the auto-open-to-show-him habit
   is off.

5. **Include a numbered "what to test" list** with the link — specific
   things to check (a breakpoint, a theme toggle, a flow to click
   through), not just "let me know what you think."

## Recording a demo instead of / alongside a live link

Default to MP4 (h264) for a recorded UI walkthrough, not animated GIF —
a verified-valid, correctly-encoded GIF has displayed as static (no
animation) in Will's chat client before; MP4 of the identical content
played first try. Only use GIF if the destination specifically
requires it, and verify it actually animates before sending. See the
`record-ui-demo` skill for the actual recording pipeline.
