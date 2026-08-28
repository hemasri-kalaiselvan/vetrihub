# VetriHub

> A personal utility Progressive Web App (PWA) that brings together several everyday tools in one place.

**Tech:** HTML, CSS, JavaScript, PWA
**Tools:** GitHub
**AI Tools:** ChatGPT, Claude

## About

- A personal utility app that keeps practical everyday tools together under one roof, rather than spread across separate apps.
- Includes three tools: a private **Cycle Tracker**, a **Recipe** collection, and a **Situations & Solutions** reference.
- Built as an installable PWA, designed to grow — new tools can be added the same way over time.
- Two kinds of pages: **public** (Recipe, Situations & Solutions) that anyone can view, and **private** (Cycle Tracker), marked with a lock icon and visible only to the owner.
- Owner sign-in is used to view private pages and to add, edit, or delete content.

## How AI Helped

- **ChatGPT** — initial design direction and the first build prompt
- **Claude** — code, debugging, and deployment
- Worked through real, specific problems — like getting the OAuth redirect right for an installed PWA
- Used Google Sign-In and Google Sheets as a lightweight backend, with no server to run

Full documentation:

- [Design Document](docs/Design-Document.md)
- [Prompt Document](docs/Prompt-Document.md)
- [Build & Deployment](docs/Build-and-Deployment.md)

---

## Tech notes

- A single-file PWA (HTML, CSS, and JavaScript) with no build step.
- Google Sign-In (OAuth) for the owner, with a restricted API key for public reads.
- Google Sheets store the data; the app creates its own files on first sign-in.
