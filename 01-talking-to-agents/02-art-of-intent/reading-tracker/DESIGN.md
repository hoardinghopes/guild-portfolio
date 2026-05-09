# Reading Tracker — Design Document

## Purpose
A personal tool for tracking reading
Single user, no accounts needed. Runs in a web browser, data stays on the device.

## Features
1. Add a habit with a name and an optional daily target (e.g. "Read — 30 minutes")
2. Mark a habit as done for today with one click
3. Show a 7-day streak view: did I do this every day this week?
4. Show a monthly calendar heat map of completion history
5. Edit a habit's name or delete it

## What It Should Do
1. Add a book with title, author, publisher, topic (tags), short review, 5 star rating, currently-reading checkbox, finished-reading checkbox. Only title and author required.
2. Display the list of books in reverse order that they were read.
3. Display by authors, and list their books
4. Search form for when the lists get too long for a single page
5. Display a 'currently reading' banner

## Technology
- HTML, CSS, JavaScript (no frameworks)
- Data stored in the browser's local storage as JSON
- Single HTML file, no build step, no server

## Interface
- Clean, minimal, mobile-friendly
- Add-book form collapses to a "+" button when not in use

## Constraints
- Must work entirely offline after first load
- Must persist data across browser refreshes
- Must be readable on a phone screen (360px wide)

## Out of Scope
- User accounts, sharing, or sync across devices
- Notifications or reminders
- Import/export (could add later)


## Success Criteria (how I know it's done)
- I can add, mark, edit, and delete books without opening devtools
- My data survives a full browser restart
- I used it for three days straight and it didn't frustrate me