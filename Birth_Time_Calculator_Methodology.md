# Birth Time Calculator — method and rationale

**What this is:** a free web tool that estimates a birth time from a short set of
questions, then hands the person into the MyNaksh app.

**What it is not:** AI. There is no model in the loop. The engine is
deterministic — real sidereal astronomy plus a fixed rule set. The same answers
always give the same result. It runs entirely in the browser, needs no server,
no login, and costs nothing per user.

---

## The design principle

Rectification inputs fall into two very different classes, and the whole design
turns on keeping them apart.

**Direct evidence** is what a person actually remembers about the clock: whether
it was dark, whether the shops were shut, whether the household was asleep. This
is imprecise but honest, and its validity rests on nothing but memory.

**Classical correspondence** is the inference from a person's body or character
to their ascendant. This may be precise if the correspondence holds, but its
validity is unproven.

An earlier build let correspondence outvote memory. Someone who answered
"evening, around sunset" was told 5:29 am. It happened in 38.7% of runs. The
fix, and the current architecture, is that **memory gates and correspondence
only breaks ties.**

---

## The flow

### Setup
An inline calendar (day, month, year views), then state, then district. All 36
states and union territories, all 763 districts, coordinates from GeoNames.
District-level precision is ample: even a 2° longitude error moves the ascendant
about 2°, against a sign that lasts two hours.

### Two hard filters

**Birth star.** If the person knows their nakshatra, only minutes when the Moon
was genuinely in it are considered. The Moon crosses roughly one nakshatra per
day, so on boundary days this alone removes a large part of the day.

**Dark or light.** Sunrise and sunset are computed for the exact date and
district. An answer of "dark" can never return a daytime result.

Neither can be outvoted. Verified over 700 adversarial runs with deliberately
contradictory votes: **0 daytime answers for "dark", 0 night answers for
"light", 0 birth-star mismatches.**

### Four soft votes, chosen by branch

Nobody is asked a question that cannot narrow their own half of the day.

| Dark branch | Range it marks |
|---|---|
| Would the shops near your home still have been open? | sunset to ~21:30 |
| Would most people nearby have already gone to sleep? | ~22:30 to sunrise |
| Had it passed midnight? | 00:00 to sunrise |
| Was it in the second half of the night, closer to morning? | midpoint of night to sunrise |

| Light branch | Range it marks |
|---|---|
| Was it before 12 noon, or after? | 00:00 to 12:00 |
| Was the sun low in the sky, with long shadows? | dawn and dusk bands |
| Would the shops near your home have opened by then? | ~09:30 to 20:00 |
| Was the sun almost straight overhead? | 11:00 to 14:00 |

Each match adds 2 points, so one wrong answer costs 2 of 8 and shifts the
estimate rather than deciding it. Each question is phrased about the *area*
rather than the family, because a household in the middle of a delivery is
behaving abnormally while the street outside keeps its usual rhythm. "Would
have" signals that we want reasoning about the hour, not recall of the day.

### The birthmark

Which part of the body carries a congenital mark. Used only to choose between
candidate minutes, never to override memory.

### The 25-minute grid

The Uttara Kalamrita day-constellation check from Sastri, ch. 3. With `V` as
vighatis (24-second units) from local sunrise:

- birth star: `(4V) mod 9`, counted from Ashwini / Magha / Mula
- weekday: `(3V) mod 7`, counted from Sunday

Both hold at the true birth time. Because 9 and 7 are coprime, compliant times
recur every **63 vighatis — 25.2 minutes**, exactly the interval the book
states. Verified against the printed tables on p26: **81 of 81 entries match**,
and a time one vighati off correctly fails.

This costs no user questions. Date and district already give sunrise, weekday
and the Moon's position.

---

## Reconciliation against Sastri's text

