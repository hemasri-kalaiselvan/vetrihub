# VetriHub

> A personal utility Progressive Web App (PWA) that brings together several everyday tools in one place.

**Tech:** HTML, CSS, JavaScript, PWA
**Tools:** GitHub
**AI Tools:** ChatGPT, Claude

## About

VetriHub is a personal utility app that keeps a set of practical everyday tools together under one roof, rather than spread across separate apps. It currently includes three tools: a private **Cycle Tracker**, a **Recipe** collection, and a **Situations & Solutions** reference. It's built as an installable PWA and designed to grow, so new tools can be added the same way over time.

The app has two kinds of pages: public ones that anyone can view (Recipe and Situations & Solutions), and private ones marked with a lock icon that only the owner can see (the Cycle Tracker). Sign-in is for the owner only — it's used to view private pages and to add, edit, or delete content.

## How AI Helped

- **ChatGPT** — initial design direction and the first build prompt
- **Claude** — assisted with code, debugging, and deployment
- **GitHub** — hosting on GitHub Pages

The full documentation of how VetriHub was designed, prompted, and built:

- [Design Document](docs/Design-Document.md)
- [Prompt Document](docs/Prompt-Document.md)
- [Build & Deployment](docs/Build-and-Deployment.md)

### The process

VetriHub started from a design and prompt drafted with ChatGPT, then was built and refined hands-on with Claude, and deployed to GitHub Pages.

### What I learned

Building VetriHub showed how a fully client-side app can use Google Sign-In and Google Sheets as a lightweight backend, and how AI can help work through real deployment challenges — like the OAuth redirect — step by step.

---

## How it works (brief)

- A single-file PWA (HTML, CSS, and JavaScript) with no build step.
- Google Sign-In (OAuth) for the owner, with a restricted API key for public reads.
- Google Sheets store the data; the app creates its own files on first sign-in.
- Public pages are readable by anyone; the private Cycle Tracker is owner-only.
