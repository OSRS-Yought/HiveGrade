<div align="center">

<img src="banner.png" alt="HiveGrade" width="720">

A hive-inspection grader that judges a colony against what's normal for the time of year.

![PWA](https://img.shields.io/badge/PWA-installable-f0a818?style=flat-square)
![Offline](https://img.shields.io/badge/works-offline-2e7d32?style=flat-square)
![Storage](https://img.shields.io/badge/data-on--device%2C%20no%20account-8a6f4f?style=flat-square)
![Audience](https://img.shields.io/badge/made%20for-hobbyist%20beekeepers-c2611c?style=flat-square)

</div>

---

## What it is

HiveGrade turns a frame-by-frame inspection into a letter grade, A through F, scored against what a colony should look like on that calendar date. You map what's on each frame — brood, honey, nectar, pollen, empty comb — log the queen signs and a mite-wash count, and it scores brood, stores, pollen, and brood pattern against the targets for the current season stage, then applies modifiers for mites, queen status, swarm cells, and nest layout.

The grade comes with every input and weight shown, and each hive keeps a saved history so you can watch the trend instead of one snapshot.

It installs to a phone or desktop, runs offline, and keeps all data on the device. No account, no server.

## Why it exists

Most inspections end in a gut call: "looks okay." That's hard to act on and impossible to track. The usual rules of thumb also mislead, because colonies are seasonal — five frames of brood and two of honey is great in May and a problem in October. Same frames, opposite reading, depending on the month.

HiveGrade replaces "looks okay" with a number measured against the stage of the year, built on benchmarks from references beekeepers already trust: Randy Oliver (ScientificBeekeeping) and David Burns (Honey Bees Online). It shows the math so you can disagree with it, and logs every inspection so the trend stays visible.

## Who it's for

Hobbyist beekeepers, especially newer and intermediate ones, who want a straight read on whether a colony is on track without a mentor at their shoulder. One hive or a handful, inspected yourself. It handles a single nuc through a triple-deep, and understands Langstroth, Apimaye, top-bar, Warré, Layens, and generic equipment.

## Regions

The season calendar isn't the same everywhere, so grading is region-specific. You pick a region and a USDA hardiness zone: the zone drives the cold-weather numbers, the region sets the flow, dearth, and overwinter calendar. Until a region is selected, HiveGrade won't produce a grade — it doesn't pretend one region's numbers are universal.

One region ships verified today: **Northwest · 8b** (maritime Puget Sound). More are added one verified profile at a time, each built from that region's own state extension service. The tables below are the Northwest · 8b calendar.

## Features

- Date-aware grading, scored against seven season stages rather than a year-round average
- Region and zone profiles — the calendar localizes; grading is gated until you pick one
- FrameMap — tap a frame, log its makeup; the brood nest and stores draw as a color-coded cross-section
- A–F grade with reasons: every factor, weight, and modifier is shown
- Any hive system — Langstroth, Apimaye, top-bar, Warré, Layens, nuc, generic; configurable and recolorable
- Saved history: every inspection stored and reopenable
- Printable one-page report and JSON export for backup
- Installable and offline, with on-device storage

---

## How the grade is built

Six steps. Every factor and modifier below is shown in the app and on the report.

### 1 · The date picks the season stage

The inspection date maps to one of seven stages. Targets are a share of the colony's drawn comb. (Shown here for the Northwest · 8b profile; the stages are universal, the months and floors are per-region.)

| Stage | Months | Brood | Stores | Pollen | Honey floor |
|---|---|---|---|---|---|
| Overwinter | Dec–Jan | 0–10% *(low is fine)* | ≥55% | 10–20% | ≥60 lb |
| Late winter | Feb | ≥10% | ≥40% | 10–20% | — |
| Spring buildup | Mar–Apr | ≥40% | ≥15% | 6–20% | — |
| Swarm season | May | ≥50% | ≥20% | 6–20% | — |
| Main flow | Jun–Jul | ≥35% | ≥30% | 4–15% | — |
| Winter-bee / dearth | Aug–Sep | ≥25% | ≥45% | 8–15% | — |
| Fall prep | Oct–Nov | 5–20% *(low is fine)* | ≥55% | 10–20% | ≥60 lb |

Pollen floors run low during buildup and the flow on purpose: standing pollen there is throughput, not a reserve, so a low reading mid-flow is flow-rate rather than a shortage. They're held higher in dearth, fall, and winter, where stored pollen is a real buffer.

### 2 · The colony is measured in deep-frame-equivalents (DFE)

Frames differ in comb area, so everything is normalized to a Langstroth deep = 1.0:

| Frame / box type | DFE |
|---|---|
| Langstroth deep · Apimaye deep · poly deep · nuc | 1.00 |
| Flow super | 0.85 |
| Langstroth medium · Apimaye medium · poly medium | 0.667 |
| Langstroth shallow · poly shallow | 0.50 |
| Top-bar | 0.80 |
| Warré | 0.60 |
| Layens | 1.40 |

A nuc and a triple-deep are judged fairly because the grade reads proportions of drawn comb, not raw frame counts. Capped honey is also estimated at about 8 lb per deep-frame, which feeds the winter-stores floor.

### 3 · Each factor is scored 0–100 against its seasonal target

```
score(actual, floor):
    if this factor is "low is fine" for the stage:   return 100      # neutral
    if actual >= floor:                              return 100      # at or above target
    else:                                            return (actual / floor) * 100

pattern(solidness):
    if brood present AND solidness logged:           return clamp((solidness - 50) / 35 * 100, 0, 100)
    else:                                            return 100      # neutral
```

At or above the floor scores 100; below it, it scales down. Stores and pollen are "more is better" and never penalized for exceeding. Pattern reaches 100 at 85% solid. In overwinter and fall, low brood is expected and goes neutral rather than dragging the grade down.

### 4 · Factors are weighted by what matters that season

| Stage | Brood | Stores | Pollen | Pattern |
|---|---|---|---|---|
| Overwinter | 0.05 | **0.60** | 0.25 | 0.10 |
| Late winter | 0.25 | **0.40** | 0.25 | 0.10 |
| Spring buildup | **0.40** | 0.15 | 0.25 | 0.20 |
| Swarm season | **0.40** | 0.20 | 0.20 | 0.20 |
| Main flow | 0.30 | **0.35** | 0.15 | 0.20 |
| Winter-bee / dearth | 0.20 | **0.35** | 0.15 | 0.30 |
| Fall prep | 0.05 | **0.55** | 0.25 | 0.15 |

```
base = brood·w_brood + stores·w_stores + pollen·w_pollen + pattern·w_pattern
```

### 5 · Modifiers adjust the score

| Modifier | Effect | Why |
|---|---|---|
| Mites | wash ≥ 3% caps the grade at a season ceiling; ≥ 2% → −8 | The cap is harshest in Aug–Sep (down to D−). It's mite-borne viruses that sink a colony, and that's the fatal window. |
| No laying evidence | no queen, no eggs, no open brood, and no capped brood → −25; with capped brood → caution only | Burns's "don't jump to queenless": capped brood means she was laying recently. |
| Swarm cells | charged or capped cells → loud flag, grade barely moves | A colony making swarm cells is healthy and about to leave. It's a management alert, not a health failure. |
| Nest arrangement | tight contiguous brood with pollen alongside → +3; scattered brood → −4 | Burns's "rainbow": brood centered, pollen beside it, honey arced above and outside. |
| Winter-stores floor | overwinter and fall, honey < ~60 lb → −6 plus a flag | Farrar's and Burns's overwintering minimum. |

Season mite ceilings: overwinter and late winter 75 · spring, swarm, flow, fall 72 · Aug–Sep dearth 60.

### 6 · The score becomes a letter and a color

| Score | Grade | Score | Grade | Score | Grade |
|---|---|---|---|---|---|
| ≥97 | A+ | ≥80 | B− | ≥67 | D+ |
| ≥93 | A | ≥77 | C+ | ≥63 | D |
| ≥90 | A− | ≥73 | C | ≥60 | D− |
| ≥87 | B+ | ≥70 | C− | <60 | F |
| ≥83 | B |  |  |  |  |

Colors: A bright green, B dark green, C gray, D dark red, F bright red.

---

## Why these numbers

The targets come from two references most beekeepers trust:

- **Overwintering minimum.** Randy Oliver, citing C.L. Farrar, holds that a good colony needs roughly 60+ lbs of ripened honey and the equivalent of 3–6 frames of pollen to winter. David Burns lands in the same place, 60–80 lbs in the top deep. Hence the 60 lb floor and pollen's real weight heading into winter.
- **Colony strength.** Oliver's "frame of bees" standard — a deep frame about 70% covered above 60 °F — anchors what strong means. Cold-adapted stock winters on a smaller cluster but explodes on spring pollen, which is why pollen is weighted up in spring.
- **Inspection priorities and nutrition.** Burns frames a healthy inspection as abundant brood, a laying queen, and enough honey *and* pollen, and stresses protein as much as sugar. Pollen is a first-class factor here, not an afterthought.
- **Don't over-diagnose.** Burns's caution to read brood across several frames and not leap to "queenless" is why capped-brood-but-no-eggs is a caution, not a penalty.

Oliver's numbers come from a large migratory California operation, so they're treated as inputs to calibrate against local conditions, not answers to copy. The full record of every value, its source, and whether it's sourced, calibrated, or a judgment call lives in the project's rubric doc.

## Worked example

> **June** — one 10-frame Langstroth deep: 5 frames brood, 2 nectar, 3 pollen, solidness not logged, queen box unchecked.
>
> - Drawn-comb composition → brood 50%, stores 20%, pollen 30%
> - Stage = Main flow (floors: brood 35%, stores 30%, pollen 4%)
> - brood → 100 · stores 20/30 → 67 · pollen → 100 · pattern (unset) → 100 (neutral)
> - base = 100·0.30 + 67·0.35 + 100·0.15 + 100·0.20 = 88
> - capped brood, no fresh eggs → caution, no penalty; arrangement neutral
> - **→ B+**
>
> The same frames in October would grade much lower: fall prep wants ≥55% stores against a 60 lb floor. Same colony, different month, different answer.

## Data and privacy

Every inspection is stored on your device in the browser's local storage. Nothing is uploaded, there's no account, and there's no server. Export to JSON or a printable report any time to back up or move your data.

## Install

HiveGrade is a Progressive Web App. Open it in a browser and choose Add to Home Screen (mobile) or Install (desktop) to run it full-screen and offline.

## Credits and disclaimer

Scoring benchmarks adapted from Randy Oliver (ScientificBeekeeping.com) and David Burns (HoneyBeesOnline.com). HiveGrade is an independent tool, not affiliated with or endorsed by either.

> **HiveGrade is a decision aid, not a diagnosis.** A grade summarizes what you logged. It does not replace your own eyes on the comb, a real alcohol-wash mite count, a disease check, or an inspector when something looks wrong.
