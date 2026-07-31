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

**Project timeline, in order:**

1. **Started Track 1** — Making the CNN More Robust with Data Augmentation, reframed around a single question: can one session, well augmented, drive reliably at any time of day?
2. **Uncropped baseline** — trained on midday data only, using DonkeyCar's default augmentation settings. Succeeded on midday (its own training conditions); failed on morning and night, as expected for an unaugmented single-session model.
3. **Progressive augmentation with Claude Code** — used Claude to analyze our own captured driving images (not just the codebase) and identify real, evidence-based failure modes, then implemented augmentations one at a time and retrained after each addition:
   - `FinalProjectModel1` (`SHADOW`, `HUE_SAT`, `RGB_SHIFT`) — succeeded on midday only
   - `FinalProjectModel2` (+ `SUNFLARE`, `GAMMA`, `CLAHE`) — succeeded on midday only
   - `FinalProjectModel3` (+ `MOTION_BLUR`, all 7 augmentations) — succeeded on midday, **partially** succeeded on night
4. **Scaling back augmentations** — suspecting that stacking too many augmentations at once was making training harder rather than helping, we pared the set down to the augmentations with the strongest evidence behind them (`HUE_SAT`, `RGB_SHIFT`, `SUNFLARE`, `GAMMA`, `CLAHE` — dropping `SHADOW`, since our own images showed no real shadows, and `MOTION_BLUR`, which was only confirmed in a single frame) and trained `FinalProjectModel4`. Result: succeeded on midday, partially succeeded on night — comparable to `FinalProjectModel3`, despite using fewer augmentations.
5. **Discovering the real problem** — using Claude to visually sample and measure frames across all three sessions, we found that static background objects (planters, a bench, stairs, curbs, a hedge) extended much deeper into the frame than the horizon alone suggested, meaning our models may have been partly learning from walls and surrounding objects rather than the track itself. We implemented an `ROI_CROP_TOP` fix based on the worst-case object depth measured consistently across sessions.
6. **Retraining with the crop applied** — this is where our results got more complicated rather than better:
   - `Baseline (Cropped)` — **failed even on midday**, its own training condition. The exact cause is still unclear; our leading hypothesis is that cropping out too much of the frame left too little visual signal for the model to learn a reliable track-following policy, but we haven't confirmed this yet.
   - `FinalProjectModel3 (Cropped)` — also failed, consistent with the cropped baseline already being broken beneath it.

**Augmentations implemented:**

We extended DonkeyCar's `ImageAugmentation` pipeline (built on the [albumentations](https://albumentations.ai/) library) with new augmentations targeting failure modes identified by analyzing our own driving images with Claude: shadows, saturation/color-temperature shifts, streetlight glare, local bright/dark contrast, and motion streaking.

| Augmentation | Targets | Config Parameter(s) |
|---|---|---|
| `SHADOW` | Random shadow regions (not observed in our own dataset — later dropped from `FinalProjectModel4`) | `AUG_NUM_SHADOWS_LIMIT` |
| `HUE_SAT` | Saturation/hue shifts between sessions | `AUG_HUE_SHIFT_LIMIT`, `AUG_SAT_SHIFT_LIMIT`, `AUG_VAL_SHIFT_LIMIT` |
| `RGB_SHIFT` | Color-temperature shifts (warm morning light vs. cool/artificial night lighting) | `AUG_R_SHIFT_LIMIT`, `AUG_G_SHIFT_LIMIT`, `AUG_B_SHIFT_LIMIT` |
| `SUNFLARE` | Streetlight glare/blown highlights at night | `AUG_SUNFLARE_SRC_RADIUS`, `AUG_SUNFLARE_NUM_FLARE_CIRCLES_RANGE`, `AUG_SUNFLARE_FLARE_ROI` |
| `GAMMA` | Non-linear midtone exposure differences | `AUG_GAMMA_LIMIT` |
| `CLAHE` | Local contrast (bright/dark coexisting in the same frame) | `AUG_CLAHE_CLIP_LIMIT`, `AUG_CLAHE_TILE_GRID_SIZE` |
| `MOTION_BLUR` | Directional streaking from longer night shutter speed + vehicle motion (confirmed in one frame only — later dropped from `FinalProjectModel4`) | `AUG_MOTION_BLUR_LIMIT` |

