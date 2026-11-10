# Repository Guidelines

## Project Structure & Module Organization

This repository is a minimal Svelte 5 / SvelteKit application for an Airbnb data visualization critique, with D3 available for charts.

- `src/routes/+page.svelte`: home page; currently contains starter content.
- `src/routes/+layout.svelte`: shared layout and favicon setup.
- `src/lib/`: reusable JavaScript, Svelte components, and imported assets. Use the configured `#lib` alias for imports.
- `static/`: public files, including `airbnb_boston_cambridge.csv`, served at `/airbnb_boston_cambridge.csv`.
- `airbnb_README.md`: dataset provenance, column definitions, and visualization considerations.
- `vite.config.js`: Vite, Svelte compiler, and deployment adapter configuration.

There is currently no test directory or automated test suite. Treat `.svelte-kit/`, `build/`, and `node_modules/` as generated files.

## Build, Test, and Development Commands

- `npm ci`: install dependencies from the committed `package-lock.json`.
- `npm run dev`: start the local development server.
- `npm run dev -- --open`: start development and open the app in a browser.
- `npm run build`: generate a production build and catch compilation errors.
- `npm run preview`: serve the production build locally after building.

No test, lint, or formatting scripts are configured.

## Coding Style & Naming Conventions

Follow existing files: tab indentation, single-quoted JavaScript strings, semicolons, and ES module imports. Keep SvelteKit route filenames such as `+page.svelte` and `+layout.svelte`. Use PascalCase for reusable Svelte components and camelCase for functions and variables. The compiler enables Svelte runes mode; use APIs such as `$props()` and `$state()` for component state and inputs.

## Testing Guidelines

Run `npm run build` before submitting changes, then inspect affected pages using the development server or production preview. For charts, verify numeric parsing, labels, tooltips, responsive layout, and categorical color distinctions. No testing framework or coverage threshold exists; document manual checks in the pull request.

## Commit & Pull Request Guidelines

The initial commit uses `scaffold: empty SvelteKit project + airbnb data`. Follow that concise, imperative pattern with a descriptive prefix, such as `feat: add price scatterplot`.

Pull requests should explain the change, include validation commands and results, link relevant issues when available, and provide screenshots for visual changes. Preserve dataset provenance and document any transformations; CSV numeric fields load as strings and need explicit conversion.

## Airbnb Critique: Project Context

### Perceptual Rules

- `room_type` is categorical (three unordered values). Use a categorical palette, never a sequential or continuous color ramp.
- Choose a categorical palette and verify it with color-vision simulations. Okabe–Ito and ColorBrewer palettes can provide starting points.
- If categories are difficult to distinguish under a color-vision simulation, revise the palette or add a redundant encoding, such as a distinct marker shape.
- Use position on a common scale for the main quantitative comparison. Check whether quantities encoded by area would be easier to compare on an axis.
- Keep each channel tied to a clear variable. Use redundant category encodings when needed for accessibility; remove encodings that distract from the task.
- Text, including axis labels and legends, must have at least 4.5:1 contrast against its background.
- Essential data marks must have at least 3:1 contrast against adjacent background colors through their fill or outline. Account for opacity when checking displayed colors.
- Set text color in CSS (e.g. `text { fill: #333; }`), rather than with a `fill` attribute on each SVG element, to make the color easier to inspect and edit. Use a contrast checker when DevTools does not provide a ratio.

### When the User Gives a Critique

- Use the user's observations and tool results as evidence when revising the rendered chart. If a result is unclear, ask for clarification or repeat the check.
- Make exactly the change requested; do not improve other things unasked.
- If asked whether a chart is colorblind-safe, report which checks you performed. Run the simulator if you have browser access; otherwise state that you cannot run it so the user can perform the check.
