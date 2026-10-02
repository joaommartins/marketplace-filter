# Marketplace Filter

A Tampermonkey userscript that adds keyword filtering and status markers to Facebook Marketplace listings.

## Installation

1. Install the [Tampermonkey](https://www.tampermonkey.net/) extension for your browser.
2. Download [`marketplace-filter.user.js`](https://github.com/ai36/marketplace-filter/raw/main/marketplace-filter.user.js) — Tampermonkey will prompt you to install it automatically.
3. Navigate to `facebook.com/marketplace/` — the panel will appear in the bottom-left corner.

Alternatively, open Tampermonkey → **Create a new script**, paste the file contents, and save (`Ctrl+S`).

---

## Filtering

The panel contains a search input. Listings that don't match the query are dimmed (opacity: 0.1). A counter at the bottom of the panel shows the number of matches.

A **×** button on the right side of the input clears the query and resets the filter.

### Query syntax

| Pattern | Description |
|---|---|
| `word` | Title contains the word |
| `word1+word2` | Both words must be in the title |
| `word1 word2` | Same as `word1+word2` — whitespace is an implicit `+` |
| `word1\|word2` | Either word must be in the title |
| `-word` | Title must **not** contain the word |
| `(word1\|word2)+word3` | Grouping with parentheses |
| `-(word1\|word2)` | Neither word may appear |

`-` negates the term or parenthesised group that follows it, so `a -b`, `-b` and `-(b|c)` all work. A `-` inside a word is literal, as in `plug-in`. A stray `-` with no term after it is ignored.

Search is case-insensitive. Price and listing ID are ignored — only the title and city are matched, so `portland` filters by city.

### Examples

```
prius+prime              → Prius Prime only
(prius|camry)+2017       → Prius or Camry, 2017
plug-in+hybrid           → any plug-in hybrid
prius -camry             → Prius, excluding Camry
-wanted -parts           → hide wanted/parts ads
-(prius|camry)           → anything that is neither
```

### Search history

Focusing the input shows a dropdown with up to 10 recent queries. If the field is not empty, the list filters to entries that match the current text. Clicking an item applies that query. The dropdown opens upward when there isn't enough space below in the viewport.

### Bulk marking

While a filter is narrowing the list, the panel shows a **Mark all N matching:** row with the three status buttons. Click one to apply that status to every matching listing at once — click the same one again to clear it from all of them.

The row appears only when the query, the status filter or the unseen filter is active *and* at least one listing matches, so it can never mean "mark the whole page". Listings the filter has dimmed are never touched.

---

## Listing statuses

Each listing card gets a small panel in the top-left corner of the photo with three buttons, plus a bookmark chip when the listing is in your Facebook Saved list (see [Saved listings](#saved-listings)):

| Icon | Status | Description |
|---|---|---|
| ✓ (green) | Good | Worth considering |
| ✗ (red) | Bad | Not a fit |
| ★ (yellow) | Later | Save for later |

**How to use:**
- Click a button to set the status (the button highlights).
- Click the same button again to clear the status.
- Only one status can be set per listing.

**Listings marked "Bad"** are dimmed based on the active filter:

| Situation | Behavior |
|---|---|
| No filter excludes the listing | opacity: 0.25; hover restores full visibility |
| The "Bad" filter is selected | full opacity (the listing matches the filter) |
| Any other filter excludes it | opacity: 0.1; no hover effect |

**Storage:** statuses are saved in Tampermonkey's GM storage and persist across browser data clears.

---

## Status filter

The controls are laid out in rows: the status filters and the view toggles share one, and the reset and trash buttons sit on the next alongside the match counter. Clicking a status toggle shows only listings with that status; others are dimmed. Multiple statuses can be selected at once. Click again to deselect.

---

## Hiding rejected listings

Two toggles in the bottom row remove the listings you marked **"Doesn't match"** from the grid rather than just dimming them. In both views the match counter stops counting them.

| Toggle | Effect |
|---|---|
| Crossed eye | **Hide.** The cards disappear but keep their slot, so nothing shifts while you work down a page. A fully rejected row still collapses its height, pulling the row below up. |
| Re-stack (chevrons) | **Hide and close the gaps.** Facebook's per-card cell is collapsed as well, so the remaining cards flow back into place. Re-stacking implies hiding. |

Selecting the "Doesn't match" status filter turns both views off, and switching either one on clears that selection — together they would empty the grid.

---

## Seen tracking

Listings you have opened are remembered, so you can tell new ones from ones you've already looked at.

- **Unseen** listings get a bold green ring.
- **Seen** listings are dimmed behind a dark veil, so new ones stand out at a glance. The veil sits under the status buttons, so their colours stay readable.
- A listing stops being new as soon as you do anything with it: open it (any click on the card outside the status panel), give it a status, or save it on Facebook. Only untouched listings keep the ring.
- Listings marked **"Doesn't match"** keep their own dimming rather than the veil — rejecting a listing already fades it, and it restores on hover.
- The **eye** toggle shows only unseen listings; everything already seen is dimmed. Click again to show all.
- The **↺** button clears the seen marks from openings. Listings you saved or triaged stay seen, since that follows from those states; the trash button clears everything.

Seen marks are stored in GM storage, one entry per listing.

---

## Freezing the list

Marketplace keeps loading listings as you scroll, which makes "everything I've looked at" a moving target. The **pause** button freezes the list at the moment you press it: anything that loads afterwards is ignored — hidden, left out of the counter, and out of reach of the sweep — until you press it again. Unfreezing brings those listings back, still unseen.

The **double-check** button marks every loaded listing as seen in one go. Its tooltip always names the exact count, and with the list frozen that count cannot change under you. Freeze first if you want the sweep to be predictable.

---

## Saved listings

A bookmark badge appears on any listing that is in your Facebook **Saved** list, so you can tell at a glance which ones you've already bookmarked.

Grid cards themselves carry no saved flag, so the script mirrors the list from Facebook's Saved page: **open `facebook.com/marketplace/you/saved/` once** and every listing rendered on that page is recorded. Scroll it to load more before leaving if you want deeper coverage — only what has actually rendered is captured. Badges then appear wherever those listings show up.

- The mirror is stored per listing in GM storage and re-read on every page.
- Saved listings also count as **seen**, so they are never outlined as new and never shown by the "unseen only" filter.
- Saved listings also read as the **Consider later** status, unless you have given them another status since. Because that follows from being saved, the ★ button cannot be cleared on them while they remain saved — give it a different status, or clear the mirror with the trash button.
- The mirror is **add-only**: un-saving a listing on Facebook does not remove its badge. The trash button clears the mirror.
- If Facebook renames the Saved route, `SAVED_PATH_RE` in the saved-tracking section is the single line to update.

---

## Clear all data

The trash icon button on the right side of the status filter row prompts for confirmation, then deletes **all** saved statuses, seen marks and the mirrored saved list. Cards update immediately without a page reload.

---

## Miscellaneous

- **Dynamic cards** — new listings loaded on scroll automatically receive status buttons and are checked against the active filter.
- **SPA navigation** — the panel reinitializes on Facebook's client-side page transitions.
- **Collapse/expand** — the `–`/`+` button in the top-right corner of the panel.

## Project structure

```
marketplace-filter/
├── marketplace-filter.user.js   # main script
└── README.md            # documentation
```

## Compatibility

- Browsers: Chrome, Firefox, Edge (with Tampermonkey installed)
- Site: `https://www.facebook.com/marketplace/*`
