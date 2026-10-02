# Marketplace Filter

A Tampermonkey userscript that adds keyword filtering and status markers to Facebook Marketplace listings.

## Installation

1. Install the [Tampermonkey](https://www.tampermonkey.net/) extension for your browser.
2. Download [`marketplace-filter.user.js`](https://github.com/ai36/marketplace-filter/raw/main/marketplace-filter.user.js) — Tampermonkey will prompt you to install it automatically.
3. Navigate to `facebook.com/marketplace/` — the panel will appear in the bottom-left corner.

Alternatively, open Tampermonkey → **Create a new script**, paste the file contents, and save (`Ctrl+S`).

---

## Filtering

The panel contains a search input. Listings that don't match the query are dimmed (opacity: 0.1). A counter below the input shows the number of matches.

A **×** button on the right side of the input clears the query and resets the filter.

### Query syntax

| Pattern | Description |
|---|---|
| `word` | Title contains the word |
| `word1+word2` | Both words must be in the title |
| `word1\|word2` | Either word must be in the title |
| `(word1\|word2)+word3` | Grouping with parentheses |

Search is case-insensitive. Price and listing ID are ignored — only the title and city are matched.

### Examples

```
prius+prime              → Prius Prime only
(prius|camry)+2017       → Prius or Camry, 2017
plug-in+hybrid           → any plug-in hybrid
```

### Search history

Focusing the input shows a dropdown with up to 10 recent queries. If the field is not empty, the list filters to entries that match the current text. Clicking an item applies that query. The dropdown opens upward when there isn't enough space below in the viewport.

### Location filter

Below the search input is a locations field. Enter a comma-separated list of allowed locations — city and/or state, case-insensitive, matched as substrings against the listing's location:

```
portland, seattle
```

While the field is non-empty, listings whose location matches **none** of the terms are dimmed. Clear the field to disable the filter. The value is saved and re-applied automatically on page load.

Location is read from each listing's `aria-label`, from the segments between the price and the listing ID (e.g. `Portland, OR`). A listing whose location can't be determined is treated as non-matching while the filter is active.

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

The bottom row of the panel has three status toggle buttons, the seen-tracking controls (below) and the trash button. Clicking a toggle shows only listings with that status; others are dimmed. Multiple statuses can be selected at once. Click again to deselect.

---

## Seen tracking

Listings you have opened are remembered, so you can tell new ones from ones you've already looked at.

- **Unseen** listings get a green outline.
- Opening a listing — any click on the card outside the status panel — marks it seen and removes the outline. Clicking a status button does **not** count as viewing.
- Listings in your Facebook Saved list always count as seen, whether or not you opened them (see [Saved listings](#saved-listings)).
- The **eye** toggle shows only unseen listings; everything already seen is dimmed. Click again to show all.
- The **↺** button clears every seen mark you set by clicking. Saved listings stay seen, since that follows from being saved; use the trash button to clear everything.

Seen marks are stored in GM storage, one entry per listing.

---

## Saved listings

A bookmark badge appears on any listing that is in your Facebook **Saved** list, so you can tell at a glance which ones you've already bookmarked.

Grid cards themselves carry no saved flag, so the script mirrors the list from Facebook's Saved page: **open `facebook.com/marketplace/you/saved/` once** and every listing rendered on that page is recorded. Scroll it to load more before leaving if you want deeper coverage — only what has actually rendered is captured. Badges then appear wherever those listings show up.

- The mirror is stored per listing in GM storage and re-read on every page.
- Saved listings also count as **seen**, so they are never outlined as new and never shown by the "unseen only" filter.
- The mirror is **add-only**: un-saving a listing on Facebook does not remove its badge. The trash button clears the mirror.
- If Facebook renames the Saved route, `SAVED_PATH_RE` in the saved-tracking section is the single line to update.

---

## Notes

Each listing card has a note field next to the status buttons. Click it to enter a short note (up to 50 characters).

- When empty, a dim `+ note` placeholder is shown
- Save: **Enter** or click outside the field
- Cancel: **Escape**
- Notes are stored in GM storage and tied to the listing ID

---

## Clear all data

The trash icon button on the right side of the status filter row prompts for confirmation, then deletes **all** saved statuses, notes, seen marks and the mirrored saved list. Cards update immediately without a page reload.

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
