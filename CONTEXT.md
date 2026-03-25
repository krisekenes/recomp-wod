# RECOMP WOD — Project Context

> This file is intended for AI assistants, future collaborators, and as a personal reference. It captures the design intent, architecture decisions, and program rationale behind this app.

---

## Why This App Exists

Built as a personal training companion for a body recomposition program running alongside GLP-1 medication (Ozempic/Wegovy). The user was experiencing a weight-loss plateau and needed:

1. A structured high-density interval program targeting thighs, glutes, and midsection
2. A gamification layer ("Beat the Book") to keep intensity high during later rounds
3. A simple, offline-capable tool that works on mobile without app store friction

---

## Program Design Rationale

### The "Recomp" Goal
- Primary: fat loss (particularly visceral/midsection) + lean muscle gain
- Secondary: posterior chain development (glutes, hamstrings, lower back)
- Constraint: must keep heart rate in fat-burning zone while providing enough mechanical tension for hypertrophy

### Why 6 Movements / 45–60s Intervals
- 6 movements per session increases total volume vs traditional 3–4 movement WODs
- 45–60s work intervals maximise time under tension for hypertrophy while maintaining metabolic stress
- 15s rest is short enough to keep heart rate elevated (fat burning) but sufficient for partial recovery
- 4–5 rounds provides ~25–35 minutes of actual work time per session

### The Finishers
3-minute post-WOD intensifiers targeting "post-activation potentiation" — exhausting specific muscle fibres so the body works overtime during recovery (increasing metabolic afterburn).

### GLP-1 Plateau Strategy
The "Beat the Book" feature directly addresses the plateau mechanism: on GLP-1, appetite suppression can lead to insufficient training intensity. Tracking Round 1 reps and targeting them in later rounds ensures intensity stays high enough to force fat oxidation.

---

## Architecture

### Single-file app (`index.html`)
Deliberate choice: zero build tooling, no dependencies, deployable anywhere, works offline after first load. The entire app is one HTML file with embedded CSS and JS.

### State Model
```
selectedWOD       — which of the 5 WODs is active
selectedFinisher  — optional 3-min finisher queued (null | 0|1|2)
workoutMode       — 'circuit' | 'station'
settings          — { work, rest, roundRest, rounds, sound }
currentPhase      — 'work' | 'rest' | 'roundRest'
currentMovement   — index into selectedWOD.movements[]
currentRound      — 1-based round counter
repHistory        — { "round_movIndex": reps } for Beat the Book
workoutLog        — persisted to localStorage, array of session entries
```

### Circuit vs Station Mode
Two fundamentally different traversal patterns over the same movement/round grid:

**Circuit** (default — original WOD intent):
```
Round 1: Mov1 → rest → Mov2 → rest → ... → Mov6 → roundRest
Round 2: Mov1 → rest → Mov2 → ... → Mov6 → roundRest
...
```

**Station** (same-movement block training):
```
Mov1: Set1 → rest → Set2 → rest → ... → SetN → roundRest
Mov2: Set1 → rest → Set2 → ... → SetN → roundRest
...
```

Station mode is better for strength-focused sessions where you want full fatigue of one pattern before switching.

### Timer Engine
- `setInterval(tick, 1000)` drives the clock
- `nextPhase()` handles all state transitions — deliberately kept as one function to make the circuit/station branching explicit and auditable
- Rounds are locked once `sessionStartTime` is set — prevents mid-session round count bugs
- Web Audio API (`AudioContext`) generates beeps programmatically (no audio files needed)

### Persistence
- `localStorage` only — no backend, no accounts
- Key: `recompLog` → JSON array of `{ date, wod, wodName, wodTag, color, rounds, duration }`
- Streak is computed at render time from the log, not stored separately

---

## WOD Data Structure

```javascript
{
  id: 1,
  name: '"The Foundation"',
  tag: 'WOD 01',
  focus: 'Thighs, Glutes & Explosiveness',
  color: '#e8ff3a',         // accent color used throughout UI for this WOD
  day: 'MON',               // prescribed day
  movements: [
    {
      name: 'Barbell Back Squats',
      equip: 'BARBELL',
      cue: '...'            // coaching cue shown during the work interval
    },
    // × 6
  ]
}
```

---

## Known Limitations / Future Work

- [ ] No cloud sync — log is device-local only
- [ ] No weight tracking per movement (future: log kg/lbs alongside reps)
- [ ] No rest-day active recovery programming
- [ ] Finisher timer is manual (user runs it mentally) — could be a dedicated 3-min countdown mode
- [ ] No progressive overload tracking across sessions
- [ ] Beat the Book only compares to Round 1 — could compare to personal best across sessions

---

## File Structure

