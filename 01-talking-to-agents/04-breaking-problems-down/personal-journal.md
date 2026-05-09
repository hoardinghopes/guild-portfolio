# A personal journal app where I can write daily entries, tag them with moods, and look back at past entries.

## Decomposing tasks

### Core
- I open the app and see a list of previous entries. I can add a new entry.

### Layers
- core: display entries; add new one
- layer: add mood tag, and filter by that
- layer: add mood search, with realtime filtering
- layer: add word search bar, with realtime filtering
- layer: edit entry
- layer: add image
- layer: layout, typography, etc.

### User story approach
- I open the app and see the list of previous entries in reverse chronological order
- I click "+" to open the new entry form, enter my details, and save. The form closes and the entry appears at the top of the list
- I click to add a mood - a list of moods is made available, with the option of creating/adding a new mood.
- I search by mood, and the list repopulates in realtime with articles of that mood
- I search by word, and the list repopulates in realtime with articles that contain that term
- I click to view the previous month's entries
- I can delete an entry.
- I can edit an entry.

