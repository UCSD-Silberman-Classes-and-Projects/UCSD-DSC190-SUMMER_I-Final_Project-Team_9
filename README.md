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

DonkeyCar's default image-only CNN pipeline (no GPS) struggles to generalize across changing lighting conditions. Our CNN relies purely on camera images - there's no GPS localization to fall back on. That means the model's only signal for staying on track is what it sees, and what it sees changes dramatically throughout the day: shifting shadows, brightness, and saturation. In prior coursework, this meant a model trained in the morning simply couldn't drive at night, and vice versa, forcing us to train three separate models - one per time of day. That workaround got us through the original assignment, but it isn't a real solution: it doesn't scale, and it sidesteps the actual problem - the model was never taught to generalize in the first place.

This project tackles that problem directly. Using data augmentation, guided by Claude as an AI coding agent, we aim to train a single, unified CNN that generalizes across all lighting conditions, and to contribute our augmentation improvements back to the open-source [DonkeyCar](https://github.com/autorope/donkeycar) project via a pull request.

---

## What We Promised

### Must Have
- Establish a baseline using the existing augmentation GUI on morning/midday/night data
- Feed Claude the codebase and previously-tried augmentations; implement and test its proposed improvements
- Test the unified model across all three lighting conditions vs. the original per-time-of-day baselines
- Open a pull request contributing our augmentation improvements back to [autorope/donkeycar](https://github.com/autorope/donkeycar)

### Nice to Have
- Improve DonkeyCar's documentation/code comments so Claude reasons better about the codebase, and compare suggestion quality with vs. without that improved documentation
- Prompt Claude for open-ended, novel augmentation strategies beyond our initial list
- Evaluate which novel augmentations meaningfully improve real-world driving performance

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

| | Augmentations | Trained On | Result |
|---|---|---|---|
| **Model 1** | original augumentations — baseline | morning + midday + night | |
| **Model 2** | SHADOW, HUE_SAT, RGB_SHIFT | morning + midday + night | |
| **Model 3** | *(e.g.,??)* | morning + midday + night | |

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
| Evelyn Na | | woojoona0720@gmail.com
| Bora Vanli | | 
| Roberto Huizar | |
