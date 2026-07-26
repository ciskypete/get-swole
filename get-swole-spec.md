# Get Swole — Gym Workout Tracker

## Build Spec for Claude Code

### What This Is

A single-page web app that serves as a gym workout tracker based on the MusclePharm "Get Swole" program. It's a 20-week program split into 5 phases (4 weeks each), with 4–6 training days per week depending on phase. The app needs to work great on a phone, load fast, persist data to GitHub, and look sharp with a dark theme.

Deploy to GitHub Pages. Use the GitHub API to persist data as a JSON file in the same repo.

---

## Architecture

### Single HTML File
Everything lives in one `index.html` — HTML, CSS, JS. No build step, no framework, no dependencies. Vanilla JS is fine. The artifact this is based on was React, but for simplicity and zero-build deployment, plain HTML/CSS/JS is better here.

### Data Persistence via GitHub API
- On app load: fetch `data.json` from the repo via GitHub API
- On any change (checkbox toggle, weight entry): commit updated `data.json` back to the repo
- Use a GitHub Personal Access Token stored in `localStorage` (one-time setup screen)
- The token only needs `repo` scope for a public repo, or `repo` for private
- Debounce saves — don't commit on every single checkbox tap. Wait 2–3 seconds of inactivity, then save. Show a subtle "saving..." / "saved" indicator.

### First-Time Setup Flow
On first load (no token in localStorage), show a simple setup screen:
1. "Paste your GitHub Personal Access Token"
2. "Enter your GitHub username"  
3. "Enter your repo name"
4. Save to localStorage, then load the app

