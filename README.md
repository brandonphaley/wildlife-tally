# Wildlife Tally — The Field Ledger

A progressive web app for tracking wildlife sightings across national park trips. Multiple trips, two play modes (team or versus), customizable animal lists per trip, persistent storage. Installs to your home screen, works offline.

## What's in here

- `index.html` — the app (single-file SPA with home / new-trip / trip views)
- `manifest.json` — PWA install metadata
- `sw.js` — service worker for offline support
- `icon-192.png` / `icon-512.png` — home screen icons
- `CLAUDE.md` — project context for any Claude session working on this

## Live URL

**https://brandonphaley.github.io/wildlife-tally/** — GitHub Pages, served from the `main` branch root of `brandonphaley/wildlife-tally`.

### Deploying an update
1. Bump the cache version in `sw.js` (change `'wildlife-tally-v18'` to `v19`, etc.) so installed devices pull the update.
2. Commit and push to `main`. Pages rebuilds automatically in about a minute.

## Install on phone

**Android (Chrome):** Open URL → tap "Install App" button at the bottom, or Chrome menu → "Install app" / "Add to Home Screen."

**iPhone (Safari only):** Open URL → Share → Add to Home Screen.

## How it works

- **Home** lists all journals. Tap a card to open it.
- **+ Begin New Journal** opens trip setup: park, mode (Team or Versus), dates, 1–6 players, animal list edit, then start.
- **Team mode**: shared scorecard, goal, animals × dates grid with adult and baby buttons.
- **Versus mode**: leaderboard at top, per-player cards with day-by-day tallies for each player's animal team.
- Auto-saves to localStorage on every tap. Counts survive phone restarts.

## Adding a new park

Open `index.html`, find the `PARKS` registry near the top of the `<script>` section, and add an entry:

```js
'great-smokies': {
  id: 'great-smokies',
  name: 'Great Smoky Mountains',
  region: 'Tennessee / North Carolina',
  defaultGoal: 35,
  animals: [
    { id: 'blackbear',  name: 'Black Bear',  subtitle: '',                     points: 4 },
    { id: 'elk',        name: 'Elk',         subtitle: 'Cataloochee Valley',   points: 3 },
    { id: 'salamander', name: 'Salamander',  subtitle: 'Park has 30+ species', points: 2 }
  ]
}
```

New park automatically appears in the park dropdown. Trips snapshot the animal list at creation, so editing a preset later won't affect existing journals.

## Data model

One `localStorage` key: `wildlife-tally-app-v1`.

```
{
  trips: {
    "trip-abc123": {
      id, journalNum, park, parkId,
      mode: "team" | "versus",
      days: [{ date: "2026-05-28" }, ...],
      players: [{ id, name, teamAnimalId }, ...],
      animals: [{ id, name, subtitle, points }, ...],
      goal,
      teamTallies,
      playerTallies,
      created, lastModified
    }
  },
  nextJournalNum: 4
}
```
