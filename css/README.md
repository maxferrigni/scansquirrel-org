# Scan Squirrel — CSS Architecture

## Files

| File | Loaded by | Prefix |
|---|---|---|
| `base.css` | Every page | — (global primitives) |
| `scan.css` | `/scan/*` pages | `.scan-` |
| `territory.css` | `/squirrels/[neighborhood]/` pages | `.ter-` |
| `legal.css` | `terms.html`, `privacy.html`, `release.html` | `.legal-` |

## Load order

**Main pages** (index, compare, squirrels hub, FAQ, pricing):
```html
<link rel="stylesheet" href="/css/base.css">
<link rel="stylesheet" href="/css/nav.css">
<!-- page-specific scoped styles stay inline -->
```

**Scan pages** (/scan/photographs, /scan/slides, etc.):
```html
<link rel="stylesheet" href="/css/base.css">
<link rel="stylesheet" href="/css/nav.css">
<link rel="stylesheet" href="/css/scan.css">
```

**Territory pages** (/squirrels/pasadena, etc.):
```html
<link rel="stylesheet" href="/css/base.css">
<link rel="stylesheet" href="/css/nav.css">
<link rel="stylesheet" href="/css/territory.css">
```

**Legal pages**:
```html
<link rel="stylesheet" href="/css/base.css">
<link rel="stylesheet" href="/css/nav.css">
<link rel="stylesheet" href="/css/legal.css">
```

## What goes where

### base.css
- `:root` tokens — one source of truth for the entire site
- Google Fonts import (Fraunces + DM Sans, weights 400/500/600/700)
- Reset and body defaults
- Type scale (H1–H4, body, section label)
- `.hero-lg` variant for marketing pages
- `.label` (section label), `.eyebrow` (media type nav)
- `.wrap` (720px), `.wrap-wide` (960px)
- `.btn`, `.btn-ghost`, `.btn-row`
- `.cta-strip` (shared dark CTA block)
- `.card`, `.card-accent`
- Utility classes (`.muted`, `.text-center`, `.bg-warm`, `.bg-dark`)

### scan.css
- Hero with illustration grid
- Spec summary row (DPI, price, turnaround)
- Process steps (auto-numbered with CSS counter)
- Scan-specific cards, grids, notes
- CTA block

### territory.css
- Hero (centered)
- 5-column media type grid with grades and links
- Coffee/meetup spot cards (teal left accent border)
- Full-width image (`.ter-img`)
- Two-up image grid (`.ter-imgs`)
- CTA block

### legal.css
- 760px document wrapper
- Header with last-updated line
- Summary box
- Table of contents (two-column on desktop)
- Content sections with dividers
- Contact box (dark)

### nav.css (unchanged)
- Nav and footer styles stay in nav.css
- Do NOT move footer styles to base.css — they're already shared via nav.css

## What stays inline

Main pages (index, compare, squirrels hub, FAQ, pricing) keep their unique
layout styles inline, scoped with page-specific prefixes:
- Compare: `.c5` prefix
- Squirrels hub: existing class names
- FAQ: accordion, category nav (unique to that page)
- Pricing: price cards, example calculator (unique to that page)

These pages load only `base.css` + `nav.css` and inline their layout CSS.
Over time, shared patterns can be promoted to base.css.

## Migration rules

When refactoring a page to use the shared CSS:

1. Remove the inline `:root` block — base.css owns all tokens
2. Remove the inline Google Fonts `<link>` — base.css imports fonts
3. Remove the inline reset (`* { margin:0; ... }`) — base.css handles it
4. Remove inline body styles — base.css handles it
5. Replace page-specific button classes with `.btn` / `.btn-ghost`
6. Replace page-specific section labels with `.label`
7. Replace page-specific wrappers with `.wrap` or `.wrap-wide`
8. Keep only truly page-specific styles inline

## Token names

Use the original names. Do NOT rename to `--ss-` prefix or any other scheme.

```
--ink            #1a1a1a    Body text only
--bone           #f0ebe3    Page background
--warm           #f5f0e8    Alt surface
--cream          #e8e1d5    Soft fill
--feather        #2a2a2a    Dark sections (not --ink)
--acorn          #8b5e3c    Primary buttons
--acorn-bright   #a0693f    Button hover
--teal           #2c6e6a    Accent only
--ash            #6b6660    Muted text
--nest           #d4c9b8    Text on dark
--branch         #8b7e6a    Secondary muted
--bark           #5c4a3a    Dark warm text
```

## Radius standard

- Cards and panels: 8px (`--radius-card`)
- Buttons and small controls: 6px (`--radius-small`)

## Font weights

400, 500, 600, 700 only. Weight 800 is NOT used anywhere.