These build on DonkeyCar's existing `BRIGHTNESS` and `BLUR` augmentations, which were already part of the default pipeline. Code: [`donkeycar/pipeline/augmentations.py`](https://github.com/wona-evelyn/donkeycar/blob/main/donkeycar/pipeline/augmentations.py) in [our fork](https://github.com/wona-evelyn/donkeycar).

**Models trained and tested:**

All models are trained on a single session (midday) only, to test whether one recording session plus augmentation can generalize across all lighting conditions — our project's central question.

| | Augmentations | Images | Result |
|---|---|---|---|
| **Baseline** | Default (none added) | Uncropped | Succeeded on midday; failed on morning/night (expected) |
| **FinalProjectModel1** | `SHADOW`, `HUE_SAT`, `RGB_SHIFT` | Uncropped | Succeeded on midday only |
| **FinalProjectModel2** | + `SUNFLARE`, `GAMMA`, `CLAHE` | Uncropped | Succeeded on midday only |
| **FinalProjectModel3** | + `MOTION_BLUR` (all 7) | Uncropped | Succeeded on midday; **partially** succeeded on night |
| **FinalProjectModel4** | `HUE_SAT`, `RGB_SHIFT`, `SUNFLARE`, `GAMMA`, `CLAHE` (5, reduced set) | Uncropped | Succeeded on midday; partially succeeded on night |
| **Baseline (Cropped)** | Default (none added) | `ROI_CROP_TOP = 70` | **Failed even on midday** — cause not yet confirmed |
| **FinalProjectModel3 (Cropped)** | All 7 augmentations | `ROI_CROP_TOP = 70` | Failed (same underlying issue as cropped baseline) |

**Gantt chart changes / lessons learned on timeline:**
- Added a full baseline re-collection step mid-project after the camera recalibration, which shifted our original timeline
- Replaced a planned "documentation-first iteration" comparison step with direct image-based diagnosis, which turned out to be a more direct way to validate augmentation choices against real evidence
- Added an extra augmentation-reduction step (`FinalProjectModel4`) after suspecting that stacking too many augmentations at once was hurting rather than helping — this wasn't in our original plan
- Added a dedicated debugging step after discovering our models may have been learning from background objects rather than the track — this led to the `ROI_CROP_TOP` fix, but also introduced a new, still-unresolved problem (the cropped baseline failing) that we didn't anticipate

---

## Challenges

**What worked:**
- Using Claude to analyze our actual captured images (not just the codebase) turned out to be far more reliable than guessing which augmentations "should" help — it caught real failure modes we hadn't noticed (streetlight glare, motion blur) and just as importantly, talked us out of an augmentation we assumed we needed (`ISONoise`) once the data didn't support it
- That same image-based diagnosis approach also caught a more fundamental issue: sampling frames across all three sessions revealed static background objects (planters, benches, curbs) intruding well into what should have been track-only frame content
- Scaling back from 7 to 5 augmentations (`FinalProjectModel4`) achieved comparable results to the full set, suggesting our earlier suspicion — that stacking too many augmentations at once may have been working against us — had some merit
- Verifying each augmentation's parameter names against our exact albumentations version before applying any code caught two breaking API changes (`RandomShadow`, `RandomSunFlare`) that would otherwise have failed silently or thrown confusing errors deep in training

