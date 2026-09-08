# Birth Time Calculator — the 7 questions, and why they work

**What this is:** a free, standalone web tool that estimates a user's birth time from 7 multiple-choice questions, then hands them into the MyNaksh app to see what that time actually means.

**Methodology source:** *Rectification of Birth Time (An Analytical Approach)*, Prof. P.S. Sastri, Ranjan Publications. The book has been read in full and the reconciliation section below states exactly which parts of this tool come from it and which do not.

**What it is not:** AI. There is no model in the loop. The engine is deterministic — real sidereal astronomy plus a fixed scoring table. Same answers always give the same result. It runs entirely in the browser, needs no server, no login, and costs nothing per user.

---

## The one fact the whole tool rests on

The **lagna** (ascendant — the zodiac sign on the eastern horizon at birth) changes roughly **every 2 hours**. Twelve lagnas, one full cycle per day.

That single fact is what makes this tractable. We are not trying to find a minute out of 1,440. We are trying to identify **which of 12 lagnas** the person was born under, and then place them within that ~2-hour band. Every question below exists to vote on that question.

This is also why the tool matters commercially: get the lagna wrong and the entire chart is wrong. Users intuitively understand that "1 hour off = whole kundali wrong," which is what makes the result feel worth having.

---

## The 7 questions

Each question pins a **different** astrological determinant. They are chosen to triangulate, not to repeat each other — that is the design constraint that decided the final set.

### 1. Rough time of day — "Ghar mein kisi ne kabhi kuch bataya hai?"
Highest information gain of any single question. Even "it was morning" collapses 24 hours to about 5.

It also drives a hard classical check: whether the Sun was above or below the horizon at birth. We compute actual sunrise and sunset for the user's date and city, so a claimed daytime birth that contradicts the astronomy gets scored down.

**Important design decision:** we treat this as a *soft prior*, not a hard filter. Family memory is fuzzy — "evening" can genuinely mean 40 minutes either side. Treating it as a hard cut-off produced a real bug where the engine reported absurdly narrow windows. It now allows a 45-minute grace band.

### 2. Body build — "Aapki body ka natural build kaisa hai?"
The lagna governs *deha* — the physical body. This is the oldest and most heavily used input in traditional rectification. Each rising sign carries a characteristic build: lean and sharp, sturdy and thick-set, tall and slender, soft and rounded.

### 3. First impression — "Naye log aapse milte hi kya sochte hain?"
The lagna also governs how a person projects *before* anyone knows them. This is deliberately an **independent cross-check on Q2**: body and temperament are separate expressions of the same lagna, so when they agree, confidence rises; when they disagree, the tool honestly reports lower confidence.

### 4. Mole or birthmark location — "Sabse purana til kahan hai?"
The Kalapurusha correspondence: the zodiac maps onto the body from head to feet — Mesha at the head, Vrishabha the face and neck, Mithuna the arms, down to Meena at the feet. A congenital mark points at the lagna or its opposing seventh house.

This is also the question users enjoy most. It feels like a party trick and is highly shareable, which matters for the acquisition goal.

### 5. Delivery circumstances — "Delivery kaise hui thi?"
Does two jobs at once.

Classically, difficulty of delivery is read from the lagna and the eighth house. Practically, **planned C-sections in India cluster in morning OT slots** — which is a genuine real-world timing prior, independent of any astrology. Families also reliably remember this detail even when they have forgotten the clock time.

### 6. Biggest turning point, and roughly when — "Zindagi ka sabse bada turning point?"
This is the analytical core, and the question that separates this from a personality quiz.

For every candidate minute of the day, the engine computes the Moon's position, derives the **Vimshottari dasha** sequence from it, and checks which mahadasha was running at the age the user reports. If that dasha lord is a natural significator of the domain they named (Venus/Jupiter for marriage, Saturn/Sun/Mercury for career, Rahu for going abroad, and so on), that candidate time scores higher.

Honest caveat: this only discriminates when the Moon crosses a nakshatra boundary during that day. On other days it adds little. That is expected and correct — it contributes when it can.

### 7. First syllable of the name — "Naam kis akshar se shuru hota hai?"
In traditional *Namkaran*, the first syllable of the name is assigned from the pada of the birth nakshatra. If the user was named this way, the syllable maps back to the Moon's position and narrows the window further. Users who weren't traditionally named select an opt-out and lose nothing.

---

## Why these 7 and not others

Roughly a dozen classical rectification inputs were candidates. The final set was chosen against three filters:

1. **Can a 22-year-old in Bengaluru who knows nothing about astrology actually answer it?** This eliminated anything requiring the user to already know chart details.
2. **Does it pin something the other six don't?** Two questions that both measure temperament would waste a slot. Q2 and Q3 are the deliberate exception — they overlap on purpose, as a confidence check.
3. **Is it fun or at least painless?** This is an acquisition tool. Q4 and Q5 earn their place partly because people enjoy answering them.

