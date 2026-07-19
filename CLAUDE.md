# CLAUDE.md

Context for any Claude session picking up this project. Read this before doing anything else.

## What this is

A wildlife tally PWA. Single-page web app for families tracking wildlife sightings on national park trips. Three modes:

- **Team mode** — everyone tallies toward one shared goal.
- **Pick an Animal mode** — each player picks one animal and only sightings of that animal score for them. (Internally still stored as `mode: 'versus'` for backward compatibility with existing saved trips — the label changed, the stored value didn't.)
- **Fantasy Team mode** — players are grouped into teams, then teams draft animals (random first pick; snake order for 3+ teams, straight alternating for 2; drafts every animal). Each animal scores for the team that drafted it. Tallying is per-animal like Team mode; the app routes each animal's points to its owning team. Stored fields on a fantasy trip: `teams[]`, `animalOwners{}` (animalId→teamId), `draft{ sequence[], pickIndex, complete }`. The draft is an interactive turn-by-turn screen shown until `draft.complete`.

Vanilla HTML/CSS/JS in a single `index.html`. No build step. No framework. Runs anywhere with HTTPS. Installable as a PWA via `manifest.json` + `sw.js`.

## Architecture rules — don't break these

- **Park registry pattern.** `PARKS` is a config object at the top of the `<script>` block in `index.html`. Adding a new park = adding one new entry with `{ id, name, region, defaultGoal, animals: [...] }`. No other code changes needed. New parks automatically show up in the new-trip dropdown.
- **Per-trip animal snapshots.** When a new trip is created, the park's animal list is deep-cloned into the trip object. Edits to a park preset later do NOT retroactively affect existing trips. This is intentional — preserve trip history.
- **Baby points = adult + 1, always.** Universal rule. Don't add per-animal baby overrides — the uniform rule is the simplification that kept the data model clean. Bison/elk with 0 adult points still gets 1 baby point because the rule applies.
- **Trail bonus = +1, always.** A sighting logged "on trail" scores one extra point. Applies to seen animals only, never to scat/tracks.
- **Sightings ledger, not counters.** Every logged sighting is its own record in `trip.sightings[]`: `{ id, animalId, day, form: 'seen'|'scat'|'tracks', age: 'adult'|'baby', trail, playerId, ts }`. ALL scores derive from this list through `sightingPoints()` — never store computed totals. `playerId` is set only in versus mode.
- **Advanced modes are per-trip flags.** `trip.modes = { scat, tracks }` chosen at trip creation. Scat/track finds score full points for `predator: true` animals, half rounded up otherwise. New modes should follow this pattern: a flag in `trip.modes`, a `form` value on sightings, scoring inside `sightingPoints()`.
- **One animal per entry, no slash-combos.** Presets list each species separately (split from the old combined entries; each kept the group's point value). Umbrella categories with one name ("Owls", "Wading Birds", "Bats") are fine. Every park preset includes Striped Skunk (2) and Snake (2). `predator: true` marks Carnivora + raptors + crocodilians + snakes.
- **One localStorage key.** `wildlife-tally-app-v2` holds the entire app state. If the data shape changes, bump the version suffix and write a migration. Don't silently break existing user data. (v1 counter data auto-migrates to v2 sightings on first load; the v1 key is left in place as a backup.)
- **Goal auto-fills from park × days.** `PARK_PER_DAY` (map near `DEFAULT_PARK`) holds a typical points-per-day estimate per park. `suggestedGoal(parkId, days)` returns `perDay × days` rounded to the nearest 5. The new-trip form pre-fills the goal from this and re-computes it when park or days change, *unless* the user types in the goal field (`formState.goalEdited` flag) — a manual value sticks until they switch parks or start a new trip. New parks without a `PARK_PER_DAY` entry fall back to `defaultGoal/3`, so `defaultGoal` is now only a fallback.
- **No frameworks, no build step.** Adding React/Vue/Vite/anything is a significant decision and should be discussed with Brandon before doing it.

## Aesthetic system — don't drift

The app is a **vintage national park field journal**. Maintain this on every screen.

**Palette** (CSS variables at top of the `<style>` block):
- `--cream` `#f1e7d0`, `--paper` `#faf3e0` — backgrounds
- `--forest` `#2b3a23`, `--forest-light` `#3d5230` — primary text and borders
- `--rust` `#b3551e`, `--rust-dark` `#8a4017` — accents, alerts
- `--gold` `#c89a3c` — highlights, baby points
- `--brown` `#3d2817` — body text

**Fonts** (self-hosted woff2 in `fonts/`, declared via `@font-face` at the top of the `<style>` block — do not reintroduce the Google Fonts CDN; the app must work fully offline):
- **Alfa Slab One** — display, badges, big numbers
- **Special Elite** — typewriter labels, captions, subtitles
- **Lora** — body text

**Visual idioms**: double-line and dashed borders, "official stamp" badges with letter-spaced caps, paper texture overlay, sticker-like offset drop shadows (2–3px solid, not blurred).

**Don't introduce**: blue/purple accents, modern flat pills, sans-serif body, decorative gradients beyond the existing gold→rust one, emoji as UI elements.

## Working preferences

- Ask before assuming. Brandon doesn't want a buffet of options — give your real recommendation with reasoning.
- Don't sound AI. Direct, conversational, no "I'd be happy to help" filler.
- Be willing to defend choices and admit when wrong. He'll pressure-test.
- Iterate in small steps over sweeping changes.
- He's a serious landscape photographer — visual polish matters. Use screenshots/mockups when relevant.

## Current state

Deployed and live at **https://brandonphaley.github.io/wildlife-tally/** (GitHub Pages, repo `brandonphaley/wildlife-tally`, `main` branch root). Fonts are self-hosted and precached — zero external network dependencies; the app is fully offline-capable once installed. Field-tested by the family for a week (July 2026); that feedback produced the sightings-ledger release: dropdown submission page with per-day ledger, Totals tab, split animal entries, trail bonus, and Scat/Tracks modes.

Deploying an update: commit, push to `main`, and bump `CACHE` in `sw.js` — Pages rebuilds automatically in ~1 minute.

## Roadmap (rough priority)

1. **Verify on Brandon's phone** — install from the live URL via Chrome, confirm offline use (airplane mode) + persistence.
2. **More parks** — only Yellowstone & Grand Teton is built. Glacier is the next most time-sensitive (Brandon has a trip there in July 2026). Smokies, Olympic, Acadia are likely candidates after that. Each needs its animal preset researched and added to `PARKS`.
3. **Trip export** — share final tally as image or PDF with the family. Versus mode standings especially are share-worthy.
4. **Per-player attribution in team mode** — currently team mode doesn't track who spotted what. A "current spotter" toggle or per-tap attribution would add depth.
5. **Photo per sighting** — attach a photo from the camera roll to a specific tally. Big feature, lots of edge cases — discuss scope before starting.

## Ask before doing

- Adding a build step or framework.
- Changing the localStorage data shape (needs a migration).
- Changing anything in the aesthetic system that's user-visible.
- Adding any network request at all — the app currently makes zero (no CDN, no analytics, no telemetry) and offline use in the parks depends on that.

## Just do (no need to ask)

- Add new park presets — research animals, propose point values, add to `PARKS`. Brandon will sanity-check the values.
- Fix bugs.
- Improve mobile UX inside the existing design system.
- Refactor JS for clarity without changing behavior.
- Bump the service worker cache version (`CACHE` constant in `sw.js`) when shipping changes — installed users need this to pull updates.
