# Workout Coach v5 — Real 3D Mannequin (weekend build plan)

**Target:** Sat 2026-07-11 → Sun 2026-07-12 · 5 focused sessions
**Branch:** `v5-3d` — ALL commits go here. NEVER push to `main` until the final
deploy step (GitHub Pages auto-deploys every push to `main`).

## Session protocol (read this first, every session)

1. `cd "Potfolio Dashboard/workout-coach-repo" && git checkout v5-3d && git pull`
2. Open this file, find the first unchecked task group below.
3. Do ONE session's scope only — do not run ahead even if budget remains.
4. Tick the boxes in this file, commit everything to `v5-3d`, push.
5. Update the "Status log" at the bottom with one line.

Preview locally: `.claude/launch.json` config `workout-coach` serves the project
folder on :5544 — copy repo `index.html` → project `workout-coach.html` to test,
or serve the repo folder directly.

## Locked architecture decisions (do not relitigate)

- **three.js vendored as a separate file** (`three.min.js`) in the repo, loaded
  via `<script>`. The HTML stays fully functional without it: if WebGL or the
  script fails, demos silently stay SVG. 3D is a progressive enhancement with a
  2D/3D toggle in the demo modal — the SVG renderer is NOT removed.
- **Mannequin from primitives** (capsule/sphere meshes per bone), not a loaded
  model file. Same skeleton proportions as the SVG engine (LEN table).
- **Reuse existing pose data.** Side-view exercises map angles directly onto the
  sagittal plane. Exercises that used 2D cheat views (front mode: lateral raise,
  upright row; flye mode) get a small `p3` override per pose adding real
  abduction angles. No exercise gets re-authored from scratch.
- **Camera:** simple custom drag-orbit + pinch zoom (no OrbitControls import),
  slow auto-rotate until first touch.
- **No audio, no tracking, no new storage keys** (except `wc-3d` toggle pref).

## Sessions

### Session 1 — Sat AM: scene foundation ✅ DONE
- [x] three.js r170 vendored as `three.module.min.js` (UMD builds discontinued
      upstream; loaded via a `<script type="module">` bridge that sets
      `window.THREE` and fires `three-ready`) + WebGL check + SVG fallback
- [x] 3D stage: renderer (pixelRatio capped at 2), hemi+directional lights,
      floor disc with PCF soft shadow, transparent canvas over stage gradient
- [x] Custom drag-orbit + pinch zoom (pointer events) + slow idle auto-rotate
- [x] Static standing mannequin from primitives (torso, hips, shoulders, neck,
      head, 2×arm, 2×leg, feet) using LEN proportions, skin/accent materials
- [x] "3D" toggle in demo modal stage (persists in `wc-3d`), Overhead Press
      only via `can3D()` gate; full dispose on toggle-off/close (verified)
- [x] Commit: "v5 s1: three.js scene, mannequin, orbit, modal toggle"

### Session 2 — Sat PM: rig + pose mapping
- [ ] Joint hierarchy (pelvis→torso→head, shoulders→elbows, hips→knees→ankles)
      posed from existing pose fields (torso/ua/fa/th/sh/ft + *2 far-side)
- [ ] Pose interpolation reusing lerpPose/ease; ping-pong loop identical to SVG
- [ ] Verify side-plane exercises animate correctly: bench press, squat,
      deadlift, curl, plank
- [ ] Lying poses: mannequin orientation on bench (torso angle drives whole-body
      rotation, same as SVG bench auto)
- [ ] Commit: "v5 s2: rigged mannequin animates side-plane exercises"

### Session 3 — Sun AM: all 28 exercises + equipment
- [ ] `p3` abduction overrides for ex-front-mode exercises (lateral raises,
      upright row) and flyes — real out-to-the-side arm movement
- [ ] Equipment meshes: dumbbell, deadlift bar plates, EZ bar, bench
      (flat/incline/decline from pose torso angle), support bench/seat props
- [ ] Sweep all 28 exercises in 3D; fix pose glitches; keep a checklist in the
      commit message of any exercise left visually rough
- [ ] Commit: "v5 s3: all exercises in 3D with equipment"

### Session 4 — Sun PM: parity polish + integration
- [ ] Working-muscle glow: emissive pulse on the relevant body mesh (map from
      ex.mus[0], reuse glowPt intent)
- [ ] Movement trace in 3D: line/tube along wrist path + arrow cone (skip when
      short, same 28px rule)
- [ ] Phase labels as HTML overlay on the 3D stage (reuse PH table)
- [ ] Workout mode: 3D stage honored there too when toggle is on
- [ ] Mobile perf pass: pause rendering when modal closed/page hidden, cap
      device pixel ratio, dispose scene on close
- [ ] Commit: "v5 s4: glow, traces, labels, workout mode, perf"

### Session 5 — Sun eve: QA + DEPLOY
- [ ] Full pass: every exercise × 2D/3D toggle × modal + workout mode; fresh
      profile flow; console clean; check total payload size (target < 1 MB)
- [ ] Update footer to v5; sync project `workout-coach.html` and `deploy/`
      from repo `index.html`; archive `workout-coach-v4.html` in project folder
- [ ] **Deploy:** merge `v5-3d` → `main`, push, curl the live URL until v5
- [ ] Update memory files; draft WhatsApp release note for the user
- [ ] Commit/merge: "v5: real 3D mannequin demos"

## Status log
- 2026-07-07: Plan created, branch `v5-3d` pushed. Nothing built yet.
- 2026-07-07 (s1): Session 1 complete. 3D scene + static mannequin + orbit +
  modal toggle all verified in preview (load, toggle both ways, dispose on
  close, toggle hidden on non-3D exercises, console clean). Note for s2: the
  mannequin in buildMannequin() is positioned directly (no joint hierarchy) —
  s2 should restructure it into pivot groups (shoulder/elbow/hip/knee) before
  posing. can3D() gate currently allows Overhead Press only — widen in s3.
