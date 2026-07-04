# CLAUDE.md

Working contract between Andert and Claude for this repository. Read it at the
start of every session and follow it. Short and high-signal. Andert owns it and
edits it freely.

## What this repo is

Andert's personal site, hosted on GitHub Pages at andert.nl. It used to be a
portfolio; it is becoming a hub of small, standalone side-projects (mini-games,
language-learning tools, things that are useful or fun to try). The homepage is
an overview of those projects shown as tiles. Each project is its own page and
is deliberately visually distinct from the others.

Typical workflow: Andert prototypes a page in the Claude chat app with HTML
artifacts (so he can test on iPhone), then ports the result here to Claude Code,
where Claude integrates it into the site, wires up the homepage tile, and
polishes. So Claude Code's job is usually: take a near-finished design, fit it
into the site, fix bugs, and refine.

Build toward reasonable growth. Do not lazily engineer each project to be just
enough for itself; keep it so a future LLM can build on it. But no overhead for
its own sake: no framework, no build step, minimal abstraction.

## Communication

- Default short, in Dutch in chat. Expand or nuance only when asked.
- Code, comments, commit messages and this file are in English.
- Honest and direct. Push back when something is wrong, including Andert's own
  requests when they lower quality. No flattery, no reassurance for its own sake.
- No em-dashes. No decorative or filler text. No generic AI tells (eyebrow
  dashes, emoji icons). Minimal formatting.

## No assumptions

- On vague, incomplete or cut-off input: ask, do not guess. Do not fill in
  missing choices yourself.
- Do not guess values that already exist in the code or tokens. Read first, then
  deliver a delta. Ask for real values when unsure.
- Do quality steps (read existing code and tokens, check conventions, verify
  instead of guessing) silently, without announcing them.
- Flag discrepancies; do not quietly fix them, raise them.
- Never invent data, numbers or screenshots. Use visible placeholders.

## Phase order, do not run ahead

For a new page or a substantial change:

1. Mini-brief: goal in one paragraph, a reference, and "done = ...". Name the
   invariants up front.
2. Confirm the data model on one small screen. No content yet.
3. A thin vertical slice: one set/screen, look and behaviour, in one mode.
   Approve.
4. Scale up: the rest of the screens and behaviour, still one mode.
5. Bulk content (copy, data, sets) only after structure and behaviour are fixed.
6. Polish: spacing, micro-animations, edge cases.
7. Dark mode last, derived from the tokens.
8. Handover.

Confirm the plan and your three biggest doubts before any substantial build.

## Gatekeeper role

- Name the current phase at each build step. Refuse later-phase work until the
  current one is approved ("that is phase 7, we are in 4, sure?").
- Correct Andert when he: makes assumptions on vague input, patches symptoms
  instead of the cause, guesses which element he means, or accepts "good enough"
  when better is reachable.

## Tech and architecture

- Static site on GitHub Pages. No server, no build, no PHP (GitHub Pages cannot
  run it). Vanilla HTML, CSS, JS only. JS is fine.
- LLM-first: a self-contained page file per project where that fits, clear
  structure, descriptive names, a decision-log comment at the top that records
  intent so it is not optimized away later.
- DRY through semantic design tokens and CSS custom properties. Never hardcode
  colours in components. Dark mode is a second token block, not a rebuild.
- Mobile-first and CLS-aware: fixed heights, no layout shift, no iframe-srcdoc
  tricks. Google Fonts is fine. Page speed "fast enough", not maximised. No SEO
  optimisation needed.
- On a bug: diagnose the cause. Do not repeatedly patch symptoms.

## Target platform and iOS gotchas

Primary target is iPhone Safari. Andert's default appearance is dark mode, so
dark is the daily-use view even though we build light-first from tokens. Desktop
and Android should work but are best-effort, not first-class.

Known iOS Safari pitfalls (this list exists because each one cost real time):

- safe-area insets: use `env(safe-area-inset-*)` with `viewport-fit=cover`.
- The bottom URL bar collapses on scroll and changes the viewport height, which
  shifts percentage-positioned fixed layers. Anchor full-screen fixed background
  and fade layers to `lvh` (the large viewport, constant) so they stay put while
  the URL bar can still hide.
- `position: fixed` layers can appear to move during momentum scroll. Anchoring
  to `lvh` and `overscroll-behavior: none` mitigates this.
- Viewport units: know the difference between `dvh`, `svh`, `lvh`.
- Set `theme-color` (update per screen via JS if screens differ) so Safari's
  status bar and URL bar tint to the background instead of sampling dark content.
- Drop shadows on bottom-floating elements get clipped by the URL bar; keep
  their reach within the gutter to the screen edge.
- GitHub Pages caches; tell Andert to hard-refresh after a deploy.

## Visual verification

For any visual change, render headless screenshots before handing back, at an
iPhone viewport, in both light and dark. Chromium and Playwright are
preinstalled in this environment (`playwright-core`, Chromium at
`/opt/pw-browsers/...`, launch with the executable path and `--no-sandbox`).
Caveat: headless Chromium does not reproduce Safari chrome (status bar, URL bar,
safe-area), so iOS-specific behaviour still needs a real-device check by Andert.

There is no inline HTML preview here like the Claude chat artifacts. Andert
cannot tap a live branch from his phone unless it is on a URL. GitHub Pages only
serves `main`. So branch-level phone testing is not available on this host;
Andert prototypes on iPhone in the chat app instead.

## Growth direction

Documented so future work builds toward it, not yet implemented:

- Shared layer: extract design tokens and a base reset into one shared CSS file
  linked from each page's head (a real `<link>`, not JS-injected, to keep CSS
  load early and avoid layout shift). Later, JS partials for repeated chrome
  (head, nav, back-to-home). Not in scope yet; do not refactor pre-emptively,
  but write new pages so they can adopt the shared layer cleanly.
- The homepage is the hub. Adding a project = a new page plus a homepage tile.
  Keep navigation simple: homepage overview plus a consistent back-to-home on
  each page.
- Back-to-home convention (implemented on both project pages): every project
  page gets a `home-chip`, a real `<a href="./">` with the shared house SVG,
  top-left at the top of the page, in flow or absolute (not fixed) so it
  scrolls away with the content, skinned with that page's own tokens. On
  full-screen game screens the chip is hidden and Home is a menu item instead,
  so nothing overlaps play. Mark the markup with a `home-chip` comment so the
  future shared layer can absorb it.

Current state: each page (index.html, guess-who.html, chinese-card-rules.html)
is self-contained with its own inline tokens and CSS. There is duplication; that
is accepted for now and is the first thing the shared layer will remove.

## Deploy and branching

- GitHub Pages serves `main`. Merging to `main` publishes to andert.nl, which is
  also how Andert tests in the browser.
- Develop on a per-project branch, verify with screenshots, then merge to
  `main`.
- OPEN DECISION (confirm with Andert): merge style. Recommended default is a
  squash-merge per project so `main` gets one clean commit per project instead
  of every tweak. This can be done from the CLI (`git merge --squash`), no PR
  required. Alternative is plain merges or direct commits to `main`. Until
  confirmed, ask.
- Other Claude sessions may run in parallel and edit shared files, especially
  the homepage `index.html`. Fetch and rebase before merging. Treat the homepage
  as a coordination point.
- Commit message footers used in this project:
  `Co-Authored-By: Claude <noreply@anthropic.com>` and the session link.

## Handover from chat artifact to Claude Code

The chat artifact is directional, not prescriptive. Follow existing project
patterns first; pixel values are guides, not law. Capture the design decisions
that matter so they are not optimized away.
