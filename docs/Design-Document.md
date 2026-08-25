# VetriHub — Design Document (Final State)

A personal-utility Progressive Web App (PWA): a single installable app
containing multiple unrelated personal tools. Currently three: **Cycle
Tracker**, **Recipe**, and **Situations & Solutions**. Built to allow more
utilities to be added the same way later.

This document describes the app **as it stands now** — not the design
history, reverted approaches, or abandoned ideas along the way.

---

## 1. Visual Design

- Dark theme. Background is deliberately neither pure black nor pure white
  (`#20242f`).
- Typography: **Sora** (headings/titles) + **Inter** (body/UI text), loaded
  from Google Fonts.
- Accent color: warm amber (`#ffb454`) against the dark slate background.
  A secondary danger/red is used only for delete actions and error states.
- No photos, illustrations, decorative graphics, or emojis anywhere.
- Icons are used only where they serve a real function: hamburger menu,
  right-arrow (Home links), `+` (add), edit/trash (row actions), a chevron
  (expand/collapse), a lock (private/locked state), a clock (pending
  filter), and a small rich-text toolbar (bold/italic/underline/bulleted
  list/quote).
- Every screen shares one header: `☰ VetriHub` — hamburger icon + app name,
  never omitted.

---

## 2. Navigation

### Header (all screens)
Hamburger icon + "VetriHub" title, sticky at the top.

### Hamburger drawer
- Home
- Cycle Tracker — shows a small lock icon and is disabled/greyed out
  until signed in as the owner.
- Recipe
- Situations & Solutions
- (divider)
- Sign in / Sign out (label toggles based on auth state)

### Home screen
Three links, each just a name + right arrow, nothing else:
- Cycle Tracker → (lock icon + greyed out when signed out, not navigable)
- Recipe →
- Situations & Solutions →

---

## 3. Authentication Model

- Single **owner** identity: one hardcoded Gmail address. Sign-in always
  forces Google's account picker; the app checks the signed-in email
  against the owner address. A mismatch immediately signs the account back
  out with a plain "No access." message — no partial or half-signed-in
  state is ever shown.
