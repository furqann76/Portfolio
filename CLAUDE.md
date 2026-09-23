# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This repository is a personal portfolio site for Furqan, a Computer Science student / Python-Django developer. It is a **single static HTML file** (`index.html`) — no build step, no package manager, no framework, no dependencies. Everything (HTML, CSS, JavaScript) lives inline in this one file.

## Development

There is no build/lint/test tooling in this repo. To work on the site:

- Open `index.html` directly in a browser, or serve it locally, e.g. `python3 -m http.server` from the repo root and visit `http://localhost:8000`.
- There is no bundler, transpiler, package.json, or test suite — changes take effect on save/refresh.
- Deployment is simply committing/pushing `index.html` (recent commit history shows direct "Update index.html" commits to `main`).

## Architecture

`index.html` is organized into three inline blocks, each with its own numbered section comments that act as a table of contents — check these comments first to jump to the right area:

1. **`<style>` block** (~line 48-940): numbered sections 1-16 (Reset & base, Design tokens, Typography, Layout helpers, Buttons & tags, Header/nav, Hero, About, Skills, Projects, Experience, Education, Contact, Footer, Scroll-reveal & motion, Responsive). All colors/fonts/spacing are CSS custom properties defined once under `:root` in section 2 ("Design tokens") — change values there rather than hardcoding new ones in component rules.

2. **Body markup** (~line 942-1575): a single-page layout with `<header>` nav + `<main>` containing sections identified by id (`#home`, `#about`, `#skills`, `#projects`, `#experience`, `#education`, `#contact`), followed by a `<footer>`. Nav links, scroll-spy, and scroll-reveal all key off these section ids and the shared `.reveal` class, so keep new sections consistent with that pattern (an `id` on the `<section>`, and `.reveal` on elements that should fade in).

   A second, normally-hidden view (`#cvView`, near line 1434) holds a print-friendly CV/resume. It's toggled via a `cv-active` class on `<body>` (see the CSS around line 924 and the `@media print` rules around line 931) rather than being a separate page. It also embeds a downloadable `.docx` resume as a base64 `data:` URI directly in an `<a>` href (`.toolbar__docx`) — if the resume content changes, this base64 blob needs to be regenerated and re-embedded, not hand-edited.

3. **`<script>` block** (~line 1576-1879): vanilla JS, no dependencies. Every feature is registered through a `safeInit(label, fn)` wrapper that try/catches independently, so one feature failing (bad selector, missing element, etc.) cannot break the others — this was a deliberate fix for a past bug where an early failure left later content stuck invisible. Features, in order: sticky header shadow, mobile nav toggle, scroll-spy (IntersectionObserver), scroll-reveal animations (with a hard `setTimeout` fallback to force visibility), hero terminal typed-JSON effect, project filtering (via `data-category`/`data-filter` attributes), contact form (builds a `mailto:` link client-side — there is no backend/email service), back-to-top button, footer year, and the CV view toggle described above. When adding a new interactive feature, follow the same `safeInit('label', () => { ... })` pattern with a null-check early return.

Other notable details:
- Structured data (`application/ld+json`, near the top of `<head>`) mirrors the page's real content for SEO/AI-crawler purposes — keep it in sync with actual About/Skills content, and never invent employer/degree/contact details that aren't stated on the page.
- The favicon is an inline SVG data URI (no separate asset file).
- The contact form has no server-side handling; "sending" just opens the visitor's email client via `mailto:`.
- `CONTACT_EMAIL` (in the script block, section "CONFIG") is the single source of truth for the contact address used by the mailto link.
