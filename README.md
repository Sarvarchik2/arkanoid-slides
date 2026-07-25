# Arkanoid — Interactive Presentation (Slidev)

An interactive presentation about my Arkanoid game built with Python/pygame.
Homework (Option 2: Interactive AI Presentation).

## Run

```bash
npm install
npm run dev
```

Opens in the browser at http://localhost:3030

## Controls

- `space` / `→` — next slide or next animation step
- `←` — previous slide
- click items in the Contents slide to jump to a slide
- slide 11 — playable mini-game (mouse moves the paddle, click to start/pause)

## Interactive elements

1. A playable mini-version of Arkanoid embedded right in a slide (canvas + Vue)
2. Clickable table of contents and jump buttons between slides
3. Step-by-step reveal animations (v-click) and animated code highlighting
4. Animated transitions between slides
5. Clickable links to the project's GitHub repository

## PDF version

A static PDF export is included: `arkanoid-presentation.pdf`
(note: interactive elements only work in the browser version).

To re-export: `npm run export`
