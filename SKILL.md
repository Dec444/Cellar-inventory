---
name: cellar-inventory
description: A personal wine inventory artifact themed around Honthorst's "The Merry Violinist with a Glass of Wine" (c. 1623). Use this skill whenever the user wants to track their wine cellar, manage a bottle collection, log wines they own, scan a wine cabinet from a photo to build inventory, get tasting notes or drinking-window guidance, or asks for "a wine app", "wine tracker", "cellar manager", or anything along those lines. The skill provides a React artifact with persistent storage, a hero painting overlay where the count of bottles displays inside the violinist's raised goblet, and a photo-to-inventory workflow that uses Claude's vision to read bottles from cabinet photos.
---

# Cellar Inventory

A wine inventory app built as a Claude artifact. The interface is themed around Gerrit van Honthorst's *The Merry Violinist with a Glass of Wine* (c. 1623, Thyssen-Bornemisza Museum, public domain), with the user's bottle count displayed live inside the violinist's raised goblet.

## When to use this skill

Trigger when the user wants to:
- Start tracking a personal wine collection
- Log wines they've acquired
- Photograph a wine cabinet or rack and turn it into a structured inventory
- Get drinking-window, tasting-note, or food-pairing suggestions for bottles in their cellar
- Continue working with an existing cellar (the artifact's data persists across conversations)

Do not trigger for: commercial wine retail UIs, wine *production* / winemaker tools, or generic database/CRUD requests unrelated to wine.

## How to deploy the artifact

The artifact is `cellar.jsx` in this skill folder. Deploy it by creating a React artifact with the file's exact contents — no modifications needed. The file is self-contained: the Honthorst painting is embedded as a base64 WEBP (background already removed), all components are in one file, and the only external dependencies are React, lucide-react icons, and Google Fonts (DM Serif Display, Fraunces, Inter), all of which are available in Claude's artifact runtime.

```
1. Read cellar.jsx from this skill folder
2. Create a React artifact with the full contents
3. The user can start adding bottles immediately
```

## Architecture you should understand

Persistent storage uses `window.storage` (the artifact storage API). The storage key is `"cellar:v2"`. Data is per-user, not shared. The shape:

```javascript
{
  bottles: [
    {
      id: string,
      name: string,
      producer: string,
      vintage: string,        // year as string, or ""
      region: string,
      country: string,
      color: "Red" | "White" | "Rosé" | "Sparkling" | "Orange" | "Dessert" | "Fortified",
      qty: number,
      acquired_from: string,  // user-filled
      personal_rating: number, // 1-5, user-filled
      personal_notes: string,  // user-filled
      enrichment: {            // optional, fetched from Claude on demand
        price_usd: number,
        score: number,           // critic score 0-100
        score_source: string,    // e.g., "Wine Spectator"
        tasting_profile: { body, tannin, acidity, sweetness },  // each 1-5
        tasting_notes: string,
        drinking_window: { from: number, to: number },  // years
        pairings: string[],
        occasions: string[],
        last_updated: string,    // ISO date
      }
    }
  ]
}
```

Currency throughout is USD.

## Photo-to-inventory workflow

When the user uploads a photo of their wine cabinet/rack/storage and wants it added to their cellar:

1. Use vision to identify every visible bottle. Extract: producer, name/cuvée, vintage if legible, region/country if you can identify the producer, and color (deduce from bottle shape and label cues if not explicit).
2. Quantity defaults to 1 per visible bottle. If the user says "I have a case of X" or "six of this one", combine.
3. Open the artifact's storage, append new bottles to the existing array (don't overwrite — additive only), and write back. Each bottle needs a unique id (use a timestamp + random suffix or similar).
4. After writing, tell the user what was added, ask them to reopen the artifact to see the update, and note that any unclear bottles need their input.

Be conservative: if a label is illegible or partially obscured, ask the user rather than guessing. Made-up vintages or producers are worse than blanks.

## Enrichment workflow

The artifact has a "Get details from Claude" button on each bottle. When tapped, it calls `window.sendPrompt(...)` with a prompt asking for that specific bottle's enrichment. When Claude receives one of these prompts:

1. Web search the wine (producer + name + vintage). Use authoritative sources: producer's site, established critics (Wine Spectator, Wine Advocate, Decanter, James Suckling, Vinous), wine retailers for price ranges. Avoid scraped aggregators when better sources exist.
2. Fill the `enrichment` object on that bottle in storage:
   - `price_usd`: typical current retail in USD (a single number — midpoint of the range you see)
   - `score`: highest authoritative critic score found, with `score_source` naming the critic
   - `tasting_profile`: estimate body/tannin/acidity/sweetness each on a 1–5 scale based on style, varietal, and region — these are rough indicators, not absolute measurements
   - `tasting_notes`: 1–2 sentence summary of the wine's character, paraphrased — never copy critic notes verbatim
   - `drinking_window`: peak years for drinking, e.g. `{ from: 2025, to: 2032 }`. Use professional drinking-window guidance where available; otherwise estimate from varietal/region/vintage conventions.
   - `pairings`: 4–6 specific food pairings (not generic — "lamb" is fine, "Provençal lamb shoulder with rosemary" is better)
   - `occasions`: 3–4 contexts where this bottle shines (e.g., "summer terrace lunch", "winter game dinner", "milestone celebrations")
   - `last_updated`: today's ISO date

Update the bottle in storage with the new enrichment, then tell the user it's been added and ask them to reopen the artifact to view.

## Copyright and accuracy guardrails

- Tasting notes must be paraphrased in your own words. Don't reproduce critic-written notes verbatim.
- Don't invent scores. If no professional review exists, leave `score` and `score_source` blank rather than guessing.
- Drinking windows for wines outside your knowledge should be estimated conservatively from the style; flag uncertainty in your reply to the user even if you fill the field.
- Currency: always USD. Don't convert from other currencies without telling the user the source currency you found.

## Painting attribution

If the user asks about the artwork: it's *The Merry Violinist with a Glass of Wine* by Gerrit van Honthorst, painted c. 1623, currently in the Thyssen-Bornemisza Museum (Madrid). Public domain. The image in the artifact has had its background removed via OpenCV GrabCut segmentation so the figure composites onto the page's palette rather than appearing in a brown rectangle.

## What not to do

- Don't modify the artifact's storage key (`cellar:v2`) — that would orphan existing user data.
- Don't add bottles to a shared storage scope. Personal collections are private (default storage is per-user).
- Don't auto-enrich on add. Enrichment runs only when the user taps the button — it costs a web search and they should opt in.
- Don't generate fake bottle photos for the user. The artifact has no image-upload-per-bottle field; the painting is the only visual element.