The book was read in full (118 pages, OCR'd scan).

| Chapter | Pages | Verdict the book reaches |
|---|---|---|
| Traditional Methods | 11–17 | Mandi, Gulika, Pranapada — all "cannot fix the time of birth" |
| **Day-Constellation Complex** | **18–67** | **"hitting the bull's eye"** — the book's primary method |
| Nadi Rectification | 68–77 | "found wanting" |
| Pre-Natal Epoch | 78–89 | Examined in detail, treated as usable |
| Directions (3 chapters) | 90–115 | "incapable of rectifying the time of Birth" |
| Symbolic Directions | 118+ | Event-based, treated as usable |

Half the book is one method, and it is arithmetic rather than descriptive. That
method is implemented and is the grid described above.

**What the book rejects and we never used:** Mandi, Gulika, Pranapada, Nadi
tables, primary and secondary directions, solar returns.

**What the book does not contain.** Zero mentions of appearance, complexion,
stature, moles, Kalapurusha or limbs across 118 pages. The birthmark question
comes from general jataka tradition, not from this text.

**A finding that cuts against the birthmark question.** Discussing whether
sign–body-part correspondences help fix a chart, the book notes the ascendant
"may be spread over two hours" and concludes that matching it to the person is
**"not very helpful."** The question is retained only as a tie-break, and this
is why it is never allowed to override the remembered window.

**Still unimplemented:** Pre-Natal Epoch (ch. 5), which the book treats as
usable. It needs gestation assumptions a casual user cannot supply, so it likely
belongs in an astrologer-assisted tier rather than the free funnel.

---

## Validation

### Astronomy
Tested against published charts before any UI existed.

| Test | Expected | Engine |
|---|---|---|
| India Independence, 15 Aug 1947, 00:00 IST, Delhi | Vrishabha ~7.9° | **Vrishabha 7.74°** |
| Gandhi, 2 Oct 1869, Porbandar | Tula | **Tula** |
| Bengaluru sunrise, 7 Sep 2026 | ~6:07 AM | **6:09 AM** |
| Ascendant cycle over 24h | 12 signs, in order | **Correct** |

A 180° quadrant error in the ascendant formula was found and fixed at this
stage. It presented as correct degrees-within-sign but an inverted sign, which
is exactly why testing against known charts mattered.

### Place data
763 districts, coordinates from GeoNames (public domain). Spot-checked against
fourteen known district seats — all matched — and the whole set bounds-checked
against India's extent. Renamed districts show both names, state-scoped so
Bihar's Aurangabad and Chhattisgarh's Bijapur are untouched.

### End-to-end accuracy

Simulated births where one true time generates the answers a truthful person
would give. **The birthmark is deliberately given no correlation to the true
ascendant**, so the figures do not depend on that correspondence holding.

| | Median | p90 | Within 2 h | Over 3 h |
|---|---|---|---|---|
| All answers truthful | **32 min** | 1.47 h | 98% | **0%** |
| One answer wrong | 62 min | 4.1 h | 71% | 18% |
| Two answers wrong | 2.5 h | 7.3 h | 43% | 45% |

Confidence labels are calibrated against these: Strong ≈ 25 min, Reasonable
≈ 37 min, Rough ≈ 52 min.

### Precision ceiling

Widest indistinguishable answer pattern: **3.1 h** on the dark branch, **3.0 h**
on the light branch. Two people whose true times fall in the same bucket give
identical answers, so no rule set can separate them. Reporting the centre of the
bucket bounds the worst case at roughly half its width.

---

## What this does not claim

- The range is never reported tighter than **25 minutes**, the grid spacing.
  Anything finer would be invented.
- Contradictory answers **widen** the range and drop the confidence rather than
  producing a precise-looking number.
- Four questions carry no redundancy, so one wrong answer costs more than it
  would in a longer set. That is a deliberate trade for brevity.
- One failure mode cannot be detected: a wrong set of answers that happens to be
  perfectly self-consistent with a different time will read as Strong and be
  wrong. No algorithm can catch that at four questions.

---

## The honest caveat

**Every accuracy figure here is simulated.** None has been tested against people
whose birth times are actually known.

Two earlier rounds of figures did not survive contact with a real user. The
first paired a random window with an unrelated ascendant profile and scored 88%
— largely *because* a bug was ignoring the remembered window. The second used a
decision tree that scored well when every tap was correct and produced a
6.28-hour median error when one tap was wrong.

The remedy is straightforward and cheap: thirty to fifty rows of date, district,
**known true time**, and the answers given. That would settle in an afternoon
what no amount of simulation can.

---

## Open items

1. **Validate against known birth times.** The single highest-value next step.
2. **License Recoleta.** The font files are demo versions that watermark certain
   glyphs including the digit 4. The page works around it, but the workaround
   should not be permanent.
3. **Astrologer review of the birthmark question**, which is the one input whose
   real predictive value is untested — and which the source text itself
   describes as "not very helpful."
4. **Publish the `.well-known` files** on mynaksh.com to enable true deep
   linking. One constant in `index.html` then switches every app button over.
