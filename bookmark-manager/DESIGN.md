# Personal Bookmark Manager — Design Document

## 1. Overview

A single-user web app to save, tag, and search bookmarks. No accounts, no login. Runs locally on my machine.

---

## 2. User Stories / Features

| # | Feature | Description |
|---|---------|-------------|
| 1 | Add bookmark | Save a URL with a title and optional notes |
| 2 | Tagging | Add one or more tags per bookmark for organization |
| 3 | Search | Free-text search across title, URL, notes, and tags |
| 4 | Filter by tag | Click a tag to see only bookmarks with that tag |
| 5 | Edit bookmark | Update the title, URL, notes, or tags |
| 6 | Delete bookmark | Remove a bookmark |
| 7 | View all | Default page shows all bookmarks newest-first |
| 8 | Copy URL | One-click copy a bookmark's URL to clipboard |

Out of scope (for now): browser extension, import/export, sharing, mobile app.

---

## 3. Pages / UI

A single-page app with three main sections:

### 3.1 Header
- App title: "Bookmarks"
- Search bar (free text, filters as you type)

### 3.2 Add Bookmark Form
- Fields: URL (required), Title (required), Notes (optional), Tags (comma-separated)
- "Save" button
- Collapsible / always-visible at the top

### 3.3 Bookmark List
- Each bookmark shows:
  - Title (clickable link, opens in new tab)
  - URL (shortened display)
  - Notes (truncated, expandable)
  - Tags as clickable pills/badges
  - Copy URL icon, Edit icon, Delete icon
- Sorted newest-first
- When a tag pill is clicked, the list filters to that tag (tag shown as active)

### 3.4 Edit Modal
- Pre-filled form (same fields as add)
- "Save Changes" and "Cancel" buttons

---

## 4. Wireframe (text)

```
┌──────────────────────────────────────────────────┐
│  📑 Bookmarks                    [🔍 Search...]  │
├──────────────────────────────────────────────────┤
│  URL:      [_________________________]           │
│  Title:    [_________________________]           │
│  Notes:    [_________________________]           │
│  Tags:     [tag1, tag2, ...]                     │
│  [Save Bookmark]                                 │
├──────────────────────────────────────────────────┤
│                                                  │
│  How to center a div  ── copy  ✏️  🗑️           │
│  https://example.com/center-a-div                │
│  Really useful for layout stuff                  │
│  [css] [layout] [beginner]                       │
│  ────────────────── 2 hours ago                  │
│                                                  │
│  Awesome React tutorial  ── copy  ✏️  🗑️       │
│  https://react-tutorial.dev                      │
│  Follow along with examples                      │
│  [react] [tutorial]                              │
│  ────────────────── yesterday                    │
│                                                  │
└──────────────────────────────────────────────────┘
```

---

## 5. Technology Stack

| Layer | Choice | Why |
|-------|--------|-----|
| **Persistence** | Browser `localStorage` | No backend, no server, no database. Data lives in the browser. |
| **Frontend** | Plain HTML + CSS + vanilla JS | One file, zero dependencies. Open in any browser. |
| **ID generation** | `crypto.randomUUID()` or `Date.now() + counter` | Simple unique IDs without a database. |

### Why this works

- **Single HTML file** — double-click `index.html` and it runs. No install, no build, no server.
- **localStorage** persists across page refreshes and browser restarts. Data is a JSON string under a single key (`bookmarks`).
- **Zero dependencies** — no npm install, no pip install, no framework to learn.
- **Portable** — put it on a USB stick, open it on any machine.

### Trade-off

localStorage is per-browser. Bookmarks won't sync between Chrome and Firefox on the same machine. For a first project that's fine — you can always add sync later.

---

## 6. Data Shape

A JSON array stored in `localStorage` under the key `"bookmarks"`:

```js
[
  {
    id: "a1b2c3",           // crypto.randomUUID()
    url: "https://...",
    title: "How to center a div",
    notes: "Really useful",
    tags: ["css", "layout"],
    created_at: "2026-05-09T10:30:00.000Z"
  }
]
```

Search is a JavaScript filter across the array:

```js
const q = query.toLowerCase();
results = bookmarks.filter(b =>
  b.title.toLowerCase().includes(q) ||
  b.url.toLowerCase().includes(q) ||
  b.notes.toLowerCase().includes(q) ||
  b.tags.some(t => t.toLowerCase().includes(q))
);
```

---

## 7. App Logic (no routes — all client-side)

| Action | Where | What it does |
|--------|-------|--------------|
| Load | `load()` | Read and parse from `localStorage`; returns array |
| Add | inline in `form` submit handler | Unshift new object onto `allBookmarks`, call `save()`, re-render |
| Edit | `startEdit(bookmark)` / `saveEdit(id)` | `startEdit` injects an inline edit form; `saveEdit` validates, updates `allBookmarks[idx]`, calls `save()`, re-renders |
| Delete | inline in delete button handler | Filters `allBookmarks` by id, calls `save()`, re-renders |
| Search/Filter | `render()` | Reads module-level `searchQuery` and `currentFilter`; no parameter |

`allBookmarks` is a module-level array initialised once from `load()` on startup. All mutations update it in memory and persist via `save()`.

---

## 8. File Structure

```
bookmarker/
└── index.html       # Everything: HTML + CSS + JS in one file
```

That's it. One file you can open in a browser.

---

## 9. Visual Design

- **Font**: System font stack (`-apple-system, BlinkMacSystemFont, "Segoe UI", ...`)
- **Colors**: Dark theme — `#121214` page background, `#1a1a1e` card/surface background, `#e1e1e3` body text, `#1e4ac8` accent blue for buttons and active states, `#93b4f8` link colour
- **Layout**: Centered single column, max-width 640px
- **Tags**: Rounded pills (`#1e1e24` background, `#aaaaaa` text, `#2a2a2e` border)
- **No CSS framework** — plain CSS using logical properties (`margin-block-*`, `padding-inline-*`, etc.)
- **Responsive**: Works on desktop and mobile via `max-width` + flexible inputs

---

## 10. Future Ideas (v2)

- Browser bookmarklet to quickly save from any page
- Import from browser bookmarks HTML export
- Pagination (if bookmarks exceed ~100)
- Backup/restore database file
