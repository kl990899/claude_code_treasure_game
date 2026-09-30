# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A single-page React 18 + TypeScript "Treasure Hunt" game built with Vite (exported from Figma Make). Three chests are shown; one randomly holds treasure (+$100), the others hold skeletons (-$50). The game ends when the treasure is found or all chests are opened.

## Commands

- `npm install` — install dependencies
- `npm run dev` — start the Vite dev server on port 3000 (opens the browser automatically)
- `npm run build` — production build to `build/` (not `dist/`)

There is no test runner, linter, or `tsconfig.json` configured. Vite uses SWC (`@vitejs/plugin-react-swc`), which strips types without type-checking, so type errors will not fail the build.

## Architecture

- All game logic and UI live in [src/App.tsx](src/App.tsx): `Box[]` state (`id`, `isOpen`, `hasTreasure`), `score`, and `gameEnded`. `initializeGame()` runs on mount and on "Play Again"; `openBox()` updates the score and checks end-of-game conditions inside the `setBoxes` updater.
- Animations use `motion/react` (Framer Motion's successor), not `framer-motion`.
- Images (`src/assets/*.png`) and sounds (`src/audios/*.mp3`) are imported as ES modules and passed to `src`/`Audio`. `src/audios/chest_open_with_evil_laugh.mp3` is meant for skeleton chests, and `src/assets/key.png` is meant as a custom cursor over closed chests.
- `src/components/ui/` holds generated shadcn/ui components (Radix + `class-variance-authority`). Only `Button` is used right now. `cn()` lives in `src/components/ui/utils.ts`.
- `vite.config.ts` aliases versioned import specifiers (for example `'sonner@2.0.3'`) to bare package names because the Figma-generated UI components import them that way. It also maps `@` to `./src`. If you add a component that imports a versioned specifier, add a matching alias.

## Styling gotcha

`src/index.css` is a **precompiled Tailwind v4 output**. Tailwind is not installed and there is no PostCSS or Tailwind build step. Only utility classes that already appear in `src/index.css` will work. A new Tailwind class (for example a `cursor-[url(...)]` arbitrary value or a color shade that isn't used yet) will silently do nothing. When you need a new style, use inline `style` props or add plain CSS rules to `src/index.css`. `src/styles/globals.css` contains the Tailwind source/theme tokens, but nothing imports it.

## Other notes

- `src/guidelines/Guidelines.md` is an empty Figma Make template with no project rules.
- The sources of the sound and image assets are credited in `Attributions.md`.
- `README.md` is a Claude Code tutorial script for this repo (adding sounds, a win/tie/loss result display, a key-cursor hover, SQLite sign-up/sign-in with a guest mode, and Vercel/GitHub Pages deploy commands under `.claude/commands/`). It describes planned exercises, not features that already exist.
