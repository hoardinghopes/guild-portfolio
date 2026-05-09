# Bookmark manager

[1st Apprentice project from the Navigators Guild](https://github.com/Navigators-Guild/apprentice-onboarding/blob/main/01-talking-to-agents/05-first-build.md)

This is a bookmark manager as a case study in communicating design requirements to Claude.

I want to create a single-page manager that lists bookmarks. I can query and filter them by search term and tag. The storage is browser localstorage.



## Core

    "Here's my design document for a bookmark manager. Let's start with just the core: I want to be able to add a bookmark with a URL and title, see it appear in a list, and have it persist when I refresh the page. Use browser local storage for saving data. Plain HTML, CSS, and JavaScript in a single file called index.html. Make the design clean and minimal."

## Layer 2: Notes and tags

    "The core bookmark list is working. Now I want to add two fields to the add form: an optional note (a text area for a brief description) and optional tags (comma-separated text that gets stored as a list). Display the note under each bookmark's title, and display tags as small labeled badges."

## Layer 3: Edit and delete

    "Now add the ability to edit and delete bookmarks. Each bookmark should have a small edit icon and a delete icon. Clicking edit should let me change the title, URL, note, and tags inline. Clicking delete should remove the bookmark after a confirmation. Make sure changes persist in local storage."

## Layer 4: Tag filtering

    "Now add tag filtering. Above the bookmark list, display all unique tags as clickable buttons. When I click a tag, the list should filter to only show bookmarks with that tag. There should be an 'All' button that removes the filter. The currently active filter should be visually highlighted."

## Layer 5: Search

    "Add a search bar next to the tag filters. It should filter the bookmark list in real time as I type, matching against bookmark titles and notes. Search and tag filters should work together. If I have a tag filter active and then search, it should search within the filtered results."


## Layer 6: Polish

    "The functionality is complete. Now let's polish the interface. I want: better spacing and typography, a subtle color scheme (dark background with light text, dark mode), smooth transitions when filtering and searching, the add form should collapse to a button when not in use to keep the interface clean, and bookmarks should show the domain name extracted from the URL as a subtle label."

All the steps worked shockingly well – I did not need to add extra comments at any point. However, if you take a look at the project's `design.md`, you'll see a level of clarity there (it was written by Claude) that I did not offer, which makes me wonder where that came from.

I'm inclined to run the project again, in a completely different directory on my computer, using a different AI to see how that fares.