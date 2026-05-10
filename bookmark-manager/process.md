# Process Log — Personal Bookmark Manager

## Overview

A record of how this project was built: the decisions made, problems hit, and things learned along the way.

## AI agents

I have 2 AI overlords: 
- Claude Code (in VSCodium) using Sonnet 4.6 
- Big Pickle Opencode Zen.

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

---

## 10. Adversarial review & fixes

After the first review fixes (from Claude) were implemented, I asked for an adversarial review from Opencode. Here's what was fixed and why.

### Critical bugs

- **`crypto.randomUUID()` crashes on Chrome via `file://`** — `file://` is not a secure context in Chrome, so `crypto.randomUUID()` throws. Added `generateId()` that tries `crypto.randomUUID()` first and falls back to `Date.now()-counter` (`index.html:383-386`).

- **Empty-state message lied** — "No bookmarks yet" was shown even when bookmarks existed but none matched the active search or tag filter. Changed to three distinct messages: "No bookmarks yet" (truly empty), "No bookmarks with this tag" (filter active), "No bookmarks match your search" (search active), and a combined variant (`index.html:508-521`).

- **Notes could not be expanded** — The spec said "(truncated, expandable)" but there was no expand mechanism. Added a "Show more" / "Show less" toggle that appears only when notes actually overflow 3 lines (`index.html:199-212`, `index.html:576-585`).

- **Dialog focus targeted a deleted DOM node** — `confirmDelete` returned focus to the delete button, then `render()` destroyed it. On confirm, focus now goes to the next bookmark's delete button or the form toggle. On cancel, focus returns to the original trigger (`index.html:426-433`).

- **`created_at` stored but never displayed** — Timestamps were in the data model and wireframe but absent from the UI. Added a `timeAgo()` function showing relative time ("2h ago", "yesterday") and a date fallback for older entries. Missing `created_at` on old bookmarks is handled gracefully (`index.html:188-192`, `index.html:436-484`, `index.html:541`).

### Security

- **`isSafeUrl` accepted credentials in URLs** — `https://user:pass@evil.com` passed validation, leaking credentials into the DOM. Now checks `parsed.username` and `parsed.password` and rejects them (`index.html:605`).

- **No Content Security Policy** — Added a CSP meta tag (`index.html:6`). It blocks external script and style sources, external connections (`connect-src 'none'`), form hijacking (`form-action 'none'`), and base tag injection (`base-uri 'none'`). However, because this is a single-file app with inline `<script>` and `<style>` blocks, both directives require `'unsafe-inline'`, which means the CSP does **not** block injected inline scripts. It is not an XSS defence. The real XSS protection here is `escapeHtml`/`escapeAttr` applied to all user-generated content before it touches the DOM. A hash-based CSP (`'sha256-...'`) would replace `'unsafe-inline'` and actually block injected scripts, but requires recomputing the hash after every edit to the script or style block — too much friction for a single-file project without a build step.

### Accessibility

- **Search input had no label** — Added a visually-hidden `<label for="search">` with `.sr-only` utility class (`index.html:300-310`, `index.html:370`).

- **No `aria-live` region** — List updates were silent for screen readers. Added `role="status"` on the list and a separate live region updated with result summaries (`index.html:373-374`, `index.html:579-584`).

- **Animation ignored `prefers-reduced-motion`** — Disabled the fadeIn animation for users who prefer reduced motion (`index.html:169-171`).

- **Filter-bar buttons had no grouping role** — Added `role="group" aria-label="Filter by tag"` (`index.html:372`).

### UX

- **Full URL never shown** — Only the domain was displayed. Added the full URL in a truncated, muted line below the domain (`index.html:188-193`, `index.html:533`).

- **Silent failure on empty edit fields** — `saveEdit` returned silently when URL or title was missing. Now shows an alert (`index.html:643`).

- **Clipboard failure was silent** — Added a `.catch()` that sets the button title to "Copy failed" (`index.html:562`).

- **Inconsistent filter/search clearing after form submit** — `searchQuery` was cleared but `currentFilter` was not, leaving new bookmarks invisible. Both are now cleared (`index.html:700-701`).

### Code quality

- **`load()` called N times per render cycle** — The delete, copy, and edit handlers each re-read localStorage. Changed all to reference the `allBookmarks` captured from the single `load()` call in `render()`. `startEdit` now accepts a bookmark object instead of calling `load()`. However, `saveEdit` was missed in this pass and continued to call `load()` independently — this was later corrected (see section 11).

- **Redundant `.form-body` toggle** — Removed the `<div class="form-body">` wrapper, its CSS, and all JS references. Visibility is now controlled solely by `.form-card.open` (`index.html:67-68`, `index.html:368-387`, `index.html:671-676`, `index.html:696-697`).

