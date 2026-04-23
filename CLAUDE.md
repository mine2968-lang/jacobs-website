# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A single-file static website with a multi-tab navigation system. Everything — HTML structure, CSS styles, and JavaScript logic — lives in [index.html](index.html). There is no build step, no framework, no dependencies, and no server.

To view the app, open `index.html` directly in a browser. No build step needed.

## Tabs

- **Home**: Landing page with welcome message
- **To Do**: Task management with Chart.js doughnut progress chart, filters, and localStorage persistence
- **Projects**: Card-based project tracker (ongoing/planned/completed) with image upload support
- **Calculator**: Standard 4-function calculator with equation history display
- **IMG Convert**: Client-side image format converter using Canvas API (JPG, PNG, WebP, etc.)
- **Screenshots**: Gallery with lightbox, images referenced by filename from `screenshots/` folder
- **Clips**: Video gallery with categories; files referenced by filename from `videos/` folder. Has two pages: `#page-clips` (category list) and `#page-clips-detail` (clip list within a category)
- **Sports**: Live scores, standings, roster, and schedule. Defaults to Yankees. Supports following multiple teams via ESPN public API. Has a roster view (`#sportsRoster`) and detail view (`#sportsDetail`) with a full schedule sub-view
- **Cards**: Trading card tracker
- **Anime**: Anime watchlist

## Architecture

### Page Structure
- Top navigation bar (`<nav>`) with brand logo (`images/jakes_website.png`), tab buttons, and a live scores ticker
- `showPage(id, btn)` handles tab routing — removes `active` from all pages/buttons, adds to target
- "⬆⬇ Data" nav button opens export/import modal for full backup to `backup.json`

### localStorage Keys
- `tasks` — `{ id: number, text: string, done: boolean }[]`
- `projects` — `{ id: number, name, description, status: 'ongoing'|'planned'|'completed', image: string }[]`
- `screenshots` — `{ id: number, filename: string, label: string }[]`
- `clipCategories` — category objects
- `clips` — `{ id: number, filename: string, label: string }[]`
- `followedTeams` — `{ id: string, sport: string, name: string, abbr: string, logo: string }[]`
- `animeWatchlist` — anime entries

IDs are always `Date.now()` timestamps.

### Rendering Model
Each feature follows the same pattern: mutation → `save*()` → `render*()`. Render functions rebuild DOM from scratch each call. User text is XSS-sanitized via `escapeHtml()` before any `innerHTML` injection.

**Exception — Sports tab**: uses content-diffing before setting `innerHTML` to avoid glitching during live auto-refresh. Check `body.dataset.lastHtml !== html` before assigning.

### Sports Tab Architecture
- `followedTeams` defaults to Yankees (`id: '10'`, `sport: 'baseball/mlb'`)
- `loadTeam(teamId, sport)` dispatches to team-specific loaders (e.g. `loadYankees()`)
- `loadYankees_inner(TEAM_ID)` fetches data from ESPN public APIs in parallel: team info, 4 weekly scoreboards (past 30 days), today's scoreboard, future scoreboard (next 14 days), then roster and season stats
- Recent games: `slice(-10).reverse()` from completed games pool
- Next game: taken from `futureSbData` (not `team.nextEvent`, which can return today's game)
- Live auto-refresh: 10s interval when game is live, 60s otherwise. Uses content-diff guard to skip DOM update when data unchanged
- Full schedule: fetched on demand via `showFullSchedule()`, splits season into ~9 weekly chunks in parallel
- Scores ticker: runs independently via `refreshTicker()`, also uses content-diff to avoid animation restarts

### Team Logo URLs
- Header logo: `https://a.espncdn.com/i/teamlogos/mlb/500-dark/scoreboard/${abbr}.png` (dark variant for navy header)
- `teamLogo(team)` helper: same ESPN dark URL for all in-game card logos
- `teamLogoUrl(t)` helper: used for followed-team grid cards (uses local `images/mlb_*.png` for MLB)

### External Dependencies
- `chart.js` loaded from CDN (`cdn.jsdelivr.net`) — only for the To Do doughnut chart (`todoChart`)
- ESPN public APIs — no auth required, used for all sports data

### Media Files
Screenshots and clips are referenced by filename only. Actual files must be placed in `screenshots/` or `videos/` folders manually.

### Adding New Tabs
1. Add `<button onclick="showPage('tabid', this)">Tab Name</button>` to nav
2. Create `<div class="page" id="page-tabid">` with page content
3. No other changes needed — `showPage()` handles routing