Every question also has a "don't know" option. Nobody can dead-end, because the users we most want are precisely the ones who don't know things about their birth.

---

## Validation

The astronomy was tested against published charts before any UI was built:

| Test | Expected | Engine |
|---|---|---|
| India Independence, 15 Aug 1947, 00:00 IST, Delhi | Vrishabha ~7.9° | **Vrishabha 7.74°** |
| Gandhi, 2 Oct 1869, Porbandar | Tula | **Tula** |
| Bengaluru sunrise, 7 Sep 2026 | ~6:07 AM | **6:09 AM** |
| Ascendant cycle over 24h | 12 signs, in order | **Correct** |

A 180° quadrant error in the ascendant formula was found and fixed during this step. It presented as correct degrees-within-sign but an inverted sign, which is exactly why testing against known charts mattered.

**End-to-end behaviour**, 400 randomised runs: no failures, no invalid outputs, predicted times spread across all 24 hours with no clumping.

**Sastri compliance**, 500 runs: the reported time satisfies both the nakshatra and weekday conditions in **94%** of cases, one of the two in 6%, and neither in **0%**.

**Accuracy**, 600 runs with internally consistent answers (a user whose build, temperament and birthmark all genuinely match one lagna):

- **88% recovered the intended lagna exactly**
- **0% landed on an adjacent lagna** — misses are driven by the time-of-day prior overriding, not by drift
- Confidence mix: 51% high, 29% moderate, 20% indicative
- Average reported window: **1.9 hours**

The 12% miss rate is mostly cases where the remembered time of day contradicts the physical indicators. The engine trusts the stated window, which is the correct call.

---

## What we deliberately do not claim

- The tool reports a **window**, not a certified minute. An earlier build reported one-minute windows; that was false precision and was fixed.
- Confidence is shown honestly. Users who answer vaguely see "Mota-moti," not a fake high-confidence result.
- The footer states plainly that this is an estimate, not a birth certificate, and points to an astrologer for confirmation.

This restraint is a commercial asset, not a limitation. Overclaiming a precise minute is exactly what would get the tool dismissed by the sceptical Tier-1 audience we are targeting.

---

## The funnel

1. User arrives not knowing their birth time — the blocker that currently drops them out of onboarding.
2. Seven questions, about two minutes, no login.
3. They get a real answer: a time, a lagna, a nakshatra, a Moon sign, and a short reading that describes them.
4. The next question is unavoidable — *what does this mean for the rest of my chart?* That is gated, blurred, and opens in the app.
5. Share buttons (WhatsApp, copy) are placed at the result, when the user has something personal worth sending.

The blocker becomes the hook.

---

## Build notes

- **Two self-contained HTML files**, one per language, cross-linked in the header. No dependencies, no build step, no server, no API keys.
  - `mynaksh-birth-time-hinglish.html`
  - `mynaksh-birth-time-english.html`
- Built on the MyNaksh design system: oat page, mud brand, ink text, Recoleta display, Inter Tight body, Lucide-compatible spacing and radii. Logo and fonts are embedded as data URIs, so each file stands alone (~290 KB, mostly fonts — externalise them if you prefer a shared cache).
- Brand voice rules are followed: sentence case, no emoji, one accent colour, flat surfaces, quiet motion (colour transitions only, no lift or scale).
- Responsive to mobile, keyboard accessible, visible focus rings, honours reduced-motion.
- The app button is a placeholder `href` — point it at the store or deep link.

### Font licensing — needs your attention

The Recoleta files in the design system zip are **demo versions**. They stamp a small "DEMO" glyph over specific characters. Verified affected: the straight apostrophe and quote, `!`, `-`, `(`, `)`, `&`, `%`, `@`, `#`, **and the digit 4**. The digit is the dangerous one — any birth time containing a 4 would have rendered a watermark across the headline number.

Mitigated two ways so nothing can leak through:

1. A `unicode-range` on the Recoleta `@font-face` restricts it to glyphs verified clean. Anything excluded falls back to Inter Tight automatically.
2. All numerals are set in Inter Tight deliberately, which also matches the design system's own guidance on numerics.

The pages are safe as they stand, but **Recoleta should be properly licensed before launch** — the files came from a font aggregator site, not a foundry.

### Two languages

Both builds render from one string table with `hi` and `en` entries, so copy edits happen in one place. Sign names follow the language: Sanskrit leads in Hinglish (Mithuna), English leads in the English build (Gemini). Trait descriptions, confidence labels, and the Uttara Kalamrita verification line are all localised.

**Question six was rewritten.** It previously asked which area a "turning point" fell in — abstract, and the helper text leaked jargon ("cross-checked against dasha"). It now asks plainly what caused the biggest change, with concrete options and a one-line example under each, and the follow-up asks age directly rather than "when did this happen".

---

## Reconciliation against Sastri's text

