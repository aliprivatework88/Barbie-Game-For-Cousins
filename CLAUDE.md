# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A single-file, zero-dependency browser game: a Barbie-style Hair Studio avatar creator. The entire app lives in `index.html` — HTML, CSS, and JavaScript are all embedded in that one file. There is no build step, no package manager, and no test suite. Open the file directly in a browser to run it.

## Running / Testing

```bash
# Open in browser (Linux)
xdg-open index.html

# Or serve locally to avoid any file:// quirks
python3 -m http.server 8080
# then visit http://localhost:8080
```

There are no automated tests. Verification is done by opening the page and exercising the UI manually.

## Architecture

Everything runs inside a single IIFE (`(() => { ... })()`). The structure within it follows this order:

### 1. State (`state` object)
Single source of truth for all mutable values: `face`, `skin`, `hairStyle`, `hairColor`, `accessory` (type/color/side), and `label` (show/text/color/size). Every user interaction updates `state` then calls `draw()`.

### 2. Constant option arrays
Defined directly below `state` — `FACE_OPTIONS`, `SKIN_TONES`, `HAIR_STYLES`, `HAIR_COLORS`, `ACCESSORY_TYPES`, `ACC_COLORS`, `NAME_COLORS`. Adding a new option means appending to the relevant array and ensuring the drawing functions handle its key.

### 3. UI builders
Dynamic DOM construction: buttons and color swatches are created programmatically from the option arrays, not hard-coded in HTML. Event listeners call `refreshSelections()` + `draw()`. The HTML contains only empty container `<div id="faces">`, `<div id="styles">` etc. as mounting points.

### 4. `refreshSelections()`
Syncs the `.sel` CSS class across all buttons and swatches to match current `state`. Must be called after any state change so the UI reflects the selection visually.

### 5. `draw()` — the rendering pipeline
Called on every state change. Draws to `<canvas id="avatar">` (720×900 px) in this fixed order:
1. Background gradient + `sparkle()` decoration
2. Ground shadow ellipse
3. `hairBack()` — back-layer hair (long, ponytail, bun, braids)
4. Neck + shoulders (skin-colored shapes)
5. `drawHead()` — face shape (round/oval/heart) + ears
6. `drawFace()` — cheeks, eyes, nose, mouth
7. `hairFront()` — front-layer hair + shine overlay
8. `drawAccessory()` — bow, headband, heart-clip, or star-clip
9. Name label bubble (if `state.label.show && state.label.text`)

The back/front split for hair is intentional: some styles need hair drawn behind the face, others in front.

### 6. Color utilities
`shade(hex, amt)` — lightens (positive) or darkens (negative) a hex color by blending toward white/black.  
`hexToRgb` / `rgbToHex` — conversion helpers used by `refreshSelections()` to match swatch background colors against `state.skin`.

### 7. Action buttons
Random, Reset, Save PNG — wired up at the bottom. Save uses `canvas.toDataURL('image/png')` and a synthetic `<a>` click; filename is `{name}-avatar.png` if a name is set.

## Key Conventions

- All canvas coordinates are expressed as fractions of `headW`/`headH` (e.g., `x - w*0.44`) so shapes scale relative to the head size, which varies by face shape.
- When adding a new hair style, implement it in **both** `hairBack()` and `hairFront()` (one or both may be no-ops for that style, but the `if(style==='...')` guard must exist in both functions).
- CSS variables are defined in `:root` — use them (`var(--accent)`, `var(--line)`, etc.) for any new UI chrome rather than hardcoding colours.
- The random name pool in `randomBtn.onclick` (`['Aisha','Sara','Maya',...]`) is intentionally culturally diverse; keep it that way when extending.
