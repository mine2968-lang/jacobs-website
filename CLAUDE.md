# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A single-file static website with a multi-tab navigation system. Everything — HTML structure, CSS styles, and JavaScript logic — lives in [index.html](index.html). There is no build step, no framework, no dependencies, and no server.

Current tabs:
- **Home**: Landing page with welcome message
- **To Do**: Task management app with filtering

To view the app, open `index.html` directly in a browser.

## Architecture

### Page Structure
- Top navigation bar (`<nav>`) with brand name and tab buttons
- Two pages: `#page-home` and `#page-todo`
- `showPage(id, btn)` function handles tab routing — switches active page and nav button

### To Do App Data & Rendering
Task state stored in `localStorage` under key `tasks` as JSON array: `{ id: number, text: string, done: boolean }`. IDs are `Date.now()` timestamps.

Rendering model: `save()` persists to localStorage → `render()` rebuilds task list DOM from scratch based on `tasks` array and active `filter` (`'all'` | `'active'` | `'done'`).

User input XSS-sanitized via `escapeHtml()` before innerHTML injection.

### Adding New Tabs
1. Add `<button onclick="showPage('tabid', this)">Tab Name</button>` to nav
2. Create `<div class="page" id="page-tabid">` with page content
3. No other changes needed — `showPage()` handles routing
