# Process Log — Personal Bookmark Manager

## Overview

A record of how this project was built: the decisions made, problems hit, and things learned along the way.

---

## 1. Design phase

Before writing any code I asked Opencode to author a design document to lock down scope and avoid scope creep mid-build.

**First instinct: server + database.** Opencode's initial design called for a Python/Flask backend with a SQLite database — three tables, five routes, six files. 

**Decision: localStorage only.** This is a personal tool running on one machine with no user accounts. Creating a backend adds setup cost (install Python, run a process, keep it running). Browser `localStorage` gives persistence for free — the data survives refreshes and restarts, lives as a JSON array under a single key, and requires zero infrastructure. The trade-off is that data is per-browser, but that's acceptable for a solo tool.

This cut the project from six files to one.

---

## 2. Build: core (add + list + persist)

Started with the minimum useful thing: save a URL and title, see it in a list, have it survive a page refresh.

The data model is a flat JSON array stored in localStorage. Each bookmark is an object with an `id`, `url`, `title`, and later `notes`, `tags`, and `created_at`. The array is read on every render and written on every mutation — simple, no sync issues.

`crypto.randomUUID()` was used for IDs. Simple, no dependency.

---

## 3. Build: notes and tags

Added an optional `<textarea>` for notes and a comma-separated text input for tags, stored as a string array. The tags input trades a proper tag-chip UI for simplicity — easier to build, easy to edit.

One thing to keep in mind: the data schema changed mid-project. Bookmarks saved before the notes/tags fields existed have no `notes` or `tags` property. This came back to bite later (see bug #2 below).

---

## 4. Build: edit and delete

**Delete** was straightforward: `confirm()` dialog, filter the bookmark out by ID, save, re-render.

**Edit** — Opencode chose nline editing over a modal. When the edit icon is clicked, the bookmark card's HTML is replaced in-place with a pre-filled form. On save, the updated values are written back; on cancel, a re-render restores the original card. This avoids the complexity of a separate modal layer and feels natural for a list-based UI.

---

## 5. Build: tag filtering

The filter bar renders all unique tags extracted from the full bookmark set. Tags in the bar and tag badges on each card are both clickable — either way sets the active filter and re-renders. Clicking the active tag a second time clears the filter (toggle behaviour), which felt more natural than requiring a separate "All" click.

---

## 6. Build: search

Search is a client-side filter over the in-memory array: `includes()` across title, URL, notes, and tags. It chains with the active tag filter — search always operates on whatever the tag filter has already narrowed down.

**Bug #1 — search not firing.** After wiring up search, it wasn't filtering when no tag filter was active. Root cause: the search input's event listener had been accidentally nested inside the tag filter click handler during an earlier edit, instead of being attached at the top level. It was only registering after a tag was clicked, not on page load. Fix: move the listener to module scope, outside `render()`.

**Bug #2 — TypeError on `b.notes`.**  Typing a search term threw `Uncaught TypeError: can't access property "toLowerCase", b.notes is undefined`. Bookmarks created before the notes field was added have no `notes` property — `b.notes` is `undefined`, not an empty string. Fixed by guarding: `(b.notes || '').toLowerCase()`.

This was a good reminder that `localStorage` data can be older than the current schema and needs defensive reads.

---

## 7. Polish: dark theme, collapsible form, domain label, animations

**Dark theme.** Switched from a light design to a dark one (`#121214` background, `#e1e1e3` text, `#3b82f6` accent blue). Dark feels more natural for a utility app I'll use all day.

**Collapsible form.** The add form collapses to a "+ Add bookmark" button when not in use. The list takes up the full screen by default, which is the primary use case (browsing saved bookmarks, not adding new ones).

**Domain label.** Extracted via `new URL(url).hostname` and shown below the title as a subtle hint. The `new URL()` constructor does the heavy lifting and handles edge cases better than a regex would.

**Fade-in animation.** Opencode initially tried CSS `transition: opacity` on bookmark cards, but transitions only work on state changes — they do nothing when elements are replaced wholesale via `innerHTML`. The fix was a `@keyframes fadeIn` animation applied to `.bookmark` directly, so every time the list re-renders each card runs the animation from scratch.

---

## 8. Review and hardening

After the initial build I asked Claude to do a full code review and work through the issues systematically.

**Security**

- `escapeAttr` was replacing `"` before `&`, which double-encoded the `&` it just introduced. Replacement order must be `&` first, then `"`.
- `javascript:` URIs pass HTML5 URL validation but are dangerous in `href` attributes. Added an `isSafeUrl()` guard that rejects anything other than `http:` and `https:` at both the add and edit paths.

**Data integrity**

- `created_at` was never being saved. Added `new Date().toISOString()` on add.
- The `saveEdit` function was writing the updated object into the array *before* running validation. If validation failed it returned early, leaving the in-memory array dirty (though re-loading from localStorage on next render masked this). Fixed by validating first, then writing.

**Missing features**

- Search was only matching title and notes — the spec called for URL and tags too. Fixed.
- The copy URL button was listed in the design document but never implemented. Added it with `navigator.clipboard.writeText()` and a brief ✓ feedback indicator.
- `localStorage.setItem()` throws `QuotaExceededError` when storage is full. Added a try/catch to surface this as a readable message rather than a silent failure.

**UX bugs**

- After saving a new bookmark, the active search query wasn't being cleared, so the new bookmark could be invisible in the filtered list.
- Tag case wasn't normalised — `"CSS"` and `"css"` produced separate filter pills. Tags are now lowercased on save.

**Accessibility**

- The form toggle button needed `aria-expanded` (updated on each click) and `aria-controls`.
- Action buttons (copy/edit/delete) needed `aria-label` — the icon glyphs are invisible to screen readers.
- The buttons were `opacity: 0` until hover, making them invisible when keyboard-focused. Added `:focus-within` so they appear for keyboard users.
- Tag filter pills needed `aria-pressed` to communicate which filter is active.
- The inline edit form needed focus management — the URL field now receives focus immediately after the form is injected, so keyboard users aren't stranded on the (now-removed) edit button.
- Replaced the native `confirm()` delete dialog with a custom `<dialog>` element. The native dialog is inaccessible in many AT configurations. The `<dialog>` with `showModal()` provides focus trapping, a backdrop, and proper `role="dialog"` semantics for free. Focus is returned to the triggering button on close.

**CSS**

- The `.form-card` rule was declared twice. The second declaration (`display: none`) overrode the first block's visual styles. Merged into one block.
- Converted all directional physical margin/padding properties (`margin-top`, `padding-bottom`, etc.) to logical equivalents (`margin-block-start`, `padding-inline`, etc.) for consistency.

---

## 9. What I'd do differently

- **Schema versioning from day one.** The `b.notes` TypeError happened because I added a field mid-project with no migration. Even for localStorage data, a version field and a migration step at startup would prevent this class of bug.
- **Validate earlier.** I wrote the data path before thinking about what invalid data looks like. The `javascript:` URI issue and the unguarded `b.notes` both stem from not thinking about bad input upfront.
- **Commit more granularly.** The initial build was one large file before I started using git. Smaller, more frequent commits would make it easier to bisect bugs.
