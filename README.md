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
sed -i 's|SITE_URL|https://rohan27052002.github.io/BirthTimeCalci/|g' index.html
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
