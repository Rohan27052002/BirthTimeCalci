# MyNaksh — Birth Time Calculator

A free standalone tool that estimates a person's birth time from a short set of
questions, then sends them to the MyNaksh app.

**Live:** https://rohan27052002.github.io/BirthTimeCalci/

## Files

| File | Purpose |
|---|---|
| `index.html` | The whole site. Fonts, logo, place data and engine all embedded. |
| `og-image.png` | Social share card, 1200×630. |
| `favicon.ico`, `favicon-32.png`, `apple-touch-icon.png` | Icons. |
| `Birth_Time_Calculator_Methodology.md` | Full method, reconciliation against Sastri, measured accuracy. |

No build step, no dependencies, no server, no API keys. One external request:
Inter Tight from Google Fonts. Everything else is inline, so the file also works
from `file://`.

## The questions

**Date and district** — an inline calendar with day, month and year views, then
state followed by district. All 36 states and union territories, all 763
districts. Impossible dates cannot be produced.

**1. Your birth star.** If they know their nakshatra, it becomes a hard filter.

**2. Dark or light.** Also a hard filter, and the single most reliable thing
anyone remembers.

**Then four questions, chosen for that half of the day.** Nobody is shown a
question that cannot narrow their own half:

| If they said dark | If they said light |
|---|---|
| Would the shops near your home still have been open? | Was it before 12 noon, or after? |
| Would most people nearby have already gone to sleep? | Was the sun low in the sky, with long shadows? |
| Had it passed midnight? | Would the shops near your home have opened by then? |
| Was it in the second half of the night, closer to morning? | Was the sun almost straight overhead? |

**Last, the birthmark** — which part of the body carries a mark they were born
with.

Every question is a two-way choice with its own answer labels, plus "not sure",
so nobody has to mentally negate anything and nothing dead-ends.

## How the estimate works

Real sidereal astronomy plus classical Vedic rectification. No AI, no
randomness — the same answers always give the same result.

**Two hard filters.** The Moon must actually be in the stated nakshatra, and the
Sun must be on the correct side of the horizon for that date and district. These
cannot be outvoted, so an answer of "dark" can never return a daytime result.

**Four soft votes.** Each is worth 2 points, so one wrong answer shifts the
estimate rather than deciding it. If the answers contradict each other, the
range widens and confidence drops instead of showing false precision.

**A 25-minute grid picks the minute.** The Uttara Kalamrita day-constellation
check from Prof. P.S. Sastri's *Rectification of Birth Time*, ch. 3. With `V` as
vighatis (24-second units) from local sunrise:

- birth star: `(4V) mod 9`, counted from Ashwini / Magha / Mula
- weekday: `(3V) mod 7`, counted from Sunday

Both must hold, so compliant times recur every 63 vighatis — 25.2 minutes.
Verified against the book's printed tables: 81 of 81 entries match.

The reported range is never tighter than 25 minutes, because that is the grid
spacing and anything finer would be invented.

## Measured accuracy

Simulated births, with the birthmark deliberately given **no** correlation to
the true ascendant, so these figures do not depend on that classical
correspondence holding:

| | Median | Within 2 h | Over 3 h |
|---|---|---|---|
| All answers truthful | **32 min** | 98% | **0%** |
| One answer wrong | 62 min | 71% | 18% |

Four questions carry no redundancy, so a wrong answer costs more than it would
with a longer set. Confidence labels are calibrated to match: Strong ≈ 25 min,
Reasonable ≈ 37 min, Rough ≈ 52 min.

Widest indistinguishable bucket: 3.1 h on the dark branch, 3.0 h on the light
branch. That is the precision ceiling for someone answering only these four.

**These are simulated numbers.** They have not been tested against people whose
birth times are actually known. Treat them as an upper bound until they have.

## Behaviour

- **Bilingual.** The header toggle switches English and Hinglish instantly, with
  no reload, keeping answers and position. `#en` and `#hi` deep-link to a
  language; with no hash it follows the browser and falls back to English.
- **Verified at parity:** 93 string keys per language, identical vote keys in the
  same order, so both languages drive the same calculation.
- **App button routes by device** — App Store on iOS, Google Play on Android and
  desktop. A dismissible bar appears once the hero scrolls away.
- **Scrolling holds still.** Questions and Back keep their position (measured
  0 px drift); finishing lands on the time near the top of the viewport.

### Deep linking (not live yet)

The app buttons go to a store listing, not into the app, because a true deep
link is not possible from mynaksh.com today. Universal Links (iOS) and App Links
(Android) require these two files:

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

Every app button then follows it. No other change needed.

## Before a wider launch

- **License Recoleta.** The font files used are demo versions that watermark
  certain glyphs, including the digit 4. The page works around this — numerals
  are set in Inter Tight and a `unicode-range` blocks the affected glyphs — but
  the font should be licensed properly.
- **Validate against known birth times.** Every accuracy figure above is
  simulated. Thirty to fifty cases of date, district, known true time and the
  answers given would settle whether the model holds up.
- **Have an astrologer review the birthmark question.** The Kalapurusha
  correspondence is classical but unproven, and it is the one input whose real
  predictive value is untested.
