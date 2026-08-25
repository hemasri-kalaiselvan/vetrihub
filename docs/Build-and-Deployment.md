# VetriHub — Build & Deployment

*Final version — describes how VetriHub is built and hosted as it currently runs, on GitHub Pages.*

---

## 1. What VetriHub is (build-wise)

A single-file Progressive Web App (PWA) with no framework and no build step:

- `index.html` — all markup, CSS, and JavaScript in one file
- `manifest.json` — PWA manifest (`start_url: "./"` so it behaves the same from a browser tab or the installed home-screen icon)
- `sw.js` — a basic offline-caching service worker
- `icons/icon-192.png` and `icons/icon-512.png` — app icons

Because it is plain HTML/CSS/JS, it runs by simply opening `index.html` — nothing to compile. This also means it deploys to any static host with no build configuration.

A `CONFIG` block at the top of `index.html` holds all the identifiers the app needs (OAuth client ID, API key, owner email, and the IDs of the Google Sheets/Doc it uses). These are filled in once, after the app creates its backing files on first sign-in.

---

## 2. Google Cloud setup

VetriHub uses Google Sign-In and Google Sheets, so it needs a Google Cloud project.

### Project and APIs
- Create a Google Cloud project (e.g. "VetriHub") at console.cloud.google.com.
- Under **APIs & Services → Library**, enable **Google Sheets API**, **Google Docs API**, and **Google Drive API**.

### OAuth consent screen
- Under **Google Auth Platform → Branding**, set user type to **External** (the only option for a personal Gmail account), with the app name and the owner's contact email.
- Left in **Testing** status, which restricts sign-in to accounts explicitly added as test users.

### Scopes
- Under **Google Auth Platform → Data Access**, the app uses the narrowest scope that works: `openid`, `email`, and `https://www.googleapis.com/auth/drive.file`.
- `drive.file` grants access only to files the app itself creates — which is why the app creates its own Sheets/Doc on first sign-in rather than using pre-made ones.

### Test users
- Under **Audience → Test users**, add the owner's own Gmail. Only added test users can complete sign-in while the app is in Testing mode.

### OAuth Client ID
- Under **Clients → Create Client**, type **Web application**.
- **Authorized JavaScript origins**: `https://hemasri-kalaiselvan.github.io` (no trailing slash, domain only).
- **Authorized redirect URIs**: `https://hemasri-kalaiselvan.github.io/vetrihub/` (full path, trailing slash).
- Client ID value is stored in the app's `CONFIG` as `<CLIENT_ID>`.

### API key (for public reads)
- Under **Credentials → Create Credentials → API key**.
- **API restrictions**: restrict to **Google Sheets API** only.
- **Application restrictions**: restrict to the site's domain (`https://hemasri-kalaiselvan.github.io/*`).
- The key value is stored in `CONFIG` as `<API_KEY>`.

> Note: The API key allows only public, read-only access to the Sheets that are deliberately shared for public reading (Recipe and Situations & Solutions). It cannot access private data or the owner's account.

---

## 3. The app's data files

On the owner's first sign-in, the app automatically creates its backing files under a single **"VetriHub" folder** in the owner's Google Drive:

- Cycles Sheet — private, owner only — `<CYCLES_SHEET_ID>`
- Recipes Sheet — public read, owner write — `<RECIPES_SHEET_ID>`
- Situations Sheet — public read, owner write — `<SITUATIONS_SHEET_ID>`
- Recipes reference Doc (write-only mirror) — `<DOC_ID>`

Each new ID is shown on-screen once when created, then copied into the app's `CONFIG` so that public (no-login) reads work for everyone. These IDs are kept as placeholders in this public document.

---

## 4. Hosting on GitHub Pages

VetriHub is hosted free on GitHub Pages, served from a project subpath (`/vetrihub/`).

1. Create a public repository named **vetrihub**.
2. Upload the app files so they sit at the **root** of the repo: `index.html`, `manifest.json`, `sw.js`, and the `icons/` folder (with both PNGs inside).
3. In the repository: **Settings → Pages → Build and deployment → Source → Deploy from a branch**, with branch **main** and folder **/ (root)**. Save.
4. Because it is a plain static site (no build step), no GitHub Actions workflow is needed.
5. After the first deploy, the app is live at: `https://hemasri-kalaiselvan.github.io/vetrihub/`

### Subfolder-safe redirect
Because the app runs from the `/vetrihub/` subpath rather than a domain root, the OAuth redirect logic resolves to the current folder (`https://hemasri-kalaiselvan.github.io/vetrihub/`). This is why the Authorized redirect URI in Google Cloud must include the full `/vetrihub/` path.

---

## 5. Making updates

- Edit files directly in the GitHub repository (or upload replacements) and click **Commit changes**.
- GitHub Pages republishes automatically within a minute or two.
- Because the service worker caches files for offline use, after a redeploy you may need to fully refresh — or, on an installed copy, remove and reinstall the home-screen icon — to guarantee the newest version is running.

---

## 6. First sign-in and the "unverified app" screen

Because the app is External and left in Testing, Google shows a "Google hasn't verified this app" screen on sign-in. As the owner (and an added test user), proceed via **Advanced → Go to VetriHub (unsafe) → Continue**, then grant the requested permissions. The "unsafe" wording is Google's standard label for any unverified app; it is the owner's own app accessing the owner's own Drive.

---

## 7. Testing checklist

1. Sign in fresh (clear site data first) — confirm the right screen and account.
2. Sign in with a different Google account — confirm "No access" and no private data leaks.
3. Add, edit, delete one entry in each tool — confirm nothing else in the list is touched.
4. Open in a private/incognito tab — confirm public tools show public-only data and private tools stay locked.
5. Switch to another app and back — confirm the session survives reasonable backgrounding.
6. Reload mid-use — confirm it returns to the same screen, not Home.
7. Check the on-screen Errors panel (owner-only, bottom-right) for anything unexpected.

---

## 8. Summary

VetriHub is a single-file PWA (HTML/CSS/JS, no build step) that uses Google Sign-In and Google Sheets as its backend, with a restricted API key for public reads. It is hosted free on GitHub Pages from a public repository, served at a `/vetrihub/` subpath, and updated by committing changes directly on GitHub. All keys and file IDs are kept as placeholders in this public document.