```
recomp-wod/
├── index.html          # Entire app — HTML + CSS + JS
├── README.md           # Public-facing documentation
├── CONTEXT.md          # This file — design intent and architecture
└── .github/
    └── workflows/
        └── deploy.yml  # GitHub Actions → GitHub Pages CI/CD
```

---

## Exercise Image Follow-up

**Source:** `yuhonas/free-exercise-db` — public domain, no API key, hosted on GitHub raw CDN.
**Base URL:** `https://raw.githubusercontent.com/yuhonas/free-exercise-db/main/exercises/`
**Format:** Each exercise folder has `0.jpg` (start position) and `1.jpg` (end position).
**Animation:** Alternate between frames on a ~900ms interval to simulate a GIF.

### EXERCISE_IMAGES map (42 movements — ready to implement)

```javascript
const EXERCISE_IMAGES = {
  // WOD 1 — The Foundation
  'Barbell Back Squats':            'Barbell_Full_Squat',
  'Kettlebell Swings':              'Kettlebell_sumo_high_pull',
  'Dumbbell Reverse Lunges':        'Dumbbell_Rear_Lunge',
  'Barbell Romanian Deadlifts':     'Barbell_Romanian_Deadlift',
  'Kettlebell Goblet Squats':       'Kettlebell_Goblet_Squat',
  'Jump Squats':                    'Jump_Squat',
  // WOD 2 — The Engine
  'Barbell Overhead Press':         'Barbell_Shoulder_Press',
  'Dumbbell Renegade Rows':         'Dumbbell_Renegade_Row',
  'Kettlebell Clean & Press':       'Kettlebell_Clean_and_Press',
  'Barbell Bent-Over Rows':         'Barbell_Bent_Over_Row',
  'Dumbbell Floor Press':           'Dumbbell_Floor_Press',
  'Weighted Russian Twists':        'Russian_Twist',
  // WOD 3 — The Hinge
  'Barbell Deadlifts':              'Barbell_Deadlift',
  'KB Suitcase Carry':              'Farmers_Walk',
  'Dumbbell Stiff-Leg Deadlifts':   'Dumbbell_Romanian_Deadlift',
  'KB Goblet Lateral Lunges':       'Lateral_Lunge',
  'Barbell Glute Bridges':          'Barbell_Hip_Thrust',
  'Hollow Body Hold':               'Hollow_Body_Hold',
  // WOD 4 — The Metabolic Torch
  'Barbell Thrusters':              'Barbell_Thruster',
  "Dumbbell Devil's Press":         'Dumbbell_Burpee',
  'KB American Swings':             'Kettlebell_sumo_high_pull',
  'Barbell Power Cleans':           'Barbell_Power_Clean',
  'DB Weighted Step-ups':           'Dumbbell_Step_Up',
  'Burpees Over the Bar':           'Burpee',
  // WOD 5 — The Finisher
  'Barbell Front Squats':           'Barbell_Front_Squat',
  'Dumbbell Goblet Lunges':         'Dumbbell_Goblet_Squat',
  'KB Single-Arm Snatch':           'Kettlebell_Snatch',
  'Barbell Upright Rows':           'Barbell_Upright_Row',
  'DB Curl to Press':               'Dumbbell_Bicep_Curl',
  'Plank to Pike':                  'Inchworm',
  // WOD 6 — The Asymmetry
  'Single-Leg Romanian Deadlift':   'Dumbbell_Single_Leg_Deadlift',
  'Bulgarian Split Squats':         'Dumbbell_Bulgarian_Split_Squat',
  'Single-Arm KB Row':              'Kettlebell_One_Arm_Row',
  'Pistol Squat Progression':       'Bodyweight_Squat',
  'Single-Arm KB Press':            'Kettlebell_One_Arm_Press',
  'Pull-Up Negatives':              'Pullup',
  // WOD 7 — The Restore
  'Barbell Good Mornings':          'Barbell_Good_Morning',
  "DB Farmer's Carry":              'Farmers_Walk',
  'Dead Hang':                      'Pullup',
  'KB Halo':                        'Kettlebell_Halo',
  'Glute Bridge (Bodyweight)':      'Glute_Bridge',
  'Bird Dog':                       'Bird_Dog',
};
```

### Implementation checklist
- [ ] Add `ex-image-wrap` HTML to `current-ex` div in timer view
- [ ] Add CSS for `.ex-image-wrap`, `.img-frame-a/b`, `.ex-image-placeholder`
- [ ] Add `loadExerciseImages(name)` function with 900ms flip interval
- [ ] Call `loadExerciseImages(mv.name)` in `updateMovementDisplay()`
- [ ] Add `clearInterval(imageFlipInterval)` to `resetTimerState()`
- [ ] Verify folder names against the live repo for any mismatches
- [ ] Handle `onerror` gracefully with placeholder fallback