- OAuth flow: a hand-rolled **full-page redirect** (not a popup, and not
  Google Identity Services' popup-based token client) — popups are
  unreliable once a PWA is installed to the home screen; a redirect works
  identically whether launched from a browser tab or the installed icon.
- Scope: `openid email https://www.googleapis.com/auth/drive.file` — the
  narrowest scope available. `drive.file` only grants access to files the
  app itself creates, which is why every backing Sheet/Doc is created
  *by* the app on first sign-in rather than pre-made manually.
- Session handling: no backend exists, so there is no way to get a truly
  permanent login — Google does not issue long-lived refresh tokens to
  pure client-side apps. The app caches the access token for its real
  ~1 hour lifetime and silently retries sign-in (no visible prompt) if a
  previous session is remembered but the cached token has expired. This
  meaningfully reduces, but cannot fully eliminate, repeated sign-in
  prompts — especially since mobile OSes can fully kill a backgrounded
  installed PWA, which forces a fresh session on next open.

---

## 4. Data Architecture

No backend server. Google Drive itself is the backend, accessed two ways:

- **Owner-authenticated calls** (OAuth bearer token) — used for every
  write, and for reads of anything private.
- **Unauthenticated calls** (a separate, restricted API key) — used only
  for public, read-only access to content that's meant to be world-
  readable (Recipe and Situations & Solutions).

| Feature | Storage | Visibility |
|---|---|---|
| Cycle Tracker | Google Sheet "VetriHub Cycles" | Private — owner only, never shared |
| Recipe | Google Sheet "VetriHub Recipes Data" | Public read (link-shared, Viewer), owner-only write |
| Recipe (reference mirror) | Google Doc "VetriHub Recipes" | Public read, but the **app never reads from it** — write-only courtesy copy |
| Situations & Solutions | Google Sheet "VetriHub Situations Data" | Public read (link-shared, Viewer), owner-only write |

All four files live inside a single **"VetriHub" folder** in the owner's
Drive, created and organized automatically by the app on first sign-in.

### Why two different save strategies exist
- **Cycle Tracker** saves by reading the whole sheet, clearing it, and
  rewriting everything on every change. This is simple but carries a real
  (if currently accepted) risk: two saves close together can race and one
  can silently overwrite the other. This was a deliberate, informed
  trade-off — left as-is because Cycle Tracker is single-entry-at-a-time
  in practice and hasn't caused problems.
- **Recipe and Situations & Solutions** use atomic, targeted operations
  instead: adding is a pure **append** (no read beforehand, so it cannot
  race against existing rows at all); editing and deleting locate the one
  specific row and touch only that row. This was a direct fix after the
  whole-sheet-rewrite pattern caused real data loss in testing.

---

## 5. Cycle Tracker (private)

- Header row: "Cycle Tracker" + `+` beside the title (not the app
  heading). No subtitle.
- Table columns: **Date | Gap (days) | Expected Next | Actions**
  - Date format: `MMM DD, YYYY` (e.g. `Oct 23, 2024`), optional time shown
    appended (`h:mm AM/PM`) if given.
  - Gap and Expected Next are calculated automatically from the dates —
    never entered manually — and also written back to the Sheet as extra
    columns purely so they're visible if you open the Sheet directly.
  - Sorted newest-first; inserting an older/missed entry re-sorts
    automatically.
  - Sticky header while scrolling.
  - Actions column: Edit + Delete icons, always visible per row, icons
    only (no text labels).
- Add/Edit: a modal with a native date picker (required) and native time
  picker (optional). Save validates, recalculates, re-sorts, closes.
  Cancel discards changes entirely.
- Delete: always requires a confirmation dialog first.
- Reachability: the whole screen — on Home and in the drawer — shows a
  lock icon and is not navigable at all until signed in as the owner.

---

## 6. Recipe (public read, owner write)

- Header row: "Recipe" + a `+` icon; owner also sees a second icon (a
  clock, with a red count badge) for filtering to **pending** recipes.
- **Search**: simple name-only search box.
- **List**: each entry shows only the **name**; tapping expands/collapses
  the instructions (no ingredients, times, servings, or other metadata).
- **Pending vs. complete**: a recipe saved with blank instructions is
  "Pending" — tagged with a small badge in the list, and title-only.
  - Public visitors only ever see recipes that have instructions.
  - The signed-in owner sees everything, with pending ones tagged.
  - This is a **view-level filter only**, not a hard security boundary —
    since the sheet itself is publicly link-shared and its ID/key are
    visible in the app's own source, a technically determined visitor
    could still query the raw data directly. Accepted as a deliberate,
    explicit trade-off rather than adding a second, genuinely private
    sheet (which was considered and rejected as unnecessary complexity).
- **Add/Edit**: name (required) + instructions (optional — blank saves as
  pending).
- **Bulk add**: an alternate mode in the same Add form — paste multiple
  recipes at once. Rule: a blank line separates each recipe; the first
  line of a block is the title, any lines after it (until the next blank
  line) are the instructions; a title with nothing after it becomes
  pending. Saved as a single atomic multi-row append.
- **Delete**: always requires confirmation first.
- **Reference mirror**: every successful save also best-effort rewrites a
  plain-text copy of all *complete* recipes into a separate Google Doc,
  purely so the owner can view/share that Doc directly in Google Docs. A
  failure here never blocks or undoes the real save.

---

## 7. Situations & Solutions (public read, owner write)

Same functional pattern as Recipe — search, add/edit/delete, expand to
read, public read / owner write, atomic save operations — with one
difference: the content field is genuinely rich text.

- **Add/Edit** form: a title field, plus a content area with a small
  toolbar — **Bold, Italic, Underline, Bulleted list, Quote**, and a
  dropdown for **Normal text / Heading (large, medium, small)**. Built
  with the browser's native `contenteditable` + `execCommand`, so no
  external editor library is required.
- Content is stored as HTML (exactly what the editor produces) and
  rendered as-is when an entry is expanded, so formatting is preserved.
- No pending/complete concept and no reference-Doc mirror for this
  feature — it wasn't requested, and the Sheet itself is public and
  readable directly if needed.

---

## 8. Cross-cutting UX details

- **Debug/error log**: a small red "Errors (n)" button appears bottom-
  right whenever something logs an error — tapping it opens a panel with
  the raw error text, so problems can be diagnosed directly on a phone
  without a computer or remote debugging. Deliberately shown **only when
  signed in as the owner**, never to public visitors.
- **Screen persistence**: the current screen is remembered across a page
  refresh (so reloading doesn't bounce back to Home).
- **No browser HTTP caching** on any data read — every fetch explicitly
  bypasses the cache, so the app always sees Google's true current state
  rather than a stale cached response.
- **Saving state**: a "Saving…" label and disabled controls prevent
  double-submits while any write is in flight.
