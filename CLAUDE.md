# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A single-file static website with a multi-tab navigation system. Everything — HTML structure, CSS styles, and JavaScript logic — lives in [index.html](index.html). There is no build step, no framework, no dependencies, and no server.

Current tabs:
- **Home**: Landing page with welcome message
- **To Do**: Task management with Chart.js doughnut progress chart, filters, and localStorage persistence
- **Projects**: Card-based project tracker (ongoing/planned/completed) with image upload support
- **Calculator**: Standard 4-function calculator with equation history display
- **IMG Convert**: Client-side image format converter using Canvas API (JPG, PNG, WebP, etc.)
- **Screenshots**: Gallery with lightbox, images referenced by filename from `screenshots/` folder
- **Clips**: Video gallery, files referenced by filename from `videos/` folder

To view the app, open `index.html` directly in a browser. No build step needed.

## Architecture

### Page Structure
- Top navigation bar (`<nav>`) with brand logo (`images/jakes_website.png`) and tab buttons
- Pages: `#page-home`, `#page-todo`, `#page-projects`, `#page-calculator`, `#page-imgconvert`, `#page-screenshots`, `#page-clips`
- `showPage(id, btn)` handles tab routing — removes `active` from all pages/buttons, adds to target
- "⬆⬇ Data" nav button opens export/import modal for full backup to `backup.json`

### localStorage Keys
All state lives in `localStorage`. Keys and shapes:
- `tasks` — `{ id: number, text: string, done: boolean }[]`
- `projects` — `{ id: number, name, description, status: 'ongoing'|'planned'|'completed', image: string }[]`
- `screenshots` — `{ id: number, filename: string, label: string }[]`
- `clips` — `{ id: number, filename: string, label: string }[]`

IDs are always `Date.now()` timestamps.

### Rendering Model
Each feature follows the same pattern: mutation → `save*()`  → `render*()`. Render functions rebuild DOM from scratch each call. User text XSS-sanitized via `escapeHtml()` before any `innerHTML` injection.

### External Dependencies
- `chart.js` loaded from CDN (`cdn.jsdelivr.net`) — used only for the To Do doughnut chart (`todoChart`). No other dependencies.

### Media Files
Screenshots and clips are referenced by filename only. Actual files must be placed in `screenshots/` or `videos/` folders manually — the app does not handle file uploads for these.

### Adding New Tabs
1. Add `<button onclick="showPage('tabid', this)">Tab Name</button>` to nav
2. Create `<div class="page" id="page-tabid">` with page content
3. No other changes needed — `showPage()` handles routing
