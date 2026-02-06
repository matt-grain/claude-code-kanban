# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**claude-code-kanban** — A real-time Kanban dashboard for observing Claude Code tasks. Express + chokidar backend, vanilla JS single-file frontend. Zero build step.

## Commands

```bash
npm start          # Start server on port 3456
npm run dev        # Start server + auto-open browser
PORT=8080 npm start  # Custom port
```

No build, lint, or test commands — the app is a single HTML file served by Express.

## Architecture

```
server.js              Express server + chokidar file watchers + SSE
public/index.html      Entire frontend (CSS + JS + HTML, ~3100 lines)
```

### Data Flow

```
Claude Code writes ~/.claude/tasks/{sessionId}/{taskId}.json
    → chokidar detects change
    → server broadcasts SSE event
    → frontend fetches updated data via REST API
    → re-renders Kanban board (with JSON comparison to skip no-op renders)
```

### Server (server.js)

- **3 chokidar watchers**: tasks dir, teams dir, projects dir — filtered to `add/change/unlink` events only
- **SSE endpoint** (`/api/events`): broadcasts `update`, `team-update`, `metadata-update` events
- **REST API**: sessions list, session tasks, all tasks, delete task (with dependency check), add note
- **Caching**: session metadata (10s TTL), team config (5s TTL)
- **Port fallback**: if default port is in use, falls back to OS-assigned random port

### Frontend (public/index.html)

Single-file app with embedded CSS and JS. Key sections:
- **Sidebar**: session list with progress bars, project/session filters, live updates feed
- **Main area**: Kanban board (Pending → In Progress → Completed) with task detail panel
- **SSE handler**: debounced (500ms tasks, 2s metadata) with JSON hash comparison to avoid unnecessary re-renders

Key functions:
- `fetchSessions()` / `fetchTasks()` — data fetching with change detection
- `renderKanban()` / `renderSessions()` — DOM rendering
- `getOwnerColor()` — consistent hash-based agent colors
- `escapeHtml()` — XSS prevention (always use for user-provided strings)

### External Dependencies (CDN)

- `marked.js` — markdown rendering for task descriptions
- `DOMPurify` — sanitizes rendered markdown HTML
- Google Fonts — IBM Plex Mono, Playfair Display

## Key Conventions

- **Single-file frontend**: all CSS, JS, HTML live in `public/index.html`. No components, no framework.
- **Observation over control**: Claude Code owns task state. The viewer only reads and displays.
- **XSS safety**: always use `escapeHtml()` for any user/task data rendered to DOM. Markdown goes through `DOMPurify.sanitize(marked.parse(...))`.
- **Dark/light theme**: CSS variables for theming, stored in localStorage.
- **npm package**: published as `claude-code-kanban` with bin entry `claude-code-kanban`.
