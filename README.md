# EDH Pod Life Tracker

A single-file, zero-dependency life tracker for **Magic: The Gathering Commander (EDH)**. It's built for one phone or tablet lying flat in the middle of the table. It tracks life totals, commander damage, and eliminations for 2–4 players, and keeps a local match history you can export and import as JSON.

No install, no build step, no account. Open `index.html` in a browser and play.

---

## Features

### Table layouts
- **2P, 3P, and 4P modes**, switched from the control bar at the bottom of the screen.
- **Seat-aware orientation.** Top-row player cards are rotated 180° so everyone around the table reads their own total right side up.
  - **4P:** 2×2 grid. Top two seats flipped.
  - **3P:** two flipped seats on top, one full-width seat on the bottom.
  - **2P:** face-to-face, one seat flipped.

### Life tracking
- Every player starts at **40 life**.
- Large **78px touch targets** for `+` / `—`, sized for quick tapping mid-game.
- Life totals can go below zero.

### Commander damage
- Each player card has a **Commander Damage Received** panel with one row per opponent.
- Rows are color-coded to match each opponent's seat theme.
- Adding commander damage **also subtracts that amount from life**, so you don't have to tap twice. Removing it gives the life back.
- When damage from any single commander reaches **21 or more**, that row turns red and pulses as a lethal warning.
- Commander damage can't go below 0.
- Rows for empty seats are hidden in 2P and 3P modes.

### Commander art
- Tap **Choose commander** under a player's name to open a searchable list of every commander-legal card (about 3,500).
- Type any part of the name. Each word matches the start of a word, so `tchalla`, `black pan`, and `sauron lord` all work. Color identity pips appear next to each result.
- Picking a commander sets that seat's background to the card's art, with an **Art: _artist_** credit. **Use land background** switches back.
- Art comes from the newest *standard* printing: available in nonfoil, not a promo, and not a showcase, extended-art, etched, or other "booster fun" variant.
- Each seat's commander is remembered on that device and recorded in **Log Game**, so match history shows who played what.
- Searching works offline because the list ships with the app (`commanders.js`). The art itself loads from Scryfall, so without internet the seat keeps its land background.

### Player names
- Tap a player's name to rename them.
- New names update automatically in every opponent's commander damage panel.
- Clearing a name falls back to that seat's default.

### Commander tax
- Each player has a **Tax** counter next to their commander. Tap **+** each time they recast their commander from the command zone.
- It counts in steps of 2 (0 → 2 → 4 → 6…), so the number is exactly the extra mana the next cast costs.
- The number turns amber once a player has tax. It can't go below 0, and it locks when the player is eliminated.
- **Reset** sets everyone back to 0, and **Log Game** records each player's final tax.
- On narrow screens (a phone in 4P), the counter moves to its own line under the commander button.

### Monarch
- Each player card has a **crown** button next to **I'm Dead**. Tap it to make that player the monarch.
- Only one player can be the monarch. Crowning someone takes the crown from whoever had it, and tapping the current monarch's crown removes it.
- The monarch's card gets a gold border and a gold crown, so the whole table can see who draws the extra card.
- If the monarch is eliminated, the crown is cleared. Tap it onto the new monarch, since the app doesn't track whose turn it is.
- **Reset** clears the monarch, and switching to a smaller layout clears it from a seat that's no longer in play.
- **Log Game** records who held the crown at the end, shown as ♛ in match history.

### Elimination
- **I'm Dead** marks a player as eliminated. The card dims, and its life and commander damage controls lock.
- The button switches to **Undo**, so an accidental tap is easy to reverse.

### Reset
- **Reset** (with a confirmation prompt) restores all players to 40 life, clears all commander damage, and revives eliminated players.
- Player names and the current layout are kept, so the same pod can go straight into the next game.

### Match history
- **Log Game** saves a snapshot of the current game: player count, timestamp, each player's name, final life, and elimination status, plus the winner(s).
- **Winner logic:**
  - If exactly one player is still alive, they win.
  - Otherwise, the surviving player(s) with the highest life total are recorded as winners. Ties produce multiple winners.
- **History** opens a slide-up panel listing every logged game, newest first. Winners are shown with 🏆, eliminated players are struck through, and survivors show their remaining life.
- History is stored in your browser's `localStorage` under the key `edh_match_history`, so it survives page refreshes on the same device and browser.

### Data portability
- **Export JSON** downloads the full match history as a timestamped `.json` file.
- **Import JSON** loads a previously exported file and merges it with your existing history. Imported games are added ahead of existing ones.
- **Clear All** permanently erases the stored history after a confirmation prompt.

---

## Seat themes

Each seat uses basic-land artwork as its background, under a tinted scrim so life totals stay readable. Eliminated players' art turns greyscale.

| Seat | Default name | Background | Tint |
|------|--------------|------------|------|
| 1 | Arcane | Plains | Mystic purple |
| 2 | Inferno | Mountain | Volcanic red |
| 3 | Forest | Forest | Deep pine green |
| 4 | Shadow | Swamp | Obsidian grey |

---

## Getting started

