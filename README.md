<div align="center">

<img src="banner.png" alt="HiveGrade" width="720">

### Know whether your colony is *actually* thriving — for the time of year.

![PWA](https://img.shields.io/badge/PWA-installable-f0a818?style=flat-square)
![Offline](https://img.shields.io/badge/works-offline-2e7d32?style=flat-square)
![Storage](https://img.shields.io/badge/data-on--device%2C%20no%20account-8a6f4f?style=flat-square)
![Audience](https://img.shields.io/badge/made%20for-hobbyist%20beekeepers-c2611c?style=flat-square)

</div>

---

## 🐝 What it is

HiveGrade is a web app that turns a frame-by-frame hive inspection into a **letter grade** — A through F — judged against what's normal for the **calendar date**. You map what's on each frame (brood, honey, nectar, pollen, empty comb), log a few details like the queen and a mite-wash count, and HiveGrade scores the colony's brood, stores, pollen, and brood pattern against season-appropriate targets, then applies modifiers for mites, queen status, swarm cells, and nest arrangement. It returns a grade with **all of its reasoning shown** — plus a saved history so you can watch a colony's health move over the season.

It installs to your phone or desktop, works offline, and keeps everything on your own device. No account, no server.

## 💡 Why it exists

Most hive inspections end in a gut call: *"looks okay, I think."* That's hard to act on and impossible to track. And the usual rules of thumb are misleading because **bee colonies are seasonal** — five frames of brood and two of honey is a fantastic reading in May and a worrying one in October. The *same frames* mean opposite things depending on the month.

HiveGrade exists to replace "looks okay" with something concrete and honest: a number and a letter, measured against what a colony *should* look like at that point in the year, built on benchmarks from references beekeepers actually trust — **Randy Oliver** (ScientificBeekeeping) and **David Burns** (Honey Bees Online). It shows its work so you can argue with it, and it logs every inspection so the trend — not just today's snapshot — is visible.

## 👤 Who it's for

**Hobbyist beekeepers** — especially newer and intermediate ones who want a straight answer about whether their colony is on track, without a mentor leaning over their shoulder. If you keep one hive or a handful, inspect them yourself, and want to *learn the seasonal logic* while you track your bees, this is built for you. It's equally happy with a single nuc or a triple-deep, and it understands Langstroth, Apimaye, top-bar, Warré, Layens, and generic equipment.

## ✨ Features

- 📅 **Date-aware grading** — scored against seven season stages, not a year-round average
- 🗺️ **FrameMap** — tap any frame, log its makeup; the brood nest and stores appear as a color-coded hive cross-section
- 🎯 **A–F grade with reasons** — every factor, weight, and modifier is shown, never a black box
- 🐝 **Any hive system** — Langstroth, Apimaye, top-bar, Warré, Layens, nuc, generic; fully configurable, with colors
- 📈 **Saved history & trend** — every inspection is stored and reopenable; watch brood strength over time
- 🖨️ **Printable report** + 🤖 **JSON export** — a clean PDF-ready one-pager, and a machine-readable summary for an AI assistant
- 📲 **Installable & offline** — a Progressive Web App with on-device storage

## 📸 Screenshots

> Screenshots make this page far more compelling — add your own once you've mapped a real inspection. Three shots worth capturing:
> 1. **The graded inspection** — the big color letter and the readout
> 2. **The FrameMap** — a brood nest mapped in, comb colors filled
> 3. **The printable report**
>
> Save them in a `docs/` folder and reference them here:
> ```markdown
> ![Graded inspection](docs/grade.png)
> ![FrameMap](docs/framemap.png)
> ![Report](docs/report.png)
> ```

---

## 📊 How the grade is built

Six steps. Every factor and modifier below is displayed in the app and on the report — nothing is hidden.

### 1 · The date picks the season stage

Your inspection date maps to one of seven stages (Northern hemisphere; a toggle flips the calendar six months). Targets are a **share of the colony's drawn comb**:

| Stage | Months | Brood | Stores | Pollen | Honey floor |
|---|---|---|---|---|---|
| Overwinter | Dec–Jan | 0–10% *(low is fine)* | ≥55% | 10–20% | ≥60 lb |
| Late winter | Feb | ≥10% | ≥40% | 10–20% | — |
| Spring buildup | Mar–Apr | ≥40% | ≥15% | 10–20% | — |
| Swarm season | May | ≥50% | ≥20% | 10–20% | — |
| Main flow | Jun–Jul | ≥35% | ≥30% | 8–15% | — |
| Winter-bee / dearth | Aug–Sep | ≥25% | ≥45% | 8–15% | — |
| Fall prep | Oct–Nov | 5–20% *(low is fine)* | ≥55% | 10–20% | ≥60 lb |

### 2 · The colony is measured in deep-frame-equivalents (DFE)

Frames differ in comb area, so everything is normalized to a Langstroth **deep = 1.0**:

| Frame / box type | DFE |
|---|---|
| Langstroth deep · Apimaye deep · nuc | 1.00 |
| Langstroth medium · Apimaye medium | 0.667 |
| Langstroth shallow | 0.50 |
| Top-bar | 0.80 |
| Warré | 0.60 |
| Layens | 1.90 |

A nuc and a triple-deep are judged fairly because the grade reads *proportions* of drawn comb, not raw frame counts. Capped honey is also estimated at **~8 lb per deep-frame**, which feeds the winter-stores floor.

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

At or above the floor scores 100; below, it scales down. Stores and pollen are "more is better" — never penalized for exceeding. Pattern reaches 100 at **85% solid**. In overwinter and fall, **low brood is expected and goes neutral** rather than dragging the grade down.

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
| **Mites** | wash ≥ 3% caps the grade at a season ceiling; ≥ 2% → −8 | The cap is harshest in Aug–Sep (down to D−) — it's mite-borne viruses that sink a colony, and that's the fatal window. |
| **No laying evidence** | no queen + no eggs + no open brood **and** no capped brood → −25; with capped brood → caution only | Burns's "don't jump to queenless" — capped brood means she was laying recently. |
| **Swarm cells** | charged/capped cells → loud flag, grade barely moves | A colony making swarm cells is *healthy* — it's about to leave. Management alert, not a health failure. |
| **Nest arrangement** | tight contiguous brood + pollen alongside → +3; scattered brood → −4 | Burns's "rainbow": brood centered, pollen beside it, honey arced above and outside. |
| **Winter-stores floor** | overwinter / fall, honey < ~60 lb → −6 + flag | Farrar's and Burns's overwintering minimum. |

Season mite ceilings: overwinter & late winter **75** · spring, swarm, flow, fall **72** · Aug–Sep dearth **60**.

### 6 · The score becomes a letter and a color

| Score | | Score | | Score |
|---|---|---|---|---|
| ≥97 **A+** | ≥80 **B−** | ≥67 **D+** |
| ≥93 **A** | ≥77 **C+** | ≥63 **D** |
| ≥90 **A−** | ≥73 **C** | ≥60 **D−** |
| ≥87 **B+** | ≥70 **C−** | <60 **F** |
| ≥83 **B** | | |

🟢 **A — bright green** · 🟢 **B — dark green** · ⚪ **C — gray** · 🔴 **D — dark red** · 🔴 **F — bright red**

---

## 📚 Why these numbers

The targets come from two references most beekeepers trust:

- **Overwintering minimum** — Randy Oliver, citing C.L. Farrar, holds a good colony needs ~**60+ lbs of ripened honey and the equivalent of 3–6 frames of pollen** to winter. David Burns lands in the same place: **60–80 lbs in the top deep**. Hence the 60 lb floor and pollen's real weight heading into winter.
- **Colony strength** — Oliver's "frame of bees" standard (a deep frame ~70% covered above 60 °F) anchors what *strong* means; cold-adapted stock winters on a smaller cluster but explodes on spring pollen, reflected in the spring pollen weighting.
- **Inspection priorities & nutrition** — Burns frames a healthy inspection as *abundant brood, a laying queen, and sufficient honey **and** pollen*, and stresses protein as much as sugar. Pollen is a first-class factor here, not an afterthought.
- **Don't over-diagnose** — Burns's caution to read brood across multiple frames and not leap to "queenless" is why capped-brood-but-no-eggs is a *caution*, not a penalty.

The seasonal backbone ties it together: every benchmark above shifts meaning across the year, so the grade is always measured against the **stage**.

## 🧮 Worked example

> **June** — one 10-frame Langstroth deep: **5 frames brood, 2 nectar, 3 pollen**, solidness not logged, queen box unchecked.
>
> - Drawn-comb composition → brood **50%**, stores **20%**, pollen **30%**
> - Stage = **Main flow** (floors: brood 35%, stores 30%, pollen 8%)
> - brood → **100** · stores 20/30 → **67** · pollen → **100** · pattern (unset) → **100** *(neutral)*
> - base = 100·0.30 + 67·0.35 + 100·0.15 + 100·0.20 = **88**
> - capped brood, no fresh eggs → caution (no penalty); arrangement neutral
> - **→ B+**
>
> The same frames in **October** would grade far lower — fall prep wants ≥55% stores against a 60 lb floor. Same colony, different month, different answer. That's the whole point.

## 💾 Data & privacy

Every inspection is stored **on your device** in the browser's local storage — nothing is uploaded, there's no account, and there's no server. Export to JSON or a printable report any time to back up or move your data.

## 📲 Install

HiveGrade is a Progressive Web App. Open it in a browser and choose **Add to Home Screen** (mobile) or **Install** (desktop) to run it full-screen and offline.

## 🙏 Credits & disclaimer

Scoring benchmarks adapted from **Randy Oliver** (ScientificBeekeeping.com) and **David Burns** (HoneyBeesOnline.com). HiveGrade is an independent tool, not affiliated with or endorsed by either.

> ⚠️ **HiveGrade is a decision aid, not a diagnosis.** A grade summarizes what you logged. It does not replace your own eyes on the comb, a real alcohol-wash mite count, a disease check, or an inspector when something looks wrong.
