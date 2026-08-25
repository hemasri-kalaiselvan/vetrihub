# VetriHub — Master Build Prompt (Final Spec)

Use this as a starting prompt for building a similar personal-utility PWA
in the future. It describes the finished requirements cleanly, without the
back-and-forth history of how they were reached.

---

Build **VetriHub**, a personal utility Progressive Web App (installable,
no build tools, plain HTML/CSS/JS) containing multiple unrelated personal
tools, designed so more tools can be added the same way later.

## Overall design

- Modern, simple, personal, minimal — a personal digital utility box, not
  a commercial site.
- Dark theme; background neither pure black nor pure white.
- Fonts: one heading font (e.g. Sora), one body font (e.g. Inter).
- One accent color used sparingly (buttons, active states, badges).
- No images, illustrations, decorative graphics, or emojis. Icons only for
  real functions (menu, add, edit, delete, expand/collapse, lock,
  filter, text formatting).
- Every screen shares one header: hamburger icon + app name.

## Navigation

- Hamburger drawer lists: Home, then one entry per tool, then a
  Sign in/Sign out control at the bottom.
- Any tool that should be private shows a lock icon and is disabled in
  the drawer/Home until the owner is signed in.
- Home lists every tool as a plain name + arrow, nothing else.

## Authentication

- Single-owner app: one person's Google account is the "owner." Anyone
  else who signs in is immediately signed back out with "No access."
- Use a full-page OAuth redirect flow (not a popup) — required for
  reliable sign-in inside an installed PWA.
- Request the narrowest OAuth scope that works. If the app needs to
  create its own Sheets/Docs, `drive.file` is enough — but note it only
  covers files the app itself creates, not ones a person pre-makes
  manually.
- Cache the access token client-side for its real lifetime (~1 hour) and
  attempt a silent (no-prompt) re-auth on return visits so sign-in isn't
  required every single time. Be upfront that a fully backend-less app
  cannot guarantee a permanent login — that needs a real server holding a
  refresh token.

## Data backend

- No backend server. Use Google Sheets as structured storage and Google
  Drive's sharing permissions as the access-control layer:
  - Private data → a Sheet that is never shared.
  - Public-readable data → a Sheet shared "Anyone with the link — Viewer,"
    read via a restricted, unauthenticated API key; writes still require
    owner OAuth.
- Auto-create each tool's backing Sheet on the owner's first sign-in
  (needed if using `drive.file` scope), then hardcode the resulting file
  ID into the app's config once — public reads need a fixed, known ID.
- For any write operation, avoid "read everything, clear it, rewrite
  everything" as the save pattern — it's simple but races badly when two
  saves happen close together. Prefer:
  - **Add** → a pure atomic append (no read beforehand at all).
  - **Edit/Delete** → look up the one specific row, then touch only
    that row.
- If a human-readable mirror (e.g. a Google Doc copy) is wanted for
  sharing/reference, make it write-only — the app should never read data
  back from it, only overwrite it after a successful save to the real
  source of truth. A mirror-sync failure must never undo or block the
  actual save.
- Always disable HTTP caching on reads used to inform a write (fetch with
  `cache: "no-store"`), and route all writes to one piece of data through
  a single in-page queue so overlapping saves can't interleave.
- Organize every file the app creates into one dedicated Drive folder,
  automatically, on first sign-in.

## Per-tool feature pattern (repeat for each list-style tool)

- Header: tool name + `+` to add.
- Search box (matches on title/name only).
- List: title only by default; tap to expand and show the body/detail.
- Add/Edit: a modal form. Only the title/name is required — allow saving
  with an empty body, and treat that as a distinct "pending"/incomplete
  state, visually tagged, filterable via its own icon (owner-only), and
  hidden from public/anonymous viewers by default (a view-level filter,
  not a hard security boundary if the backing sheet is public anyway —
  say so explicitly rather than overpromising true privacy).
- Delete: always behind a confirmation dialog.
- Offer a "bulk add" alternate mode: paste many entries at once,
  blank-line-separated, first line of each block = title, remaining
  lines = body (empty body = pending). Save as one atomic multi-row
  append.
- If rich formatting is needed for the body, use a `contenteditable` +
  `execCommand` toolbar (bold/italic/underline/lists/quote/heading sizes)
  rather than pulling in an external editor library — store the resulting
  HTML directly and render it as-is.

## Cross-cutting requirements

- Build an on-screen, owner-only error/diagnostic log (a small button
  that shows recent errors as plain text) — mobile users can't easily open
  browser dev tools, so this is the only practical way to debug live
  issues on a phone.
- Persist which screen the user was on across a page refresh.
- Disable relevant buttons and show a "Saving…" state during any write to
  prevent double-submits.
- Package as a PWA: manifest.json (`start_url: "./"` — a root path,
  consistent whether launched from a browser tab or the installed icon),
  a basic offline-caching service worker, and two icon sizes (192/512).