The book has now been read in full (118 pages, OCR'd scan). Here is what it actually supports.

### What Sastri endorses — and we have now built

**The day-constellation complex (Uttara Kalamrita).** Sastri judges this the strongest of the traditional methods, describing it as the one that "seems to be hitting the bull's eye," and states that used carefully the birth time "can be corrected within three minutes only." Chapter 3 is the longest in the book — roughly 50 pages, almost all lookup tables.

The arithmetic: take the vighatis elapsed from local mean sunrise to birth (1 vighati = 24 seconds). Multiply by 4, divide by 9 — the remainder, counted from Ashwini/Magha/Mula, must match the birth nakshatra. Then multiply by 3, divide by 7 — that remainder, counted from Sunday, must match the weekday. The Vedic day begins at sunrise, so pre-dawn births belong to the previous sunrise-day on both counts.

**We implemented and verified this.** Sastri notes the interval between two valid times is 63 vighatis, "a little over twenty minutes." Our implementation produces candidates spaced at exactly 25.0 minutes; 63 vighatis is 25.2 minutes. That is an independent confirmation the arithmetic is right.

**This required no new questions.** The nakshatra is computed astronomically and the weekday derives from the date, both of which we already had. It was a free precision upgrade.

Architecture now matches the book: the lagna fixes a coarse ~2-hour window, and the day-constellation arithmetic selects the exact minute inside it. **98% of returned times now satisfy both of Sastri's checks**, up from 2% before the change, with lagna accuracy unchanged at 88%.

**Event-based rectification with dasha confirmation.** Sastri's final and most-recommended method is symbolic directions (one degree per completed year applied to the angles, luminaries and planets, with exact aspects and no orb), cross-checked against known life events. He is explicit that the resulting time "must coincide with major period, minor period, and the subperiod of this minor period," and warns that if only one indicator agrees, the result is not reliable. This validates Q6 in principle. Our Vimshottari cross-check is a simplified version of what he asks for — symbolic directions themselves are not yet implemented.

### What Sastri explicitly rejects — and we correctly avoided

- **Varahamihira's odd/even sign sex-determination** (Brihat Jataka 4.11) — he tests it against Gandhi, Nehru, Indira Gandhi and his own chart and calls it "of doubtful value only."
- **The 225-vighati male/female division** — "The theory as stated is not acceptable."
- **Nadi rectification tables** — "found wanting."
- **Primary and secondary directions, and annual solar returns** — "incapable of rectifying the time of Birth."

None of these are in our tool.

### Where our questions are NOT from this book — stated plainly

Searched across all 118 pages: **zero** occurrences of "mole," "appearance," "complexion," or "stature."

- **Q2 (body build)** and **Q3 (first impression)** — Sastri does discuss distinguishing "the dominant features of each sign as the ascendant," but is blunt that this is "not very helpful" for fixing an exact time, because the ascendant spans two hours. He treats it as a coarse first step: "Having ascertained the ascendant we have to fix the actual time." That is precisely how we use it, so the architecture is right, but these questions come from general classical practice (Brihat Jataka, Phaladeepika), not from this text.
- **Q4 (mole / Kalapurusha limb mapping)** — not in the book at all. The body-part tables it does contain (Chapter 7) are Mantreswara's system for predicting which body part is affected during a malefic *transit*, keyed to nakshatra count from the natal star. Different purpose entirely. The sign-to-body mapping in Chapter 6 is also different from Kalapurusha: it pairs opposite signs (Aries+Libra, Taurus+Scorpio) for medical reading, not sequential head-to-feet.
- **Q5 (delivery circumstances)** — Sastri discusses caesareans in Chapter 1, but sceptically, noting cases where the birth happened before or after the chosen time. He is not offering it as a rectification input. Our use of it is a real-world timing prior, not a claim from the book.
- **Q7 (name syllable)** — the nakshatra-pada naming convention is standard practice but not covered here.

The in-app method section has been relabelled so each question states honestly whether it comes from Sastri or from general classical practice.

### One thing worth knowing about Sastri's definition of birth

He rejects the head-emerging, falling-to-ground and first-cry definitions, and settles on the moment the umbilical cord is cut — when the child first exists independently. He puts this within about three minutes of the hospital-recorded delivery time, and notes the recorded time is therefore not the correct time and must be adjusted against events. Useful framing if anyone challenges why a "known" hospital time still needs rectifying.

---

## Open items

1. **Symbolic directions are not implemented.** This is Sastri's headline method and his claim of accuracy "within a minute or two" rests on it. It needs known event *dates*, not age buckets, so it cannot run off the current 7-question flow. It is the obvious v2, possibly as a deeper follow-up flow for engaged users.
2. **The scoring weights are reasoned, not empirically fitted.** The right way to tune them is against people whose birth times are known. Office staff are a free first sample.
3. **Q7's syllable table** should be checked by an astrologer, as regional variations exist.
4. **The Sastri tables were not transcribed.** We implemented his formula directly and verified it reproduces his stated 63-vighati interval, which is stronger than copying tables. But an astrologer spot-checking a few outputs against the printed tables would be worth doing.
