# GutCheck — Chrome Extension

A friction layer for Amazon that intercepts impulse purchases. Built for a Type and Interaction class project by Michael French.

## What It Does

On any Amazon product page (`/dp/` URLs), the extension injects a "Gut Check" panel directly above the Add to Cart button. The panel prompts the user to reconsider the purchase and offers alternative search links:

- **Search Google** (Google Shopping, pre-filtered)
- **Search Nearby** (Google Maps)
- **Secondhand Listings** (Facebook Marketplace)
- **Local Bookstores** (IndieBound)

When the user hovers the Add to Cart or Buy Now buttons, the GutCheck panel shakes and highlights in Amazon orange to redirect attention. When a search link is clicked, the panel icon swaps to a checkmark.

## Project Structure

| File | Purpose |
|---|---|
| `manifest.json` | Chrome Extension Manifest V3 config; matches Amazon product page URLs |
| `content.js` | All extension logic — DOM injection, mutation observer, event handlers |
| `style.css` | Extension styles; borrows Amazon's design tokens (Amazon Ember font, spacing, border radii) |
| `images/` | Extension icons at 16/32/48/128px |

## Architecture

**No build step.** This is vanilla JS/CSS loaded as a Chrome content script. There are no dependencies, no bundler, and no npm.

**Content script entry points** (`content.js`):
- `insertPopup()` — finds `#add-to-cart-button`, builds the panel HTML via `createPopup()`, injects it with `insertAdjacentHTML("beforebegin", ...)`, and wires up event listeners
- `watchAmazonChanges()` — `MutationObserver` on `document.body` that re-runs `insertPopup()` when Amazon's variant selectors change (color, size, etc.) or if the popup node is removed; debounced with a 500ms timer
- `gutcheckIconSwap()` — swaps the cart icon SVG for a checkmark SVG on first search-link click
- `triggerHighlight()` — binds `mouseenter`/`mouseleave` on Amazon's buy buttons to add/remove the `gutcheck-highlight` class

**CSS approach**: The extension uses Amazon's own class names (`.a-box`, `.a-button`, `.a-icon`) so the panel blends into the native UI. Custom styles are scoped under `#gutcheck`. The highlight animation is a CSS keyframe shake (`jump-shaking`) triggered by adding a class via JS. The dropdown blur overlay uses a `body::before` pseudo-element controlled by `:has()` selectors.

## Design Tokens

Defined as CSS custom properties in `:root`:

```
--gutcheck-button-bg: #2EC4B6   (teal — primary action color)
--amazon-orange: #ffa41c        (highlight border on hover)
--gutcheck-shadow-color         (box shadow on highlight state)
```

Amazon's spacing and border radius values are also captured as variables (`--amazon-padding`, `--amazon-button-height`, `--amazon-border-radius`) to stay consistent with the surrounding Amazon UI.

## Loading the Extension Locally

1. Open `chrome://extensions`
2. Enable **Developer mode**
3. Click **Load unpacked** and select this directory
4. Navigate to any Amazon product page (URL must match `*://www.amazon.com/*/dp/*` or `*://www.amazon.com/dp/*`)

No build or install step needed.

## Key Constraints

- **Manifest V3** — no background scripts; everything runs as a content script
- **Amazon DOM dependency** — the popup insertion relies on `#add-to-cart-button` being present; pages without this element are silently skipped
- **Styles must not break Amazon's layout** — use `z-index` carefully (panel at 197, overlay at 196, dropdown at 198)
- **No external requests** — all search links open in new tabs; the extension itself makes no network calls
