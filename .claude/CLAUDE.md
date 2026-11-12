# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

The personal site at alexerlandsson.com: one page, `index.html`, with its CSS and JavaScript inline. No framework, no dependencies, no build step, no tests, no linter. The only external request is the Fraunces font from Google Fonts.

## Commands

- Preview: `python3 -m http.server 8000` from the repo root, then open `http://localhost:8000/`. Asset paths are root-absolute (`/favicon.svg`, `/site.webmanifest`), so opening the file directly breaks the icons and manifest.
- Deploy: push to `main`. GitHub Pages serves the branch root as is (legacy branch build), with Cloudflare in front of the apex domain.

## Everything in the repo root is published

- Each file in the root becomes a URL on the live site. Do not add notes, drafts or tooling files there.
- Pages runs the default Jekyll build (there is no `.nojekyll`), which skips dotfiles and dot-directories. That is why this file lives in `.claude/` and the README in `.github/` (GitHub still shows it on the repo page). Adding `.nojekyll` would publish both.
- `CNAME` holds the custom domain. Do not delete or rename it.

## How `index.html` works

Two inline scripts with separate jobs:

**Palette script (in `<head>`)** runs before first paint. It picks one of the `palettes` pairs at random (never the same as the last visit, tracked in `localStorage` under `ground-index`), then sets `--ground` and `--ink`, the `theme-color` meta and the favicon (an SVG data URL). Each pair is `[darker, lighter]`: dark mode puts the darker colour on the field, light mode flips them, and a `prefers-color-scheme` change repaints in place. Every pair must keep a contrast ratio of 6:1 or better.

**Wave script (end of `<body>`)** turns the sentence into letters that ride a wave:

- The `<p id="sentence">` is the single source of the text. Without JavaScript it is set as ordinary text. With JavaScript the `.js` class hides it visually (it stays for screen readers and search engines) and the script builds `aria-hidden` glyph spans in `.letters`.
- `measure()` reads each letter's advance from a hidden `.measure` span using `Range` rects and creates enough copies of the sentence to fill the viewport. It reruns on resize and when fonts finish loading (`dirty` flag).
- `frame()` runs every animation frame: it sums three sines into the base line, adds the pointer pull (a spring) and tap ripples, samples that line every 6px into `Y` (height) and `S` (running arc length), then places each letter by arc length with a binary search over `S`, so letter spacing stays even however the line bends.
- `flow` is the scroll offset along the line (steady drift plus `fling` from flicks and the wheel). `energy` rises with pointer movement and makes the water choppier.
- Motion: `calm` follows `prefers-reduced-motion` until the visitor uses the play / pause button (`chosen`). `live` eases between 0 and 1 so pausing settles instead of cutting. The button is `hidden` in the markup and shown by the script.
- Tunables sit at the top of the script: `DIRECTION`, `SPEED`, `WAVE_HEIGHT`.

## `404.html`

GitHub Pages serves it for any unknown path, at any depth, so every URL in it must start at the root. A "404" drifts on a 45 degree path and turns back at every edge, like the logo on an idle DVD player, and each bounce moves the page on to the next palette pair (at most two changes a second). It starts on the fallback pair, which the CSS paints by `prefers-color-scheme`, so nothing flashes on load. The play / pause button and the `calm` / `chosen` / `live` logic are copied from the front page; with reduced motion or without JavaScript the number sits still in the middle.

## Things that must stay in sync

- **The sentence**: `#sentence` in `index.html`, the quote in `llms.txt`, and the meta description, Open Graph, Twitter and JSON-LD descriptions. `og-image.png` shows its opening words.
- **The fallback colours** (`#2B35E8` on `#F6F1E4`, the first palette): `:root` in the CSS, `theme-color` meta, `favicon.svg`, `site.webmanifest`, `404.html`, and the PNG icons and `og-image.png`.
- **The palette list**: `palettes` in the head script of `index.html` and in the script of `404.html`.
- **The favicon letter path** is duplicated in `favicon.svg` and inside `paint()` in the palette script.
- **Dates**: on a content change, update `dateModified` in the JSON-LD and `lastmod` in `sitemap.xml`.
- **`llms.txt`** describes how the site behaves (no framework, no tracking, no cookies, pausable, colour changes each visit). Update it if any of that stops being true.

## Conventions

- JavaScript is ES5-style inside an IIFE: `var`, function declarations, no arrow functions, with feature checks and `try`/`catch` around optional APIs (`localStorage`, `matchMedia`, pointer capture). Match it.
- Comments explain intent in plain sentences ("one finger plays with the wave, two fingers still zoom"). Keep that voice.
- Icon-only controls carry their name as `aria-label` or visually hidden text, and are 44px touch targets.
- Movement on the controls (scale, blur) is wrapped in `@media (prefers-reduced-motion: no-preference)`. Only the opacity cross-fade runs without it.
- Keep the print styles and the no-JavaScript fallback working when changing markup or CSS.