- **No `type="button"` on icon buttons** — Copy, edit, and delete buttons now explicitly set `type="button"` to prevent accidental form submission (`index.html:543-545`).

- **`class="delete"` fragile naming** — Renamed to `delete-btn` in both CSS and JS selectors (`index.html:276`, `index.html:433`, `index.html:545`, `index.html:549`).

### Edge cases

- **Duplicate tags stored** — `"css, css"` produced `["css", "css"]`, skewing badge display. Wrapped both tag inputs in `[...new Set(...)]` to deduplicate (`index.html:663`, `index.html:691`).

- **Data corruption silently reset** — Corrupted localStorage returned `[]` with no notification. Now backs up the raw string to a second key, alerts the user, and logs details to console (`index.html:404-405`, `index.html:441-455`).

---

## 11. Second adversarial review & fixes

A second adversarial review was run after section 10's fixes were applied. Key corrections:

- **`saveEdit` still called `load()` independently** — The section 10 fix to reduce `load()` calls was only partially applied: `startEdit` was updated, but `saveEdit` continued to call `load()` on its own. This was corrected as part of a broader refactor (see below).

- **Module-level `allBookmarks` cache** — `render()` was calling `load()` (a synchronous localStorage read and JSON parse) on every invocation, including every search keystroke. `allBookmarks` was promoted to module scope, initialised once at startup via `load()`, and updated in-place by every mutation (add, edit, delete). `render()` now reads from the in-memory array with no storage access. This also completed the fix for `saveEdit`, which now mutates `allBookmarks` directly instead of calling `load()`.

- **Search debounce** — The search input called `render()` on every keystroke with no throttling. Added a 180ms debounce so rapid typing batches into a single render.

- **Corrupted backup silently overwritten** — A fixed `BACKUP_KEY` constant meant repeated corruption events overwrote the previous backup with no warning. Changed to a timestamped key (`bookmarks_corrupt_backup_${Date.now()}`) so each event is preserved separately.

- **`saveEdit` validated after writing** — Validation ran after the updated object was already written into the array. If validation failed and returned early, the in-memory array was left dirty. Fixed by validating first, then writing.

---

## 12. Third review & fixes

A third adversarial review identified issues that had been missed in the earlier passes.

### UX and accessibility

- **Delete dialog didn't name the bookmark** — The confirmation message was the static string "Delete this bookmark?". A keyboard user who tabbed into the dialog had no way to know which bookmark they were about to delete. `confirmDelete` now accepts a `title` argument and sets the dialog text to `Delete "${title}"?`, falling back to `"this bookmark"` if the ID can't be resolved. The message is reset to the neutral fallback in `cleanup()` so it's never stale.

- **Delete confirm button used primary blue styling** — The "Delete" button in the confirmation dialog shared the same `btn-primary` blue style as "Save Bookmark". Destructive actions should be visually distinct. Added a `btn-danger` class with a dark red background (`#991b1b`, ~8.3:1 contrast with white — WCAG AAA) and a darker hover state (`#7f1d1d`, ~10:1).

- **No landmark structure** — The page had no ARIA landmark elements. Three changes were made: (1) the outer `<div class="container">` was changed to `<main class="container">` so screen reader users can jump directly to content; (2) the add-bookmark `<form>` was given `aria-label="Add bookmark"`, promoting it from a generic container to a named form landmark; (3) the search input and filter bar were wrapped in a `<search>` element (HTML 5.3, broadly supported since late 2023), giving the filtering area its own landmark. Without these, every screen reader user had to tab through the entire page from the top on every visit.

### Page title

- **`document.title` never reflected active state** — The browser tab always read "Bookmarks" regardless of any active filter or search. `render()` now updates `document.title` on every call, before the early-return path, so it is always accurate: `Bookmarks — css` when a tag filter is active, `Bookmarks — "flexbox"` when searching, `Bookmarks — css · "flexbox"` when both are active, and plain `Bookmarks` when neither is.

### Data integrity

- **Per-tag length not enforced** — The tags field had `maxlength="200"` on the input, but a single 200-character string with no commas was stored as one valid tag. The `maxlength` prevented storage overflow but did nothing to validate the shape of individual tags. Extracted tag parsing into a `parseTags()` helper used by both the add and edit paths. It splits on commas, trims, lowercases, deduplicates (via `Set`), and rejects any individual tag longer than 50 characters, alerting the user if any were dropped rather than silently discarding them.

- **No cross-tab sync** — Two tabs open simultaneously would diverge: a deletion or addition in one tab was invisible in the other. A `storage` event listener was added at startup. When another tab writes to `STORAGE_KEY`, the listener reloads `allBookmarks` from storage and calls `render()`. The `storage` event only fires in tabs that did *not* make the change, so this adds no overhead to the current tab's own mutations.
