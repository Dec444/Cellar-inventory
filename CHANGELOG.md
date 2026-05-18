# Changelog

All notable changes to this project will be documented here. Format loosely follows [Keep a Changelog](https://keepachangelog.com/).

## [0.1.0] — Initial release

### Added
- Single-file React artifact (`cellar.jsx`) for personal wine inventory tracking.
- Hero painting: Honthorst's *The Merry Violinist with a Glass of Wine* (c. 1623, public domain), background removed and embedded as base64 WEBP.
- Dynamic SVG glass overlay with two layers — multiply-blended wine fill clipped to a tapered cone matching the painted goblet, plus a separate normal-blended layer for the count number.
- Bottle count displayed live inside the violinist's raised goblet in DM Serif Display.
- Schema for bottles: core fields (name, producer, vintage, region, country, color, qty), optional enrichment (price, score, tasting profile, drinking window, pairings, occasions), and personal fields (rating, notes, acquired-from).
- Persistent storage via the artifact storage API (key: `cellar:v2`).
- Photo-to-inventory workflow: Claude reads bottles from a cabinet photo and writes them into the cellar.
- On-demand enrichment via `window.sendPrompt` — Claude web-searches and fills the enrichment object on a bottle when the user taps the button.
- `SKILL.md` teaching Claude when to trigger and how to deploy the artifact.

[0.1.0]: https://github.com/REPLACE-WITH-YOUR-USERNAME/cellar-inventory/releases/tag/v0.1.0
