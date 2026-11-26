# AGENTS.md

This file provides guidance to agents and LLM's when working with code in this repository.

## Repository overview

This is a static GitHub Pages site for Sam Goldfield: an interactive Game Boy–style portfolio page originally created in 2018. The site is implemented as a single `index.html` with inline JavaScript and a single stylesheet `css/style.css`, plus font assets and favicon.

Key files:
- `index.html`: HTML document, SEO/meta tags, JSON-LD person schema, Game Boy markup, and inline JavaScript for button/keyboard interaction.
- `css/style.css`: All layout and styling for the Game Boy UI and surrounding text, including responsive media queries.
- `font/Alert.ttf`, `font/PublicoText.woff`: Custom fonts used by the page.
- `favicon.ico`, `favicon-night.ico`, `favicon-pixel.png`: Favicons (day, night, and a pixel-art version).
- `CNAME`: Custom domain for GitHub Pages (`samgdf.com`).
- `keybase.txt`: Keybase domain-ownership proof; leave untouched.

There is no build system, bundler, or JavaScript package manager in use; the site is served directly as static assets by GitHub Pages.

## How to run and develop locally

Because this is a pure static site, you can use any simple HTTP server to test changes locally.

Recommended approaches:
- Open `index.html` directly in a browser for a quick check (note: some browser security settings may restrict font loading or future additions like AJAX; use an HTTP server for the most accurate behavior).
- Run a simple local server from the repo root:
  - Using Python 3:
    - `python3 -m http.server 8000`
    - Then open `http://localhost:8000/` in a browser.
  - Using Node (if installed globally):
    - `npx serve .`

There is no automated test or lint command configured in this repository at present.

## Code structure and behavior

### HTML structure (`index.html`)

- **Head section**
  - Standard meta tags for charset, viewport, theme color, and mobile web app capabilities.
  - Open Graph and description meta tags pointing to `https://www.samgdf.com`.
  - JSON-LD `<script type="application/ld+json">` block defining a `Person` schema with name, job title, employer, and social profiles.
  - Links to `favicon.ico` (swapped to `favicon-night.ico` at night), a preload for `font/Alert.ttf`, and the main stylesheet `css/style.css`.
  - An inline `<style>` block with the game-specific styles: the `.gameboy__screen.game` canvas screen (260px, exact 2x pixel scale, padding removed), the `#dialog` text box at the bottom of the screen (Alert font, typewriter "done" arrow), and `h3 i { font-size: inherit }` so key caps match the side text.

- **Body content**
  - `<h3>` (left): intro and instructions, then the SELECT → LinkedIn and START → GitHub lines. `<h4>` (right): favorite quotes. Both separate paragraphs with `<br><br>`; key names (A, B, SELECT, START) use `<i>` key caps.
  - `.gameboy` container with nested `.wrapper`, `.gameboy__screen.game`, `.gameboy__actions`, `.gameboy__arrows`, and `#options` elements forming the visual Game Boy interface.
  - The screen holds `<canvas id="game">` (130×130 game pixels, shown at 260×260) and the `#dialog` message box.
  - A/B buttons (`.action`) carry `data-btn="a|b"`; D-pad arrows (`.gameboy__arrow--*`) carry `data-dir="up|down|left|right"`.
  - `SELECT` and `START` are `<a class="btn">` links to LinkedIn and GitHub respectively.

