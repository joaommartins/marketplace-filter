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

---

## Listing statuses

Each listing card gets a small panel in the top-left corner of the photo with three buttons:

| Icon | Status | Description |
|---|---|---|
| ✓ (green) | Good | Worth considering |
| ✗ (red) | Bad | Not a fit |
| ★ (yellow) | Later | Save for later |

**How to use:**
- Click a button to set the status (the button highlights).
- Click the same button again to clear the status.
- Only one status can be set per listing.

**Listings marked "Bad"** are dimmed based on the active status filter:

| Active filter | Behavior |
|---|---|
| No filter selected | opacity: 0.4; hover restores full visibility |
| "Bad" filter selected | full opacity (the listing matches the filter) |
| Any other filter selected | opacity: 0.1; no hover effect |

**Storage:** statuses are saved in Tampermonkey's GM storage and persist across browser data clears.

---

## Status filter

The bottom row of the panel has three status toggle buttons. Clicking a toggle shows only listings with that status; others are dimmed. Multiple statuses can be selected at once. Click again to deselect.

---

## Notes

Each listing card has a note field next to the status buttons. Click it to enter a short note (up to 50 characters).

- When empty, a dim `+ note` placeholder is shown
- Save: **Enter** or click outside the field
- Cancel: **Escape**
- Notes are stored in GM storage and tied to the listing ID

---

## Clear all data

The trash icon button on the right side of the status filter row prompts for confirmation, then deletes **all** saved statuses and notes. Cards update immediately without a page reload.

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
