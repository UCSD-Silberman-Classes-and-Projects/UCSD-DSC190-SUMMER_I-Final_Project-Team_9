# Making the Donkeycar CNN More Robust with Data Augmentation

DSC 190 SU1 2026 Final Project

UC San Diego | Halıcıoğlu Data Science Institute (HDSI) | Jacobs School of Engineering

## Team Members

| Name | Department |
|---|---|
| Evelyn Na | Halıcıoğlu Data Science Institute |
| Bora Vanli | Halıcıoğlu Data Science Institute |
| Roberto Huizar | Department of Mathematics |

---

## Abstract

DonkeyCar's default image-only CNN pipeline (no GPS) struggles to generalize across changing lighting conditions. Our CNN relies purely on camera images — there's no GPS localization to fall back on. That means the model's only signal for staying on track is what it sees, and what it sees changes dramatically throughout the day: shifting shadows, brightness, and saturation. In prior coursework, this meant a model trained in the morning simply couldn't drive at night, and vice versa, forcing us to train three separate models — one per time of day. That workaround got us through the original assignment, but it isn't a real solution: it doesn't scale, and it sidesteps the actual problem — the model was never taught to generalize in the first place.

This project tackles that problem directly, but with a stricter target than simply combining more data: can a **single dataset, collected at any one time of day**, augmented well, produce a model that drives reliably across *all* lighting conditions — without ever needing to collect data at multiple times of day at all? We train on data from a single session (in our case, midday), apply data augmentation guided by Claude as an AI coding agent, and test the resulting model across morning, midday, and night. If successful, this would mean future cohorts only need to collect one recording session — regardless of which time of day they happen to record it — instead of three. We also aim to contribute our augmentation improvements back to the open-source [DonkeyCar](https://github.com/autorope/donkeycar) project via a pull request.

---

## What We Promised

### Must Have
- Establish a baseline by training on a single session's data only (in our case, midday), using DonkeyCar's existing (default) augmentation settings, and test it across all three lighting conditions (morning, midday, night)
- Feed Claude the codebase and previously-tried augmentations; implement and test its proposed improvements
- Train an augmented model on the **same single session's data only** and test it across all three lighting conditions, comparing against the single-session baseline
- Open a pull request contributing our augmentation improvements back to [autorope/donkeycar](https://github.com/autorope/donkeycar)

### Nice to Have
- Train separate morning- and night-only baseline models (in addition to our single-session baseline) as a reference point, showing how single-session models perform outside their own lighting condition
- Improve DonkeyCar's documentation/code comments so Claude reasons better about the codebase, and compare suggestion quality with vs. without that improved documentation
- Prompt Claude for open-ended, novel augmentation strategies beyond our initial list
- Document whether single-session-plus-augmentation can reliably replace multi-session data collection for future cohorts, regardless of which time of day that single session is recorded

---

## Accomplishments

*(To be filled in as the project progresses)*

**What we have done:**
-

**Augmentations implemented:**

We extended DonkeyCar's `ImageAugmentation` pipeline (built on the [albumentations](https://albumentations.ai/) library) with three new augmentations targeting our specific failure modes: shadows, saturation shifts, and color-temperature differences between sessions.

| Augmentation | Targets | Config Parameter(s) |
|---|---|---|
| `SHADOW` | Random shadow regions (tree/building shadows) | `AUG_NUM_SHADOWS_LIMIT` |
| `HUE_SAT` | Saturation/hue shifts between sessions | `AUG_HUE_SHIFT_LIMIT`, `AUG_SAT_SHIFT_LIMIT`, `AUG_VAL_SHIFT_LIMIT` |
| `RGB_SHIFT` | Color-temperature shifts (warm morning light vs. cool/artificial night lighting) | `AUG_R_SHIFT_LIMIT`, `AUG_G_SHIFT_LIMIT`, `AUG_B_SHIFT_LIMIT` |

These build on DonkeyCar's existing `BRIGHTNESS` and `BLUR` augmentations, which were already part of the default pipeline. Code: [`donkeycar/pipeline/augmentations.py`](https://github.com/wona-evelyn/donkeycar/blob/main/donkeycar/pipeline/augmentations.py) in [our fork](https://github.com/wona-evelyn/donkeycar).

**Models trained and tested:**

Our core comparison is between the two single-session models (Model 1 vs. Model 2, both trained on midday data as our chosen single session) — the central question of this project. The morning/night single-session baselines are a supporting reference point showing how a model performs entirely outside its own recorded lighting condition.

| | Augmentations | Trained On | Tested On | Result |
|---|---|---|---|---|
| **Model 1 — Single-Session Baseline (midday)** | Default (none added) | midday only | morning, midday, night | |
| **Model 2 — Single-Session + Augmentation (midday)** | `SHADOW`, `HUE_SAT`, `RGB_SHIFT` | midday only | morning, midday, night | |
| Model 3 — Morning Baseline *(reference)* | Default (none added) | morning only | morning, midday, night | |
| Model 4 — Night Baseline *(reference)* | Default (none added) | night only | morning, midday, night | |

**Gantt chart changes / lessons learned on timeline:**
-

---

## Challenges

**What worked:**
-

**What did not work as expected, and why:**
-

**How would we solve the issue(s):**
-

**If we had another week, we would:**
-

---

## Final Project Videos

### Final Presentation Slides
*(Link to slides)*

### Video
*(Movie or live demonstration link)*

---

## Acknowledgements

- [DonkeyCar](https://github.com/autorope/donkeycar) — open-source autonomous vehicle framework
- [Claude Code](https://claude.com/claude-code) — AI coding agent used throughout development
- DSC 190 course staff, UC San Diego Halıcıoğlu Data Science Institute

---

## Contacts

| Name | Email |
|---|---|
| Evelyn Na | woojoona0720@gmail.com |
| Bora Vanli | |
| Roberto Huizar | |