- **Inline JavaScript: a small tile-based adventure game** (script at the end of `<body>`)
  - **Sprites** are 16×16 string grids drawn by `makeSprite(rows, mirror)` using the `PAL` color letters; white touching the border becomes transparent. The player (`SPRITES`, from `favicon.ico`) has idle and walk frames per direction; also `WIFE` (four directions) and `GIRL` (sitting, seen from behind). Small items (`ITEMS`: golden key, silver key, remote) are drawn with `art()`.
  - **Rooms** live in `ROOMS` (`office`, `hall`, `living`, `bath`, `outside`). Each has a `floor` style, a `map` of tile characters (legend in the comment above `ROOMS`), and `exits` keyed by tile character, returning the destination room and position. Only the current room is drawn: `enterRoom()` swaps the map and `buildWorld()` redraws it into an offscreen canvas (also called whenever `state` changes something visible).
  - **State**: `state` holds door/item/TV flags, `pocket` holds at most one item, and `follower` is that item trailing one tile behind the player (it takes the tile the player just left, so walking into it swaps places). `solid()` handles collisions, including state-dependent doors and the NPCs from `npcs()`.
  - **Movement**: one tile per step (`tryStep`, `STEP_MS`); holding a direction keeps walking. The camera centers on the player and clamps at the room edges (rooms smaller than the screen are centered), and nudges down while a message is open so the player stays above the text box.
  - **Interaction**: A calls `interact()`, which checks the NPC or tile the player faces and calls `say(text)` (typewriter text in `#dialog`). A/B finish or close a message; arrow keys, the D-pad, or clicking/tapping the screen close it immediately.
  - **Input**: arrow keys, A (also Enter/Space), B (also Escape), plus pointer events on the D-pad and A/B buttons.

To add a room, add an entry to `ROOMS` and an exit pointing to it; to add something to interact with, give it a tile character, draw it in `buildWorld()`, add it to `SOLID` if it blocks, and add a `case` in `interact()`.

- **Night mode**: an inline script in `<head>` adds `.night` to `<html>` from 18:00 to 06:00 local time (re-scheduling itself for the next switch) and swaps the favicon to `favicon-night.ico` (sleeping sprite). In `css/style.css`, page colors are the `--bg`/`--fg` custom properties; `.night` swaps them inside `@media (min-width: 481px)`, so the Game Boy itself and the full-screen phone layout are unchanged.

### Styling and layout (`css/style.css`)

- Global styles:
  - `body` background color and adjustments for mobile via media queries.
  - Custom fonts `Alert` and `Publico` loaded via `@font-face` from the `font/` directory.
  - Typography for the `h3`/`h4` side columns and `<i>` key caps.

- **Game Boy UI components**
  - `.gameboy` and `.wrapper` define the outer shell, dimensions, margin, and base layout.
  - `.gameboy__screen` styles the screen area with greenish background, inset shadow, border framing, and small decorative element via `::after`.
  - `.gameboy__actions` and `.action` classes style the A/B buttons as circular, pressable buttons with shadows and active state.
  - `#options` and nested `.btn` elements style the `SELECT` and `START` buttons, including rotated orientation.
  - `.gameboy__arrows` and `.gameboy__arrow` (+ modifier classes like `--up`, `--down`, `--left`, `--right`, `--center`) define the D-pad; each arrow is two stacked CSS triangles (`::before` highlight, `::after` shadow) colored via a `--c` custom property.

- **Responsive behavior**
  - At high resolutions (`min-width: 1824px`): text gets larger and the Game Boy screen font size increases.
  - At medium widths (`max-width: 1287px`): the side text columns (`h3`, `h4`) shrink.
  - At narrow widths (`max-width: 976px` and `max-width: 480px`): side text is hidden, background and Game Boy container adjust to present a mobile-friendly layout.

Future layout changes should consider the existing responsive breakpoints and preserve legibility on both desktop and mobile.

## Conventions and guidance for future changes

- Keep the site lightweight and static: avoid introducing heavy build tooling unless necessary. If a build system is added, update this file with new build/test commands.
- If you refactor the inline JavaScript into separate files or modules, document the new structure here so future agents understand where interaction logic lives.
- When adding new interactive elements, follow the existing pattern of pairing DOM elements with event listeners and ensure keyboard accessibility (e.g., support both clicks and key presses where appropriate).
- Preserve or update SEO and structured data (JSON-LD) in `index.html` when changing personal details like job title or employer.
