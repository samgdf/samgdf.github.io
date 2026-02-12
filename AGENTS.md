# AGENTS.md

This file provides guidance to agents and LLM's when working with code in this repository.

## Repository overview

This directory contains a standalone, static HTML experience: an interactive landscape scene with sky, drifting clouds, tall grass, and a running river. The scene exposes two primary calls to action (LinkedIn and GitHub links) and implements a dynamic favicon that reflects user interactions.

There is a single entry point `index.html` that includes:
- `<head>` with SEO/meta tags, Open Graph data, and JSON-LD `Person` schema for Sam Goldfield.
- Inline `<style>` defining all layout, animation, and visual styling.
- Inline `<script>` implementing all interactivity (favicon rendering, keyboard/hover/mouse behavior, scene animation).

No bundler, package manager, or external assets are referenced from within this directory; it is served as static HTML.

## How to run and develop locally

From the `new` directory you can either:
- Open `index.html` directly in a browser for very fast iteration.
- Or, for behavior that more closely matches production hosting, serve it via a simple HTTP server from the parent repo root and navigate to `/new/`:
  - Using Python 3 from the repo root:
    - `python3 -m http.server 8000`
    - Open `http://localhost:8000/new/` in your browser.

There are no configured build, lint, or test commands for this directory.

## High-level architecture and behavior (`index.html`)

### Head and metadata

- Standard meta viewport and mobile web app tags.
- Theme and status bar colors tuned to the sky color (`#87CEEB`).
- Open Graph tags and `description` reflecting the interactive landscape and external links.
- JSON-LD `Person` schema containing name, URL, job title, employer, and social profiles. If personal details change (e.g., job title or employer), keep this JSON-LD block in sync.

### Layout and styling (inline `<style>`)

- Global reset (`*`) removes margins/padding and hides scrollbars via `overflow: hidden` to keep the scene full-viewport.
- `body` is a full-viewport gradient background: sky → distant landscape → foreground grass.
- **Cloud system**
  - `.cloud` elements with `::before`/`::after` pseudo-elements approximate cloud shapes.
  - `@keyframes float-cloud` moves clouds horizontally across the viewport while fading them in/out.
- **Water / river**
  - `#water` is a horizontal band near the bottom of the viewport, with gradient fill mimicking water.
  - `.water-wave` elements animate via `@keyframes flow` to create moving light streaks.
- **Links and reflections**
  - `#links-container` is absolutely centered and contains the two main anchors (LinkedIn and GitHub).
  - Link elements have fade-out animations unless hovered; CSS classes (`hovered`, `no-animation`, `faded`) control visibility and timing.
  - `.reflection` elements mirror each link’s text, vertically flipped with blur and reduced opacity to simulate reflections in the river.
  - `.hover-zone` invisible divs extend the hover area vertically above each link, so moving through the sky can trigger link hover behavior.
- **Grass system**
  - `#grass-container` overlays the scene and holds multiple `.grass-blade` elements.
  - Each blade is a thin rectangle with a vertical gradient and transform origin at its base to support swaying.

### JavaScript structure and responsibilities

All interaction logic is implemented in one inline `<script>` block. Key subsystems:

#### Dynamic favicon system

- Uses a 32×32 `<canvas>` to draw favicon frames, referenced by the existing `<link rel="icon">` tag.
- `updateFavicon(state, animationFrame)` renders different scenes depending on `state`:
  - `default`: sky, small cloud, water, ground, and static grass blades.
  - `grass-moving`: emphasizes animated grass and a subtle wind indicator.
  - `water-ripple`: focuses on water with an expanding ripple.
  - `link-hover`: highlights a glowing dot and simple text lines to indicate active links.
- `setFaviconState(state, duration)` animates the favicon in the given state for `duration` ms, then returns to `default`. It uses an interval to increment `faviconAnimationFrame` and re-render.

When modifying favicon behavior, keep the contract that `state` is a small set of string identifiers and that unknown states fall back to the default rendering.

#### Scene setup