### Run locally
1. Download or clone this repository.
2. Open `index.html` in any modern browser (Chrome, Safari, Firefox, Edge).

### Use it at the table
For the best experience, host it so it opens like an app on your phone or tablet:

1. Enable **GitHub Pages** for this repo (Settings → Pages → deploy from the `main` branch).
2. Open the Pages URL on your device.
3. Add it to your home screen:
   - **iOS (Safari):** Share → *Add to Home Screen*
   - **Android (Chrome):** ⋮ menu → *Add to Home screen*

The page disables pinch-zoom and text selection so stray touches during play don't shift the layout.

---

## Updating the commander list

When a new set comes out:

1. Open `tools/update-commanders.html` in your browser and click **Build list**. It takes about a minute.
2. Your browser downloads a new `commanders.js`.
3. Move it into the app folder, replacing the old one.

## Match history data format

Exported files are a JSON array of match records:

```json
[
  {
    "id": "match_1760897280000",
    "timestamp": "10/19/2026, 2:08:00 PM",
    "mode": "4 Players",
    "players": [
      { "id": 1, "name": "Arcane",  "lifeLeft": 12, "isDead": false, "isMonarch": true, "commanderTax": 4, "commander": "T'Challa, the Black Panther" },
      { "id": 2, "name": "Inferno", "lifeLeft": 0,  "isDead": true  },
      { "id": 3, "name": "Forest",  "lifeLeft": -3, "isDead": true  },
      { "id": 4, "name": "Shadow",  "lifeLeft": 5,  "isDead": true  }
    ],
    "winners": [1]
  }
]
```

| Field | Description |
|-------|-------------|
| `id` | Unique ID built from the save time in milliseconds |
| `timestamp` | Human-readable local date and time the game was logged |
| `mode` | `"2 Players"`, `"3 Players"`, or `"4 Players"` |
| `players` | One entry per active seat: seat `id`, `name`, final `lifeLeft`, `isDead`, `isMonarch`, `commanderTax`, and `commander` (name, or `null` for land background; missing in games logged before this feature) |
| `winners` | Array of winning seat IDs (more than one on a tie) |

---

## Project structure

```
.
├── index.html          # The entire app: markup, styles, and script
├── plains.png          # Arcane background
├── mountain.png        # Inferno background
├── forest-cropped.png  # Forest background (letterbox bars removed)
├── forest.png          # Original forest art
├── swamp.png           # Shadow background
├── commanders.js       # Bundled commander list (name, Scryfall id, artist, color identity)
├── tools/
│   └── update-commanders.html  # Rebuilds commanders.js from Scryfall
└── README.md
```

Keep the images in the same folder as `index.html`. If an image is missing, that seat falls back to its solid theme color.

All the logic lives in one HTML file: inline CSS for layout and theming, and vanilla JavaScript for state, commander damage logic, and the history engine. No frameworks, no external requests.

---

## Known limitations

These are worth knowing about before a game night, and are good candidates for future work:

- **In-progress games aren't saved.** Only logged games persist. Refreshing or closing the page mid-game resets life totals, commander damage, and names.
- **Winner edge case.** The highest-life check compares against *all* players, including eliminated ones. If an eliminated player has a higher life total than every survivor (for example, someone killed by 21 commander damage while at high life), no winner is recorded unless only one player is left alive. Marking everyone else dead before logging avoids this.
- **21 commander damage doesn't auto-eliminate.** The row shows the lethal warning, but the player still needs to tap **I'm Dead**.
- **One commander per player.** Partner commanders and backgrounds don't get separate commander damage rows, art, or tax counters yet.
- **No other counters.** Poison, commander tax, energy, and similar counters aren't tracked yet.
- **Switching layouts doesn't reset.** Hidden seats keep their state, and only the active seats are included when a game is logged.
- **Import doesn't de-duplicate.** Importing the same file twice adds those games twice.
- **History is per device.** `localStorage` isn't synced between browsers or devices. Use Export/Import to move it.

---

## Roadmap ideas

- Persist the in-progress game across refreshes
- Auto-eliminate at 21 commander damage or 0 life (optional toggle)
- Partner commander rows
- Poison counters
- Initiative / Undercity tracker (works like the monarch toggle)
- Turn tracker and game timer
- Win-rate and stats view built from match history (per player and per commander)
- Partner and background pairs in the commander picker
- Configurable starting life (e.g. 20 for duels, 30 for Brawl)

---

## Contributing

Issues and pull requests are welcome. Because the app is one file, the easiest way to test a change is to edit `index.html` and reload it in your browser, ideally with your browser's device emulation set to a phone or tablet.

## License

Add a license of your choice (MIT is a common pick for small projects like this).

---

*Card data and art crops are provided by [Scryfall](https://scryfall.com). Card art is credited to its artists and is copyright Wizards of the Coast. This project is free and unofficial, as permitted by the [Wizards of the Coast Fan Content Policy](https://company.wizards.com/en/legal/fancontentpolicy).*

*Magic: The Gathering is a trademark of Wizards of the Coast LLC. This project is fan-made and isn't affiliated with or endorsed by Wizards of the Coast.*
