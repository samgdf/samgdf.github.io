# AGENTS.md

This file provides guidance to agents and LLM's when working with code in this repository.

## Repository overview

This is a static GitHub Pages site for Sam Goldfield: an interactive Game Boy–style portfolio page originally created in 2018. The site is implemented as a single `index.html` with inline JavaScript and a single stylesheet `css/style.css`, plus font assets and favicon.

Key files:
- `index.html`: HTML document, SEO/meta tags, JSON-LD person schema, Game Boy markup, and inline JavaScript for button/keyboard interaction.
- `css/style.css`: All layout and styling for the Game Boy UI and surrounding text, including responsive media queries.
- `font/Alert.ttf`, `font/PublicoText.woff`: Custom fonts used by the page.

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
  - Links to `favicon.ico` and the main stylesheet `css/style.css`.

- **Body content**
  - Two introductory text blocks (`<h4>` and `<h3>`) explaining the portfolio, with links to GitHub and a set of favorite quotes.
  - `.gameboy` container with nested `.wrapper`, `.gameboy__screen`, `.gameboy__actions`, `.gameboy__arrows`, and `#options` elements forming the visual Game Boy interface.
  - `#text__box` is the main screen area that displays a rotating set of messages.
  - A and B buttons (`#next_button`, `#prev_button`) and arrow buttons (`#up_button`, `#down_button`, `#left_button`, `#right_button`) are clickable divs styled via CSS.
  - `SELECT` and `START` buttons are links to LinkedIn and an email `mailto:` link respectively.

- **Inline JavaScript behavior**
  - Defines an array `arr` of messages shown on the Game Boy screen.
  - Maintains a current index `i` and exposes two functions:
    - `nextItem()`: increments `i` modulo `arr.length` and returns the next message.
    - `prevItem()`: decrements `i` with wraparound and returns the previous message.
  - On `window.load`:
    - Attaches click listeners to `#next_button` and `#right_button` and `#down_button` to advance messages.
    - Attaches click listeners to `#prev_button` and `#left_button` and `#up_button` to go backwards.
    - Initializes `#text__box` with the first message (`arr[0]`).
  - Global `document.body.onkeyup` handler:
    - A / Right / Down arrow keys advance messages.
    - B / Left / Up arrow keys go backwards.

Future agents editing behavior should respect these bindings: changes to button IDs, structure, or key bindings must be mirrored consistently in both HTML and JavaScript.

### Styling and layout (`css/style.css`)

- Global styles:
  - `body` background color and adjustments for mobile via media queries.
  - Custom fonts `Alert` and `Publico` loaded via `@font-face` from the `font/` directory.
  - Typography for `h2`, `h3`, `h4`, `p`, and `<i>` elements.

- **Game Boy UI components**
  - `.gameboy` and `.wrapper` define the outer shell, dimensions, margin, and base layout.
  - `.gameboy__screen` styles the screen area with greenish background, inset shadow, border framing, and small decorative element via `::after`.
  - `.gameboy__actions` and `.action` classes style the A/B buttons as circular, pressable buttons with shadows and active state.
  - `#options` and nested `.btn` elements style the `SELECT` and `START` buttons, including rotated orientation.
  - `.gameboy__arrows` and `.gameboy__arrow` (+ modifier classes like `--up`, `--down`, `--left`, `--right`, `--center`) define the D-pad; arrows are drawn using `::before`/`::after` pseudo-elements.

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