- DOM references: `grass-container`, `clouds-container`, and the global `clouds` and `grassBlades` arrays.
- `createClouds()`
  - Creates a fixed number of `.cloud` elements with randomized sizes, vertical positions, and animation durations/delays.
  - Appends clouds to `#clouds-container` and registers them via `trackCloud`.
- `createGrass()`
  - Computes `numBlades` based on viewport width.
  - For each blade, randomizes horizontal position, height, base rotation, and z-index, and stores per-blade metadata (`baseRotation`, `offset`) in `grassBlades`.

If you adjust cloud/grass densities or visuals, preserve the separation between DOM elements and per-element metadata to keep animations manageable.

#### Cloud–link interaction (brightness)

- `updateCloudBrightness()`
  - Disabled on very narrow screens to avoid layout/measurement issues.
  - For each cloud with a visible bounding box, iterates over visible links in `#links-container`.
  - Computes center–center distance and adds a `brightened` class to the cloud when within a threshold distance of any visible link.

This system depends on:
- `clouds` array containing all active cloud elements.
- Visibility of links being encoded via CSS opacity; links with opacity ≤ 0.5 are treated as effectively hidden.

#### Keyboard-driven grass movement

- Uses a `keys` dictionary keyed by event `.key` values.
- `keydown` listener:
  - Tracks movement keys: arrow keys and WASD.
  - On first press of any movement key, calls `setFaviconState('grass-moving', 2000)`.
- `keyup` listener:
  - Clears the corresponding entry in `keys`.
- `updateGrass()` (called on every animation frame):
  - Computes `moveX`/`moveY` from current key state.
  - Advances `time` and calculates per-blade `windEffect` (sinusoidal) and `moveEffect` (driven by movement direction).
  - Updates each blade’s `transform` rotation accordingly.

If you add new keyboard interactions, be careful not to conflict with the existing movement keys and their use in `updateGrass()`.

#### Link reflections and extended hover behavior

- `createReflections()`
  - For each link in `#links-container`:
    - Creates a `.reflection` element with matching text and a `.hover-zone` element extending vertically above the link.
    - `updatePositions()` keeps reflection and hover-zone aligned to the link on resize.
  - Hovering either the link or its hover zone:
    - Adds `hovered`/`no-animation` classes, removes `faded`, and triggers `setFaviconState('link-hover', 3000)`.
  - On hover end, re-applies fading.

This subsystem tightly couples CSS classes (`hovered`, `faded`, `no-animation`) across both the main links and their reflections; if you refactor these names, update both CSS and JS consistently.

#### Water ripple interaction

- Click handler on `#water`:
  - Computes click coordinates relative to the water element.
  - Creates a `.ripple` element at that point, with a fixed size and CSS `ripple-animation` keyframe driving its expansion and fade out.
  - Calls `setFaviconState('water-ripple', 1500)` to sync the favicon animation with the ripple.
  - Removes the ripple element after the animation completes.

#### Animation loop and initialization

- `animate()` uses `requestAnimationFrame` to continuously:
  - Call `updateGrass()`.
  - Call `updateCloudBrightness()`.
- On initial load, the script calls:
  - `updateFavicon('default')` to seed the favicon.
  - `createClouds()`, `createGrass()`, `createReflections()`, and then `animate()`.

The main loop is intentionally simple; if you add more per-frame work, consider performance implications on low-power devices.

## Guidance for future changes

- If you split CSS or JavaScript out of `index.html` into separate files, update this document to describe the new structure and ensure any new build/serve commands are captured in the "How to run" section.
- When adding new interaction modes (e.g., additional keyboard shortcuts or mouse gestures), aim to reuse existing patterns:
  - Use small, named favicon `state` values routed through `setFaviconState`.
  - Keep one primary `requestAnimationFrame` loop and extend it rather than introducing multiple, competing loops.
- Maintain the alignment between visual elements (links, reflections, hover zones, clouds) and their JavaScript controllers by updating CSS class names and selectors in lockstep.
