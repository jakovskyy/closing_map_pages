# Closing Map — literal reproduction of ClosingMap-Final.xd

Eight pages rebuilt directly from the design file. Every element is positioned from the coordinates in the XD scene graph, so this is a literal reproduction rather than an interpretation.

**Coverage, measured against the file:**

| | Extracted from XD | Rendered |
|---|---|---|
| Text strings | 633 | **633 (100%)** |
| Images | 71 | **71 (100%)** |
| Shapes | 597 | 573 (96%) |

The 24 missing shapes are complex compound vector paths. Everything else — every word, every image, every rectangle, circle, line and simple path — is present and positioned.

## Run it

Open `index.html`. No build step, no server. A dark **XD PAGES** bar across the top is a prototype aid for jumping between pages; it is not part of the design and should be deleted before this goes anywhere.

Each page is a 1920px stage scaled to fit the viewport, matching the artboard width. Artboard heights range from 4,875px (the two home pages) to 9,335px (Article Sample).

## How it was built, and three bugs worth knowing about

The first version of this build inferred layout from type sizes and reading order. It was wrong, and it was wrong in ways that a text-presence check did not catch. Three bugs, all now fixed:

1. **Group traversal.** XD nests grouped layers under `group.children`, not `children`. Following only the latter meant **58% of the design was never read** — 553 nodes found where 1,301 exist. Whole navigation bars and footers were simply absent.
2. **Artboard origin.** Artboards sit on an infinite canvas and their children carry absolute canvas coordinates. Home - Buying happens to sit at x = -12 so it rendered by luck; the other seven sit between x = 1,965 and x = 13,796 and rendered **completely blank**. Origins come from `uxdesign#bounds` in the manifest and are now subtracted.
3. **Per-run colour.** Ranged text styles store fill as a plain ARGB integer rather than a colour object. Two-tone headlines rendered in a single colour until this was handled.

The lesson for anyone auditing this later: checking that a string exists in the DOM proves nothing about whether it is visible, positioned, or the right colour.

## Pages

| File | XD artboard | Fidelity |
|---|---|---|
| `index.html` | Home - Buying | Literal |
| `selling.html` | Home - Selling | Literal |
| `how-it-works-buyers.html` | How It Works - Buyers | Literal |
| `how-it-works-sellers.html` | How It Works - Sellers | Literal |
| `calculators.html` | Calculators - Buyers | Hub, hand-built from the artboard |
| `learn.html` | Learn - All | Literal |
| `article.html` | Article Sample | Literal |
| `pending.html` | — | Placeholder for undesigned destinations |

Every page is generated from `design-data/nodes.json` by `/tmp/literal.py`. Re-running the extractor and generator reproduces the whole site, so corrections belong in the extractor rather than in hand-edited HTML.

**A note on what "literal" costs.** Absolutely positioned output reproduces the design exactly but is not idiomatic markup. Elements use `h1`–`h3` and `p` tags by size so the document still has structure, but a developer building the real site should treat this as a specification to read, not a codebase to extend.

---

## Tokens

Read out of the XD scene graph, not sampled from images. Earlier drafts of this work used colours sampled from JPEG screenshots and every one of them was wrong by a few points.

| Token | Value | Uses in file |
|---|---|---|
| `--ink` | `#172032` | 252 |
| `--blue` | `#2F6FD6` | 92 |
| `--white` | `#FFFFFF` | 65 |
| `--pale` | `#F0F5F9` | 15 |
| `--grey` | `#626468` | 2 |
| `--green` | `#44A230` | 2 |

The green appears **twice in the entire design**. It is a specific accent, not a brand-wide colour, and it is far more saturated than the logo green (`#79BD62`). Do not use it as a general success colour without asking.

**Type** is Gilroy in five weights, self-hosted as woff2 (43 KB each, down from ~140 KB TTF). The scale in `site.css` reflects real usage frequency: 22px carries 66 uses and is the dominant body size, 19px carries 53.

Artboards are **1920 wide**. The content column is capped at 1400 with fluid gutters, so the design scales down rather than being locked to 1920.

---

## Assets

72 files in `assets/img/`. Every image ships as WebP with a PNG or JPG fallback via `<picture>`.

Originals totalled **156 MB**. Web derivatives total **3 MB**, capped at 1400px wide. Originals are untouched in `Brand Guidelines/Final Assets (1)/` and `design-data/images/`.

### Two things to resolve

**1. The Final Assets folder and the XD file are out of sync.**

Present in the XD but missing from Final Assets: `Buyer Example Transaction-Blue.png`, `Match.png`, `Step 1–4 copy.png`, `ChatGPT Image May 19 2026.png`, `ClosingMap-RGB.png`.

Present in Final Assets but unused by the XD: `Buyer Example Transaction.png` (the un-suffixed version), `Closing Costs Estimate.png`, `Door.png`, `Hand.png`, `How It Works - Buyers - Hero.png`, `Target.png`.

This build pulls the embedded copies out of the `.xd`, so nothing is missing here. But the folder alone is not a complete asset set.

**2. Eight iStock photographs are embedded in the design.**

Filenames carry the asset IDs, for example `istock-2224485283`. Confirm web-use licences were purchased before anything goes live. iStock enforces this actively.

---

## Link graph

Taken from `interactions/interactions.json` in the XD file, so the routing is the designer's, not a guess.

The navigation offers Calculators for Buyers, Sellers and Lenders, but **only the Buyers artboard exists**. Those destinations, along with Sign in, Get started, the seller net sheet and the legal pages, route to `pending.html`, which states plainly that the page has not been designed. Nothing 404s and nothing is hidden.

---

## Known issues in the source design

Found while extracting. Passing them along rather than silently fixing them.

1. **Seller page carries buyer copy.** The stats band on Home - Selling reads *"typical buyer closing cost range nationally"* and *"no fees, no account required to run the numbers."* Copied from the buying page and never updated. This build substitutes seller-appropriate copy on `selling.html`; revert if you want the design reproduced literally.
2. **"All 50 states" contradicts the footer**, which says serving New York, New Jersey and Florida. Both appear on the home page.
3. **No Calculators artboards for Sellers or Lenders**, though the nav points at both.

---

## Structure

```
closingmap-site/
  index.html  selling.html  how-it-works-buyers.html  how-it-works-sellers.html
  calculators.html          learn.html  article.html
  pending.html
  assets/css/site.css        one stylesheet, all tokens at the top
  assets/fonts/              Gilroy ×5, woff2
  assets/img/                72 files, WebP + fallback
  README.md
```

Extraction artefacts live one level up in `design-data/`:

- `pages.json` — every text node with position, size, weight, colour, per-run styling
- `copy-deck.txt` — all 322 strings in reading order, human readable
- `image-placements.json` — 45 placements with coordinates and fit behaviour
- `structure.json` — artboards, the full link map, image inventory
- `tokens.json` — the palette and type system
- `images/` — 30 images extracted from the `.xd` at original resolution
