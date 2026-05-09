# Bookmark Manager — Design Document

## What It Does

A personal bookmark manager that runs locally in a browser. No accounts, no server, no login — just you and your links.

### Core Features

**Saving bookmarks**
- Add a URL, a title, and optional notes
- Tag each bookmark with one or more labels (e.g. "rust", "recipes", "read-later")
- Date is recorded automatically when you save

**Browsing bookmarks**
- See all bookmarks in a list, newest first
- Click any bookmark to open the link in a new tab

**Searching and filtering**
- Search by title, URL, or notes text
- Filter by tag — click a tag to see all bookmarks with that tag
- Combine search and tag filter together

**Editing and deleting**
- Edit the title, URL, notes, or tags on any saved bookmark
- Delete bookmarks you no longer want

That's it. No import/export, no folders, no sharing — those can come later if you want them.

---

## What It Looks Like

A single-page app with three zones:

```
┌─────────────────────────────────────────────┐
│  HEADER: search bar + "Add Bookmark" button │
├──────────────┬──────────────────────────────┤
│              │                              │
│  SIDEBAR     │  BOOKMARK LIST               │
│  tag list    │  card per bookmark           │
│              │                              │
└──────────────┴──────────────────────────────┘
```

**Header** — always visible at the top. Contains the search input and the button to add a new bookmark.

**Sidebar** — a list of all your tags. Clicking one filters the list. An "All" option clears the filter.

**Bookmark list** — the main content area. Each bookmark is a card showing:
- Title (as a clickable link)
- URL (truncated if long)
- Tags as small pills
- Notes (if any), collapsed by default
- Edit and Delete buttons

**Add/Edit form** — appears as a modal (a popup overlay) when you click "Add Bookmark" or "Edit". Contains fields for URL, title, notes, and tags. Tags can be typed as comma-separated words.

### Visual style

Clean and minimal. No decorative imagery. Dark text on a white or very light grey background. Tags use a soft accent colour (e.g. a muted blue or green). The goal is something that looks tidy and stays out of your way.

---

## Technology

### The short answer

Plain HTML, CSS, and JavaScript. No frameworks, no build tools, no install step. Open the `index.html` file in a browser and it works.

### Why this stack

This is your first project. The less infrastructure between you and the running app, the faster you'll learn and the less that can go wrong. Frameworks like React are useful later, but they add concepts that would distract from the fundamentals right now.

### Data storage

Your bookmarks are saved in the browser using **localStorage** — a built-in browser feature that keeps data on your machine between sessions. No database, no server needed.

The tradeoff: data lives in that specific browser on that specific machine. If you clear your browser data, you lose your bookmarks. That's fine for a personal tool you're learning with.

### File structure

```
05-first-build/
├── index.html      ← the whole app lives here
├── style.css       ← all the visual styles
└── app.js          ← all the logic
```

Three files. That's the entire project.

### What each file does

**index.html** — defines the structure of the page: the header, sidebar, list area, and modal. Links to `style.css` and `app.js`.

**style.css** — controls layout, colours, typography, and how things look on small screens. We'll use CSS custom properties (variables) for colours so they're easy to change.

**app.js** — handles everything the app *does*: saving to localStorage, reading from it, rendering the list, filtering by tag, searching, and wiring up button clicks.

---

## What We're Not Building (Yet)

- User accounts or login
- Syncing across devices
- Browser extension to save links with one click
- Import from other bookmark managers
- Keyboard shortcuts
- Dark mode

These are all possible later additions. Leaving them out keeps the first version simple enough to actually finish.

---

## Definition of Done

The app is complete when you can:

1. Open `index.html` in a browser with no extra setup
2. Add a bookmark with a URL, title, notes, and tags
3. See it appear in the list immediately
4. Search for it by title
5. Filter to it by clicking its tag
6. Edit it and see the changes saved
7. Delete it
8. Close the browser, reopen it, and find the bookmark still there