Include brief instructions on how to create a token (link to GitHub's token page).

### `data.json` Structure

```json
{
  "lastUpdated": "2026-07-26T14:30:00Z",
  "sessions": {
    "p1-d0-o0-w3": {
      "checked": {
        "0-0": true,
        "0-1": true,
        "1-0": true
      },
      "weights": {
        "0-0": 135,
        "0-1": 185,
        "1-0": 50
      }
    }
  }
}
```

Key format: `p{phaseId}-d{dayIdx}-o{optionIdx}-w{weekNumber}`
- `weekNumber` is 1–4 within each phase (user picks or it auto-increments)
- `checked` maps `{groupIdx}-{exerciseIdx}` to boolean
- `weights` maps same keys to numbers (lbs)

---

## UI Design

### Color Scheme (match exactly)
```
Background:        #0a0a0a
Surface:           #141414
Card:              #1a1a1a
Green (primary):   #8BC34A
Green dark:        #689F38
Green glow:        rgba(139,195,74,0.15)
Text:              #e8e8e8
Text dim:          #888888
Text muted:        #555555
Border:            #2a2a2a
Superset color:    #8BC34A (green)
Tri-set color:     #FF9800 (orange)
Gauntlet color:    #f44336 (red)
28 Method color:   #9C27B0 (purple)
Circuit color:     #00BCD4 (teal)
```

### Typography
- Primary font: `JetBrains Mono` from Google Fonts for headings, labels, set/rep counts
- Body/system: `-apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif`
- Exercise names: 15px, weight 500, JetBrains Mono
- Set/rep values: 13px, weight 600, colored by group type
- Labels/phase info: 11–12px

### Layout (mobile-first, max-width 540px centered)
The overall structure from top to bottom:

#### Sticky Header
- **Title**: "GET SWOLE" (GET in green, SWOLE in white)
- **Subtitle**: "MUSCLEPHARM × BODYBUILDING.COM" in dim text
- **RESET button**: top right, subtle bordered button, clears current day/week
- **Phase selector**: row of buttons P1–P5, active phase gets green background
- **Phase info bar**: green-tinted box showing "Weeks 1–4 — Foundation: Supersets, no rest between pairs" etc.
- **Week selector**: buttons W1–W4 so the user can pick which week of the phase they're on
- **Day selector**: row of buttons MON/TUE/WED/THU (and FRI/SAT for phases 4–5), active day gets green border + glow background

#### Day Content (scrollable)
- **Day title**: "Monday — Chest + Back" with Monday in green
- **Workout option toggle** (Phase 4 only): two buttons for "Workout #1" and "Workout #2"
- **Progress bar**: shows X/Y exercises done, 4px green bar, "COMPLETE" label when full
- **Exercise groups**: stacked cards with 10px gap between them

#### Exercise Group Card
- Cards have `border-radius: 12px`, dark background (#1a1a1a)
- If the group has a type (Superset, Tri-Set, etc.), show a colored left border (3px) and a labeled header bar with a glowing dot
- If no type, show a subtle gray left border
- When all exercises in a group are checked, the card fades to 60% opacity

#### Exercise Row (inside a card)
- Tappable — entire row is the click/tap target
- **Checkbox**: 22px square, rounded 6px, border colored by group type when checked, checkmark SVG inside
- **Exercise name**: to the right of checkbox
- **Set/rep info**: below the name, colored by group type
- **Tip** (if any): small italic pill next to set info (e.g., "5-sec twist, then 4 more")
- When checked: row fades to 45% opacity, name gets strikethrough
- **Weight input**: small field below or to the right of the set info. Shows "Last: 135 lbs" if there's data from the previous week. Tapping opens a number input. If no previous data, shows a subtle "＋ lbs" placeholder.

### Weight Tracking Behavior
- Weight input is optional — never required, never blocks checking off
- When entering a weight, use a number input with `inputmode="numeric"` for phone keyboard
- Display previous week's weight as a reference ("Last: 135"). Pull from `w{n-1}` data for same phase/day/exercise
- If phase is week 1, no "Last" shown (no prior data)
- Inline editing — tap the weight area to edit, tap away or press enter to confirm

### Save/Sync Indicator
- Small text in the header area: "Saved ✓" / "Saving..." / "Offline" 
- Debounce commits to 2–3 seconds after last interaction

---

## Workout Data

All 5 phases of exercise data are below. Bake this directly into the JS as a data structure.

### Phase 1 — Weeks 1–4
"Foundation — Supersets, no rest between pairs"

**Monday: Chest + Back**
- SUPERSET: Wide Grip Pull-Ups (5 × failure) + Flat Bench (5 × 12)
- SUPERSET: Incline Dumbbells (5 × 12) + Seated Row (5 × 12)
- SUPERSET: Wide Grip Pull-Ups (5 × failure) + Chest Fly (5 × 12)
- Single Dumbbell Pull-Over Across Bench (6 × 12)

**Tuesday: Legs + Abs**
- SUPERSET: Squats (5 × 12) + Leg Press (5 × 12)
- SUPERSET: Stiff Leg Dead Lift (5 × 12) + Hamstring Curl (5 × 12)
- SUPERSET: Barbell Lunges (5 × 12) + Weighted Calf Raises (5 × 12)
- Ab Wheel (50 reps)

**Wednesday: Arms**
- SUPERSET: Arnold Cheat Curls — Straight Bar (6 × 8) + Incline French Press (6 × 8)
- SUPERSET: Incline Alternating DB Curls (5 × 6, tip: "5-sec twist, then 4 more") + Straight Bar Push-Downs (5 × 20, tip: "20 × ¼ rep")
- SUPERSET: Bench Dips (5 × failure) + Preacher Curl Machine (5 × 30)

**Thursday: Shoulders + Abs**
- SUPERSET: Military Press — Bar (5 × 12) + Upright Rows (5 × 12)
- SUPERSET: Lateral Raises (5 × 12) + Full Frontals (5 × 12)
- Ab Wheel (1 × 100)

Fri–Sun: Rest + Recover

### Phase 2 — Weeks 5–8
"Pyramid sets + cardio on rest days"

**Monday: Chest + Back**
- Under Grip Pull-Ups (4 × failure)
- Incline Bench Press (12, 10, 8, 6)
- Seated Row (12, 10, 8, 8, 8)
- Flat Bench (5 × 5)
- Wide Grip Pulldowns (15, 12, 10)
- Chest Fly (5 × 5, tip: "4 count stretch at bottom")

**Tuesday: Legs**
- Squats (20, 15, 12, 10, 8)
- Leg Press (20, 15, 12, 10, 8)
- Leg Extension (25, 20, 15, 10)
- Hamstring Curl (20, 15, 10, 5 × 5)
- Calf Raises (25, 20, 25, 20)

**Wednesday: Arms**
- SUPERSET (Larry Scott Preacher Curls Circuit): 5 × 5 full + 5 × 5 half (tip: "DB first, straight bar second, reverse curl third")
  - → Dumbbells (5 full, then 5 half)
  - → Straight Bar (5 full, then 5 half)
  - → Reverse Curls (5 full, then 5 half)
- Bicep Curl Machine (30 reps)

**Thursday: Shoulders + Abs**
- Military Press — Bar (15, 12, 10)
- Military Press — Dumbbells (12, 10, 8)
- Lateral Raises (20, 15, 12, 10)
- Full Frontals (5 × 5)
- Barbell Shrug (5 × 5, tip: "5 count top & bottom")
- Abs (100 reps)

Fri–Sun: Rest + Recover + Cardio (20–30 min intervals: 1 min hard / 1 min easy)

### Phase 3 — Weeks 9–12
"Tri-sets — 3 exercises back-to-back, no rest"

**Monday: Chest + Back**
- TRI-SET: Dumbbell Press (4 × 12) + Incline Dumbbell Press (4 × 12) + Dumbbell Fly (4 × 12)
- TRI-SET: Flat Bench (4 × 12) + Cable Crossover (4 × 15) + Incline Dumbbell Fly (4 × 12)
- TRI-SET: Pull-Ups (4 × 15) + Dumbbell Pull-Overs (4 × 15) + Seated Rows (4 × 15)
- TRI-SET: Pull-Downs (4 × 15) + T-Bar Row (4 × 15) + Stiff Arm Cable Crossover (4 × 15)

**Tuesday: Legs**
- TRI-SET: Squats (3 × 15) + Leg Press (3 × 15) + Leg Extension (3 × 15)
- TRI-SET: Stiff Leg Deadlift (3 × 15) + Hamstring Curls (3 × 15) + Walking Lunges (3 minutes)

**Wednesday: Arms**
- TRI-SET: Preacher Curls (4 × 15) + Forehead Curls (4 × 15) + Hammer Curls (4 × 15)
- TRI-SET: 3-Way Skull Crushers (3×20 nose / 3×20 forehead / 3×20 behind) + Close Grip Preacher Curls (30 reps) + Straight Bar Push-Downs (3 × 30)

**Thursday: Shoulders + Abs**
- Arnold Press (4 × 20)
- TRI-SET: Dumbbell Military Press (4 × 20) + Lateral Raises (4 × 20) + Front Raises (4 × 20)
- Shrugs (20 reps, tip: "2 count top & bottom")
- Ab Wheel (100 reps)

Fri–Sun: Rest + Recover + Cardio (20–30 min intervals)

### Phase 4 — Weeks 13–16
"Pick one of two workouts per day — 6 training days"

Each day has TWO workout options. User picks one via toggle.

**Monday: Chest + Back**

*Workout #1:*
- SUPERSET: Wide-Grip Chin-Ups (5 × 15) + Flat Bench (5 × 12, tip: "Slow, 3 count down & up")
- SUPERSET: Under-Grip Chin-Ups (5 × 12) + Incline Barbell Press (5 × 12, tip: "Slow, 3 count down & up")
- TRI-SET: Chest Fly (4 × 15) + Dips (4 × 12) + Alternate: DB Pull-Over / Cable Cross / Hammer Machine (3–5 × 15)
- Abs — choose one: Weighted Crunch / Ab Wheel / Kneeling Crunch (100 reps)

*Workout #2:*
- SUPERSET: Bench Press (5 × 12) + T-Bar Rows (5 × 12)
- SUPERSET: Incline Barbell Press (5 × 12) + Under-Grip Weighted Chin-Ups (5 × 12)
- SUPERSET: Dumbbell Pull-Over (5 × 12) + Cable Crossover (5 × 20)
- Abs — choose one (100 reps)

**Tuesday: Arms**

*Workout #1:*
- SUPERSET: Straight Bar Curls (5 × 15) + Bench Dips (5 × 12)
- SUPERSET: Incline Curls (5 × 8, tip: "5 count twist then 4 more") + Bench Dips (5 × 30)
- SUPERSET: Preacher Curls (5 × 12) + Tricep Push-Down (5 × 20)
- Forearm Curls (3 × 20)
- Abs — choose one (100 reps)

*Workout #2 — Arm Gauntlet (5 rounds through all):*
- Tricep Push-Down (20 reps)
- Tricep Band Press-Down (20 reps)
- Straight Bar Curls (15 reps)
- Preacher Curl (15 reps)
- Skull Crusher (20 reps)
- Bench Dips (20–30 reps)
- Incline DB Curls (8 reps, tip: "twist 5 count, then 4 more")
- Abs — choose one (100 reps)

**Wednesday: Legs**

*Workout #1:*
- Squat (12 × 12)
- Leg Extensions (10 × 12)
- Hamstring Curls (10 × 10, tip: "Heavy as possible")
- Seated Calf Machine (28 method, 2–3 sets)
- Standing Calf Machine (28 method, 2–3 sets)
- Abs — choose one (100 reps)

*Workout #2:*
- Heavy Deep Squats (8 × 8)
- Leg Press (4 × 20)
- SUPERSET: Leg Extensions (4 × 15) + Hamstring Curl (4 × 15)
- SUPERSET: Seated Calf Machine (28 method, 2–3 sets) + Standing Calf Machine (28 method, 2–3 sets)
- Abs — choose one (100 reps)

**Thursday: Chest + Back** — Same two options as Monday

**Friday: Arms** — Same two options as Tuesday

**Saturday: Legs**

*Workout #1:* Same as Wednesday Workout #1

*Workout #2:*
- Heavy Deep Squats (8 × 8)
- Leg Press (4 × 20)
- SUPERSET: Leg Extension (4 × 15) + Hamstring Curl (4 × 15)
- SUPERSET: Seated Calf Machine (5 × 15) + Standing Calf Machine (5 × 15)
- Abs — choose one (100 reps)

Sunday: Rest + Recover

### Phase 5 — Weeks 17–20
"5×5 heavy + 28 method finisher on each lift"

The 28 method: 7 full reps → 7 super slow reps → 7 top-half reps → 7 bottom-half reps = 28 total.

**Monday: Chest / Triceps / Abs**
- 28 METHOD: Incline Barbell Bench Press (5 × 5, then 1 × 28 method)
- 28 METHOD: Flat Bench Press (5 × 5, then 1 × 28 method)
- 28 METHOD: Flat DB Fly (5 × 5, then 1 × 28 method, tip: "4 count stretch")
- Push-Ups (1 × 100)
- SUPERSET: Incline Bench Bodyweight Skulls (5 × 15) + Bench Dips (5 × 20)
- SUPERSET: Ab Wheels (4 × 25) + Kneeling Cable Crunches (4 × 25) + Weighted Crunches (4 × 25)

**Tuesday: Back / Biceps / Abs**
- 28 METHOD: T-Bar or Bent Over Rows (5 × 5, then 1 × 28 method)
- Pull-Ups Wide Weighted (5 × 8)
- 28 METHOD: Pull-Down (1 × 28 method)
- 28 METHOD: Seated Rows (5 × 5, then 1 × 28 method)
- Arnold Chest Curls (5 × 5)
- 28 METHOD: Barbell Curls (1 × 28 method)
- Dave Draper Forehead Curls (5 × 5)
- 28 METHOD: Barbell Curls (1 × 28 method)
- SUPERSET: Hanging Knee-Ups (5 × 15) + Toes to Bar (5 × 15)

**Wednesday: Legs / Abs**
- 28 METHOD: Squats (5 × 5 deep, then 1 × 28 easy squat)
- 28 METHOD: Dead Lifts (5 × 5, then 1 × 28 back extensions)
- 28 METHOD: Leg Extensions (5 × 5, then 1 × 28 method)
- 28 METHOD: Leg Curls (5 × 5, then 1 × 28 method)
- Calf Raises Weighted (30, 20, 10, 5)
- Decline Sit-Ups (4 × 25–50)
- Hanging Knee-Ups (3 × 25–50)
- Kneeling Cable Crunches (4 × 12)
- Stick Twists (3 × 50)

**Thursday: Shoulders / Abs**
- 28 METHOD: Military Press Barbell (5 × 5, then 1 × 28)
- 28 METHOD: Shrugs (5 × 5, then 1 × 28 method)
- 28 METHOD: Rear Delt Flys (5 × 8–12, then 1 × 28 method)
- 28 METHOD: Lateral Raises (5 × 5, then 1 × 28 method)
- 28 METHOD: Front Raises (5 × 5, then 1 × 28 method)
- Full Lateral Raises (1 × 100)
- Weighted Crunches (4 × 25)

**Friday: Cardio / Abs**
- CIRCUIT: Jump Rope (2 min) → Double Unders (50) → Jump Rope (2 min) → Double Unders (40) → Jump Rope (2 min) → Double Unders (30) → Jump Rope (2 min) → Double Unders (20)
- Decline Sit-Ups (4 × 25–50)
- Hanging Knee-Ups (3 × 25–50)
- Kneeling Cable Crunches (4 × 12)
- Stick Twists (3 × 50)

Sat–Sun: Rest + Recovery

---

## Deployment Checklist for Claude Code

**Repo:** `ciskypete/get-swole` (https://github.com/ciskypete/get-swole) — already created, you have write access.

1. Build `index.html` with all code inline in the root of `ciskypete/get-swole`
2. Create an initial empty `data.json`: `{"lastUpdated": null, "sessions": {}}`
3. Enable GitHub Pages on the repo (main branch, root)
4. Final URL will be: `https://ciskypete.github.io/get-swole/`
5. Test that the page loads, setup screen works, data saves and loads

---

## Key Implementation Notes

- **Mobile-first**: viewport meta tag, touch targets at least 44px, `inputmode="numeric"` on weight fields
- **Add to Home Screen ready**: include a `<link rel="manifest">` with a simple manifest for PWA-like behavior (name, icons, display: standalone, theme color #0a0a0a)
- **Offline fallback**: cache the workout data in localStorage as well so the app loads even without network. Only the save/load of user data needs the API. If offline, queue saves and sync when back online.
- **No delete protection**: the RESET button should confirm before wiping ("Reset this day?")
- **Week selector**: W1–W4 buttons below the phase selector. This determines which week's data slot you're writing to. Defaults to W1.
- **The original PDF is included in this repo** for reference: `bb_getswole_workout_.pdf`
