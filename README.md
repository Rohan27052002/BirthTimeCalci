# MyNaksh — Birth Time Calculator

A free standalone tool that estimates a person's birth time from seven questions,
then sends them to the MyNaksh app.

## Publish

1. Commit these files to the repo root.
2. Settings → Pages → Deploy from a branch → `main` / `/ (root)`.

### One edit before you publish

Five meta tags contain the placeholder `SITE_URL`. Crawlers need absolute URLs,
so WhatsApp and LinkedIn previews will not work until it is replaced:

```bash
sed -i 's|SITE_URL|https://YOURUSER.github.io/YOURREPO/|g' index.html
```

Everything else works as-is.

## Files

| File | Purpose |
|---|---|
| `index.html` | The whole site. Fonts and logo embedded. |
| `og-image.png` | Social share card, 1200×630. |
| `favicon.ico`, `favicon-32.png`, `apple-touch-icon.png` | Icons. |
| `.nojekyll` | Stops GitHub Pages running Jekyll. |

## Behaviour

- **Bilingual.** The header toggle switches English and Hinglish instantly, with
  no reload, and keeps quiz answers and position. `#en` and `#hi` deep-link to a
  language; with no hash it follows the browser and falls back to English.
- **App buttons route by device.** App Store on iOS, Google Play on Android and
  desktop. A dismissible bar pinned to the bottom appears once the hero scrolls
  away, so the app CTA is reachable from anywhere on the page, not only at the
  result.

### Deep linking (not live yet)

The bottom bar and the result CTA both go to a store listing, not into the app,
because a true deep link is not possible from mynaksh.com today. Universal Links
(iOS) and App Links (Android) require these two files:

```
https://mynaksh.com/.well-known/apple-app-site-association
https://mynaksh.com/.well-known/assetlinks.json
```

Both currently return the site's SPA fallback HTML rather than JSON, so neither
platform can verify the domain. Once your mobile team publishes them, set one
constant near the top of the script in `index.html`:

```js
var APP_LINK = 'https://mynaksh.com/app/kundli';   // whatever path the app claims
```

Every app button then follows it: the app opens if installed, and the URL falls
back to the web page if not. No other change needed.
- One external request: Inter Tight from Google Fonts. Everything else is inline.

### Question order

The seven questions are ordered for completion, not for how the engine uses
them — the engine reads all answers at once, so order is purely a UX decision.

Easy self-observation comes first (how people read you, your build), then the
mildly novel one (a birthmark), then family facts. The recall-heavy question —
whether anyone remembers a time — sits at five, once someone is invested, rather
than at one where "no idea" reads as failing at the first step. The reflective
question about life changes is sixth, and an easy factual one closes.

Each screen carries a short line marking progress, so the run feels finite.

### Name sound coverage

The classical pada table holds **93 distinct syllables**. Question seven groups
them into **17 sound groups** that cover all 93 with no gaps and no overlaps —
including the G, L, V, Y, B and H sounds and the vowels, all of which are common
in Indian names.

The stored answer is the group, and the engine matches the pada's syllable
against the whole group. An earlier version stored a single syllable while
displaying several, so a name beginning "Ki" was scored as if it began "Ka".

This question only shifts the result when the user's sound matches one of the
pada syllables the Moon occupied that day — usually one or two groups out of
seventeen. That is the method working as intended, not a bug: it contributes
strongly for those users and stays neutral for everyone else.

### Place coverage

State first, then district — two dropdowns, no free typing, so every answer is a
known value with known coordinates.

- **36 states and union territories**, **all 763 districts**
- MECE by construction: every place in India sits in exactly one district
- Largest list is Uttar Pradesh at 75 districts, which a native select handles fine
- Renamed districts show both names, e.g. `Bengaluru Urban (Bangalore Urban)`,
  `Chhatrapati Sambhajinagar (Aurangabad)`. Renames are state-scoped, so Bihar's
  Aurangabad and Chhattisgarh's Bijapur are untouched — those were never renamed.

Coordinates come from **GeoNames** (public domain), preferring the district seat
where the names agree, rounded to two decimals. Spot-checked against fourteen
known reference points and bounds-checked against India's extent.

District-level precision is ample here: even a 2° longitude error shifts the
ascendant by about 2°, against a sign that lasts roughly two hours.

## How the estimate works

Real sidereal astronomy plus classical Vedic rectification. No AI, no randomness —
the same answers always give the same result.

The strongest constraint is the Uttara Kalamrita day-constellation check from
Prof. P.S. Sastri's *Rectification of Birth Time* (ch. 3). With `V` = vighatis
(24-second units) from local sunrise to birth:

- birth star: `(4V) mod 9`, counted from Ashwini / Magha / Mula
- weekday: `(3V) mod 7`, counted from Sunday

Both must hold, so compliant times recur every 63 vighatis (25.2 minutes).
Verified against the book's printed tables: 81 of 81 entries match.

Measured on 600 runs with internally consistent answers: the intended ascendant is
recovered 88% of the time, with no adjacent-sign errors. Across 500 runs, 94% of
reported times satisfy both Sastri conditions and 0% satisfy neither.

See `Birth_Time_Calculator_Methodology.md` for the full rationale, the
reconciliation against Sastri's text, and where the questions depart from it.

## Before a real launch

- **License Recoleta.** The font files in the design system are demo versions that
  watermark certain glyphs, including the digit 4. The page works around this —
  numerals are set in Inter Tight and a `unicode-range` blocks the affected
  glyphs — but the font should be licensed properly.
- **Scoring weights are reasoned, not fitted.** Test against people whose birth
  times are actually known before trusting the accuracy figures in production.
