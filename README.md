# Cellar Inventory

A personal wine inventory app, built as a Claude artifact, themed around Gerrit van Honthorst's *The Merry Violinist with a Glass of Wine* (c. 1623). Your bottle count lives inside the violinist's raised goblet — wine fills the glass as your collection grows.

![A 17th-century painting by Honthorst of a smiling violinist raising a glass; the count of wine bottles in your cellar appears inside the glass.](https://upload.wikimedia.org/wikipedia/commons/9/91/Gerard_van_Honthorst_-_The_Happy_Violinist_with_a_Wine_Glass_-_WGA11653.jpg)

> *Replace the image above with a screenshot of the running artifact once you've taken one.*

## What this is

A single-file React artifact (`cellar.jsx`) plus a Claude skill (`SKILL.md`) that teaches Claude how to deploy and extend it. Together they give you:

- **A wine cellar tracker** that runs as an artifact inside Claude.ai. No server, no signup. Data persists across conversations via the artifact storage API.
- **Photo-to-inventory:** drop a photo of your cabinet into chat, and Claude reads the bottles into your cellar.
- **On-demand enrichment:** tap a bottle to get pricing, critic scores, tasting profile, drinking window, food pairings, and occasion suggestions — fetched live from the web by Claude.
- **A dynamic painting:** Honthorst's violinist is the hero, his goblet showing your live bottle count in cream gold over dark wine.

## What's in this repo

```
.
├── cellar.jsx     The artifact — a single React file, self-contained
├── SKILL.md       Claude skill instructions for deploying and using it
└── README.md      You are here
```

The `cellar.jsx` file embeds the painting as a base64 WEBP with the background already removed (via OpenCV GrabCut). This makes the artifact fully portable: no external image dependencies, works in any sandboxed environment, never breaks because a CDN went down.

## How to use it

### Option A: as a Claude skill (recommended)

If you have Claude with skills enabled:

1. Place this folder where Claude can read it as a skill (the location depends on your Claude setup; check `https://docs.claude.com` for the current path).
2. In Claude, say something like *"I want to start tracking my wine cellar"* or *"help me catalog my wines"*. The skill triggers automatically.
3. Claude deploys the artifact and you start adding bottles.

### Option B: paste into a Claude.ai conversation

If you just want to try it without setting up a skill:

1. Copy the contents of `cellar.jsx`.
2. In Claude.ai, ask Claude to create a React artifact and paste the code in as the contents. (Or upload `cellar.jsx` and ask Claude to render it as an artifact.)
3. The artifact opens. Start adding bottles.

### Option C: adapt for your own app

The file is a vanilla React component with `lucide-react` icons. To run it outside Claude:

1. Replace `window.storage` calls with whatever persistence you want (`localStorage`, IndexedDB, a backend).
2. Replace the `window.sendPrompt` callback (used for enrichment) with calls to your own LLM endpoint.
3. The painting is embedded as base64 — no asset pipeline changes needed.

## Architecture

- **Storage key:** `cellar:v2`. Schema documented in `SKILL.md`.
- **Currency:** USD throughout.
- **Tech:** React (hooks), `lucide-react` for icons, Google Fonts (DM Serif Display for the count, Fraunces for headings, Inter for UI), inline SVG for the dynamic glass overlay.
- **Glass overlay:** two stacked SVG layers — wine fill with `mix-blend-mode: multiply` so it integrates with the painted goblet's highlights, and the count number on top with normal blending so it stays crisp. The wine is clipped to a tapered cone path matching the goblet's actual shape.
- **Bottle schema:** core fields are auto-extracted (name, producer, vintage, region, country, color, qty), enrichment fields are fetched on demand (price, score, tasting profile, drinking window, pairings, occasions), personal fields are user-filled (rating, notes, acquired-from).

## Attribution

The hero painting is *The Merry Violinist with a Glass of Wine* by Gerrit van Honthorst, painted around 1623, currently in the [Thyssen-Bornemisza Museum](https://www.museothyssen.org/) in Madrid. The work is in the **public domain** (the artist died in 1656). The image was sourced from Wikimedia Commons.

The background of the embedded version was removed using OpenCV's GrabCut algorithm with region-seeded foreground hints, so the figure composites cleanly onto the page rather than appearing in the painting's original brown ground.

## License

MIT. See `LICENSE` if you add one; otherwise, MIT is the default for this project.

You're free to copy, modify, and redistribute the code. Attribution is appreciated but not required.

## Contributing

This is a small personal project that's open-sourced as a reference / template. PRs welcome but the maintainer may not respond quickly. If you build something interesting on top of this, drop a link in the issues — I'd love to see it.

## Acknowledgments

- Gerrit van Honthorst, for painting a man this happy about wine four hundred years ago.
- The Thyssen-Bornemisza Museum, for stewarding the painting.
- Anthropic, for the Claude artifact runtime that makes a self-contained client-side app like this possible.