**What did not work as expected, and why:**
- Our uncropped models (`FinalProjectModel1`–`4`) all generalized to midday, and our best ones (`FinalProjectModel3`, `4`) partially generalized to night — but none fully solved the original problem of one model working reliably across all three lighting conditions
- More surprisingly, after identifying that background objects were likely contaminating training and applying an `ROI_CROP_TOP` fix, our cropped baseline **failed even on midday** — worse than the uncropped baseline it was meant to improve on. We don't yet have a confirmed root cause; our current best guess is that the crop may have removed too much of the frame, leaving the model without enough visual context to learn a stable driving policy, but this hasn't been verified
- This means the crop fix, while based on solid evidence (the object-intrusion measurements were consistent across all three sessions), did not translate into a better-performing model — a reminder that a correct diagnosis doesn't automatically guarantee a correct fix

**How would we solve the issue(s):**
- Systematically vary `ROI_CROP_TOP` across several values (rather than jumping straight to the most conservative worst-case estimate) and retrain a baseline at each, to see whether a smaller crop still removes enough background while preserving enough track visibility to train successfully
- Visually inspect a sample of cropped training images directly (not just the crop math) to confirm the track surface is still clearly visible and check for any obvious loss of context
- Compare training loss curves between the uncropped and cropped baselines to see whether the cropped model is failing to converge at all, or converging to a poor policy — that difference would point to different next steps

**If we had another week, we would:**
- Debug the cropped baseline failure first, since it currently blocks us from properly evaluating whether cropping combined with augmentation is the right fix
- Once a working cropped baseline exists, re-run the augmentation comparison (5 vs. 7 augmentations) on top of it
- Repeat the single-session experiment starting from a session recorded at a different time of day (e.g., morning instead of midday), to confirm any result generalizes and isn't specific to midday lighting
- Complete the documentation-first iteration comparison from our original proposal, which we deprioritized in favor of direct image-based diagnosis this time around

**Lessons learned:**

*On working with Claude/AI agents:*
- Having Claude analyze our actual captured images, not just the codebase, led to far more accurate diagnoses than working from assumptions
- Version-sensitive libraries (like albumentations) need their exact installed version checked before applying any AI-suggested code — this caught two breaking API changes before they caused silent failures
- When Claude pushed back on an augmentation we assumed we needed (`ISONoise`), trusting that pushback over our own assumption was the right call — verified evidence should outrank instinct

*On engineering and debugging:*
- A correct diagnosis doesn't guarantee a correct fix — we correctly identified that background objects were contaminating training, but the crop fix we applied based on that diagnosis made performance worse, not better
- Ruling out causes systematically (config, then hardware, then data) was far more efficient than changing multiple things at once and hoping something worked
- More augmentations isn't automatically better — dropping from seven to five augmentations produced comparable results, suggesting weakly-justified augmentations can hurt training rather than help it

*On project management:*
- A mid-project hardware change (camera recalibration) triggered far more downstream work than expected — full data re-collection and baseline re-validation, not just a quick adjustment
- Replacing our original plan (documentation-first iteration) with a more direct approach (image-based diagnosis) turned out to be the right call — plans should flex when a better method becomes obvious mid-project

---

## Final Project Videos

### Final Presentation Slides

https://docs.google.com/presentation/d/1RcToqMur0FWXHRwW5vj3S9iEqYiJRJFE3sEL4WN-_zg/edit?usp=sharing

### Video

Demo video of FinalProjectModel3 auto-driven during midday
- https://drive.google.com/file/d/1xk6jBapw42spCtV5l6vIbB4MbvuVLnxW/view?usp=sharing

Failure video of FinalProjectModel3 auto-driven at night
- https://drive.google.com/file/d/1YHL8rXfIcc2Zc41kUO9KOl3c50I5Yuit/view?usp=sharing

---

## Acknowledgements

- [DonkeyCar](https://github.com/autorope/donkeycar) — open-source autonomous vehicle framework
- [Claude Code](https://claude.com/claude-code) — AI coding agent used throughout development
- DSC 190 course staff, UC San Diego Halıcıoğlu Data Science Institute, UC San Diego Jacobs School of Engineering

---

## Contacts

| Name | Email |
|---|---|
| Evelyn Na | woojoona0720@gmail.com |
| Bora Vanli | bvanli@ucsd.edu |
| Roberto Huizar | rohuizar@ucsd.edu |
