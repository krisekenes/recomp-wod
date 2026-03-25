# RECOMP — WOD Training System

A personal workout app built around a 5-WOD recomposition program targeting hypertrophy, fat loss, and posterior chain development. Designed to be used alongside GLP-1 medication with a specific focus on thighs, glutes, and midsection.

Live at: **https://krisekenes.github.io/recomp-wod**

---

## The Program

**Format:** 45s work / 15s rest (default: 60s / 15s) · 4–5 rounds · 60s rest between rounds

| Day | WOD | Focus |
|-----|-----|-------|
| Mon | WOD 1 — "The Foundation" | Thighs, Glutes & Explosiveness |
| Tue | WOD 2 — "The Engine" | Push, Pull & Midsection Stability |
| Wed | WOD 3 — "The Hinge" | Posterior Chain & Deep Core |
| Thu | — | Rest |
| Fri | WOD 4 — "The Metabolic Torch" | Max Caloric Burn & Stamina |
| Sat | WOD 5 — "The Finisher" | Hypertrophy & Midsection Volume |
| Sun | — | Rest |

### Burn-It-Down Finishers (3 min, post-WOD)
- **"Midsection Vacuum"** — KB Suitcase Carries + Hollow Body Rocks
- **"Thigh & Glute Pump"** — Goblet Squat Pulses + Reverse Lunges + Wall Sit
- **"Metabolic Redline"** — KB Swings + Burpees + KB Swings

---

## App Features

- **Interval Timer** — configurable work/rest/round-rest durations, inline on the timer screen
- **Two Workout Modes**
  - **Circuit** — cycle all 6 movements per round, repeat for X rounds
  - **Station** — complete all sets of one movement before moving to the next
- **Rep Tracking** — log reps per movement per round with +/− controls
- **Beat the Book** — from Round 2 onward, shows your Round 1 rep count as a target
- **Finisher Selection** — choose a 3-min finisher to queue before starting
- **Session Log** — persisted to localStorage; tracks sessions, total time, streak, favourite WOD
- **Streak Badge** — live streak count in the header
- **Sound Alerts** — countdown beeps and phase-change tones via Web Audio API
- **Weekly Schedule** — shows today's prescribed WOD

---

## Tech Stack

- Vanilla HTML / CSS / JavaScript — zero dependencies, zero build step
- Web Audio API for sound
- `localStorage` for session persistence
- GitHub Actions for CI/CD deployment to GitHub Pages

---

## Deployment

Pushes to `main` automatically deploy to GitHub Pages via `.github/workflows/deploy.yml`.

To deploy manually:
```bash
git clone https://github.com/krisekenes/recomp-wod
cd recomp-wod
# make changes to index.html
git add . && git commit -m "update" && git push
```

GitHub Pages will rebuild within ~60 seconds.

---

## Local Development

No build step required — just open `index.html` in a browser:

```bash
open index.html
# or
python3 -m http.server 8080
```
